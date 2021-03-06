---
layout: post
title: TIL - DB 서버 아키텍쳐
comments: true
tags:
- Architecture
- TIL
---

> 이 포스팅은 [데이터베이스 첫걸음](http://www.kyobobook.co.kr/product/detailViewKor.laf?barcode=9788968487316) 이란 책을 보고 정리한 내용임을 밝힙니다.

#### DB 서버 아키텍처 역사

* Stand-alone(~ 1980)
* Client / Server (1990 ~ 2000)
* Web 3 Layer (2000 ~ 현재)


#### Stand-alone

데이터베이스가 동작하는 머신(데이터베이스 서버)이 LAN 이나 인터넷 등의 네트워크에 접속하지 않고 독립되어 동작하는 구성

* DBMS나 애플리케이션이 같은 DB 서버에서 동작한다.
* 복수의 사용자가 동시에 작업할 수 없다.
* 가용성이 낮다 (시스템 장애시 서비스 정지)
* 확장성이 부족하다 (서버가 한대라 성능을 높이기에 한계가 있다)
* 보안이 좋다
* 구축이 간단해 테스트용도나 소규모 작업용으로 적당하다.

#### Client / Server

데이터베이스 서버 1대에 복수 사용자의 단말이 접속하는 구성

* 클라이언트에서 동작하는 Application 을 업데이트후, 배포할 때 클라이언트의 환경에 대응해야 하는데 클라이언트가 많아 질수록 비용이 급속도로 올라간다.

#### Web3 Layer

Client / Server 아키텍쳐의 단점을 해결하기 위해서 Application을 서버에서 관리해 비용을 절감하자는 요구로 탄생한 아키텍쳐

`웹서버 계층`, `애플리케이션 계층`, `데이터베이스 계층` 으로 나누어짐

* 웹 서버 계층 : 클라이언트의 접속 요청을 받아 애플리케이션 계층에 넘김, 애플리케이션 계층의 결과를 받아 클라이언트에 반환

* 애플리케이션 계층 : 비즈니스 로직을 구현한 애플리케이션이 동작하는 층, 필요하면 데이터베이스 계층에 접속해서 데이터를 추출하고, 가공 그 결과를 웹서버 계층으로 반환

* 데이터베이스 계층 : 애플리케이션 운영이 필요한 데이터를 저장

#### 정지하지 않는 시스템을 만들기 위한 전략

`가용성`을 높이려고 할때, 취할 수 있는 방법은 크게 2가지가 있습니다.
> 시스템이 서비스 제공시간에 장애 없이 서비스를 계속 지속할 수 있는 비율을 가용성(Availability) 라고 한다.

* 심장전략 - 시스템을 구성하는 컴포넌트의 신뢰성을 높여서 장애 발생률을 낮게 한다. (소수정예)
* 신장전략 - 시스템을 구성하는 컴포넌트의 여분을 충분히 준비해 둔다 (물량작전)

현재는 부품 한 개 한 개의 신뢰성을 높이기 보다는 저품질 이라도 양으로 보완한다는 신장전략 쪽이 우세하다.


##### DB 서버의 다중화

* `클러스터` : 동일한 기능의 부품들을 병렬화 하는 것 <br/>
* `리플리케이션` : 클러스터 기술중 하나로, 하나의 부품이 고장날때 를 대비해 복사본을 유지하는 것 (동기화 필요) <br/>
* `다중화` : 클러스터로 시스템의 가동률을 높이는 것 <br/>

데이터베이스는 대량의 데이터를 영구적으로 보존하고 그에 따른 성능도 요구되기 때문에 DB서버 내의 로컬 저장소나 메모리로는 이러한 요건을 충족시키기 어렵다. 그래서 DB 서버의 아키텍쳐는 전용의 외부 저장소와 묶어서 생각해야 한다.

데이터는 항상 갱신되기 때문에 DB서버를 클러스터링 할 때 `데이터 정합성`도 중요하게 의식해야 한다.
> 데이터 정합성이란 데이터가 일치되어야 한다는 말입니다.
> 예를 들어서 `리플리케이션` 을 할때 원본(master)데이터와 복사본(slave) 데이터가 동기화 되지 않으면 데이터의 정합성이 깨집니다.

* Active - Active : 클러스터를 구성하는 컴포넌트를 동시에 가동함, 시스템 다운 시간이 짧고 성능이 좋다. 비싸다
  * Shared Disk : 여러 DB 서버가 하나의 저장소를 공유, 병목현상이 나타날 수도 있음
  * Shared Nothing : 각각의 DB 서버가 각각의 저장소를 가지고 있음, 병목현상이 없지만 하나의 서버가 죽었을때 다른 서버가 이를 이어받아 처리할 수 있는 구성(Covering)을 고려해야 한다.

* Active - Standby : 클러스터를 구성하는 컴포넌트 중 실제 가동하는 것은 Active, 남은 것은 대기(Standby) 하고 있는다. 저렴하다.
  * Cold - Standby : 평소에는 Standby DB 가 작동하지 않다가 Active DB 가 다운된 시점에 작동하는 것 (상대적으로 전환시간이 길고, 저렴하다)
  * Hot - Standby : 평소에도 Standby DB 가 작동하는 구성 (상대적으로 전환시간이 짧고, 비싸다)
