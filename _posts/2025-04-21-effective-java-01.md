---
title: Effective Java - 01 정적 팩토리 메서드
author: milktea
date: 2025-03-10 15:00:00 +0800
categories: [Java]
tags: [JVM]
pin: true
math: true
mermaid: true
---

# 1. 정적 팩토리 메서드 개요

정적 팩토리 메서드는 생성자 외 객체를 생성하는 또 하나의 방법이다.
아래와 같이 정적 메서드로 새로운 객체를 생성하는데 기존 생성자 방식과 비교해보자.

```java
//정적 팩토리 메서드
public static Boolean valueOf(boolean b) {
  return b ? Boolean.TRUE : Boolean.FALSE;
}

//생성자
public Boolean(boolean value) {
  this.value = value;
}
```

---

# 2. 정적 팩토리 장점
## 1) 이름을 가질 수 있다

이름을 가진다는 것은 협업과 유지보수에서 꽤나 중요한 특징이 될 수 있다.

```java
//생성자
BigInteger prime = new BigInteger(bitLength, certainty, random);

//정적 팩토리
BigInteger prime = BigInteger.probablePrime(bitLength, random);
```

생성자 방식은 적절한 변수 이름을 가져 BigInteger(int, int, Random)이 소수를 만드는 생성자임을 추측할 수도 있다.
하지만 정적 팩토리 방식이 반환될 객체의 특성을 쉽게 묘사할 수 있다.

또한 생성자 방식은 **메서드의 매개변수 타입과 순서가 동일한 다른 생성자**를 만들 수 없다.
꼭 맞는 예는 아니지만 소수를 생성할 때 어떨 땐 로깅하고 어떨 땐 로깅하지 않도록 해야한다고 하자.

```java
//생성자
BigInteger prime = new BigInteger(bitLength, certainly, random); //로깅함
BigInteger primeWithoutLogging = new BigInteger(bitLength, random, certainly); //로깅하지 않음

//정적 팩토리
BigInteger prime = BigInteger.probablePrime(bitLength, random);
BigInteger primeWithoutLogging = BigInteger.probablePrimeWithoutLogging(bitLength, random);
```

순서를 바꾸는 생성자 방식은 순서에 따라 로깅 방식이 달라짐을 항상 인지하고 있어야 한다.
그렇지 않다면 엉뚱한 생성자를 호출하게 될 것이다.

하지만 정적 팩토리 방식은 이러한 제약이 없고 어떤 정적 팩토리를 호출할지 명확하다.

## 2) 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다

생성자는 호출될 때마다 새로운 인스턴스를 생성해야 하지만 정적 팩토리 방식은 그렇지 않아도 된다.
정적 팩토리는 미리 만든 인스턴스를 반환하거나 캐싱하는 방식으로 불필요한 객체 생성을 피할 수 있다.

이러한 정적 팩토리 방식은 **인스턴스를 통제**할 수 있으며 이러한 인스턴스 통제 개념은 많은 디자인 패턴이나 전략의 근간이 된다.

- 싱글톤 디자인 패턴: 인스턴스를 통제
- 인스턴스화 불가한 객체
- 불변값 클래스: a == b일 때만 a.equals(b)가 성립하도록 할 수 있다
- 플라이웨이트 패턴
- Enum

## 3) 반환 타입의 하위 타입 객체를 반환할 수 있다

정적 팩토리 메서드에서 인터페이스를 반환하게 되면 여러 하위 타입 객체를 반환할 수 있다.

### (1) 인터페이스 정적 메서드 선언 사례

자바 8 이전에는 인터페이스에 정적 메서드를 선언할 수 없었다.
따라서 Type이란 인터페이스를 반환하는 정적 메서드가 필요하면 Types라는 클래스를 만들어 내부에 정적 팩토리로 반환하는 것이 관례였다.

```java
interface Type {}

class Types {
  private Types {}
  
  public static Type getType() {
    //Type의 여러 구현 클래스 반환
  }
}
```

지금은 인터페이스가 정적 메서드를 가질 수 있기 때문에 이러한 클래스를 둘 필요가 없다고 생각할 수 있다.
하지만 여전히 JAVA 21에서도 인터페이스에서 **정적 필드와 정적 멤버 클래스는 public이어야 하기 때문에**
package-private 접근 제한을 두고 싶은 경우 **인스턴스 불가능한 클래스와 정적 팩토리 메서드를 활용**할 수 있다. 

### (2) Collections 사례

Collections는 위의 Type 인터페이스와 Types 클래스 관계를 실제로 도입한 사례로 볼 수 있다.
Collection은 수정 불가나 동기화 등의 기능을 덧붙인 총 45개의 구현 클래스가 있다.
하지만 이 구현체 대부분을 Collections라는 인스턴스 불가능한 클래스와 정적 팩토리 메서드를 통해 얻을 수 있다.

```java
List<String> emptyList = Collections.emptyList();
List<String> singletonList = Collections.singletonList("Hello");
List<String> synchronizedList = Collections.synchronizedList(emptyList);
List<String> unmodifiableList = Collections.unmodifiableList(emptyList);
```

Collections 클래스와 정적 팩토리 메서드를 이용하면 List의 여러 하위 클래스(SingletonList, SynchronizedList 등)을 얻을 수 있다.
이러한 방식의 장점은 다음과 같다.

- Collections 하위 클래스(총 45개)를 직접 공개하지 않기 때문에 외부 노출되는 API를 작게 유지할 수 있다.
- 정적 팩토리 메서드 장점인 이름을 통해 어떤 객체를 얻을 수 있을지 쉽게 확인할 수 있다
- 정적 팩토리를 사용하는 클라이언트는 해당 객체를 인터페이스 만으로 다루게 된다. 위 4가지 방법으로 얻은 다양한 구현체를 클라이언트는 List 인터페이스 만으로 다룬다.

인터페이스에 정적 메서드가 허용된 이후 `List.of` 처럼 인터페이스의 정적 메서드로도 List의 하위 타입을 생성할 수도 있지만
여전히 Collections 또한 많이 쓰인다.

###  

