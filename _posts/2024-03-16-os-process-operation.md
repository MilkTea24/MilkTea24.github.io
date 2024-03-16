---
title: 프로세스(Process)의 연산
author: milktea
date: 2024-03-16 10:00:00 +0800
categories: [CS]
tags: [OS]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/posts/cs/os/process/thumbnail.png
---
# 요약

- 멀티 프로세스는 어떤 장점을 가지나요?

멀티 프로세스 환경은 여러 프로세스를 병렬로 수행할 수 있습니다.
자식 프로세스를 생성하여 부모 프로세스와 또 다른 프로세스를 생성합니다.
이 과정에서 부모 프로세스와 자식 프로세스는 분리된 자원을 사용하므로 독립성을 유지하고 모듈화할 수 있습니다.
하지만 프로세스 사이 통신 비용과 Context Switching 비용이 크므로 이를 잘 고려해야 합니다.

- Linux에서 자식 프로세스의 실행과 종료는 어떻게 수행되나요?

부모 프로세스에서 fork() System Call을 호출하여 자식 프로세스를 생성합니다.
부모 프로세스에서는 fork()는 자식 프로세스의 pid를 반환하고 자식 프로세스에서는 0을 반환합니다.
이를 통해 부모 프로세스와 자식 프로세스가 수행할 작업을 다르게 정의할 수 있습니다.
자식 프로세스가 모든 작업을 수행하면 exit()가 실행됩니다.
이 때 부모 프로세스에서 wait() System Call로 자식 프로세스의 상태를 얻고 자식 프로세스의 메모리 할당을 해제합니다.


# 서론
멀티 프로세스라는 말을 한 번쯤 들어본 적이 있을 것이다.
현대의 CPU는 모두 코어와 쓰레드가 여러 개이다. 
이 말은, 동시에 여러 가지 일을 처리할 수 있다는 말이다.
따라서 멀티 프로세스는 하나의 일을 병렬로 처리함으로써 작업을 더 빨리 수행할 수 있다.

이 멀티 프로세스 작업을 위해 자식 프로세스를 생성한다.

# 자식 프로세스 생성
자식 프로세스도 하나의 프로세스이다.
하나의 프로세스는 독립된 Stack, Heap, Data, Text 메모리 공간을 가진다.
그리고 **하나의 프로세스는 고유한 프로세스 식별자(pid)**를 가진다.
따라서 자식 프로세스는 부모 프로세스와 별도의 메모리 공간을 할당받으며 pid 값도 다르다고 보면 된다.

## LINUX의 자식 프로세스 생성
자식 프로세스는 생성될 때 부모 프로세스의 프로그램과 데이터를 복사하여 별도의 메모리 공간에서 실행한다.
부모 프로세스의 프로그램을 복사한다는 말은 자식 프로세스와 부모 프로세스가 동일한 코드라는 말이다.

자식 프로세스의 생성과 종료를 다이어그램으로 보면 다음과 같다.

![img3.png](/assets/img/posts/cs/os/process-operation/child-process-lifecycle.png)

1. fork() System Call로 새로운 자식 프로세스가 생성된다.
   - 부모 프로세스는 fork() System Call의 반환값으로 자식 프로세스의 pid 값을 얻는다.
   - 자식 프로세스는 생성된 후 fork() System Call의 반환값으로 0을 얻는다.
2. 부모 프로세스는 wait() System Call로 자식이 종료될 때까지 대기한다.
3. 자식 프로세스는 작업을 수행한다.
   - 부모 프로세스에서 자식이 수행할 작업을 if (pid == 0) 블럭 내부에 정의할 수 있다.
   - 또는 exec System Call을 이용하여 자식 프로세스가 수행할 프로그램을 지정할 수 있다.(exec란 System Call은 없고 execl, execlp와 같은 여러 System Call을 묶어 exec 그룹이라고 한다.)
   exec가 수행되면 기존 부모 프로세스에서 물려받은 데이터는 수행할 프로그램으로 완전히 대체된다.
4. 자식 프로세스가 종료되면 부모 프로세스는 작업을 이어서 수행한다.


### 코드
```c
int main() {
  pid_t pid;
  
  /* 자식 프로세스는 fork()의 결과값으로 0 */
  /* 부모 프로세스는 fork()의 결과값으로 자식 프로세스의 pid를 얻는다. */
  pid = fork();
  
  /* 비정상 상황 */
  if (pid < 0) {
    /* 에러 처리 */
    ...
  }
  else if (pid == 0) {
    /* 자식 프로세스는 /bin/ls 프로그램을 실행 */
    execlp("/bin/ls", "ls", NULL);
  else {
    /* 부모 프로세스는 자식이 종료될 때까지 기다림 */
    wait(NULL); 
  }
}
```

exec 그룹 중 하나인 execlp() System Call은 실행되면 부모 프로세스에서 물려받은 데이터를 새로운 프로그램의 데이터로 완전히 교체한다.

### LINUX 프로세스 트리

![img3.png](/assets/img/posts/cs/os/process-operation/process-tree.png)

systemd는 운영체제 실행 시 가장 먼저 실행되는 프로세스이다. 
부팅 및 서비스, 로그 관리를 담당하며 모든 프로세스는 systemd의 자식 프로세스이다.

## Windows 자식 프로세스 생성
Windows는 Linux의 fork와 비슷한 CreateProcess System Call을 가진다.
fork는 부모 프로세스부터 주소 공간을 상속받아 자식 프로세스를 생성하지만 CreateProcess는 자식 프로세스가 생성될 때 새로운 주소 공간을 생성한다.

# 자식 프로세스 종료
프로세스가 모든 작업을 수행하면 exit() System Call을 통해 프로세스를 종료한다.
반면 부모 프로세스가 자식 프로세스를 abort() System Call로 강제로 종료할 수 있다.

프로세스가 마지막 구문까지 실행되면 exit() System Call을 이용하여 종료한다.
이 exit() System Call은 부모 프로세스에게 프로세스 상태 정보를 wait()의 반환 값으로 전달한다.
그리고 프로세스가 할당받은 메모리 공간이 해제된다.

abort() System Call은 여러 상황이 있다.

- 자식 프로세스가 할당된 리소스 이상을 사용한다.
- 특정 OS는 부모 프로세스가 종료될 때 남아있는 자식 프로세스를 종료시킨다.(**Cascading Termination**)

## Orphan Process, Zombie Process
wait() System Call이 적절하게 실행되지 않을 때 Orphan Process와 Zombie Process가 생성될 수 있다.

### Zombie Process
자식 프로세스가 종료되면 부모 프로세스는 wait() System Call로 자식 프로세스를 회수해야 한다.
하지만 부모 프로세스가 이를 회수하지 않으면 커널은 자식이 종료되더라도 부모 프로세스가 접근할 수 있게 이 자식 프로세스의 최소한의 정보를 가지고 있다.
이 최소한의 정보를 좀비 프로세스라 하며 불필요한 메모리 리소스를 사용하게 된다.

### Orphan Process
wait() System Call의 정상적인 호출 없이 부모 프로세스가 종료되면 자식 프로세스는 고아 프로세스가 된다.
이 고아 프로세스도 메모리가 할당되어 있다.
따라서 이를 종료하기 위해 init 프로세스(ex: systemd)를 고아 프로세스의 부모 프로세스로 변경한다.
이 init 프로세스는 주기적으로 wait() System Call을 수행하여 고아 프로세스를 종료한다.

# 멀티 프로세스
자식 프로세스를 이용한 멀티 프로세스는 여러 이점을 가진다.

- 병렬 처리 : 시스템의 작업 능력이 향상된다.
- 자원의 독립성 : 프로세스는 독립적인 공간을 가지므로 자식 프로세스가 부모 프로세스에 영향을 미치지 않는다.
- 리소스 자동 해제 : 자식 프로세스가 종료되면 메모리 공간이 자동으로 해제되므로 리소스 누수를 막을 수 있다.

## Chrome
이전 포스트에서 Chrome을 실행하면 여러 프로세스가 실행됨을 확인할 수 있다.
Chrome은 각 탭마다 별개의 프로세스를 실행한다.
따라서 자원의 독립성이 보장되기 때문에 하나의 탭에 문제가 생겨도 다른 탭에 영향이 가지 않는다.
또한 Render, Plug-in도 별개의 프로세스하여 병렬성과 자원의 독립성을 활용한다.

# 결론
프로세스는 자식 프로세스를 활용함으로써 멀티 프로세싱을 수행한다.
멀티 프로세싱은 자원의 독립성과 병렬 처리와 같은 여러 장점을 가진다.
하지만 프로세스가 자원의 독립성이 보장된다는 말은 한 프로세스에서 다른 프로세스에 접근하기 힘들다고도 볼 수 있다.

하나의 일을 병렬 처리를 수행할 때 서로 정보를 교환해야 할 때 어떻게 할까?
다음 포스트에서 이와 관련하여 IPC를 알아볼 예정이다.

# Reference
2022 1학기 운영체제 수업 자료

Operating System Concepts, 8th Edition. Abraham Silberschatz, Peter Baer Galvin, Greg Gagne. Wiley

[https://kim-dragon.tistory.com/202](https://kim-dragon.tistory.com/202)

[https://velog.io/@buna1592/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C-%EC%A2%80%EB%B9%84-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%EC%99%80-%EA%B3%A0%EC%95%84-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-Zombie-Orphan-Process](https://velog.io/@buna1592/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C-%EC%A2%80%EB%B9%84-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%EC%99%80-%EA%B3%A0%EC%95%84-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-Zombie-Orphan-Process)
