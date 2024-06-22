---
title: Spring - 위상정렬로 생성자 주입을 지원하는 빈 생성 구현하기
author: milktea
date: 2024-05-10 03:00:00 +0800
categories: [Spring]
tags: [Bean, TopologicalSort]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/posts/thumbnail/spring.png
---


# 서론
[Mini-SpringBoot 프로젝트](https://github.com/MilkTea24/mini-spring-boot)에서 빈 생성을 구현할 때 생성자 주입을 함께 구현해야 하는 상황이였다.
필드 주입은 모든 빈을 생성한 후 의존성을 연결해도 되지만 생성자 주입은 무조건 생성 시 의존하는 빈들을 같이 전달해야 한다.
이를 위해서는 **두 가지 제약사항을 만족**해야 한다.

1. 빈을 생성할 때 의존하는 빈들은 이미 생성된 빈이여야 한다.
2. 생성자 주입은 순환 참조가 발생하면 안된다.

처음에는 어떻게 구현해야 할 지 도저히 생각나지 않았다.
그래서 이 문제를 잠시 내버려두고 백준 문제를 풀던 중 우연히도 위상 정렬 문제와 마주치게 되었다!

1. 선수 과목을 모두 이수해야 이 과목을 이수할 수 있다 => 의존하는 빈들을 모두 생성해야 이 빈을 생성할 수 있다.
2. 위상 정렬은 사이클을 탐지할 수 있다. => 생성자 주입 시 순환 참조를 탐지할 수 있다.

백준 문제를 푼 후 해결 방법이 바로 떠올라 빈 생성 기능을 구현할 수 있었다.

# 빈 생성 구현하기
## 위상 정렬 복습
간단하게 복습하고 넘어가자. 자세한 설명은 [여기](https://milktea24.github.io/posts/algorithm-topological-sorting/)에서 확인할 수 있다.
1. 간선을 저장한다.
2. 간선의 dest를 바탕으로 진입 차수를 초기화한다.
3. 큐에 진입 차수가 0인 노드를 추가한다.
4. 큐에서 하나를 꺼내어 방문처리하고 이와 연결된 인접 노드의 진입 차수를 1 감소시킨다.
5. 3 ~ 4를 큐가 빌 때까지 반복한다.
6. 만약 3~5 과정을 반복한 후에도 방문하지 않은 노드가 있다면 이 그래프는 사이클이 존재한다.

## 1. 간선을 저장한다.

![img.png](/assets/img/posts/my-springboot/topological/bean-graph.png)

SingletonBeanFactory의 [createBeanGraph](https://github.com/MilkTea24/mini-spring-boot/blob/e78b73bcbeee9dcca10b79af76923cdbacda5050/src/main/java/com/milktea/myspring/boot/web/ioc/SingletonBeanFactory.java#L57) 메서드로 의존 정보를 beanGraph에 저장하였다.
위와 같은 그래프를 코드로 나타내면 다음과 같다.

```java
class A {
    B b;    C c;
    
    @Autowired
    public A(B b, C c) { ... }
}

class B {
    D d;
    
    @Autowired
    public B(D d) { ... }
}

class C {
    B b;    E e;
    
    @Autowired
    public C(B b, E e) { ... }
}

class D {}

class E {}

```

처음 BeanDefinition을 스캔할 때 A는 B와 C에 의존한다는 정보가 이미 저장되어 있다.
**이 떄 주의할 점은 위상 정렬의 경우 간선의 방향이 반대이므로 새로 beanGraph에 저장해준다.**
그림의 beanGraph에서 B가 생성되면 A와 C의 진입 차수를 1 감소해야 한다는 것을 알 수 있다.


## 2. 각 클래스의 진입 차수 초기화

![img.png](/assets/img/posts/my-springboot/topological/in-degree.png)

TopologicalSortConstructorDiStrategy의 [createBeans](https://github.com/MilkTea24/mini-spring-boot/blob/e78b73bcbeee9dcca10b79af76923cdbacda5050/src/main/java/com/milktea/myspring/boot/web/ioc/TopologicalSortConstructorDiStrategy.java#L12)에서 저장한 간선의 정보인 beanGraph를 순회하면서 간선의 dest인 클래스인 경우 진입차수를 1 더해주었다.

```java
Map<Class<?>, Long> inDegree = new HashMap<>();

for (Map.Entry<Class<?>, List<BeanDefinition>> entry : beanGraph.entrySet()) {
    if (!inDegree.containsKey(entry.getKey())) inDegree.put(entry.getKey(), 0L);

    for (BeanDefinition bd : entry.getValue()) {
        if (!inDegree.containsKey(bd.getBeanType())) inDegree.put(bd.getBeanType(), 1L);
        else inDegree.put(bd.getBeanType(), inDegree.get(bd.getBeanType()) + 1L);
    }
}
```


## 3. 큐에 진입차수가 0인 노드를 추가한다.

![img.png](/assets/img/posts/my-springboot/topological/queue.png)

저장한 진입 차수를 가진 맵을 순회하면서 진입 차수가 0인 경우 큐에 추가하였다.
또한 진입 차수의 값을 -1로 변경하여 방문 처리를 해준다.
큐에는 생성할 빈의 Class 클래스를 넣는다.

```java
Map<Class<?>, Object> beans = new HashMap<>();
Queue<Class<?>> queue = new LinkedList<>();
        
for (Map.Entry<Class<?>, Long> entry : inDegree.entrySet()) {
    if (entry.getValue() != 0) continue;

    queue.add(entry.getKey());
    inDegree.put(entry.getKey(), -1L);
}
```

## 4. 큐에서 하나 꺼내어 빈을 생성한다.

![img.png](/assets/img/posts/my-springboot/topological/bean.png)

**진입 차수가 0인 클래스인 경우 의존하는 빈들이 모두 생성되었거나 의존하는 빈이 없다는 의미다.** 따라서 빈을 생성할 수 있는 상태이므로 다음과 같은 과정을 거쳐 빈을 생성한다.

1. BeanDefinition에서 @Autowired가 정의된 Constructor를 가져온다.
2. Constructor의 Parameter를 getParameters 메서드로 가져온다.
3. Parameter를 순회하며 이미 생성된 빈과 Parameter 타입이 일치하면 해당 빈을 arguments에 추가한다.
4. Constructor의 `newInstance` 메서드와 가져온 빈들을 이용해서 새로운 빈을 생성하여 생성된 빈 리스트에 추가한다.

```java
//TopologicalSortConstructorDiStrategy.java
    Class<?> current = queue.poll();
    beans.put(current, createBean(beans, beanDefinitions.get(current)));

    ...

private Object createBean(Map<Class<?>, Object> beans, BeanDefinition current) {
    try {
        Constructor<?> autowiredConstructor = current.getAutowiredConstructor();
        if (autowiredConstructor == null) autowiredConstructor = current.getBeanType().getDeclaredConstructor();

        Parameter[] constructorParameters = autowiredConstructor.getParameters();
        List<Object> arguments = new ArrayList<>();

        for (Parameter parameter : constructorParameters) {
            arguments.add(beans.get(parameter.getType()));
        }

        return autowiredConstructor.newInstance(arguments.toArray());
    } catch (Exception e) {
        e.printStackTrace();
        throw new RuntimeException("빈 생성 시 문제가 발생하였습니다.");
    }
}
```

## 5. 인접 노드의 진입차수를 1 감소시킨 후 진입차수가 0인 클래스를 큐에 추가한다.

![img.png](/assets/img/posts/my-springboot/topological/near-node-queue.png)

```java
for (BeanDefinition bd : currentBeanDependencies) {
    inDegree.put(bd.getBeanType(), inDegree.get(bd.getBeanType()) - 1);
}

for (Map.Entry<Class<?>, Long> entry : inDegree.entrySet()) {
    if (entry.getValue() != 0) continue;

    queue.add(entry.getKey());
    inDegree.put(entry.getKey(), -1L);
}
```
빈을 생성한 클래스에 의존하는 클래스가 있다면 빈을 생성하였으므로 의존하는 클래스의 진입차수를 1 감소시켜야 한다.
이 후 진입차수가 0인 클래스는 모든 의존하는 빈들이 생성되었다는 의미이므로 큐에 추가한다.
이 과정을 큐가 빌때까지 반복한다.

## 6. 빈의 순환 참조 탐지

![img.png](/assets/img/posts/my-springboot/topological/cycle.png)

만약 큐가 비었는데 여전히 방문하지 않은 노드가 있다면 순환 참조가 있다는 뜻이므로 예외를 생성한다.
```java
for (Map.Entry<Class<?>, Long> entry : inDegree.entrySet()) {
    if (entry.getValue() == -1) continue;

    throw new CircularReferencesException("순환 참조가 발생하였습니다.");
}
```

# 테스트
## 성공 테스트

![img.png](/assets/img/posts/my-springboot/topological/success-test.png)

### 테스트 케이스

```java
//ClassList.java
public static class D {
        public E e;
        public F f;

        @Autowired
        public D(E e, F f) {
            this.e = e;
            this.f = f;
        }
    }
    
    public static class E {
        public G g;

        @Autowired
        public E (G g) {
            this.g = g;
        }
    }
    
    public static class F {}

    public static class G {}
```

### 테스트 코드
```java
@DisplayName("생성자 주입 빈 주입 성공 테스트")
@Test
void singleton_bean_factory_constructor_di_success_test() {
    //given
    ...
    
    //when
    singletonBeanFactory.createBeans();

    //then
    SingletonBeanRegistry instanceRegistry = singletonBeanFactory.getBeanRegistry();
    ClassList.D instanceD = (ClassList.D) instanceRegistry.getSingleton(ClassList.D.class);
    Assertions.assertNotNull(instanceD.e);
    Assertions.assertNotNull(instanceD.f);
    ClassList.E instanceE = instanceD.e;
    Assertions.assertNotNull(instanceE.g);
}
```

### 실행 결과

![img.png](/assets/img/posts/my-springboot/topological/success-test-result.png)

## 실패 테스트

![img_1.png](/assets/img/posts/my-springboot/topological/circular.png)

### 테스트 케이스
```java
//ClassList.java
public static class H {
  public I i;
  @Autowired
  public H(I i) {
    this.i = i;
  }
}

public static class I {
  public H h;
  @Autowired
  public I(H h) {
    this.h = h;
  }
}
```

### 테스트 코드
```java
@DisplayName("생성자 주입 빈 순환 참조시 실패 테스트")
@Test
void singleton_bean_factory_constructor_di_fail_test() {
    //given
    ...

    //when, then
    Assertions.assertThrows(CircularReferencesException.class, singletonBeanFactory::createBeans);
}
```

### 테스트 결과

![img.png](/assets/img/posts/my-springboot/topological/circular_fail_test.png)

# 결론
이렇게 위상 정렬을 이용하면 빈을 생성해야 할 순서를 정렬하여 하나씩 생성할 수 있다.
의존하는 빈이 생성되었는지 진입 차수를 체크하므로 의존하는 빈이 존재하지 않음에도 빈을 생성하려는 경우를 막을 수 있다.

기존의 프로젝트에서는 알고리즘을 적용할 기회가 많이 없었는데 이번에 내 기준으로 난이도가 있는 알고리즘을 처음 적용해볼 수 있는 좋은 기회가 되었다.
어떻게 구현해야 할지 막막했던 기능이 알고리즘을 사용해서 꽤나 쉽게 풀리는 경험은 나에게 아주 신선했다.
그래서 이 기능은 개인적으로 정말 재미있게 구현했었고 알고리즘 공부를 더 열심히 해야겠다는 생각이 들었다!
