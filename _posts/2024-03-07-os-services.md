---
title: 운영체제 서비스
author: milktea
date: 2024-03-07 10:00:00 +0800
categories: [CS]
tags: [OS]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/posts/cs/os/services/thumbnail.png
---
# 요약


# 서론
앞서 운영체제 개요에서 운영체제는 사용자가 편리하고 효율적으로 컴퓨터를 사용할 수 있게 한다고 하였다.
이 때 운영체제가 이를 위해 제공하는 모든 것을 **서비스**라고 한다.

운영체제는 정보를 화면에 띄어주는 서비스를 제공하여 우리는 모니터를 통해 정보를 볼 수 있다.
운영체제는 프로그램을 실행하는 서비스를 제공하여 우리는 여러 프로그램들을 실행하여 컴퓨터를 사용할 수 있다.
이제 이 서비스에 대해서 자세히 알아보자.


# 운영체제 서비스
운영체제의 서비스는 다음과 같이 분류할 수 있다.
- User Interface
- Program Execution
- I/O Operations
- File-system Manipulation
- Communications
- Error Detections
- Resource Allocation
- Logging
- Protection and Security

![img1.png](/assets/img/posts/cs/os/services/services.png)

User Interface는 다른 운영체제의 서비스들과는 달리 사용자, 응용 프로그램과 직접 상호작용한다.
User Interface를 통해 다른 서비스를 사용할 수 있는 것이다.

## User Interface
컴퓨터에게 명령할 때 기계어로 입력해야 한다면 편리성과는 거리가 멀 것이다.
운영체제는 이를 위해 **Shell**이라는 프로그램을 제공하고 있다.
Command Line Shell은 사용자가 입력한 커맨드를 해석하여 OS의 서비스들을 실행할 수 있다.
예를 들어 리눅스의 Command Line Shell에서 mkdir이라는 커맨드를 입력하면 OS Service 중 file System의 디렉토리 생성 기능을 사용할 수 있는 것이다.

이 외에도 운영체제는 GUI, 터치스크린 등의 User Interface를 제공한다.

## System Calls
아까의 예시를 다시 생각해보자. 커맨드를 입력하면 OS Service를 호출할 수 있다고 했었다.
운영체제는 Service를 호출할 때 **System Call**이라는 프로그래밍 인터페이스를 활용한다.
System Call은 운영체제에서 API로 제공되면 이 함수를 실행하면 어떤 기능이 수행될지 미리 정의가 되어 있다.

예시를 정리하면 다음과 같다.
1. 리눅스 Command Line Shell에서 mkdir 커맨드를 입력하면 mkdir 프로그램이 실행된다.
2. mkdir 프로그램은 내부에서 mkdir System Call을 활용하여 디렉토리를 생성한다.

이러한 System Call은 운영체제마다 명세가 달라진다.
Windows에서는 Win32 API, UNIX 계열은 POSIX API, Java Virtual Machine은 Java API라는 System Call 명세가 존재한다.

### System Call이 왜 있을까?
System Call도 API이므로 API가 왜 사용되는지 생각해보면 된다.
우리는 다른 시스템의 API를 사용할 때 이 API의 기능만 알면 된다.
API가 어떤 식으로 구현되어 있는지 전혀 알 필요가 없다.

운영체제도 마찬가지로 하드웨어를 어떻게 조작하는지 자세한 사항은 숨기고 그에 대한 결과만 알려준다.
또한 System Call을 통해서만 하드웨어를 조작할 수 있으므로 의도하지 않은 하드웨어 접근을 막을 수 있는 장점도 있다.

![img1.png](/assets/img/posts/cs/os/services/systemcall.png)

# 결론


# Reference
2022 1학기 운영체제 수업 자료

Operation System Concepts, 8th Edition. Abraham Silberschatz, Peter Baer Galvin, Greg Gagne. Wiley
