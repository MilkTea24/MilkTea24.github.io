---
title: Spring - 스프링 컨테이너 코드로 파악하기 3) 의존성 주입과 전처리
author: milktea
date: 2024-10-11 10:00:00 +0800
categories: [Spring]
tags: [Container, ApplicationContext]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/posts/thumbnail/spring.png
---

> 상당 부분이 직접 코드를 분석하여 정리한 내용이라 틀린 부분이 있을 수도 있습니다! 잘못된 부분은 알려주시면 바로 수정하겠습니다.🙇‍♂️
{: .prompt-warning }

# 1. 의존성 정리

모든 의존성 주입 공통으로 하나의 클래스나 인터페이스를 상속받는 빈이 여러 개 있을 경우 그 클래스나 인터페이스에 의존성을 주입할 때 @Qualifier로 정확한 빈을 지정해주어야 한다.
부모 클래스를 상속받는 빈이 여러 개 있는데 스프링이 결정할 수 없기 때문이다.

## 1) 필드 주입
```java
@Controller
public class ControllerA {
  @Autowired
  private ServiceA serviceA;
}
```

필드에 @Autowired 어노테이션을 달아 필드에 의존성을 주입하는 방식이다.
간단하게 의존성 주입을 정의할 수 있는 장점이 있다.
하지만 필드 주입은 순환 참조를 허용하고 테스트 시 스프링 컨테이너의 도움 없이는 주입이 어렵다는 단점이 있다.
그리고 필드 주입, 세터 주입 공통으로 final 키워드로 불변성을 보장할 수 없다는 문제가 있다.
주입 시점이 생성 후이기 때문에 final 키워드를 붙일 수 없다.

## 2) 세터 주입
```java
@Controller
public class ControllerA {
  private ServiceA serviceA;
  
  @Autowired
  public void setServiceA(ServiceA serviceA) {
    this.serviceA = serviceA;
  }
}
```

세터에 @Autowired 어노테이션을 붙여 의존성을 주입하는 방식이다.
스프링 컨테이너 없이도 setter를 사용하여 테스트 시 의존성을 쉽게 주입할 수 있다는 장점이 있다.
하지만 여전히 순환 참조를 허용하고 불변성을 보장할 수 없다는 문제가 있다.
특히 개발자가 setter를 실수로 호출하면 객체가 변경될 수 있다는 큰 문제점이 존재한다.

## 3) 생성자 주입
```java
@Controller
public class ControllerA {
  private final ServiceA serviceA;
  
  @Autowired
  public ControllerA(ServiceA serviceA) {
    this.serviceA = serviceA;
  }
}
```

앞의 두 의존성 주입과 달리 생성 시점에 의존성 주입도 같이 수행된다.
따라서 final 키워드로 불변성을 보장할 수 있으며 생성자를 이용하여 테스트 시 주입이 간단하다는 장점이 있다.
또한 순환 참조를 허용하지 않으므로 개발자가 의도하지 않은 순환 참조로 인한 위험이 감소한다는 장점 또한 있다.


---

# 2. 의존 설정 관련 코드

## 1) 순환 참조 탐지하기

DefaultSingletonBeanRegistry의 [isDependent](https://github.com/spring-projects/spring-framework/blob/b3cc9a219e79cd7c99c528229be502f55fa9c97b/spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultSingletonBeanRegistry.java#L503)에서 순환 참조를 감지한다.
isDependent는 종속된 빈이 해당 빈이나 그 빈의 이행적 의존성(간접 의존) 중 하나에 종속되었는지 확인하는 메서드다.


```java
//AbstractBeanFactory의 doGetBean
String[] dependsOn = mbd.getDependsOn();
if (dependsOn != null) {
  for (String dep : dependsOn) {
    if (isDependent(beanName, dep)) {
        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
    }
registerDependentBean(dep, beanName);
...
  }
}
  

private boolean isDependent(String beanName, String dependentBeanName, @Nullable Set<String> alreadySeen) {
	//이미 의존성 탐색한 빈이면 더이상 할 필요 없음
  if (alreadySeen != null && alreadySeen.contains(beanName)) {
		return false;
	}
  
  //해당 빈의 의존성을 찾음
	String canonicalName = canonicalName(beanName);
	Set<String> dependentBeans = this.dependentBeanMap.get(canonicalName);
	if (dependentBeans == null || dependentBeans.isEmpty()) {
		return false;
	}
  
  
	if (dependentBeans.contains(dependentBeanName)) {
		return true;
	}
  
	if (alreadySeen == null) {
		alreadySeen = new HashSet<>();
	}
	alreadySeen.add(beanName);
	for (String transitiveDependency : dependentBeans) {
		if (isDependent(transitiveDependency, dependentBeanName, alreadySeen)) {
			return true;
		}
	}
	return false;
}
```

isDependent를 보면 DFS의 사이클 탐지와 비슷하게 의존성을 확인하는 점을 알 수 있다.

> 다만 isDependent(beanName, dep)가 아닌 isDependent(dep, beanName)처럼 반대 방향 의존성을 검사하여 순환 참조를 감지 해야 되지 않나..?라는 의문이 든다.
왜 isDependent(beanName, dep)로 순환 참조를 탐지하는지 정확한 이유를 찾지 못했다. {: .prompt-warning }

## 2) 순환 참조를 해결하는 캐시

처음에는 필드 주입과 세터 주입이 순환 종속성을 탐지할 수 없다고 알고 있었다.
하지만 doCreateBean의 코드를 확인하니 기존 알고있었던 지식과 다른 점을 찾을 수 있었다.
스프링은 필드 주입과 세터 주입에서 순환 종속성을 탐지할 수 없는 것이 아니라 **스프링 내부에서 순환 참조가 있더라도 의존성을 주입할 수 있도록 따로 처리를 해주는 것이였다.**

```java
  	boolean earlySingletonExposure = (mbd.isSingleton() 
  && this.allowCircularReferences 
  && isSingletonCurrentlyInCreation(beanName));
```

[DefaultSingletonBeanRegistry](https://github.com/spring-projects/spring-framework/blob/main/spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultSingletonBeanRegistry.java)(DefaultListableBeanFactory의 4번째 부모 클래스)는 순환 참조가 있더라도 의존성을 주입해 줄 수 있도록 캐시 역할을 하는 여러 필드를 가지고 있다.

- `singletonObjects` 필드: 완전히 생성된 싱글톤 객체를 저장하는 첫번째 캐시
- `earlySingletonObjects` 필드: 'early' 싱글톤 객체를 저장하는 두번째 캐시. 아직 완성이 되지 않은 객체를 캐싱한다.
- `singletonFactories` 필드: 필요한 시점에 빈을 생성하거나 미완성된 빈을 프록시로 반환하는 싱글톤 팩토리를 저장하는 세번째 캐시. 순환참조를 해결할 때 프록시 처리와 같은 추가적인 과정이 필요한 경우 사용한다.

필드 주입, 세터 주입인 경우 earlySingletonObjects와 singletonFactories를 사용하여 순환 참조 문제를 해결할 수 있다.


## 3) 생성자 주입 과정

생성자 주입은 필드 주입과 달리 빈 생성 시점에서 생성자 주입이 이루어진다.
빈 인스턴스 과정을 더 자세히 확인해보면 아래 코드에서 생성자 주입을 먼저 수행하고 빈의 인스턴스를 생성함을 유추할 수 있다.

```java
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
	// Make sure bean class is actually resolved at this point.
	Class<?> beanClass = resolveBeanClass(mbd, beanName);

  ...

	// Preferred constructors for default construction?
	ctors = mbd.getPreferredConstructors(); //@Autowired 등의 방식으로 지정된 생성자
	if (ctors != null) {
		return autowireConstructor(beanName, mbd, ctors, null);
	}
	// No special handling: simply use no-arg constructor.
	return instantiateBean(beanName, mbd);
	}
```


### (1) 생성자 주입 순환 참조

생성자 주입은 필드 주입과 달리 여러 캐시를 사용해서 순환 참조를 허용하도록 지원하지 않는다.
DefaultSingletonBeanRegistry에서 순환 참조를 체크하는 로직을 참고할 수 있다.

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
  ...
  beforeSingletonCreation(beanName);
  ...
}

protected void beforeSingletonCreation(String beanName) {
	if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
		throw new BeanCurrentlyInCreationException(beanName);
	}
}
```

A -> B -> A 순환 참조가 있다고 가정하자.
1. A는 생성 중인 싱글톤(singletonsCurrentlyInCreation)으로 추가된다.
2. B는 생성 중인 싱글톤으로 추가된다.
3. 다시 A로 이동해서 A를 생성 중인 싱글톤으로 추가하려고 하니 A가 이미 생성 중인 싱글톤으로 등록이 되어 있다.

따라서 A 빈을 가져올 때 해당하는 빈이 없는 상태라면 DefaultSingletonBeanRegistry에서 이를 생성하는데 이 과정에서 순환 참조가 발생하여 BeanCurrentlyInCreationException이 발생하게 된다.

A -> B 참조만 있다고 가정하면

1. A는 생성 중인 싱글톤으로 추가된다.
2. B는 생성 완료된 싱글톤으로 추가된다.
3. A는 생성 완료된 B 빈을 주입받아 생성을 완료한다.

## 4) 필드 주입 과정

A -> B -> C -> D -> A 순환 참조가 있다고 가정하자.

![img_6.png](/assets/img/posts/my-springboot/spring-container-3/img_6.png)

1. doCreateBean()으로 A를 생성한다. 이 때 A는 초기화되지 않았으므로 singletonFactories 캐시에 담는다.
2. doCreateBean() 내에서 populateBean() 메서드를 실행하여 의존성을 주입한다.
3. populateBean() 내에서 doResolveDependency() 메서드를 실행하여 캐시에 의존하는 빈이 있는지 찾는다.
4. 없으므로 doCreateBean으로 B를 생성한다.
5. 이 과정을 반복하여 D까지 생성한다.
6. D를 생성한 doCreateBean()의 populateBean()의 doResolveDependency() 메서드를 실행하면 A를 singletonFactories 캐시에서 찾을 수 있다.

### doResolveDependency() 메서드가 A를 원본 객체로 반환

7. doResolveDependency() 메서드를 실행하여 원본 A를 반환했다면 D는 이를 바탕으로 객체를 완성하고 C는 완성된 D를 바탕으로 객체를 완성하고... 이 과정이 A까지 이루어진다.
8. initializeBean()에서 최종 빈을 결정하는데 원본 A이므로 성공적으로 순환 참조를 처리할 수 있다.

### doResolveDependency() 메서드가 A를 프록시 객체로 반환

A가 AOP 기능이 적용되어 있거나 프록시 객체가 필요한 상황이라면 A를 프록시 객체로 반환한다.

7. 프록시 A를 반환했다면 D는 이를 바탕으로 객체를 완성하고 C는 완성된 D를 바탕으로 객체를 완성하고... 이 과정이 A까지 이루어진다.
8. initializeBean()에서 최종 빈을 결정하는데 이 때 프록시가 여러 번 중첩되면 BeanCurrentlyInCreationException이 발생한다.


# Reference

[https://dev-coco.tistory.com/70](https://dev-coco.tistory.com/70)

[https://www.alibabacloud.com/blog/spring-circular-dependency_600191](https://www.alibabacloud.com/blog/spring-circular-dependency_600191)


