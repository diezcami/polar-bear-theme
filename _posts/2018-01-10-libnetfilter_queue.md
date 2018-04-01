---
layout: post
title:  "libnetfilter_queue 다운로드, nfqnl_test.c 실행"
date:   2018-01-10 08:43:60
author: garimoo
categories: Study
---
<br/>
## libnetlink project, libnetfilter_queue 다운로드

```
sudo wget http://www.netfilter.org/projects/libnfnetlink/files/libnfnetlink-1.0.1.tar.bz2
```

명령어로 패키지 다운로드 (최신 버전이 나왔을 수 있으니 http://www.netfilter.org/projects/libnfnetlink/downloads.html 여기서 확인)

```
tar -xvf libnfnetlink-1.0.1.tar.bz2
cd libnfnetlink-1.0.0/
./configure
make
sudo make install

```

다운로드가 끝났으면 http://www.netfilter.org/projects/libnetfilter_queue/downloads.html 에서 libnetfilter_queue 같은 방식으로 다운로드.

(나는 다운로드 중에 libmnl인가? 필요하다고 해서 더 다운받았다. http://www.netfilter.org/projects/libmnl/index.html )

<br/><br/>
## sample 코드 이해하기(nfqnl_test.c)

http://www.netfilter.org/projects/libnetfilter_queue/doxygen/group__LibrarySetup.html
http://www.netfilter.org/projects/libnetfilter_queue/doxygen/group__Queue.html

여기서 코드에 대한 설명을 볼 수 있다.

```c

//nfqnl_test.c


#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <netinet/in.h>
#include <linux/types.h>
#include <linux/netfilter.h>		/* for NF_ACCEPT */
#include <errno.h>
#include <libnetfilter_queue/libnetfilter_queue.h>


/* returns packet id */


static u_int32_t print_pkt (struct nfq_data *tb)
{

	int id = 0;
	struct nfqnl_msg_packet_hdr *ph;
	struct nfqnl_msg_packet_hw *hwph;
	u_int32_t mark,ifi;
	int ret;
	unsigned char *data;
	ph = nfq_get_msg_packet_hdr(tb);

	if (ph) {
		id = ntohl(ph->packet_id);
		printf("hw_protocol=0x%04x hook=%u id=%u ",
			ntohs(ph->hw_protocol), ph->hook, id);
	}

	hwph = nfq_get_packet_hw(tb);

	if (hwph) {
		int i, hlen = ntohs(hwph->hw_addrlen);
		printf("hw_src_addr=");

		for (i = 0; i < hlen-1; i++)
			printf("%02x:", hwph->hw_addr[i]);

		printf("%02x ", hwph->hw_addr[hlen-1]);
	}

	mark = nfq_get_nfmark(tb);

	if (mark)
		printf("mark=%u ", mark);

	ifi = nfq_get_indev(tb);

	if (ifi)
		printf("indev=%u ", ifi);

	ifi = nfq_get_outdev(tb);

	if (ifi)
		printf("outdev=%u ", ifi);

	ifi = nfq_get_physindev(tb);

	if (ifi)
		printf("physindev=%u ", ifi);

	ifi = nfq_get_physoutdev(tb);

	if (ifi)
		printf("physoutdev=%u ", ifi);

	ret = nfq_get_payload(tb, &data);

	if (ret >= 0)
		printf("payload_len=%d ", ret);

	fputc('\n', stdout);
	return id;
}


static int cb(struct nfq_q_handle *qh, struct nfgenmsg *nfmsg,
	      struct nfq_data *nfa, void *data)
{
	u_int32_t id = print_pkt(nfa);

	printf("entering callback\n");
	return nfq_set_verdict(qh, id, NF_ACCEPT, 0, NULL);
}


int main(int argc, char **argv)
{

	struct nfq_handle *h;
	struct nfq_q_handle *qh;
	struct nfnl_handle *nh;
	int fd;
	int rv;
	char buf[4096] __attribute__ ((aligned));

	printf("opening library handle\n");

	h = nfq_open();

	if (!h) {
		fprintf(stderr, "error during nfq_open()\n");
		exit(1);
	}

	printf("unbinding existing nf_queue handler for AF_INET (if any)\n");

	if (nfq_unbind_pf(h, AF_INET) < 0) {
		fprintf(stderr, "error during nfq_unbind_pf()\n");
		exit(1);
	}

	printf("binding nfnetlink_queue as nf_queue handler for AF_INET\n");

	if (nfq_bind_pf(h, AF_INET) < 0) {
		fprintf(stderr, "error during nfq_bind_pf()\n");
		exit(1);
	}

	printf("binding this socket to queue '0'\n");

	qh = nfq_create_queue(h,  0, &cb, NULL);

	if (!qh) {
		fprintf(stderr, "error during nfq_create_queue()\n");
		exit(1);
	}

	printf("setting copy_packet mode\n");


	if (nfq_set_mode(qh, NFQNL_COPY_PACKET, 0xffff) < 0) {
		fprintf(stderr, "can't set packet_copy mode\n");
		exit(1);
	}

	fd = nfq_fd(h);

	for (;;) {
		if ((rv = recv(fd, buf, sizeof(buf), 0)) >= 0) {
			printf("pkt received\n");
			nfq_handle_packet(h, buf, rv);
			continue;
		}

		/* if your application is too slow to digest the packets that
		 * are sent from kernel-space, the socket buffer that we use
		 * to enqueue packets may fill up returning ENOBUFS. Depending
		 * on your application, this error may be ignored. Please, see
		 * the doxygen documentation of this library on how to improve
		 * this situation.
		 */

		if (rv < 0 && errno == ENOBUFS) {
			printf("losing packets!\n");
			continue;
		}

		perror("recv failed");
		break;
	}

	printf("unbinding from queue 0\n");
	nfq_destroy_queue(qh);


#ifdef INSANE

	/* normally, applications SHOULD NOT issue this command, since
	 * it detaches other programs/sockets from AF_INET, too ! */

	printf("unbinding from AF_INET\n");
	nfq_unbind_pf(h, AF_INET);

#endif


	printf("closing library handle\n");
	nfq_close(h);

	exit(0);
}
```

코드 컴파일 방법은 다음과 같다.

`gcc -Wall -o test nfqnl_test.c -lnfnetlink -lnetfilter_queue`


코드 실행 방법은 다음과 같다.

`sudo ./test`


나는 sudo 안붙혀서 계속 에러나서 이틀동안 고생했다...
계속 error during nfq_unbind_pf()가 안된다고 하는거ㅠㅠㅠ
알고보니 root 권한으로 실행시키면 되는 쉬운 문제였다.
<br/><br/>

암튼 이걸 실행시키기 위해서 두개의 shell이 필요하다.

**첫번째 쉘**

```
1. sudo iptables -A OUTPUT -p icmp -j NFQUEUE --queue-num 0

# this create a queue and rediect icmp to this queue

2. ping www.cceye.com

# this create icmp traffic , note at this stage, all ICMP traffic are blocked, since no queue consumer process.
```

**두번째 쉘**

```
3. let the ping continue run, and in new shell, run nfqnl_test,

sudo ./test
```
