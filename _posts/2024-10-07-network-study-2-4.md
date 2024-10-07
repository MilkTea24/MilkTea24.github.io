---
title: 네트워크 - DNS

author: milktea
date: 2024-10-07 07:00:00 +0800
categories: [Network]
tags: [DNS]
pin: true
math: true
mermaid: true
---

# 1. DNS(Domain Name System)

## Domain Name

서로 다른 네트워크 상에서 호스트를 유일하게 식별하기 위해서 IP를 사용한다.
하지만 사용자는 IP 만으로는 이 호스트가 어떤 기능을 하는 호스트인지 구별하는 것은 쉽지 않다.
유일하게 식별할 수 있는 주민등록번호가 있지만 이름을 부르는 것처럼 사람이 읽기 쉬운 텍스트를 활용하여 호스트를 구별할 수 있는 식별자는 **도메인 이름**이라고 한다.

## DNS

IP와 호스트의 매핑 정보는 DNS의 여러 서버에서 관리된다.
이 DNS는 분산화된 데이터베이스로 많은 네임 서버들이 계층을 이루고 있다.

---
# 2. DNS 작동 방식

DNS의 작동 과정은 다음과 같다.
어떤 호스트가 `www.naver.com` 웹서버에게 HTTP Request 메세지를 보내려고 하자.
호스트는 이 웹서버의 IP 주소를 알아야만 통신을 할 수 있다.

1. 호스트 브라우저는 URL로부터 호스트 이름 `www.naver.com`을 추출
2. 호스트 OS의 네트워크 스택(Transport + Network + Network Interface 계층)은 이 호스트 이름을 DNS 리졸버에게 전달
3. DNS 리졸버는 로컬 DNS 서버로 호스트 이름에 대한 IP 주소를 요청하는 쿼리를 송신
4. DNS 리졸버는 로컬 DNS 서버로부터 호스트 이름에 대한 IP 주소 응답 수신
5. DNS 리졸버는 IP 주소 정보를 OS와 브라우저에게 전달
6. 브라우저가 DNS 리졸버로부터 IP 주소를 받으면 해당 IP 주소와 80번 포트에 있는 프로세스에게 TCP 연결을 요청한다.

## Transport 프로토콜

[RFC 1035](https://datatracker.ietf.org/doc/html/rfc1035#section-4.2)에서 DNS는 UDP와 TCP 모두 사용할 수 있다고 명시되어 있다.
TCP와 UDP 모두 포트 53번을 사용한다.

TCP는 연결과 연결 해제 과정에서 핸드쉐이크 과정이 필요한데 UDP는 이러한 과정이 없어 오버헤드가 적다.
따라서 연결된 클라이언트가 아주 많고 단순 질의 로직이 대부분인 DNS 특성 상 신뢰성보다 성능이 우선되는 UDP를 많이 활용한다.

하지만 DNS에서 UDP로 전송가능한 데이터 크기는 512바이트로 제한되며 이후 초과되는 메세지는 잘린다.
따라서 512바이트보다 큰 데이터를 전송하거나 백업 DNS 서버와 동기화하는 역할인 영역 전달(Zone Transfer) 과정에서는 TCP를 사용한다.


## 캐싱

DNS 서버는 자주 조회하는 IP나 다른 DNS 서버 주소를 캐싱할 수 있다.
따라서 여러 번 동일한 도메인 이름을 조회하면 주소, IP가 캐싱되어 위의 질의 과정을 일부 생략할 수 있다.

- 로컬 DNS 서버에서는 최근에 접속했던 도메인 주소와 IP를 캐싱하여 질의없이 바로 반환할 수 있다.
- 각각의 recursive DNS 서버에서도 하위 계층의 DNS 서버를 캐싱하여 하위 계층 질의 단계를 건너뛸 수 있다.

---

# 3. 쿼리 처리 방식

DNS 질의의 3번 과정에서 쿼리를 받고 DNS 서버가 쿼리를 처리하는 방식이 크게 두 가지가 존재한다.
하나는 Iterated Query 방식이고 나머지 하나는 Recursive Query 방식이다.

## Iterated Query

![img.png](/assets/img/posts/network/study-2-4/img.png)

1. 클라이언트가 로컬 DNS 서버(근거리 서버, 보통 LAN 네트워크 당 하나 이상)에 호스트의 IP를 질의한다.
2. 로컬 DNS 서버가 루트 DNS 서버에 질의하고 하위 계층의 TLD DNS 서버 주소를 알려준다.
3. 로컬 DNS 서버가 TLD DNS 서버에 질의하고 실제 IP 주소를 알고 있는 DNS 서버 주소를 알려준다.
4. 로컬 DNS 서버가 최종(Authoritative, 권한 있는) DNS 서버에 질의하여 IP 주소를 얻고 호스트에게 반환한다.

### 장점 

- 요청을 받은 외부 DNS 서버는 주소를 반환하면 끝이기 때문에 서버 부하가 감소된다.
- 외부 DNS 서버 하나가 실패해도 다른 경로를 선택할 수 있는 유연성을 가진다.
- 루트 DNS 서버, TLD DNS 서버와 같이 중간 단계 결과까지 로컬 DNS 서버에 캐싱할 수 있다.

### 단점

- 로컬 DNS 서버에서 질의하는 로직이 복잡해진다.
- 각각의 외부 DNS 서버에 새 연결을 구성하므로 네트워크 오버헤드가 Recursive 방식보다 크다.

### Recursive Query

![img_1.png](/assets/img/posts/network/study-2-4/img_1.png)

1. 클라이언트가 로컬 DNS 서버에게 호스트 IP 주소를 질의한다.
2. 로컬 DNS 서버는 가지고 있는 루트 DNS 서버 주소를 사용하여 루트 DNS 서버에게 질의한다.
3. 루트 DNS 서버는 가지고 있는 TLD DNS 서버 주소를 사용하여 TLD DNS 서버에게 질의한다.
4. TLD DNS 서버는 가지고 있는 최종 DNS 서버 주소를 사용하여 최종 DNS 서버에게 질의한다.
5. 최종 DNS 서버 -> TLD DNS 서버 -> 루트 DNS 서버 -> 로컬 DNS 서버 -> 클라이언트 경로로 IP를 반환한다.

### 장점

- 클라이언트는 요청을 하고 기다리기만 하면 되므로 단순하다.
- 중간의 recursive DNS(최종 DNS 서버가 아닌 서버들)가 캐싱을 통해 빠른 응답을 할 수 있다.

### 단점

- 외부 DNS 서버는 질의까지 해야하므로 서버 부하가 증가한다.
- recursive DNS는 모두 실패 지점이 될 수 있다.

---

# 4. DNS 레코드

DNS는 분산화된 데이터베이스로 매핑 정보를 **레코드** 방식으로 저장하게 된다.

- .com TLD 서버 레코드 예시

| Name           | Value          | Type |
|----------------|----------------|------|
| hello.com      | dns1.hello.com | NS   |
| dns1.hello.com | 212.212.212.1  | A    |
| helllo.com     | dns2.hello.com | NS   |
| dns2.hello.com | 212.212.212.2  | A    |

dns1.hello.com과 dns2.hello.com은 최종 서버에 질의하여 얻은 IP 주소를 캐싱한다.
따라서 최종 서버에 질의하지 않아도 .com TLD 서버는 IP 주소를 바로 반환할 수 있다.

- hello 최종 서버

| Name           | Value              | Type  |
|----------------|--------------------|-------|
| hello.com      | realname.hello.com | CNAME |
| hello.com      | mail.hello.com     | MX    |
| mail.hello.com | 212.212.71.4       | A     |

## 레코드 Type

레코드가 저장하고 있는 정보 특성에 따라 여러 개의 타입으로 분류할 수 있다.

1. type = A(Address): IP 주소를 저장하는 레코드
2. type = AAAA: IPv6 주소를 저장하는 렠드
3. type = NS(Name Server): 하위 계층의 DNS 서버 주소를 저장하는 레코드
4. type = CNAME(Canonical Name): 한 도메인 이름의 별칭을 저장하는 레코드. 여러 도메인 이름이 하나의 IP를 가르키도록 할 수 있다.
5. type = MX(Mail Exchange): 해당 도메인의 메일 서버 이름을 저장하는 레코드

---
# Reference

[https://kingofbackend.tistory.com/198](https://kingofbackend.tistory.com/198)

[https://dev.dwer.kr/2020/03/dns-tcp.html](https://dev.dwer.kr/2020/03/dns-tcp.html)

James F. Kurose,Keith W. Ross 편저, 최종원 외 7인 옮김, 컴퓨터 네트워킹 하향식 접근 8판, 퍼스트북
