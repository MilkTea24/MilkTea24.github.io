---
title: 메서드 리팩토링하기
author: milktea
date: 2024-06-22 04:00:00 +0800
categories: [Software Engineering]
tags: [Refactoring]
pin: true
math: true
mermaid: true
---

# 요약

# 서론

# Extract Method
비슷한 기능을 하는 코드를 메서드로 추출한다.

## 문제 상황
```java
void printAll() {
    //print Banner
    System.out.println("==== Banner ====");
    
    //print Details
    System.out.println("name: " + name);
    System.out.println("amount: " + amount);
  }
```
우리는 주석에 대해서 경계할 필요가 있다.
주석에 이 코드가 어떤 기능을 하는지 작성하는 경우 주석 대신 메서드로 추출할 수 있다는 점을 고려해야 한다.
이렇게 함수 내에 여러 기능을 하는 코드를 메서드로 분리하지 않으면 **응집도**가 떨어져 다음과 같은 문제가 발생한다.

1. 메서드로 추출하지 않으면 다른 메서드에서 재사용할 수 없다.
2. 코드의 가독성이 떨어져 코드의 흐름을 파악하기 어려워진다.

## 해결하기
```java
void printAll() {
    printBanner();
    printDetails(name, amount);
  }
  
void printDetails(String name, double amount) {
    System.out.println("name: " + name);
    System.out.println("amount: " + amount);
  }
```
이처럼 메서드로 분리하면 printAll 함수는 Banner와 Detail을 출력함을 한 눈에 파악할 수 있어 가독성(Analysability)이 향상된다.
또한 printBanner와 printDetails를 다른 메서드에서 사용되어 재사용 가능성(Reusability)이 높아진다.
이 때 코드의 동작 의도를 잘 나타내는 메서드명을 사용해야 코드의 흐름을 파악하기 용이하다.

# Replace Temp with Query
## 문제 상황
```java
    double basePrice = _quantity * _itemPrice;
    if (basePrice > 1000)
        return basePrice * 0.95;
    else
        return basePrice * 0.98;
```
얼핏 보아서는 큰 문제가 없어 보인다.
하지만 이 때 basePrice라는 임시 변수는 메서드 내에서만 유효하기 때문에 다른 메서드에서 basePrice를 계산하는 코드가 중복된다.
계산을 위한 코드가 중복될 경우 계산 방법이 변경될 때 중복된 모든 코드를 일일히 변경해야 하므로 유지보수성이 떨어지게 된다.

이 때 basePrice를 클래스의 필드로 선언하는 방법을 고려해볼 수 있지만 이 방법도 상황에 따라 적절하지 않을 수 있다.
basePrice가 특정 메서드 내에서만 한정적으로 사용하는 변수라면 이를 필드로 만드는 방법은 객체의 상태를 복잡하게 만든다.

## 해결하기
```java
    if (basePrice() > 1000) 
      return basePrice() * 0.95;
    else 
        return basePrice() * 0.98;
    
double basePrice() {
    return _quantity * _itemPrice;
  }
```
이렇게 basePrice를 계산하는 식을 메서드로 선언하게 되면 클래스 내 여러 메서드에서 이를 재사용할 수 있다.
하지만 이 경우 연산이 여러 번 수행되므로 개발 시에는 성능과 유지보수성 사이 적절한 균형을 맞추어 결정하는 것이 중요하다.

# Introduce Explaining Variable
## 문제 상황
```java
if ((platform.toUpperCase().indexOf("MAC") > -1)&&
      (browser.toUpperCase().indexOf("IE") > -1)&&
       wasInitialized() && resize > 0 )
```
이 코드를 보고 어떤 조건문인지 한번에 파악하기는 쉽지 않다.
조건이 더 복잡해지거나 조건의 수가 더 많아진다면 코드는 더 이해하기 어려울 것이다.

이 경우 메서드나 쿼리로 추출하는 방법도 고려할 수 있다.
하지만 어떠한 표현식이 여러 로컬 변수로 구성된 복잡한 식일 때 이를 메서드로 추출할 때는 많은 비용이 든다.

## 해결하기
```java
final boolean isMacOs = platform.toUpperCase().indexOf("MAC") > -1;
final boolean isIEBrowser = browser.toUpperCase().indexOf("IE") > -1;
final boolean wasResized = resize > 0;

if (isMacOs && isIEBrowser && wasInitialized() && wasResized)
```
final로 선언된 임시 변수를 설정하고 이 임시 변수에 연산의 결과를 대입한다.
이렇게 의미 있는 변수명을 통해 코드의 가독성을 향상시킬 수 있다.

여러 로컬 변수로 구성된 복잡한 식일 때는 메서드로 추출하는 방법 대신 먼저 Explaining Variable을 도입한 후 이 임시 변수를 쿼리로 변환할 수 있는지 파악하는 것이 좋을 수 있다.

# Replace Method with Method Object

# 결론
리팩토링은 베드 스멜을 식별하여 개선함으로써 유지보수성을 향상시키는 과정이라 할 수 있다. 
이 때 유지보수성이 좋은 코드란 요구사항의 변경에 대응하기 쉽고 기능을 확장하기 용이한 코드로 볼 수 있다.
다음 포스트에서 여러 리팩토링 방법들을 소개할 예정이다.

# Reference
2024 1학기 소프트웨어시스템설계 수업 자료

[https://iso25000.com/index.php/en/iso-25000-standards/iso-25010/57-maintainability](https://iso25000.com/index.php/en/iso-25000-standards/iso-25010/57-maintainability)
