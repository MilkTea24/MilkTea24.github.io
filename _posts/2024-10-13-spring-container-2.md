---
title: Spring - 스프링 컨테이너 코드로 파악하기 2) 빈 생성
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

# 1. 빈 생성

객체 생성은 컨테이너 생성 시 실행되는 AbstractApplicationContext의 [refresh](https://github.com/spring-projects/spring-framework/blob/0f25c75b9e4a1c2f3385044bcf0183241be30c76/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L587) 메서드에서 확인할 수 있다.

refresh() 메서드에서 BeanFactory, BeanPostProcessor 사전 설정 작업들을 수행한 후 아래의 코드에서 빈을 생성한다.

```java
public void refresh() throws BeansException, IllegalStateException {
...

// Instantiate all remaining (non-lazy-init) singletons.(실제 싱글톤 빈 생성)
finishBeanFactoryInitialization(beanFactory);
  ...
}
```

실제 빈을 생성하는 [finishBeanFactoryInitialization](https://github.com/spring-projects/spring-framework/blob/0f25c75b9e4a1c2f3385044bcf0183241be30c76/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L938) 메서드의 일부이다.
빈의 생성은 ConfigurableListableBeanFactory가 담당하고 있다.

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
  ...
  
  // Allow for caching all bean definition metadata, not expecting further changes.
	beanFactory.freezeConfiguration();
  
  // Instantiate all remaining (non-lazy-init) singletons.
	beanFactory.preInstantiateSingletons();
}
```

***beanFactory.preInstantiateSingletons()*** 메서드에서 빈의 메타데이터인 BeanDefinition을 바탕으로 싱글톤 빈의 생성이 이루어진다.

## 1) BeanFactory가 doCreateBean을 호출하는 과정

DefaultListableBeanFactory는 ConfigurableListableBeanFactory를 구현하는데 DefaultListableBeanFactory의 부모 클래스인 AbstractAutowireCapableBeanFactory까지 이동하면 [doCreateBean](https://github.com/spring-projects/spring-framework/blob/b3cc9a219e79cd7c99c528229be502f55fa9c97b/spring-beans/src/main/java/org/springframework/beans/factory/support/AbstractAutowireCapableBeanFactory.java#L554) 메서드를 찾을 수 있다.

1: preInstantitateSingletons가 동일한 클래스의 [preInstantitateSingleton](https://github.com/spring-projects/spring-framework/blob/b3cc9a219e79cd7c99c528229be502f55fa9c97b/spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultListableBeanFactory.java#L1064)을 호출

2: preInstantitateSingleton이 동일한 클래스의 [instantitateSingleton](https://github.com/spring-projects/spring-framework/blob/b3cc9a219e79cd7c99c528229be502f55fa9c97b/spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultListableBeanFactory.java#L1114)을 호출

3: instantitateSingleton에서 최종적으로 동일한 클래스의 [getBean](https://github.com/spring-projects/spring-framework/blob/b3cc9a219e79cd7c99c528229be502f55fa9c97b/spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultListableBeanFactory.java#L369)를 호출한다.

```java
private void instantiateSingleton(String beanName) {
    //팩토리 빈이면
    if (isFactoryBean(beanName)) {
      ...
    }
    //팩토리 빈이 아닌 일반적인 빈이면
    else {
    	  getBean(beanName);
    }
}
```

4: getBean에서 동일한 클래스의 [resolveBean](https://github.com/spring-projects/spring-framework/blob/b3cc9a219e79cd7c99c528229be502f55fa9c97b/spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultListableBeanFactory.java#L515)을 호출한다.

```java
public <T> T getBean(Class<T> requiredType, @Nullable Object... args) throws BeansException {
  ...
		Object resolved = resolveBean(ResolvableType.forRawClass(requiredType), args, false);
  ...
		return (T) resolved;
	}

```

5: resolveBean에서 최종적으로 동일한 클래스의 [resolveNamedBean](https://github.com/spring-projects/spring-framework/blob/b3cc9a219e79cd7c99c528229be502f55fa9c97b/spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultListableBeanFactory.java#L1481)을 호출한다.

6: resolveNamedBean에서 AbstractBeanFactory 클래스의 [getBean](https://github.com/spring-projects/spring-framework/blob/b3cc9a219e79cd7c99c528229be502f55fa9c97b/spring-beans/src/main/java/org/springframework/beans/factory/support/AbstractBeanFactory.java#L221)을 호출한다.

7: AbstractBeanFactory 클래스의 getBean이 [doGetBean](https://github.com/spring-projects/spring-framework/blob/b3cc9a219e79cd7c99c528229be502f55fa9c97b/spring-beans/src/main/java/org/springframework/beans/factory/support/AbstractBeanFactory.java#L239)을 호출한다.

8: doGetBean이 최종적으로 AbstractAutowireCapableBeanFactory의 [createBean](https://github.com/spring-projects/spring-framework/blob/b3cc9a219e79cd7c99c528229be502f55fa9c97b/spring-beans/src/main/java/org/springframework/beans/factory/support/AbstractAutowireCapableBeanFactory.java#L486)을 호출한다.

```java
protected <T> T doGetBean(
			String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)
			throws BeansException {

    String beanName = transformedBeanName(name);
    Object beanInstance;

	// Eagerly check singleton cache for manually registered singletons.
	Object sharedInstance = getSingleton(beanName);
	if (sharedInstance != null && args == null) {
     ...
	}
	else {
		// Fail if we're already creating this bean instance:
		// We're assumably within a circular reference.
     ...
		// Check if bean definition exists in this factory.
		BeanFactory parentBeanFactory = getParentBeanFactory();
		if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
			// Not found -> check parent.
       ...
		}
		if (!typeCheckOnly) {
			markBeanAsCreated(beanName);
		}

		StartupStep beanCreation = this.applicationStartup.start("spring.beans.instantiate")
				.tag("beanName", name);
		try {
			if (requiredType != null) {
				beanCreation.tag("beanType", requiredType::toString);
			}
			RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
			checkMergedBeanDefinition(mbd, beanName, args);
			// Guarantee initialization of beans that the current bean depends on.
            // 의존하는 빈들을 먼저 생성해야 생성할 수 있도록
			String[] dependsOn = mbd.getDependsOn();
            //순환 참조 검사
			if (dependsOn != null) {
				for (String dep : dependsOn) {
					if (isDependent(beanName, dep)) {
						throw new BeanCreationException(mbd.getResourceDescription(), beanName,
								"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
					}
					registerDependentBean(dep, beanName);
					try {
						getBean(dep);
					}
					catch (NoSuchBeanDefinitionException ex) {
             ...
					}
					catch (BeanCreationException ex) {
             ...
					}
				}
			}
        
        // 싱글톤 빈 생성
			// Create bean instance.
			if (mbd.isSingleton()) {
				sharedInstance = getSingleton(beanName, () -> {
					try {
						return createBean(beanName, mbd, args);
					}
					catch (BeansException ex) {
						// Explicitly remove instance from singleton cache: It might have been put there
						// eagerly by the creation process, to allow for circular reference resolution.
						// Also remove any beans that received a temporary reference to the bean.
						destroySingleton(beanName);
						throw ex;
					}
				});
				beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
			}

        //프로토타입 스코프 빈 생성
			else if (mbd.isPrototype()) {
         ...
			}
       
       //유효한 스코프 이름이 아님
			else {
         ...
			}
		}
     //빈 생성 과정 에러 처리
		catch (BeansException ex) {
       ...
		}
		finally {
			beanCreation.end();
			if (!isCacheBeanMetadata()) {
				clearMergedBeanDefinition(beanName);
			}
		}
	}
		return adaptBeanInstance(name, beanInstance, requiredType);
}
```


9: createBean이 doCreateBean(String, RootBeanDefinition, Object[])을 호출한다.

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {
    ...

	try {
     //=====빈 인스턴스 생성=====
		Object beanInstance = doCreateBean(beanName, mbdToUse, args);
		if (logger.isTraceEnabled()) {
			logger.trace("Finished creating instance of bean '" + beanName + "'");
		}
		return beanInstance;
	}
	catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
		// A previously detected exception with proper bean creation context already,
		// or illegal singleton state to be communicated up to DefaultSingletonBeanRegistry.
		throw ex;
	}
	catch (Throwable ex) {
		throw new BeanCreationException(
				mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
	}
}
```

## 2) doCreateBean

doCreateBean에서 실제 객체의 생성, 의존성 주입, refresh 메서드에서 등록된 BeanPostProcessor를 사용하여 빈의 전처리, 후처리가 수행된다.

```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {
  // 1. 빈 생성
	// Instantiate the bean.
   ...
	if (instanceWrapper == null) {
		instanceWrapper = createBeanInstance(beanName, mbd, args);
	}
  ...
  
  // 2. 순환 참조 해결을 위해 완성되지 않은 경우 캐싱
	// Eagerly cache singletons to be able to resolve circular references
	// even when triggered by lifecycle interfaces like BeanFactoryAware.
	boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
			isSingletonCurrentlyInCreation(beanName));
	if (earlySingletonExposure) {
    ...
		addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
	}
		// Initialize the bean instance.
	Object exposedObject = bean;
	try {
    //3. 의존성 주입
		populateBean(beanName, mbd, instanceWrapper);
    //4. 빈 초기화 및 BeanPostProcessor 적용
		exposedObject = initializeBean(beanName, exposedObject, mbd);
	}
	catch (Throwable ex) {
    ...
	}
  
    //5. 순환 참조 해결 시도. ExposedObject가 미완성 빈인지 확인하고 캐시로 해결.
		if (earlySingletonExposure) {
		Object earlySingletonReference = getSingleton(beanName, false);
		if (earlySingletonReference != null) {
			if (exposedObject == bean) {
				exposedObject = earlySingletonReference;
			}
      // 순환 참조 해결할 수 없을 때
			else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
        ...
				if (!actualDependentBeans.isEmpty()) {
					throw new BeanCurrentlyInCreationException(beanName,
							"Bean with name '" + beanName + "' has been injected into other beans [" +
							StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
							"] in its raw version as part of a circular reference, but has eventually been " +
							"wrapped. This means that said other beans do not use the final version of the " +
							"bean. This is often the result of over-eager type matching - consider using " +
							"'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.");
				}
			}
		}
	}
		
  //6. 빈이 소멸할 때 처리할 작업이 있다면 등록
  // Register bean as disposable.
	try {
		registerDisposableBeanIfNecessary(beanName, bean, mbd);
	}
	catch (BeanDefinitionValidationException ex) {
		throw new BeanCreationException(
				mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
	}
		return exposedObject;
}
```

다음 포스트에서 doCreateBean 내부의 빈 라이프사이클 수행 과정을 조금 더 자세하게 알아볼 예정이다.


