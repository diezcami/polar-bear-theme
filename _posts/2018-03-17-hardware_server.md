---
layout: post
title:  "[H/W] 서버"
date:   2018-03-13 08:43:59
author: garimoo
categories: Study
---
<br/># 하드웨어 - 서버 교육

## 회사별 서버 모델
## 메모리

### UDIMM (Unbuffered DIMM)

* 버퍼가 없는 DIMM(Dual In-line Memory Module)

### RDIMM (Registered DIMM)

![Image](/assets/20180317/1.png)

* 클럭과 주소 등의 제어 신호를 버퍼 회로가 가져온다. 버퍼 회로가 끼어들어 레이턴시가 증가하므로 UDIMM보다 액세스 속도가 떨어진다.
* `대용량 메모리`나 `안정적`으로 운영이 필요한 서버용 메모리로 자주 사용된다.

### LRDIMM(Load Reduced DIMM)

* RDIMM을 발전시킨 방식, 메모리 컨트롤러와 메모리 칩 사이의 모든 통신이 버퍼 회로를 매개로 이루어진다.
* 메모리 버스 전체의 부하를 줄여서, 모듈 하나당 메모리 칩 수를 늘릴 수 있다.
* 대용량과 고속 전송을 실현한다.

### 랭크

![Image](/assets/20180317/2.png)

메모리 컨트롤러가 DRAM에서 데이터를 입출력하는 단위가 랭크이다. 하나의 랭크는 `64bit` 단위로 입출력한다.

* `1Rx8` : 8bit DRAM 칩을 8개(+ ECC 1개) 구성한 싱글 랭크 메모리
* `1Rx4` : 4bit DRAM 칩을 16개(+ ECC 2개) 구성한 싱글 랭크 메모리
* `2Rx8` : 8bit DRAM 칩을 16개(+ECC 2개) 구성한 듀얼 랭크 메모리

## 하드디스크

### SAS vs SATA

![Image](/assets/20180317/3.png)
SAS에는 SATA에는 없는 port가 존재하고, 이는 데이터 전송시 오류의 발생 처리를 빠르게 해결해준다. 따라서 SATA는 전반적으로 속도가 느리다.

* `S-ATA` (Serial ATA) : 대역폭이 높고 가격이 싸지만 성능은 떨어진다.
* `SAS` (Serial attached SCSI) : SAS는 RPM이 높지만 가격이 비싸다.
* `NL SAS`(Near-Line SAS) : SATA 하드디스크에 SAS 인터페이스를 사용하는 구조이다. SAS의 이점은 살리면서 대용량을 구성할 수 있는 장점을 가지고 있다.

SAS 및 SATA 드라이브는 동일한 RAID 어레이에서 함께 사용 하면 안 되며 동일한 후면판에 혼합 되어서는 안된다.

## OOB (Out Of Band)

### DELL - iDRAC

* iDRAC은 Dell의 원격 액세스 컨트롤러이다.
* 서버가 꺼져 있는 경우에도 관리자가 Dell 시스템을 업데이트하고 관리하도록 해 준다.
* 원격 관리 작업을 수행할 수 있도록 웹/콘솔 인터페이스를 제공한다.

## Q&A

**Q. RAID 1+0과 0+1의 차이, 1+0을 많이 쓰는 이유는?**
A. RAID 0+1은 데이터가 완전하지 않다. 속도를 위해 이렇게 사용. 에러가 났을 때 복구가 어렵다.
![Image](/assets/20180317/4.png)
![Image](/assets/20180317/5.png)

## 출처

* https://extrememanual.net/2446
* https://www.intel.co.kr/content/www/kr/ko/support/articles/000005782/server-products.html
