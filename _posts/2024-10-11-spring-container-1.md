---
title: Spring - 스프링 컨테이너 코드로 파악하기 1) 빈 스캔
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

# 1. 객체 컨테이너

스프링 컨테이너는 객체의 생성과 초기화, 빈들의 의존성 주입을 담당한다.
크게 BeanFactory 인터페이스와 ApplicationContext 인터페이스가 존재한다.
ApplicationContext는 BeanFactory의 빈 관리 컨테이너 기능을 확장하여 여러 기능을 추가로 지원하는 인터페이스라고 생각하면 된다.
일반적인 SpringBoot 프로젝트를 수행할 때에는 ApplicationContext 구현체 중 AnnotationConfigServletWebServerApplicationContext를 사용한다.
AnnotationConfigServletWebServerApplicationContext은 어노테이션을 기반으로 객체를 스캔하여 정보를 얻어 등록할 수 있고 웹 서버가 내장되어 있어 단독으로 실행할 수 있다.

## 1) DI, IoC와 스프링 컨테이너

DI, IoC에 대한 설명은 [여기](https://milktea24.github.io/posts/spring-intro/#iocinversion-of-container--didependency-injection)에서도 확인할 수 있다.

IoC는 개발자가 아닌 프레임워크나 컨테이너가 객체의 제어권을 가지는 방식으로 Spring은 이에 대한 여러 구현 방식 중 주로 DI를 활용한다. DI는 외부에서 객체를 생성하여 이를 주입하는 방식으로 의존하는 객체의 생성과 관련된 유지보수가 편해지는 장점이 있다.
Spring에서 IoC에 대한 구현은 DI 뿐만 아니라 FactoryBean에게 객체 생성을 위임하는 방식 또한 있다.
따라서 DI는 IoC의 구현 방법 중 하나라고 설명할 수 있다.


## 2) 빈의 라이프 사이클

> 객체 생성 -> 의존 설정 -> 초기화 -> 소멸

스프링 컨테이너는 위의 모든 라이프 사이클을 관리한다.

## 3) 주요 클래스, 인터페이스

![img_1.png](/assets/img/posts/my-springboot/spring-container-1/img_1.png)

- BeanFactory는 빈을 인스턴스화하고 구성하고 관리하는 컨테이너이다.
- ApplicationContext는 BeanFactory를 확장하여 이벤트 발행, MessageSource(다국어 처리) 등의 고급 기능을 지원한다.
- BeanDefinition은 빈을 생성하기 위한 메타 정보를 보관한다. 빈을 스캔할 때 생성되며 빈 하나당 메타 정보가 하나씩 있어 실제로 빈을 생성할 때 BeanDefinition을 활용한다. 스프링이 다양한 형태의 설정을 지원할 수 있는 이유는 **빈의 메타 데이터를 BeanDefinition으로 추상화**하기 때문이다.
- BeanDefinitionRegistry는 스캔하여 얻은 BeanDefinition을 저장하는 보관소이다.
- AnnotationConfigRegistry는 어노테이션 기반 설정을 지원하는 인터페이스이다. scan 메서드가 있어 패키지를 스캔하여 빈을 검색할 수 있다.

### (1) ListableBeanFactory

```java
//동일한 클래스를 여러 별칭으로 빈을 만드는 경우
@Configuration
public class AppConfig {

  @Bean
  @Qualifier("A")
  public MyClass myClassA() {
    return new MyService();
  }

  @Bean
  @Qualifier("B")
  public MyClass myClassB() {
    return new MyService();
  }
}

//하나의 부모 클래스나 인터페이스를 상속한 빈들이 있을 경우
public abstract class Animal {}

@Component
public class Dog extends Animal {}

@Component
public class Cat extends Animal {}
```

하나의 동일한 클래스를 별칭을 달리하여 여러 빈을 생성하거나, 한 클래스, 인터페이스를 상속한 빈들이 있을 때 관련된 기능은 ListableBeanFactory에서 제공한다.
getBeanNamesForType, getBeansOfType 등 하나의 타입에 여러 가지 빈의 이름을 검색하거나 빈 자체를 얻을 수 있는 기능을 제공한다.


## 4) 컨테이너 초기화

컨테이너가 실행되는 과정들은 AbstractApplicationContext의 [refresh()](https://github.com/spring-projects/spring-framework/blob/0f25c75b9e4a1c2f3385044bcf0183241be30c76/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L587) 메서드에서 확인할 수 있다.
refresh 메서드에서는 빈 생성 뿐만 아니라 BeanFactory 관련 설정, BeanPostProcessor 관련 설정 등 컨테이너를 실행하기 위한 다양한 사전 작업들을 수행한다.

```java
try {
  this.startupShutdownThread = Thread.currentThread();

  StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

  // Prepare this context for refreshing.
  prepareRefresh();

  // Tell the subclass to refresh the internal bean factory.(BeanFactory 재설정)
  ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

  // Prepare the bean factory for use in this context.
  prepareBeanFactory(beanFactory);

  try {
    // Allows post-processing of the bean factory in context subclasses.
    postProcessBeanFactory(beanFactory);

    StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
    // Invoke factory processors registered as beans in the context.(BeanFactoryPostProcessor 실행)
    invokeBeanFactoryPostProcessors(beanFactory);
    // Register bean processors that intercept bean creation.(BeanPostProcessor 등록)
    registerBeanPostProcessors(beanFactory);
    beanPostProcess.end();

    ...

    // Instantiate all remaining (non-lazy-init) singletons.(실제 싱글톤 빈 생성)
    finishBeanFactoryInitialization(beanFactory);

    // Last step: publish corresponding event.
    finishRefresh();
  }
```

- [BeanFactoryPostProcessor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html): 빈을 인스턴스화하기 전에 동적으로 빈 정의를 수정할 수 있다.
  - ConfigurationClassPostProcessor: @ComponentScan을 처리하여 BeanDefinition을 생성할 수 있는 하위 클래스이다.
- [BeanPostProcessor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html): 빈을 초기화하기 전과 초기화 후의 작업을 정의할 수 있다. 이는 빈을 생성한 후 관련 처리를 정의할 수 있다.


---

# 2. 빈 스캔

### (1) @ComponentScan
@ComponentScan 어노테이션으로 하위 패키지를 스캔하여 빈을 찾는다.
@SpringBootApplication 어노테이션에는 **@ComponentScan이 메타 어노테이션으로 존재**하기 때문에 스프링부트 프로젝트에서 @ComponentScan을 명시하지 않아도 빈을 얻을 수 있는 이유다.
또한 @EnableAutoConfiguration과 @Configuration도 메타 어노테이션으로 포함되어 있어 자동으로 빈을 스캔, 등록하고 빈을 설정할 수 있다.

![img_3.png](/assets/img/posts/my-springboot/spring-container-1/img_3.png)

@SpringBootConfiguration에서는 @Configuration을 메타 어노테이션을 가진다.

![img_4.png](/assets/img/posts/my-springboot/spring-container-1/img_4.png)

### (2) ConfigurationClassPostProcessor

[ConfigurationClassPostProcessor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)는 BeanFactoryPostProcessor의 구현체이므로 빈이 생성되기 전에 빈 정의를 등록하는 역할을 한다.
ConfigurationClassPostProcessor의 경우 @Configuration을 포함하여 @ComponentScan 등 여러 어노테이션을 처리하여 빈 정의를 스프링 컨테이너에 등록한다. 다른 BeanFactoryPostProcessor와 비교하여 특별한 점은 **모든 BeanFactoryPostProcessor가 실행되기 전에 먼저 실행되어 BeanDefinition을 등록**한다.

![img_5.png](/assets/img/posts/my-springboot/spring-container-1/img_5.png)

```java
//ConfigurationClassPostProcessor.java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
  ...
  ConfigurationClassParser parser = new ConfigurationClassParser(
    this.metadataReaderFactory, this.problemReporter, this.environment,
    this.resourceLoader, this.componentScanBeanNameGenerator, registry);
}

```
[ConfigurationClassParser](https://docs.spring.io/spring-framework/docs/4.0.0.M1_to_4.2.0.M2/Spring%20Framework%204.2.0.M2/org/springframework/context/annotation/ConfigurationClassParser.html)는 ComponentScan을 즉각적으로 처리하여 BeanDefinition을 등록하기 위해 [ComponentScanAnnotationParser](https://docs.spring.io/spring-framework/docs/3.2.0.M1_to_3.2.0.M2/Spring%20Framework%203.2.0.M1/org/springframework/context/annotation/ComponentScanAnnotationParser.html)을 가지고 있다.

```java
//ComponentScanAnnotationParser.java
public Set<BeanDefinitionHolder> parse(AnnotationAttributes componentScan, String declaringClass) {
  ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(this.registry,
    componentScan.getBoolean("useDefaultFilters"), this.environment, this.resourceLoader);

  ...
 
  return scanner.doScan(StringUtils.toStringArray(basePackages));
}
```

[ClassPathBeanDefinitionScanner](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ClassPathBeanDefinitionScanner.html)가 ComponentScanAnnotationParser 내부에서 스캔을 하면서 그 결과로 BeanDefinition들을 반환한다.
이 BeanDefinition들은 이후 BeanFactory에서 실제 빈으로 생성되고 관리된다.



# Reference

[https://mangkyu.tistory.com/210](https://mangkyu.tistory.com/210)

[https://pplenty.tistory.com/6](https://pplenty.tistory.com/6)
