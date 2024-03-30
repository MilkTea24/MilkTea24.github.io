---
title: Spring의 등장 배경으로 Spring 알아보기
author: milktea
date: 2024-03-26 10:00:00 +0800
categories: [Spring]
tags: [POJO, Spring]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/posts/thumbnail/spring.png
---
# 요약
Spring은 엔터프라이즈용 Java 애플리케이션 개발을 편하게 할 수 있게 해주는 오픈소스 경량급 애플리케이션 프레임워크이다.
기존의 JAVA EE의 EJB가 미리 정의된 기술을 구현하는 방식에서 어노테이션을 이용한 방식으로 전환함으로써 POJO의 장점을 최대한 활용할 수 있도록 하였다.
또한 DI, IoC를 도입하여 개발자는 객체 관리를 Spring에 맡기고 비즈니스 로직에만 집중할 수 있다.
AOP는 코드의 중복을 줄여 유지보수성을 높이고 PSA는 데이터베이스, 캐시와 같은 기능에서도 기술에 종속되지 않는 코드를 작성할 수 있게 해준다.
이와 같은 Spring의 여러 개발자 편의 기능 덕분에 아직까지도 인기있는 웹 프레임워크 중 하나가 되었다.


# 서론
Spring은 무엇일까?라는 질문을 받은 적이 있었다.
평소 Spring을 써왔지만 Spring이 무엇일까에 대한 대답은 쉽지 않았다.

그래서 이에 대한 적절한 대답이 무엇일까 알아보던 중 Spring의 탄생과 관련한 아주 [흥미로운 글](https://okky.kr/articles/415474?note=1302409)을 찾았다.
이 참조 자료를 바탕으로 조사한 후 정리하였다.

먼저 Spring의 등장 배경을 소개하고 이후 Spring의 핵심 특징을 정리한다.

# POJO
## JAVA EE와 EJB
Spring이 등장하기 이전에는 JAVA EE가 있었다.
JAVA EE(JAVA Enterprise Edition)는 자바로 서버사이드 개발을 위해 기존의 JAVA SE에서 여러 확장 기능을 지원하는 에디션이다.
이 JAVA EE의 등장으로 자바를 사용한 웹 애플리케이션 개발이 대중화된다.

![img1.png](/assets/img/posts/my-springboot/j2ee.png)

JAVA EE로 개발한 대규모 애플리케이션 구조를 위 그림과 같이 정리할 수 있다.
Java Servlet, JSP, EJB는 JAVA EE에서 제공하는 **여러 확장 기능 중 하나**이다.
Java Servlet, JSP는 Spring을 사용하면 볼 수 있는 익숙한 기술들이다.
그런데 EJB는 앞의 기술들에 비해 생소할 수도 있다.

간단하게 말하자면 이 EJB가 가진 기술적 단점으로 인해 Spring이 등장하게 되었다.
**당시의 EJB가 가진 문제점에 반발하여 나온 사상이 바로 POJO**이다.
(JAVA EE 진영도 이 문제를 인식하고 있었고 현재의 EJB는 POJO를 채택하고 있다.)

## EJB의 문제점
그 당시 EJB에 어떤 문제가 있었는지 간단하게 확인해보자.
무려 2000년 6월 9일에 작성된 [EJB 2.0 튜토리얼](https://www.infoworld.com/article/2076103/read-all-about-ejb-2-0.html)의 코드를 가져왔다.
```java
public abstract EmployeeBean implements javax.ejb.EntityBean {
    // instance fields
    EntityContext ejbContext;
    // container-managed persistent fields
    public abstract void setIdentity(int identity);
    public abstract int getIdentity();
        ...

    // container-managed relationship fields
    public abstract void setContactInfo(ContactInfo info);
    public abstract ContactInfo getContactInfo();
}
```
이름과 메서드로 Employee의 데이터를 가지고 있는 Entity 객체임을 짐작할 수 있다.
그런데 이 클래스는 javax.ejb.EntityBean이라는 인터페이스를 구현하고 있다.

당시 EJB의 빈들은 EJB Container가 관리함으로써 여러 로우과 관련한 처리를 대신해주고 개발자 입장에서는 비즈니스 로직에만 집중하면 된다는 장점이 있었다.
하지만 다음과 같은 단점들도 같이 가지고 있었다.

1. 애플리케이션이 특정 기술인 EJB에 종속된다. 이는 확장과 변화에 유연하지 못하다.
2. 객체는 항상 특정 기술을 구현하고 상속해야 하므로 객체가 무거워진다.

만약 비즈니스 로직을 담당하는 객체들이 기술에 종속되면 어떻게 될까?
다른 기술로 변경해야 하는 경우 기술 종속적인 코드를 전부 수정해야 한다.
또한 상속을 이용한 기술 종속의 경우 다중 상속이 어려워진다는 문제점도 있었다.

결국 EJB는 자바의 객체 지향 설계 구현이 어려워진다는 단점이 있었다.
이러한 문제를 해결하기 위해 POJO라는 개념이 등장하였다.

## POJO(Plain Old Java Object)
POJO는 자바 외의 어떠한 특정한 기술에 종속되지 않은 오브젝트이다.
미리 정의된 기술을 상속(Extend), 구현(Implement) 하면 안된다.
원칙적으로, 미리 정의된 어노테이션의 사용도 POJO로 보지 않는다.
하지만 이상적인 POJO 구현의 어려움도 있고 어노테이션을 사용할 때 위의 기술 종속적인 객체의 문제가 적기 때문에 많은 프레임워크가 어노테이션 방식을 채택하고 있다.

Spring은 JAVA EE의 Servlet, JSP와 같은 개념은 채택하되 **기술 종속적인 객체의 문제를 해결하기 위해 POJO의 장점을 최대한 활용하는 어노테이션을 사용한다.**

# Spring
스프링의 등장 배경을 바탕으로 스프링이 무엇인가에 대한 답을 정리하면 다음과 같다.

> 엔터프라이즈용 Java 애플리케이션 개발을 편하게 할 수 있게 해주는 오픈소스 경량급 애플리케이션 프레임워크
{: .prompt-tip }

Spring이 인기를 얻게 된 이유는 엔터프라이즈용 Java 애플리케이션 개발이 당시 JAVA EE보다 더 편하고 경량이였기 때문으로 요약할 수 있다.
여기서 경량이라고 하는 의미는 기존 JAVA EE의 EJB에 비해서 개발자가 더 간편하게 코드를 작성할 수 있다는 말이다.
또한 JAVA EE와 달리 오픈소스이므로 생태계 활성화로 인해 빠른 개선과 기능 추가가 가능하였다.
그래서 많은 개발자들이 JAVA EE에서 Spring으로 넘어오게 된 것이다. 

Spring이 어떻게 엔터프라이즈용 Java 애플리케이션의 개발을 편하게 해주는지 다음과 같은 핵심 개념들을 살펴보면서 설명하겠다.
아래의 개념들은 [이 글](https://www.codestates.com/blog/content/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8)을 바탕으로 작성하였다.

## IoC(Inversion of Container) / DI(Dependency Injection)
IoC는 객체의 생성과 삭제, 의존 관계를 스프링 컨테이너가 대신 해주는 개념이다.
DI는 객체 사이의 의존관계를 외부에서 주입해주는 개념이다.

### 의존(Dependency)과 DI
**클래스 사이의 의존 관계는 두 클래스 사이 연산 간의 호출 관계를 표현**한다.
A 클래스가 B 클래스에 의존한다고 할 때 B 클래스의 변경이 A 클래스에 영향을 미침을 뜻한다.

![img2.png](/assets/img/posts/my-springboot/dependency.png)

```java
class Class3 {
    public void op31(Class4 arg) {
        arg.op4() ;
    }
    public void op32() {
        Class5 c5 = new Class5() ;
        c5.op5() ;
    }
}
```
의존 관계 발생은 위와 같이 이뤄진다.
이 때 호출할 때마다 클래스를 생성하는 op32()에 주목하자.
예를 들어 100개의 메서드가 Class5에 의존하는데 이를 Class6으로 의존성을 변경해야 한다.
100개의 메서드를 op32()처럼 짠 경우 100개 모두 일일이 변경해야 할 것이다.

반면 **클래스를 외부에서 생성해서 op31()처럼 넘겨주는 경우 Class5 대신 Class6을 넘겨주는 것으로 변경**하기만 하면 된다.
이처럼 외부에서 객체를 생성하여 전달하는 방식으로 의존성을 구현하는 방법을 DI라고 한다.

### IoC
IoC는 클래스의 외부에서 객체 생성도 개발자에게 맡기지 않는다.
스프링 컨테이너가 빈이라고 하는 객체를 생성하여 이를 의존성으로 전달한다.
개발자는 클래스 A와 클래스 B가 의존성이 있다고 명시하면 스프링 컨테이너는 이 정보를 바탕으로 알아서 객체를 생성하여 의존성 주입까지 담당한다.

엔터프라이즈 애플리케이션에서는 규모가 크므로 당연히 의존하는 클래스의 수가 많을 것이다.
따라서 이 경우 각 객체마다 의존성 주입을 개발자가 담당하게 된다면 휴먼 에러, 중복되는 코드 작성 등의 문제점이 발생할 수 있다.
Spring은 객체의 생성과 삭제를 개발자가 아닌 Container가 담당하여 **개발자는 객체의 생성과 의존성 연결을 스프링에게 맡기고 비즈니스 로직에만 집중**할 수 있다.

## AOP(Aspect Oriented Programming)
엔터프라이즈 애플리케이션을 개발할 때에는 여러 기능에 공통적인 부분 기능이 들어가는 경우가 많다.
예를 들어 보안과 로깅 같은 부분 기능들은 대부분의 기능에 필수적으로 들어간다.
이러한 부분 기능들을 **공통 관심 사항** 그 외 비즈니스 로직을 **핵심 관심 사항**이라고 한다.
이러한 공통 관심 사항을 따로 분리하지 않으면 코드의 길이가 길어지고 유지보수가 어려워진다.

스프링에서는 **공통 관심 사항을 분리하여 코드의 중복을 줄일 수 있는 AOP**를 지원한다.

## PSA(Portable Service Abstraction)
추상화된 인터페이스를 제공함으로써 어떤 서비스를 코드의 큰 수정 없이 이용할 수 있게 하는 기능이다.
스프링에서 지원하는 여러 PSA 서비스 중 Spring Data JPA의 예시를 확인해보자.

엔터프라이즈 애플리케이션을 개발하던 중 MySQL에서 MariaDB로 데이터베이스를 변경해야 한다고 하자.
이 경우 Spring Data JPA를 통해 데이터베이스 연결을 추상화하면 개발 도중에도 데이터베이스의 변경을 쉽게 할 수 있다.
만약 Spring이 Spring Data JPA나 JDBC와 같이 추상화된 인터페이스를 사용하지 않았다면 특정 데이터베이스에 종속되어 POJO에서 설명한 기술 종속과 관련된 문제점이 발생할 수 있다.


# 결론
Spring이 지원하는 POJO, IoC, DI, AOP, PSA 개념 모두 엔터프라이즈 개발을 편하게 해주기 위한 장치이다.
따라서 개발자는 Spring을 이용하면 불필요한 코드의 중복과 로우레벨 코드 작성을 최대한 줄어 비즈니스 로직 작성에만 집중할 수 있다.

각각의 개념에 대한 자세한 내용은 직접 구현해본 MySpringBoot 프로젝트를 바탕으로 소개할 예정이다.

# Reference
[https://okky.kr/articles/415474?note=1302409](https://okky.kr/articles/415474?note=1302409)

[https://www.codestates.com/blog/content/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8](https://www.codestates.com/blog/content/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8)

[https://azderica.github.io/00-java-pojo/#google_vignette](https://azderica.github.io/00-java-pojo/#google_vignette)
