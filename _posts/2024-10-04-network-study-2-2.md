---
title: 네트워크 - HTTP 메세지 포맷

author: milktea
date: 2024-10-04 08:00:00 +0800
categories: [Network]
tags: [HTTP]
pin: true
math: true
mermaid: true
---

# 1. HTTP 메서드 포맷

## 요청 메세지
```text
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```


![img.png](/assets/img/network/study-2-2/img.png)

HTTP/1.1에서 전형적인 요청 메세지이다.
첫 줄은 **요청 라인(Request Line)** 이고 이후의 줄들은 **헤더 라인(Header Line)** 이라고 부른다.
요청 라인은 메서드 + URL + HTTP 버전으로 구성되어 있고 헤더는 필드 이름-값 쌍으로 구성되어 있다.
또한 POST의 경우 바디(Entity Body)에 데이터를 담아 서버에 전송할 수 있다.

## 응답 메세지

```text
HTTP/1.1 200 OK
Connection: close
Data: Tue, 18 Aug 2015 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tue, 18 AUg 2015 15:11:03 GMT
Content-Length: 6821
Content-Type: text/html
...
```

응답 메세지는 첫 줄에 상태 라인(Status Line), 헤더 라인, 바디로 구성되어 있다.
상태 라인은 버전 필드 + 상태 코드 + 상태 메세지로 구성되어 있다.

## 헤더

헤더는 요청 메세지, 응답 메세지에 포함되어 있는 메타데이터이다.

- Connection: HTTP/1.1까지 버전에서 지속 연결, 비지속 연결을 결정하는데 사용
- Authorization: 서버에 인증 정보를 전달할 때 사용된다.
- Content-Type: 본문의 미디어 타입을 명시한다.
- Cache-Control: 캐싱 동작을 제어한다.
- Set-Cookie: 서버가 클라이언트에게 쿠키를 설정한다.

---

# 2. HTTP 메서드 종류

HTTP 메서드에는 다음과 같은 속성이 있다.

- 안전한 메서드 : 호출해도 서버 데이터를 변경하지 않는다.
- 멱등(Idempotent) : 몇 번을 호출하든 결과는 동일하다.
- 캐시 가능 : 응답 결과 리소스를 캐시해서 사용해도 되는 메서드인지를 나타낸다.

예를 들면 GET은 리소스를 조회하는 메서드이므로 데이터를 변경하지 않아 안전하다.
또한 하나의 데이터를 두 번 조회한다고 해서 결과가 달라지지 않으므로 멱등이다.

하지만 POST의 경우 서버의 리소스를 수정할 수 있다.
또한 서버의 리소스를 업데이트하므로 두 번 업데이트하면 이전과 결과가 달라질 수 있다.



| 메서드     | 안전 | 멱등 | 캐시 가능    |
|---------|----|----|----------|
| GET     | O  | O  | O        |
| HEAD    | O  | O  | O        |
| POST    | X  | X  | O(특수 상황) |
| PUT     | X  | O  | X        |
| PATCH   | X  | X  | O(특수 상황) |
| DELETE  | X  | O  | X        |
| CONNECT | X  | X  | X        |
| OPTIONS | O  | O  | X        |
| TRACE   | O  | O  | X        |

## 1) GET

GET 메서드는 특정 경로의 리소스를 조회하는 용도로 사용되는 메서드이다.
데이터를 검색할 때 기본으로 사용되는 메서드이다.
GET은 멱등하고 캐시될 수 있으므로 재사용이 가능해 최적화의 이점을 누릴 수 있다.

GET은 원칙적으로 바디에 데이터를 담아 전송하는 것을 허용하지 않는다.
따라서 정보가 URL의 쿼리 스트링에 노출되므로 민감한 정보로 데이터를 조회해야 할 때 문제가 발생할 수 있다.
RFC 9110에서는 이러한 상황에서 대안으로 POST를 사용해서 암호화한 결과를 본문에 추가하거나 데이터를 필터링, 변환하는 방법을 제시하고 있다.

### 쿼리 파라미터 vs 경로 매개변수 vs 본문

- 쿼리 파라미터(Query Parameter): 정렬이나 필터링 시에 사용한다.
- 경로 파라미터(Path Variable): 어떤 resource를 식별할 때 사용한다.
- 본문: 길이의 제한이 없고 경로에 데이터가 보이지 않기 때문에 보안에 약간 더 유리한 면이 있다.

## 2) POST

POST 메서드는 form을 제출하여 서버에서 처리하거나 데이터의 추가, 업데이트를 위해 사용되는 메서드이다.
서버의 데이터를 변경할 수 있으므로 안전하지 않고 동일한 요청을 여러 번 전송하면 다른 결과를 받을 수 있어 멱등성을 만족하지 않는다.

POST는 요청한 후 서버에서 반환된 업데이트된 데이터를 캐싱하면 이후 GET 요청에서 이 데이터를 재활용할 수 있다.
다만 이 경우 Content-Location 헤더와 Cache-Control 헤더를 통해 명시적으로 캐시 기간과 캐시된 리소스 정보를 명시해야만 재활용할 수 있다.

하지만 이론상으로는 캐시할 수 있지만 멱등하지 않는 상황에 대해서는 거의 사용되지 않는다고 보면 된다.

## 3) PUT

PUT 메서드는 리소스의 상태를 요청한 상태로 변경할 때 사용되는 메서드이다.
만약 변경할 데이터가 없는 경우 서버는 데이터를 생성하고 201 상태 코드로 생성됨을 알릴 수 있다.

PUT은 데이터를 수정하므로 안전하지 않지만 기존 리소스를 PUT으로 전송한 모든 데이터로 교체하므로 여러 번 실행해도 동일한 결과를 얻으므로 멱등하다.

### POST와 PUT의 차이

POST는 클라이언트가 서버에 "이 데이터를 바탕으로 알아서 만들어주세요"라는 뉘앙스이다.
클라이언트는 서버가 리소스를 처리하는 방식을 서버에 맡겨 서버가 알아서 새로운 리소스를 생성하는 것이다.

반면 PUT은 클라이언트가 서버에 "서버에 있는 **그 데이터**를 내가 보내는 데이터로 바꾸어주세요"라는 뉘앙스이다.
따라서 PUT 메서드는 클라이언트가 변경을 원하는 특정 리소스의 URI를 명시적으로 지정해야 한다.

물론 멱등성의 차이 또한 존재한다.

## 4) PATCH

PATCH 메서드는 서버에 있는 리소스를 어떻게 수정되어야 할지에 대한 동작을 정의할 때 사용되는 메서드이다.

text
{
"operation": "increment",
"height": 10
}


PATCH는 이와 같이 리소스의 특정 데이터를 어떻게 수정할 지에 대한 동작을 정의할 수 있다.
이러한 동작이 더하고 빼는 등의 연산이라면 **멱등하지 않고** 새로운 값으로 교체하는 연산이라면 **멱등** 하다.
따라서 기본적으로 PATCH는 멱등하지 않다.

### PUT과 PATCH의 차이

PUT은 기존 데이터를 없애고 새로운 데이터로 완전히 변환하기 때문에 멱등하고 PATCH는 다양한 연산을 사용할 수 있으므로 멱등하지 않을 수 있다.
또한 PUT은 명세상으로 캐싱이 불가능하고 PATCH는 POST처럼 명시적으로 캐시 기간과 캐시된 리소스 정보를 명시하고 PATCH 응답이 전체 데이터를 반환하는 경우 캐시할 수 있다.

하지만 PATCH도 POST와 마찬가지로 캐싱 기능은 거의 사용되지 않는다고 보면 된다.

## 5) DELETE

특정 URI와 연괸된 리소스를 삭제하는 요청을 보낸다.
DELETE 요청이 처리되더라도 이 리소스를 실제로 삭제할 지, 아니면 아카이브에 저장할 지에 대한 여부는 서버에게 달려있다.

DELETE 요청은 GET 요청과 마찬가지로 본문을 포함하지 않는다.

## 6) 그외

- HEAD: GET과 동일하지만 본문은 반환하지 않는다.
- CONNECT: 프록시 서버와 터널을 생성할 때 사용되며 HTTPS 통신을 할 때 주로 사용한다.
- OPTIONS: 서버가 특정 URI에서 어떤 HTTP 메서드와 옵션을 지원하는지 확인하는데 사용된다.
- TRACE: 서버는 요청을 그대로 반환하여 메세지의 경로를 추적하는데 사용된다.

---

# 3. 상태 코드

HTTP 상태 코드는 클라이언트가 서버에 요청을 보냈을 때 서버가 응답으로 보내는 상태를 나타내는 숫자 코드이다.
상태 코드는 3자리 숫자로 구성되며 각각의 상태 코드는 요청이 어떻게 처리되었는지 알려준다.

- 1xx(정보): 요청을 받았으며 처리를 계속해도 됨을 의미한다.
- 2xx(성공): 요청이 성공적으로 처리되었음을 의미한다.
- 3xx(리다이렉션): 요청을 완료하기 위해 추가적인 작업이 필요함을 의미한다.
- 4xx(클라이언트 오류): 클라이언트 요청에 오류가 있음을 의미한다.
- 5xx(서버 오류): 서버에서 요청을 처리하는 도중 문제가 발생했음을 의미한다.

| 상태 코드 | 이름                    | 설명                                 |
|-------|-----------------------|------------------------------------|
| 100   | Continue              | 클라이언트가 계속 요청해도 됨                   |
| 101   | Switching Protocols   | 프로토콜 변경 요청을 수락함                    |
| 200   | OK                    | 요청이 성공적으로 처리됨                      |
| 201   | Created               | 새로운 리소스가 성공적으로 생성됨                 |
| 204   | No Content            | 요청은 성공했지만 응답 본문은 없음                |
| 301   | Moved Permanently     | 요청한 리소스가 영구적으로 다른 URI로 이동함         |
| 302   | Found                 | 요청한 리소스가 일시적으로 다른 URI로 이동함         |
| 304   | Not Modified          | 현재 요청에 대한 응답은 변경되지 않아 캐시를 사용할 수 있음 |
| 400   | Bad Request           | 클라이언트가 서버에게 잘못된 요청을 보냄             |
| 401   | Unauthorized          | 인증되지 않았거나 잘못된 자격 증명임               |
| 403   | Forbidden             | 인증되었지만 권한이 없음                      |
| 404   | Not Found             | 요청한 리소스를 찾을 수 없음                   |
| 500   | Internal Server Error | 서버에 예상치 못한 오류가 발생                  |
| 502   | Bad Gateway           | 게이트웨이, 프록시 서버가 잘못된 응답을 받음          |
| 503   | Service Unavailable   | 서버가 현재 사용할 수 없는 상태(과부하, 점검 등)      |
| 504   | Gateway Timeout       | 게이트웨이, 프록시가 적시에 응답을 받을 수 없음        |

---
# Reference

[https://rachel0115.tistory.com/entry/HTTP-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%A0%95%EB%A6%AC-GET-POST-PUT-PATCH-DELETE](https://rachel0115.tistory.com/entry/HTTP-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%A0%95%EB%A6%AC-GET-POST-PUT-PATCH-DELETE)

[https://datatracker.ietf.org/doc/html/rfc9110#section-9.3.4](https://datatracker.ietf.org/doc/html/rfc9110#section-9.3.4)

[https://datatracker.ietf.org/doc/html/rfc5789](https://datatracker.ietf.org/doc/html/rfc5789)

[https://developer.mozilla.org/ko/docs/Web/HTTP/Status](https://developer.mozilla.org/ko/docs/Web/HTTP/Status)

[https://ryan-han.com/post/translated/pathvariable_queryparam/](https://ryan-han.com/post/translated/pathvariable_queryparam/)

James F. Kurose,Keith W. Ross 편저, 최종원 외 7인 옮김, 컴퓨터 네트워킹 하향식 접근 8판, 퍼스트북
