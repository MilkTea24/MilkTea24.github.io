---
title: 운영체제 - System Call
author: milktea
date: 2024-03-08 14:00:00 +0800
categories: [Operating System]
tags: [SystemCall]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/posts/cs/os/services/thumbnail.png
---
# 요약
- System Call은 무엇인가요?

System Call은 사용자 프로그램이 커널의 운영체제 서비스를 호출하기 위해 사용해야 하는 API입니다.
System Call은 커널이 제공하는 운영체제의 서비스를 직접 사용자가 조작하는 일을 방지합니다.
이를 통해 사용자는 운영체제 서비스의 세부 구현을 알 필요 없고 운영체제 서비스를 안전하게 사용할 수 있도록 합니다.

- System Call은 어떻게 구현되나요?

System Call Interface는 사용자 프로그램에서 System Call의 실행을 지원합니다.
System Call Interface는 인덱스와 System Call의 세부 구현이 매핑되어 있어 인덱스를 호출하면 System Call을 실행한 후 결과값을 반환합니다.

이 때 System Call도 API, 즉 함수이므로 매개 변수를 가지는 경우가 있습니다.
매개 변수를 전달하기 위해 여러 방법을 사용할 수 있습니다.
예를 들면 Linux는 메모리에 매개 변수 데이터를 담고 그 주소를 레지스터에 담아 전달하는 방법을 사용합니다.

# 서론
앞서 운영체제 개요에서 운영체제는 사용자가 편리하고 효율적으로 컴퓨터를 사용할 수 있게 한다고 하였다.
이 때 운영체제가 이를 위해 제공하는 모든 것을 **서비스**라고 한다.

운영체제는 정보를 화면에 띄어주는 서비스를 제공하여 우리는 모니터를 통해 정보를 볼 수 있다.
운영체제는 프로그램을 실행하는 서비스를 제공하여 우리는 여러 프로그램들을 실행하여 컴퓨터를 사용할 수 있다.
이 서비스를 호출하기 위해서는 System Call을 거쳐야 한다는 특별한 제약 사항이 있다.

이 포스트에서는 운영체제의 서비스가 무엇인지보다 System Call이 무엇인지, 어떻게 구현되어 있는지 간략하게 알아보기로 한다.


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


### System Call 구현 

![img2.png](/assets/img/posts/cs/os/services/systemcall.png)

**System Call Interface**는 테이블에서 인덱스와 실제 System Call 구현부를 매핑한다.
따라서 실행할 System Call의 인덱스를 입력하면 그 System Call이 실행된다.
이 후 System Call의 실행 결과를 Caller(사용자 프로그램 등)에게 반환한다.

![img3.png](/assets/img/posts/cs/os/services/systemcall-example.png)

위의 예시는 Linux에서 C 프로그램이 prinf 함수를 실행할 때의 과정을 보여준다.
여기서 standard C library가 System Call Interface로 볼 수 있다.
prinf가 실행되면 standard C library는 write와 같은 적절한 System Call을 실행하고 System Call로 받은 결과를 적절히 가공하여 호출한 사용자 프로그램에 반환한다.

### System Call의 매개변수 전달
이 때 System Call에 입력값을 전달해야 하는 경우도 있을 것이다.
다음과 같은 세가지 방법이 일반적으로 사용된다.

1. 레지스터로 직접 매개 변수를 전달한다. 매개 변수의 용량이 레지스터의 용량보다 클 수 없다는 단점이 있다.
2. Memory에 매개 변수를 담은 후 레지스터에 Memory의 주소를 넣어 전달한다. Linux가 사용하는 방식이다.
3. 매개 변수를 Stack에 넣고(push) 운영체제가 System Call을 실행할 때 가져온다(pop).

### System Call의 종류
아래의 분류에 어떤 운영체제 서비스를 호출한다는 설명은 사실 명확하게 선을 그어 분리할 수는 없다.
따라서 System Call이 커널의 운영체제 서비스를 호출한다는 점에 주목하면 된다.

- Process Control : Program Execution, Resource Allocation 등의 운영체제 서비스를 호출한다.
- File Management : File-system Manipulation 등의 운영체제 서비스를 호출한다.
- Device Management : I/O Operations 등의 운영체제 서비스를 호출한다.
- Information Maintenance : Logging 등의 운영체제 서비스를 호출한다.
- Communications : Communications 등의 운영체제 서비스를 호출한다.
- Protection : Protection and Security 등의 운영체제 서비스를 호출한다.

# 결론
System Call은 운영체제의 서비스를 호출하기 위해 API라는 제약 사항을 두고 사용자가 세부 구현을 알 필요없이 적절하게 사용할 수 있도록 도와준다.

# Reference
2022 1학기 운영체제 수업 자료

Operating System Concepts, 8th Edition. Abraham Silberschatz, Peter Baer Galvin, Greg Gagne. Wiley
