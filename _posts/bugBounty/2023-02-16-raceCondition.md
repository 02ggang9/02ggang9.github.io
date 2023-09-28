# 버그 바운티 | Race Condition

---

## 계기

처음으로 버그 바운티를 해본 날 입니다. DreamHack 사이트를 모의해킹 했는데 프로그램이 시작된지 1년이 지나 많은 보고서가 나와 있었고 해킹을 가르치는 사이트라 보안이 잘 되어 있었습니다. 그래서 취약점을 찾지 못했고 다른 분들의 보고서를 보면서 어떤 취약점이 있었고 공격 루트와 대응 방안이 궁금했습니다. 그 중에서 최고 상금 50만원을 받은 취약점이 바로 **Race Condition** 입니다.

(아래는 50만원 타가신 보고서 입니다.)

[Dreamhack : Race Condition in Content Payment](https://patchday.io/reports/109)

## 이번 버그 바운티로 얻은 경험들

이번 버그 바운티로 정말 많은 경험을 얻었습니다.

- 모의해킹 프로젝트를 실시할 때 주통기반 / 금취분평 가이드로만 점검 해서 버그 바운티도 이 목록을 중심으로 했는데 평가 항목에 없는 취약점도 많았고 중요하구나를 깨달았습니다. 대표적으로 IDOR, Race Condition 등이 있었습니다.
    
    
- 다른 분들의 사고 방식을 이해할 수 있었습니다.
    
    다른 분들의 보고서를 보고 이런 기능에 의문을 가졌고 분석하는 방법과 실현 과정에 대해서 배울 수 있었습니다.
    
- 실제 운영되는 사이트는 정말 복잡하고 분석하는 시간이 길 수 밖에 없구나!
모의해킹 프로젝트를 진행했을 때는 소스 코드의 길이가 짧았고 다들 실제 일하는 개발자의 수준이 아니여서 읽기 쉬웠고 소스 코드의 양도 굉장히 작았습니다.

- SQL Injection 같이 최상위 위험도를 가지고 있는 취약점은 정말 찾기 어렵구나!
모의해킹 프로젝트 보고서를 보면 (대표적으로 SB 커뮤니티, Club Manager 웹 서비스) SQL Injection 취약점을 20개, 25개씩 찾았습니다. 그 이유는 대부분 Prepared Statement를 쓰지 않았기 때문입니다. 하지만 실제 운영되는 사이트는 코드 리뷰를 통해 이런 허점은 존재하지 않겠죠. 또 대학교 교육도 Prepared Statement 사용을 전제로 깔고 진행하기 때문에 이제는 더욱 SQL Injection을 찾기 어려울 것 같다고 개인적으로 새각 했습니다.

## CWE 란?

Common Weakness Enumeration의 약자로, 나열된 단어 자체만 본다면 “일반적인 결함의 나열” 이라고 볼 수 있습니다. CWE는 SQL Injection, XSS, BOF 등 다양한 소프트웨어의 결함을 식별하기 위해 결함의 종류 목록을 체계화하여 제공하고 있습니다. CWE를 사용한다면 보안 전문가 등은 다음과 같은 장점을 얻을 수 있다고 합니다.

- 결함 검사 도구 등 소프트웨어의 보안을 향상시키기 위한 도구의 표준 척도로 사용할 수 있습니다.
- 결함의 원인을 인식하고 결함을 감소시키며, 재발의 방지를 위한 공통의 기준으로서 활용할 수 있습니다.

## Race Condition | CWE-362 정의

CWE-362 : Concurrent Execution using Shared Resource with Improper Synchronization (”Race Condition”)

Race Condition은 경함 조건, 경쟁 상태 등 여러가지로 불리고 있으며, 시스템의 실질적인 동작이 다른 통제 불가능한 이벤트의 순서나 타이밍에 따라 달라지는 전자 장치, 소프트웨어 또는 기타 시스템의 조건입니다.

## 예시 | 은행 입출금 문제

가장 대표적인 Race Condition 문제의 예시입니다. 평소 프로그래밍을 할 때 쓰는 언어(Python, C, Java)는 고급 언어라고 불립니다. 이런 언어로 은행의 입출금 코드를 짜면 다음과 같이 짤 수 있습니다.

```python
balance = balance + amount
balance = balance - amount 
```

이렇게 간단하게 2줄로 짤 수 있습니다. 매우 간단하게 보일 수 있죠. 아래의 코드는 자바 언어로 만든 소스 코드입니다. 

```java
class Test {
	public static void main(String[] args) throws InterruptedException {
		BankAccount b = new BankAccount();
		Parent p = new Parent(b);
		Child c = new Child(b);
		p.start();   // start(): 쓰레드를 실행하는 메서드
		c.start();
		p.join();    // join(): 쓰레드가 끝나기를 기다리는 메서드
		c.join();
		System.out.println("balance = " + b.getBalance());
	}
}

// 계좌
class BankAccount {
	int balance;
	void deposit(int amount) {
		balance = balance + amount;
	}
	void withdraw(int amount) {
		balance = balance - amount;
	}
	int getBalance() {
		return balance;
	}
}

// 입금 프로세스
class Parent extends Thread {
	BankAccount b;
	Parent(BankAccount b) {
		this.b = b;
	}
	public void run() {   // run(): 쓰레드가 실제로 동작하는 부분(치환)
		for (int i = 0; i < 100; i++)
		  b.deposit(1000);
	}
}

// 출금 프로세스
class Child extends Thread {
	BankAccount b;
	Child(BankAccount b) {
		this.b = b;
	}
	public void run() {
		for (int i = 0; i < 100; i++)
		  b.withdraw(1000);
	}
}
```

이 소스코드를 실행하고 나면 아래와 같은 결과를 얻을 수 있습니다.

```java
balance = 0
```

이 결과는 지극히 정상적인 결과라고 볼 수 있습니다. 하지만 실제 운영되는 사이트에서는 시간의 지연이 있기 마련입니다. 또, 저급 수준의 언어 (어셈블리어)로 코드를 변경하게 되면 두 줄로 끝나지 않습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1fbe9a78-d9bc-40bb-be8c-bc00063765d7/Untitled.png)

/Users/leesoobeen/Desktop/02ggang9.github.io/_posts/images/bugBounty/1.png

![쿼리횟수](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/bugBounty/1.png?raw=true)


위의 어셈블리어를 보면 레지스터는 구분되지만 Balance라는 공유 자원에 서로 접근하고 있습니다. 그리고 코드가 서로 뒤엉켜 실행이 될 수 있기 때문에 실행 결과가 달라질 수 있습니다.

정리하자면 시간지연과 어셈블리어로 변환 후 코드 뒤엉킴, 동시에 공유 자원 접근 및 조작 때문에 예상되는 실행 결과로 나오지 않을 수 있다! 입니다. 실제 시간지연 적용시켜 코드를 실행시키면 다음과 같은 결과가 나옵니다.

```java
balance = 1000000
```

이렇게 약간의 시간 지연을 준 것만으로도 여러 쓰레드가 하나의 공유 자원을 사용하는 프로그램은 망가집니다.

### Mutual Exclusion | DeadLock | Starvation

위의 은행 입출금 문제와 코드로 Race Condition의 문제점을 간단하게 알아봤는데요. 좀 더 깊숙히 들어가면 아래의 세가지 문제에 직면하게 됩니다.

- Mutual Exclusion

Race Condition을 막기 위해서는 두 개 이상의 프로세스가 공용 데이터에 동시에 접근 하는것을 막아야 합니다. 죽, 한 프로세스가 공용 데이터를 사용하고 있으면 그 자원을 사용하지 못하도록 막거나, 다른 프로세스가 그 자원을 사용하지 못하도록 막으면 이 문제를 피할 수 있습니다. 이것을 상호 배제라고도 부릅니다.

- DeadLock

그러나 위의 같은 상호 배제를 시행하면 추가적인 제어 문제가 발생합니다. 하나는 교착상태 여기서 말하는 DeadLock 문제입니다. 프로세스는 각자 프로그램을 실행하기 위해서 두 자원 모두에 엑세스 해야 한다고 가정을 해보면 프로세스는 두 자원 모두를 필요로 하기 때문에 두 리소스를 사용하여 프로그램을 실행하기 전 까지 소유한 리소스를 해제하지 않습니다. 이러한 상황에서 두 프로세스는 교착 상태에 빠지게 되는 문제가 발생할 수 있습니다.

- Starvation

이 제어 문제는 “기아 상태”라고도 합니다. 즉 프로세스들이 더 이상 진행을 하지 못하고 영구적으로 블록되어 있는 상태입니다. 이 문제점은 시스템 자원에 대한 경쟁(Race) 도중에 발생할 수 있고 프로세스 통신 과정에서도 발생할 수 있는 문제 과정입니다. 두 개 이상의 작업이 서로 상대방의 작업이 끝나기만을 기다리고 있기 때문에 결과적으로는 아무것도 완료되지 못하는 상태가 되는 것 입니다.

요약하자면 Race Condition을 해결하기 위해서는 한 프로세스가 공용 데이터를 사용하고 있으면 다른 프로세스가 그 자원을 사용하지 못하도록 막아야 한다. → 영구적으로 프로그램이 멈출 수 있는 상태가 될 수 있다.

### 대응 방안

- Semaphore(세마포어)

세마포어는 앞에서 말씀드린 Mutual Exclusion 문제를 해결하기 위한 방법입니다. 공유된 자원의 데이터를 여러 프로세스가 접근하는 것을 막는 알고리즘 입니다. 세마포어 변수는 일반적으로 정수형 변수를 사용하며 세마포어의 종류는 **이진형 세마포어**, **계수형 세마포어**가 있습니다. 이진형 세마포어는 0과 1의 값, 한 개의 공유자원을 상호 배제하며, 계수형 세마포어는 0과 양의 정수, 여러 개의 공유자원을 상호 배제합니다.

- Mutex(뮤텍스)

뮤텍스도 앞에서 말씀드린 Mutual Exclusion 문제를 해결하기 위한 방법입니다. 세마포어와의 차이점은 세마포어는 Signaling 메커니즘으로 락을 걸지 않은 쓰레드도 signal을 사용해 락을 해제할 수 있습니다. 하지만 뮤텍스는 Locking 메커니즘으로 락을 걸은 쓰레드만이 임계 영역을 나갈 때 락을 해제할 수 있습니다. 세모포어의 카운트를 1로 설정하면 뮤텍스처럼 활용할 수 있습니다.

### WarGame | Race Condition

Race Condition 개념 정리하고 바로 실습 문제를 풀어보았습니다!

[Challenge - 60 | Race Condition](https://www.notion.so/Challenge-60-Race-Condition-693d565e5edd41ac9289ca38958b8756?pvs=21) 

[DreamHack | LEVEL2 | login - 1 | Race Condition](https://www.notion.so/DreamHack-LEVEL2-login-1-Race-Condition-f5ee80d4d042479290b5ca44b9c4c91f?pvs=21) 

### 출처

[https://velog.io/@codemcd/운영체제OS-8.-프로세스-동기화-1#:~:text=프로세스 동기화는 여러 프로세스,을 유지시켜주어야 한다](https://velog.io/@codemcd/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9COS-8.-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EB%8F%99%EA%B8%B0%ED%99%94-1#:~:text=%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%20%EB%8F%99%EA%B8%B0%ED%99%94%EB%8A%94%20%EC%97%AC%EB%9F%AC%20%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4,%EC%9D%84%20%EC%9C%A0%EC%A7%80%EC%8B%9C%EC%BC%9C%EC%A3%BC%EC%96%B4%EC%95%BC%20%ED%95%9C%EB%8B%A4)