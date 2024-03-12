---
title: 프로세스(Process)
author: milktea
date: 2024-03-11 14:00:00 +0800
categories: [CS]
tags: [OS]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/posts/cs/os/services/thumbnail.png
---
# 요약
- 프로세스는 무엇인가요?

  프로세스는 프로그램이 실행되어 메모리에 적재된 상태입니다.
  이러한 프로세스는 프로세스마다 독립된 메모리 영역을 가집니다.
  메모리 영역을 Stack, Heap, Data, Text 영역으로 세분화할 수 있습니다. 
  프로세스에 관한 여러 정보는 PCB라는 객체가 가지고 있습니다.
  
- 프로세스의 상태는 어떤 것이 있나요?

  프로세스의 상태는 New, Running, Waiting, Ready, Terminated 상태가 있습니다.
  실행중인 프로세스가 Interrupt나 이벤트로 중단되어 상태가 변경될 수 있습니다.
  이러한 과정에서 CPU 사용률을 극대화하기 위해 다른 프로세스로 전환하는 Context Switching이 일어납니다.
  여러 프로세스의 Context Switching은 Process Scheduler가 담당합니다.

# 서론

![img1.png](/assets/img/posts/cs/os/process/chrome.png)

Windows의 작업 관리자를 열면 프로세스를 확인할 수 있다는 사실은 운영체제를 공부하지 않아도 보통 알고 있다.
이 때 구글의 크롬 브라우저 프로세스를 자세히 보면 하나의 프로세스가 아니다.
분명 크롬 브라우저에서 하나의 창을 띄었음에도 13개의 프로세스가 돌아간다!

여기서 궁금증이 생긴다. 프로그램과 프로세스는 어떤 차이가 있을까?
분명 크롬 프로그램을 한번 실행했는데 이 무수히 많은 프로세스들은 무엇일까?
이에 대한 설명은 프로세스부터 멀티 프로세스, 그리고 프로세스 간의 통신까지 하나씩 차근차근 알아볼 것이다.

이번 포스트에서는 프로세스만을 설명한다.

# 프로세스(Process)
우리는 보통 메모장을 클릭하여 실행하면 "프로그램"을 실행한다고 한다.
하지만 실행한 메모장을 작업 관리자의 프로세스에서 확인할 수 있다.
결론부터 말하면 코드인 **프로그램이 실행되어 메모리에 적재되면 이게 바로 프로세스**가 된다.

메모장을 여러 개 열 수 있는 것처럼 하나의 프로그램은 여러 개의 프로세스가 될 수 있다.
또한 크롬의 예시처럼 하나의 기능을 수행함에도 여러 개의 프로세스가 실행될 수 있다.
프로세스는 메모리에 적재되므로 프로세스가 메모리에 어떻게 저장되는지가 중요할 것이다.

## 프로세스의 4가지 구성 요소
프로세스는 메모리 내에서 보통 다음과 같이 4가지 구성 요소를 가진다.
프로세스마다 독립된 4가지 구성 요소를 가지며 이는 나중에 소개할 쓰레드와 다른 점이다.
참고로 멀티쓰레드 환경에서 쓰레드는 Heap, Data, Text 영역을 공유한다.

![img2.png](/assets/img/posts/cs/os/process/process-component.png)

- Stack : 함수 호출과 함께 할당되며 함수의 지역 변수, 매개 변수를 저장한다.
- Heap : 사용자가 동적으로 할당한 영역이다.
- Data : 프로세스 시작과 함께 할당되는 전역 변수와 정적 변수를 저장한다.
- Text : 실행한 프로그램의 코드가 저장된다.

```c
int y = 15; //data 영역

int main(int argc, char * argv[]) {
  int *values; //stack 영역
  
  //malloc으로 할당된 메모리 공간은 heap 영역
  values = (int *)malloc(sizeof(int)*5); 
  ...
}
```

## 프로세스의 상태
프로세스는 active하므로 종료될 때까지 여러 상태를 가질 수 있다.

![img3.png](/assets/img/posts/cs/os/process/process-state.png)

- New: 프로세스가 생성된 상태이다.
- Running: 명령어가 실행되고 있는 상태이다.
- Waiting: 특정 이벤트(I/O 처리 등)로 대기하고 있는 상태이다.
- Ready: 준비가 완료되어 프로세서가 할당만 하면 바로 실행할 수 있는 상태이다.
- Terminated: 프로세스가 종료된 상태이다.

프로세스는 모든 연산을 마칠 때까지 실행되는 게 아니라 Process Scheduling 등의 과정으로 중단될 수 있다.


## PCB
PCB(Process Control Block)는 각 프로세스에 대한 정보를 가지고 있다.
여러 정보가 있는데 이를 대표적으로 나열하면 다음과 같다.

- Process State: 위에서 언급한 프로세스의 상태를 저장한다.
- Program Counter: 다음에 실행할 명령어의 위치를 저장한다.
- Register: 프로세스의 연산에 필요한 메모리 공간이다.

그 외 프로세스 id, IO 상태 정보, 메모리 관리 정보 등 여러 정보를 저장한다.

# Process Scheduling
CPU가 일하지 않고 있는 시간을 줄여야 최대한의 성능을 낼 수 있다.
하지만 프로세스가 I/O 처리를 해야 한다던지 등의 이유로 Waiting 상태가 될 수 있다.
이 때 CPU가 이 프로세스의 이벤트가 끝날 때까지 기다리는 것 보다는 그 동안 다른 프로세스를 처리하면 CPU 사용률을 극대화할 수 있다.

이렇게 **CPU는 처리할 다음 프로세스를 선택해야 하는데 어떻게 선택할 지 Process Scheduler가 관여**한다.
Process Scheduler는 다음에 처리할 프로세스를 저장하기 위해 Ready Queue와 Wait Queue 혹은 혼합된 형태의 Queue를 가진다.
이 Queue는 프로세스의 정보인 PCB를 가지고 있다.

![img4.png](/assets/img/posts/cs/os/process/process-scheduling.png)

1. 프로세스에서 IO 요청이 발생하면 요청이 끝날때까지 Wait Queue에서 대기한다.
2. 프로세스에게 할당된 시간이 지나면 바로 Ready Queue에서 대기한다.
3. 프로세스가 자식 프로세스를 만들면 자식이 종료될 때까지 Wait queue에서 대기한다.
4. Interrupt가 발생하면 기존 프로세스는 Wait Queue에서 대기한다.

Process Scheduler는 기존 실행 중인 프로세스가 Ready Queue나 Wait Queue로 들어가면 Ready Queue에서 대기하고 있는 프로세스를 실행한다.

## Context Switch
한 프로세스에서 다른 프로세스로 전환이 되는 과정을 **Context Switching**라 한다.
Process Scheduler는 CPU 사용률을 극대화하기 위해 Context Switching를 활용한다.
또한 하나의 CPU에 여러 프로세스를 실행하는 멀티태스킹에서도 Context Switching이 일어난다.

우리가 작성하던 문서를 종료하고 다른 문서를 작성해야 한다고 예시를 들어보자.
이 때 문서를 어디까지 작성했는지 저장해야 나중에 다시 작성할려고 불러올 때 문제없이 작성할 수 있다.

프로세스에서 어디까지 실행했는지에 대한 정보가 프로세스의 **Context**이다.
따라서 다른 프로세스를 불러오기 전에 프로세스의 상태와 Context를 PCB에 저장하는 과정을 거쳐야 한다.
이러한 과정은 오버헤드이기 때문에 너무 잦은 Context Switching은 성능 저하가 발생할 수 있다.


# 결론
프로세스는 프로그램이 메모리에 적재되어 실행되는 상태이다. 
프로세스마다 stack, heap, data, text 영역이 독립적으로 존재한다.

프로세스는 처음 실행된 후 I/O 작업이나 interrupt 등으로 중지될 수 있다.
이 때 Process Scheduling으로 기존의 프로세스를 중단하고 대기하고 있는 프로세스를 실행하는 작업을 수행한다.
기존의 프로세스를 중단하고 대기하고 있는 프로세스를 실행하는 과정을 Context Switching이라 한다.

지금까지 소개한 내용이 프로세스의 기본적인 개념이다.
다음 포스트에서 프로세스의 연산인 생성과 삭제에 대해서 알아볼 예정이다.


# Reference
2022 1학기 운영체제 수업 자료

Operating System Concepts, 8th Edition. Abraham Silberschatz, Peter Baer Galvin, Greg Gagne. Wiley
