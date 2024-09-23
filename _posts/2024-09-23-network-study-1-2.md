---
title: 네트워크 - 계층화, 캡슐화

author: milktea
date: 2024-09-23 12:00:00 +0800
categories: [Network]
tags: [TCP/IP, OSI]
pin: true
math: true
mermaid: true
---

# 1. 프로토콜 계층화

인터넷은 다양한 네트워크 장치, 수많은 호스트들로 구성된 매우 복잡한 시스템이다.
이러한 복잡한 시스템을 어떻게 효율적으로 조작할 수 있을까?

네트워크 구조에서는 **이러한 복잡성을 계층구조로 해결**하였다.

## 계층 구조

Spring 어플리케이션으로 개발을 하다보면 Controller, Service, Repository 등과 같은 방법으로 하나의 API를 여러 개의 층으로 나누는 것을 보았을 것이다.
이와 비슷하게 네트워크도 여러 계층으로 분리되어 각 계층은 각자의 역할을 수행하고 인접 계층에 결과만을 전달한다.
이러한 **관심사의 분리**는 여러 장점이 있다.

1. 모듈화로 다른 계층에 영향이 적어 변경과 확장에 유리하다.
2. 시스템을 작은 부분으로 나누어 문제를 쉽게 파악하고 관리할 수 있다.

컴퓨터 네트워크도 이러한 계층화를 적용하여 복잡한 시스템을 쉽게 파악하고 또한 변경과 확장이 가능하도록 **TCP/IP 모델, OSI 7 모델** 두 가지 계층화 모델이 존재한다.
우리가 흔히 인터넷을 쓸 때 사용하는 Wifi와 이더넷(유선랜)은 물리 계층과 데이터 링크 계층에서 다르게 구현되어 있다.
하지만 우리가 인터넷 서비스를 사용할 때 Wifi로 접속하나 이더넷으로 접속하나 사용자 경험에는 큰 차이가 없다.
네트워크 계층 이상의 상위 계층에서는 하위 계층의 구현 방식에 의존하지 않아 하위 계층에서 Wifi에서 이더넷으로 변경되더라도 큰 영향을 받지 않는다.


---

# 2. 계층 모델

하나의 계층이 어떠한 역할까지 수행할 지에 따라 각 기관, 단체마다 계층의 수가 다르다.
OSI는 7 계층, TCP/IP는 4 계층으로, "컴퓨터 네트워킹 하향식 접근"에서는 5 계층으로 설명하고 있다.
이 책에서는 TCP/IP의 네트워크 인터페이스 계층을 데이터 링크와 물리 계층으로 구분한다.

![img.png](/assets/img/network/study-1-2/img.png)

## OSI 7 계층

국제표준화기구에서 분류한 모델이다. 
상이한 컴퓨터 시스템이 서로 통신할 수 있는 표준을 제공한다.

| 계층 | 계층명           | 설명                                  |
|----|---------------|-------------------------------------|
| 7  | Application   | 사람과 직접 상호작용하여 네트워크 서비스에 접근할 수 있는 계층 |
| 6  | Presentation  | 데이터를 사용 가능한 형식으로 변환하고 암호화           |
| 5  | Session       | 연결을 유지하며 포트와 세션 관리                  |
| 4  | Transport     | TCP/UDP 등의 전송 프로토콜을 사용해 데이터 전송      |
| 3  | Network       | 데이터를 전송할 물리적 경로를 설정                 |
| 2  | Data Link     | 네트워크 상에서 데이터의 형식을 정의, 오류 처리         |
| 1  | Physical      | 물리적인 매개체를 통해 비트 단위로 데이터 전송          |

## TCP/IP 계층

미국 국방부 산하 기관(ARPA)에서 시작한 프로젝트에서 기원하며 TCP/IP와 함께 고안된 실용적인 모델이다.
OSI 7 계층을 4 계층으로 줄여 더 추상화하고 간략화하였다.
실용적인 TCP/IP 4 계층을 바탕으로 많은 프로토콜들이 고안되고 채택되었다.

| 계층 | 계층명               | OSI 7 계층과 비교                         | 프로토콜 예시                | 데이터 단위 |
|----|-------------------|--------------------------------------|------------------------|--------|
| 4  | Application       | Application + Presentation + Session | HTTP, FTP, SMTP, DNS   | 메세지    |
| 3  | Transport         | Transport                            | TCP, UDP               | 세그먼트   |
| 2  | Internet          | Network                              | IPv4, IPv6, ICMP       | 패킷     |
| 1  | Network Interface | Data Link + Physical                 | Ethernet, 802.11(wifi) | 프레임    |

---

# 3. 캡슐화

![img_1.png](/assets/img/network/study-1-2/img_1.png)

위의 이미지에서 네트워크 전송 시 데이터를 하위 계층으로 전달할 때 각 계층은 데이터에 자신의 헤더를 추가하여 전달하는 것을 볼 수 있다.
이 헤더에는 각각의 계층이 기능을 수행하기 위한 데이터가 포함되어 있다.

데이터를 전달받을 때에는 각 계층이 헤더를 보고 주어진 역할을 수행한다.
이후 상위 계층으로 데이터를 전달한다.

따라서 **각 계층의 역할을 수행하기 위해 필요한 정보를 헤더에 포함**하며 이를 **캡슐화**하고 목적지에서 **역캡슐화**하는 과정을 통해 각 계층의 역할이 수행된다.


---
# Reference

[https://www.guru99.com/ko/difference-tcp-ip-vs-osi-model.html](https://www.guru99.com/ko/difference-tcp-ip-vs-osi-model.html)

[https://www.cloudflare.com/ko-kr/learning/ddos/glossary/open-systems-interconnection-model-osi/](https://www.cloudflare.com/ko-kr/learning/ddos/glossary/open-systems-interconnection-model-osi/)

James F. Kurose,Keith W. Ross 편저, 최종원 외 7인 옮김, 컴퓨터 네트워킹 하향식 접근 8판, 퍼스트북
