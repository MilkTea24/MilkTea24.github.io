---
title: 메서드 리팩토링(Composing Methods)
author: milktea
date: 2024-06-24 14:00:00 +0800
categories: [Software Engineering]
tags: [Refactoring]
pin: true
math: true
mermaid: true
---

# 요약
- Extract Method는 언제 사용하고 어떤 효과가 있나요?

메서드의 길이가 길다면 일부를 새로운 메서드로 추출하고 전체 메서드 크기를 작게 유지할 수 있습니다.
이 경우 코드의 가독성이 좋아지고 추출한 메서드를 다른 메서드에서 재활용할 수 있다는 장점이 있습니다.

- Introduce Explaining Variable는 언제 사용하고 어떤 효과가 있나요?

대부분의 상황에서 Extract Method는 효과적이지만 지역 변수가 많을 때 메서드로 추출하면 추출한 메서드로 값을 전달하고 받기 복잡해지는 문제가 있습니다.
이 때에는 Explaining Variable을 도입하면 메서드를 도입하지 않고 가독성을 향상할 수 있습니다.
이후 Replace Temp with Query나 Replace Method with Method Object 등을 도입하여 메서드를 더 작게 유지하며 가독성을 향상할 수 있습니다. 

- Replace Temp with Query는 언제 사용하고 어떤 효과가 있나요?

지역 변수는 다른 메서드에서 사용할 수 없기 때문에 재사용성이 떨어지는 문제가 있습니다.
이 경우 지역 변수의 결과를 반환하는 메서드인 Query를 작성함으로써 이 문제를 해결할 수 있습니다.
여러 메서드에서 Query를 사용하여 재사용할 수 있고 계산 방법이 변경되더라도 Query의 코드만 변경하면 되므로 수정 용이성(Modifiability)도 향상됩니다.

- Replace Method with Method Object는 언제 사용하고 어떤 효과가 있나요?

지역 변수가 아주 많은 경우 Replace Temp with Query를 수행하면 많은 Query가 필요해 클래스의 메서드가 많아지는 문제가 있습니다.
이 때 이 메서드를 독립된 객체로 분리하면 지역 변수가 객체의 필드가 되므로 쉽게 Extract Method를 적용할 수 있습니다.


# 서론
Refactoring 책에 소개된 모든 메서드 리팩토링(Composing Methods) 방법은 다음과 같다.
- Extract Method
- [Inline Method](https://refactoring.guru/inline-method)
- [Inline Temp](https://refactoring.guru/inline-temp)
- Replace Temp with Query
- Introduce Explaining Variable
- [Split Temporary Variable](https://refactoring.guru/split-temporary-variable)
- [Remove Assignments to Parameters](https://refactoring.guru/remove-assignments-to-parameters)
- Replace Method with Method Object
- [Substitute Algorithm](https://refactoring.guru/substitute-algorithm)

이 모든 방법 중 수업에서 비중있게 다뤘던 4가지를 중심으로 소개할 예정이다.
이 포스트에서 설명하지 않는 리팩토링 방법은 링크를 따로 달았다.


# Extract Method

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

# Introduce Explaining Variable
## 문제 상황
```java
if ((platform.toUpperCase().indexOf("MAC") > -1)&&
      (browser.toUpperCase().indexOf("IE") > -1)&&
       wasInitialized() && resize > 0 )
```
이 코드를 보고 어떤 조건문인지 한번에 파악하기는 쉽지 않다.
조건이 더 복잡해지거나 조건의 수가 더 많아진다면 코드는 더 이해하기 어려울 것이다.

대부분의 상황에서는 Extract Method를 적용할 수 있다.
하지만 지역 변수가 많은 경우 Extract Method를 적용하기 어려울 수 있다.
이 경우는 Introduct Explaining Variable로 해결할 수 있다.

> If I’m in an algorithm with a lot of local variables, I may not be able to easily use Extract Method (110). In this case I use Introduce Explaining Variable (124) to help me understand what is going on.

## 해결하기
```java
final boolean isMacOs = platform.toUpperCase().indexOf("MAC") > -1;
final boolean isIEBrowser = browser.toUpperCase().indexOf("IE") > -1;
final boolean wasResized = resize > 0;

if (isMacOs && isIEBrowser && wasInitialized() && wasResized)
```
final로 선언된 임시 변수를 설정하고 이 임시 변수에 연산의 결과를 대입한다.
이렇게 의미 있는 변수명을 통해 코드의 가독성을 향상시킬 수 있다.
또한 이 Introduce Explaining Variable은 **다른 리팩토링 방법을 적용하기 전 중간 단계로 사용**할 수 있는 특징이 있다.
의미 있는 변수로 가독성이 향상되면 Replace Temp with Query나 Replace Method with Method Object를 더 쉽게 적용할 수 있다.

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
하지만 이 때 basePrice라는 임시 변수는 메서드 내에서만 유효하기 때문에 다른 메서드에서 basePrice를 사용할 때 코드가 중복된다.
계산을 위한 코드가 중복될 경우 계산 방법이 변경될 때 모든 코드를 일일히 변경해야 하므로 유지보수성이 떨어지게 된다.

이 때 basePrice를 클래스의 필드로 선언하는 방법을 고려해볼 수 있지만 이 방법도 상황에 따라 적절하지 않을 수 있다.
basePrice가 특정 메서드 내에서만 한정적으로 사용하는 변수라면 이를 필드로 만드는 방법은 객체의 상태를 복잡하게 만든다.
객체의 필드는 객체가 필수적으로 가져야 하는 속성을 저장하는 만큼 특정 메서드 내에서만 한정적으로 사용하는 변수를 필드로 선언하는 방법은 좋지 않다.

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
또한 basePrice를 계산하는 방법이 변경되더라도 basePrice 메서드 내 코드만 한번 변경하면 되므로 기존보다 수정용이성(Modifiability)이 향상된다.

이 경우 연산이 여러 번 수행되므로 개발 시에는 성능과 유지보수 사이 적절한 균형을 맞추어 결정하는 것이 중요하다.
그렇지만 여러 현대 컴파일러는 이 상황에 대한 간단한 최적화도 제공하므로 일반적인 경우 리팩토링을 고려하는 것이 더 좋다.

# Replace Method with Method Object
## 문제 상황
```java
class Order {
  ...

    double price() {
        double primaryBasePrice;
        double secondaryBasePrice;
        double tertiaryBasePrice;
        // long computation;
      ...
    }
}
```
위와 같이 많은 지역 변수를 사용하는 길이가 긴 price 메서드가 있다.
이렇게 메서드의 길이가 긴 경우 가장 먼저 Extract Method를 생각할 수 있다.
하지만 이 상황에서는 지역 변수가 많기 때문에 Extract Method를 적용하기는 어렵다.
아래처럼 Extract Method를 적용하면 추출한 메서드에 전달할 매개 변수의 수가 많아지게 되어 로직이 복잡해진다.

```java
class Order {
  ...

    double price() {
        double primaryBasePrice;
        double secondaryBasePrice;
        double tertiaryBasePrice;
        computationFirstStep(
                primaryBasePrice, secondaryBasePrice, tertiaryBasePrice
        );
        computationSecondStep(
                primaryBasePrice, secondaryBasePrice, tertiaryBasePrice
        );
      ...
    }
}
```

Extract Method 대신 Replace Temp with Query를 적용하면 어떨까?
일반적으로 Replace Temp with Query로 효과를 볼 수 있지만 때로는 메서드가 많이 복잡하여 분리하기 어려울 수 있다.
임시 변수가 6개라면 이 임시 변수를 대체하는 모든 쿼리를 하나씩 만들어 클래스 내에 메서드가 많아지게 된다.
이러한 상황이라면 이 복잡한 메서드를 아예 새로운 클래스를 만드는 방법이 더 좋을 수 있다.


## 해결하기
```java
class Order {
    double price() {
        return new PriceCalculator(this).compute();
    }
}

class PriceCalculator {
  double primaryBasePrice;
  double secondaryBasePrice;
  double tertiaryBasePrice;
  
  compute() {
      computationFirstStep();
      computationSecondStep();
  }
}
```
위와 같이 클래스로 분리하면 **메서드의 지역 변수가 클래스의 필드로 정의된다**.
이 때 Extract Method를 적용하면 매개 변수로 지역 변수를 전달하는 대신 클래스의 필드로 접근할 수 있어 코드가 더 간결해진다.

# 결론
메서드의 길이가 길다면 대부분 Extract Method를 적용하여 코드의 가독성과 재사용성을 향상할 수 있다.
하지만 로컬 변수가 많은 경우 메서드로 추출하기 어렵다는 문제가 있다.
이 때에는 Introduce Explaining Variable를 먼저 적용한다.
이후 메서드의 로컬 변수가 아주 많고 메서드의 길이가 길다면 Replace Method with Method Object를, 그렇지 않다면 Replace Temp with Query를 사용하여 메서드 길이를 최소화하고 유지보수성을 향상할 수 있다.

이렇게 메서드를 리팩토링하면 메서드의 크기가 작아진다.
그러므로 가독성이 향상되고 메서드를 재사용 할 수 있는 가능성이 높아진다.
따라서 코드를 작성할 때 메서드가 길어진 경우 리팩토링을 항상 염두해 두어야 한다.

# Reference
2024 1학기 소프트웨어시스템설계 수업 자료

Refactoring: Improving the Design of Existing Code 1판 - Martin Fowler, Kent Beck 외 3인, Addison-Wesley Professional

[https://refactoring.guru/refactoring/techniques/composing-methods](https://refactoring.guru/refactoring/techniques/composing-methods)
