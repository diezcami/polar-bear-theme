---
layout: post
title:  "Micro Service Architecture"
date:   2018-03-16 12:43:00
author: garimoo
categories: Study
---
<br/>

# Micro Service Architecture

## 애플리케이션 로직 진화와 마이크로서비스 아키텍처의 출현

### 기존 비즈니스 로직 - Monolith

* 애플리케이션 확장성 애로: 민첩성 저해
* 장기 개발 사이클: 혁신 저해
* 특정 모듈 장애시 운영 어려움, 신규 기능 추가 어려움

### 마이크로서비스를 위한 전제 조건

![Image](/assets/20180316/1.png)

### 마이크로서비스란?

**`service-oriented architecture` composed of `loosely coupled elements` that have `bounded contexts`**

* 서비스들이 네트워크를 통해 서로 API로 통신한다
* 서비스는 독자적으로 업데이트하며, 서로 영향을 주지 않는다
* 다른 서비스의 내부 구조를 알지 못해도, 내 서비스 코드를 업데이트할 수 있다.

### 하나의 마이크로서비스 구성 패턴

![Image](/assets/20180316/2.png)

## 마이크로서비스 운영 시 개발 문화 베스트 프랙티스

### 마이크로서비스 아키텍처를 위한 개발 문화

![Image](/assets/20180316/3.png)

### 마이크로서비스의 이점

* 개별 마이크로 서비스 확장 가능 : 빠른 운영으로 민첩성이 높아진다.
* 빠른 빌드/테스트/베포 가능
* 자율적 운영 가능 : 신규 기능 빠르게 추가 가능

## 출처

* [마이크로서비스 기반 클라우드 아키텍처 구성 모범 사례 - 윤석찬](https://www.slideshare.net/awskorea/microservices-architecuture-on-aws)
