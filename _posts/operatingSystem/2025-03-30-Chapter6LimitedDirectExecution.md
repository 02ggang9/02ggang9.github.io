---
published: true
title:  "[운영체제] Chapter 6 - 제한적 직접 실행 원리"
categories:
  - operatingSystem
---

Limited Direct Execution Chapter에서는 User mode, Kernel mode에 관한 내용과 System call 키워드가 등장하면서 CPU 가상화에 대한 내용입니다.

## 기본 원리 : 제한적 직접 실행

제한적 직접 실행은 사용자 프로그램을 효율적으로 실행시키고 **시스템을 안전하게 제어**하기 위한 실행 방식입니다. 시스템을 안전하기 제어해야 하는 이유는 악성 프로세스가 하드웨어의 자원을 장악할 수 있기 때문입니다. 
Kernel mode와 User mode를 도입함으로써 시스템을 안전하게 제어할 수 있습니다.

Kernel mode : 모든 권한을 가진 상태로 특수한 명령어들을 실행할 수 있음 (I/O 작업 또는 시스템 콜)

User mode : Kernel mode와 반대로 제한된 명령어만 사용할 수 있음

시스템 콜은 User Level의 Application이 파일I/O, 프로세스나 스레드 생성, 소켓과 같은 특수한 명령어를 사용하고 싶을 때 사용하는 인터페이스입니다.
시스템 콜을 호출하기 위해서는 trap이라는 특수한 명령어를 실행해야 합니다. trap은 소프트웨어 인터럽트라고도 불리며 인터럽트를 의도적으로 발생시켜 커널모드로 승격하게 만드는 것입니다.

1. Trap 0x80 ID에 있는 System call Trap 실행
2. 커널 모드로 승격 (운영체제가 CPU를 점유)
3. Kernel Stack에 PC, 레지스터 값 등을 저장
4. Trap Table을 참조해 Trap Handler 위치로 분기
5. 작업 완료 후 return-from-trap 특수 명령어를 사용해 Kernel Stack에 있는 값을 pop해서 원래의 프로그램 실행 흐름으로 복귀
6. User mode로 하향

위의 과정에서 Trap Table와 Trap Handler 라는 용어가 나왔습니다.

Trap Table은 트랩 번호에 따라 실행시킬 트랩 핸들러 주소 정보입니다. Trap Table은 운영체제가 부팅될 때 초기화됩니다.
Trap Table을 사용하는 이유는 보안 때문입니다. System Call도 Trap의 한 종류인데, User mode application이 Trap table을 거치지 않고 바로 System call로 점프한다는 것은 커널 내부의 원하는 지점으로 바로 점프한다는 의미이기 때문입니다.

아래의 그림은 위에서 살펴본 과정을 요약해서 나타내줍니다.

![Limited Direct Execution protocol](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fnc1Rr%2FbtqG61X5zP5%2F1EiBJe57GwE0cpUMimKcM1%2Fimg.jpg)

## 프로세스 간 전환

프로세스가 CPU를 점유해서 동작하고 있다는 것은 운영체제는 동작을 멈췄다는 의미입니다. 운영체제는 프로세스를 계속 실행시킬지 다른 프로세스로 전환을 시킬지 결정을 해야 하는 중요한 임무가 있는데, 어떻게 해야 할까요?

### 협조적인 방식 : 시스템 콜 기다리기

동작 중인 프로세스가 System call trap을 실행시켜서 운영체제가 CPU를 점유할 수 있도록 하는 방식입니다. 
또는 프로세스가 접근할 수 없는 메모리 영역에 접근한다거나 값을 0으로 나눈다는 비정상적인 행위를 하는 경우 trap이 발생해 운영체제가 CPU 제어권을 받게 됩니다.
하지만 이 방식은 프로세스가 악의적으로 system call trap을 실행하지 않으면 프로세스에게 계속 CPU 제어권이 있는 문제가 발생합니다.

### 비협조적인 방식 : 운영체제가 전권을 행사

비협조적인 방식에는 타이머 인터럽트(timer interrupt)를 발생시켜 운영체제에게 CPU 제어권을 넘기는 방식이 있습니다. 
부팅될 때 운영체제가 타이머 인터럽트 발생 시 실행해야 할 코드를 알려줍니다. 

## 문맥의 저장과 복원

위에서 배운 2가지 방식을 통해서 운영체제가 CPU 제어권을 넘겨 받으면 계속해서 User level의 프로세스를 실행시킬지 아니면 다른 프로세스가 CPU를 점유해 사용할 수 있도록 할지 스케줄링을 하게 됩니다.
현재 실행 중이었던 프로세스를 다른 프로세스로 전환하기로 결정하면 운영체제는 Context Switch (문맥 교환) 작업을 수행하게 됩니다.
현재 실행 중인 프로세스의 PC, 레지스터 값을 Kernel Stack에 저장하고 실행시킬 프로세스의 Kernel Stack에서 값을 복원하는 과정을 거칩니다.

부팅 할 때 운영체제가 Trap Table을 초기화하고 타이머 인터럽트를 실행하는 과정부터의 그림은 아래와 같습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fn3QlK%2FbtqG5LheuLe%2FxZQyQ1sdyq3tna4JcL3Ov0%2Fimg.jpg)

문맥을 교환할 때는 저수준의 어셈블리 코드로 PC, 레지스터 등을 저장한다고 합니다. 문맥을 교환할 때 걸리는 시간은 3 GHz 프로세서의 경우 1 마이크로초 미만이라고 합니다.

문맥을 교환하는 어셈블리 코드는 아래와 같습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbyU6a4%2FbtqG3kEtdO2%2FwkW0aVTXhAT6K1u0Bvb4r1%2Fimg.jpg)









