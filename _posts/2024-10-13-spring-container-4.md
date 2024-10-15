---
title: Spring - 스프링 컨테이너 코드로 파악하기 4) 초기화, 소멸
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

# 1. 초기화, 소멸 개념

빈의 생성 후 초기화와 소멸 전 콜백은 다양한 상황에서 활용할 수 있다
예를 들어 빈이 외부 시스템이나 리소스를 사용할 때 빈을 생성할 때 연결하는 코드를 작성하고 소멸 전에 리소스를 닫는 작업 등을 명시할 수 있다.
초기화와 소멸 전 콜백은 빈의 라이프 사이클 동안 딱 **한 번** 수행 됨이 보장된다.

빈의 초기화와 소멸은 크게 세 가지 방식이 존재한다.

## 1) 인터페이스 활용
```java
public interface initializingBean {
  void afterPropertiesSet() throws Exception;
}

public interface DisposableBean {
  void destroy() throws Exception;
}
```

빈이 initializingBean 인터페이스를 구현하고 afterPropertiesSet을 정의함으로써 초기화를 구현할 수 있다.
소멸은 Disposablebean 인터페이스를 구현하고 destroy를 정의하면 구현할 수 있다.
하지만 이러한 방식은 Spring에 종속된 인터페이스를 구현해야 하므로 비즈니스 로직이 Spring에 종속되어 POJO를 만족하지 않는다.
그러므로 다른 프레임워크에서 재사용할 수 없다.

## 2) AutoCloseable 구현

빈이 AutoCloseable을 구현하는 경우 close() 메서드가 소멸 콜백으로 자동으로 실행된다.
AutoCloseable은 자바 표준 인터페이스이므로 특정 프레임워크에 종속되지 않는다.

## 3) 초기화, 소멸 메서드 지정

```java
@Configuration
static class Config {
  @Bean(initMethod = "init", destroyMethod = "destroy")
  public BeanA beanA() {
    return beanA;
  }
}

public class BeanA {
  public void init() throws Exception {
    System.out.println("init");
  }
  
  public void close() throws Exception {
    System.out.println("destroy");
  }
}
```

@Configuration 설정으로 어떤 메서드가 초기화 메서드, 소멸 메서드인지 지정할 수 있다.
메서드의 이름은 어떠한 이름이라도 사용할 수 있다.
이 방법은 외부 라이브러리에도 초기화, 소멸 동작을 지정할 수 있는 장점이 있다.

특이한 점은 destroyMethod는 생략할 수 있다. 
생략하게 된다면 스프링 컨테이너는 close(), shutdown()라고 이름 붙은 메서드를 자동으로 찾아서 등록한다.

## 4) @PostConstruct, @PreDestroy

```java
public class BeanA {
  @PostConstruct
  public void init() throws Exception {
    System.out.println("init");
  }
    
  @PreDestroy
  public void close() throws Exception {
    System.out.println("destroy");
  }
```

간편하게 초기화 메서드, 소멸 메서드를 지정할 수 있다.
또한 @PostConstruct는 스프링이 아닌 Jakarta EE 표준이므로 Spring 컨테이너가 아닌 다른 컨테이너에서도 동작한다.
따라서 POJO의 장점을 그대로 가져올 수 있다.

하지만 외부 라이브러리에는 지정할 수 없으므로 외부 라이브러리에 초기화, 소멸 동작을 지정하기 위해서는 두 번째 방법을 사용하는 것이 좋다.



---

# 2. 생성 후, 소멸 전 콜백 관련 코드

@PostConstruct 및 @PreDestroy를 활용한 전처리, 후처리 과정은 CommonAnnotationBeanPostProcessor가 담당한다.
CommonAnnotationBeanPostProcessor의 부모 클래스인 InitDestroyAnnotationBeanPostProcessor에서 [코드](https://github.com/spring-projects/spring-framework/blob/main/spring-beans/src/main/java/org/springframework/beans/factory/annotation/InitDestroyAnnotationBeanPostProcessor.java#L216)를 확인할 수 있다.

반면 InitializingBean을 구현하는 방식과 @Bean(initMethod=)로 지정하는 방식은 AbstractAutowireCapableBeanFactory에 구현되어 있다.
DisposableBean과 @Bean(destroyMethod=) 방식은 DisposableBeanAdapter 클래스에서 구현된다.

## 1) 초기화 시 인터페이스 구현, 메서드 지정 방법 처리

AbstractAutowireCapableBeanFactory의 [invokeInitMethods](https://github.com/spring-projects/spring-framework/blob/b3cc9a219e79cd7c99c528229be502f55fa9c97b/spring-beans/src/main/java/org/springframework/beans/factory/support/AbstractAutowireCapableBeanFactory.java#L1841)에서 초기화 메서드를 찾고 실행할 수 있다.

```java
protected void invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd)
		throws Throwable {
    
 //InitializingBean을 구현하는 빈이면
	boolean isInitializingBean = (bean instanceof InitializingBean);
	if (isInitializingBean && (mbd == null || !mbd.hasAnyExternallyManagedInitMethod("afterPropertiesSet"))) {
		if (logger.isTraceEnabled()) {
			logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
		}
		((InitializingBean) bean).afterPropertiesSet();
	}

	if (mbd != null && bean.getClass() != NullBean.class) {
		String[] initMethodNames = mbd.getInitMethodNames();
		if (initMethodNames != null) {
			for (String initMethodName : initMethodNames) {
				if (StringUtils.hasLength(initMethodName) &&
						!(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
						!mbd.hasAnyExternallyManagedInitMethod(initMethodName)) {
           //@Bean(initMethod = ) 방식이면
					invokeCustomInitMethod(beanName, bean, mbd, initMethodName);
				}
			}
		}
	}
}
```

## 2) 소멸 시 인터페이스 구현, 메서드 지정 방법 처리

[DisposableBeanAdapter](https://docs.spring.io/spring-framework/docs/5.0.14.RELEASE_to_5.0.15.RELEASE/Spring%20Framework%205.0.14.RELEASE/org/springframework/beans/factory/support/DisposableBeanAdapter.html)에서 소멸 처리가 구현되어 있다.

```java
	public void destroy() {
		if (!CollectionUtils.isEmpty(this.beanPostProcessors)) {
			for (DestructionAwareBeanPostProcessor processor : this.beanPostProcessors) {
				processor.postProcessBeforeDestruction(this.bean, this.beanName);
			}
		}

		if (this.invokeDisposableBean) {
      ...
			try {
        //DisposableBean 인터페이스 구현하는 경우 destroy 실행
				((DisposableBean) this.bean).destroy();
			}
			catch (Throwable ex) {
        ...
			}
		}

		if (this.invokeAutoCloseable) {
      ...
			try {
        //빈이 AutoCloseable을 구현하고 있으면 close 메서드 실행
				((AutoCloseable) this.bean).close();
			}
			catch (Throwable ex) {
        ...
			}
		}
    //@Bean(destroyMethod=)방식을 구현하고 있으면 커스텀 메서드 실행
		else if (this.destroyMethods != null) {
			for (Method destroyMethod : this.destroyMethods) {
        
				invokeCustomDestroyMethod(destroyMethod);
			}
		}
		else if (this.destroyMethodNames != null) {
			for (String destroyMethodName : this.destroyMethodNames) {
				Method destroyMethod = determineDestroyMethod(destroyMethodName);
				if (destroyMethod != null) {
					destroyMethod = ClassUtils.getPubliclyAccessibleMethodIfPossible(destroyMethod, this.bean.getClass());
					invokeCustomDestroyMethod(destroyMethod);
				}
			}
		}
	}
```

DisposableBeanAdapter의 [inferDestroyMethodsIfNecessary](https://github.com/spring-projects/spring-framework/blob/b3cc9a219e79cd7c99c528229be502f55fa9c97b/spring-beans/src/main/java/org/springframework/beans/factory/support/DisposableBeanAdapter.java#L412) 메서드에서 DestroyMethodName을 지정하지 않아도 close와 shutdown 메서드를 자동으로 찾아 소멸 전 콜백으로 지정하는 코드를 찾을 수 있다.

```java
if (destroyMethodName == null) {
	destroyMethodName = beanDefinition.getDestroyMethodName();
	boolean autoCloseable = (AutoCloseable.class.isAssignableFrom(target));
	if (AbstractBeanDefinition.INFER_METHOD.equals(destroyMethodName) ||
			(destroyMethodName == null && autoCloseable)) {
		// Only perform destroy method inference in case of the bean
		// not explicitly implementing the DisposableBean interface
		destroyMethodName = null;
		if (!(DisposableBean.class.isAssignableFrom(target))) {
      //AutoCloseable을 구현하면 close() 메서드를 사용
			if (autoCloseable) {
				destroyMethodName = CLOSE_METHOD_NAME;
			}
			else {
				try {
            //DestroyMethodName이 null이면 close() 메서드를 찾음
					destroyMethodName = target.getMethod(CLOSE_METHOD_NAME).getName();
				}
				catch (NoSuchMethodException ex) {
					try {
              //DestroyMethodName이 null이고 close() 메서드도 없으면 shutdown() 메서드를 찾음
						destroyMethodName = target.getMethod(SHUTDOWN_METHOD_NAME).getName();
					}
					catch (NoSuchMethodException ex2) {
						// no candidate destroy method found
					}
				}
			}
		}
	}
	beanDefinition.resolvedDestroyMethodName = (destroyMethodName != null ? destroyMethodName : "");
}
```


## 3) @PostConstruct 생성 후 콜백

생성하고 초기화 전 postProcessBeforeInitialization을 실행하여 @PostConstruct 메서드 동작을 수행한다.

```java
//InitDestroyAnnotationBeanPostProcessor.java
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
	LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass());
	try {
		metadata.invokeInitMethods(bean, beanName);
	}
	catch (InvocationTargetException ex) {
		throw new BeanCreationException(beanName, "Invocation of init method failed", ex.getTargetException());
	}
	catch (Throwable ex) {
		throw new BeanCreationException(beanName, "Failed to invoke init method", ex);
	}
	return bean;
}
```

## 4) @PreDestroy 소멸 전 콜백

소멸 전 invokeDestroyMethod를 실행하여 @PreDestroy와 같이 소멸 전 수행될 동작이 지정된 메서드를 실행한다.

```java
	@Override
	public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
		LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass());
		try {
			metadata.invokeDestroyMethods(bean, beanName);
		}
		catch (InvocationTargetException ex) {
			String msg = "Destroy method on bean with name '" + beanName + "' threw an exception";
			if (logger.isDebugEnabled()) {
				logger.warn(msg, ex.getTargetException());
			}
			else if (logger.isWarnEnabled()) {
				logger.warn(msg + ": " + ex.getTargetException());
			}
		}
		catch (Throwable ex) {
			if (logger.isWarnEnabled()) {
				logger.warn("Failed to invoke destroy method on bean with name '" + beanName + "'", ex);
			}
		}
	}
```


# Reference

[https://blog.naver.com/gngh0101/221691289433](https://blog.naver.com/gngh0101/221691289433)
