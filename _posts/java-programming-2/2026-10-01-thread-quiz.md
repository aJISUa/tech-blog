---
title: "[자프실2] Thread 퀴즈: 아기돼지 join부터 짱구네 계좌까지"
date: 2026-10-01 20:10:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - Thread
  - MultiThread
  - synchronized
  - 퀴즈
excerpt: "4주차 Thread 범위로 퀴즈 네 문제를 만들었다. join 위치에 따른 실행 시간, 안 멈추는 interrupt, 잠에서 못 깨는 공주, 마이너스가 되는 짱구네 계좌."
toc: true
toc_sticky: true
---

3주차 [제네릭과 컬렉션 퀴즈]({{ site.baseurl }}{% post_url java-programming-2/2026-09-20-generic-collection-quiz %})에 이어 4주차 Thread 범위로 퀴즈를 만들었다. 실습하다가 헷갈렸거나 직접 돌려보고 놀랐던 것들을 문제로 바꿨다. 이번에도 괄호 채우기 대신 실행 결과 예측, 에러 이유, 고치는 방법을 묻는 문제로 만들었다.

Thread 문제는 실행할 때마다 결과가 바뀌어서 "실행 결과를 예측하라"고 내기가 까다로웠다. 그래서 순서가 아니라 **걸린 시간**, **멈추는지 안 멈추는지**, **최종 값**처럼 매번 같은 답이 나오는 것을 묻도록 만들었다. 문제 코드는 `JAVA2` 프로젝트 `quiz` 패키지에 넣고 전부 실제로 실행해서 확인했다.

이론은 [Thread 상태제어와 동기화]({{ site.baseurl }}{% post_url java-programming-2/2026-09-30-thread-sync-control %}), 실습은 [4주차 과제 2]({{ site.baseurl }}{% post_url java-programming-2/2026-10-01-week4-assignment2 %})에 있다.

| 번호 | 유형 | 연결되는 것 |
| --- | --- | --- |
| Quiz #1 | 실행 결과(시간) 예측 | Lab#7 아기돼지, join 위치 |
| Quiz #2 | 실행 결과 예측 + 고치기 | Lab#4 자동차 경주, interrupt 신호가 지워지는 문제 |
| Quiz #3 | 컴파일 에러와 실행 에러의 이유 | Lab#8 잠자는 숲속의 공주, wait와 lock |
| Quiz #4 | 설계와 리팩토링 | Lab#6 FamilyAccount, check-then-act |

## Quiz #1: join을 어디서 부르느냐

1초, 2초, 3초 일하는 Worker 세 개를 두 가지 방식으로 실행한다.

```java
static class Worker extends Thread {
	private final int sec;

	Worker(String name, int sec) {
		super(name);
		this.sec = sec;
	}

	@Override
	public void run() {
		System.out.println(getName() + " 시작 (" + sec + "초)");
		try {
			Thread.sleep(sec * 1000L);
		} catch (InterruptedException e) {
			return;
		}
		System.out.println(getName() + " 끝");
	}
}

public static void main(String[] args) throws InterruptedException {
	// (A) start 바로 뒤에서 join
	long start = System.currentTimeMillis();
	for (int i = 1; i <= 3; i++) {
		Worker w = new Worker("A" + i, i);
		w.start();
		w.join();
	}
	System.out.println("(A) 걸린 시간: " + (System.currentTimeMillis() - start) / 1000 + "초");

	// (B) 전부 start한 다음 한꺼번에 join
	start = System.currentTimeMillis();
	Worker[] ws = new Worker[3];
	for (int i = 0; i < 3; i++) {
		ws[i] = new Worker("B" + (i + 1), i + 1);
		ws[i].start();
	}
	for (Worker w : ws) {
		w.join();
	}
	System.out.println("(B) 걸린 시간: " + (System.currentTimeMillis() - start) / 1000 + "초");
}
```

1. (A)와 (B)는 각각 몇 초 걸릴까?
2. (A)에서 "시작"과 "끝" 줄은 어떤 순서로 찍힐까? (B)는?
3. (A)처럼 짜면 MultiThread로 만든 의미가 있을까?

### 풀이

![Quiz #1 실행 결과]({{ '/assets/images/java-programming-2/t4-quiz1-result.png' | relative_url }})

**(A)는 6초, (B)는 3초**다.

(A)는 start() 하자마자 join()으로 그 Worker가 끝날 때까지 main이 멈춘다. 그래서 A1이 끝나야 for문이 다음으로 넘어가서 A2를 만든다. 시작, 끝, 시작, 끝이 차례대로 찍히고 시간은 1 + 2 + 3 = 6초다.

(B)는 셋을 먼저 다 출발시킨 다음 기다린다. 세 Worker가 동시에 자고 있으니 가장 오래 걸리는 3초면 다 끝난다. 시작 세 줄이 먼저 나오고(이 셋의 순서는 매번 다를 수 있다) 끝은 1초, 2초, 3초짜리 순서로 나온다.

3번은 **의미가 없다.** (A)는 Thread를 세 개나 만들었지만 한 번에 하나만 일한다. 그냥 main에서 순서대로 호출한 것과 다를 게 없고, Thread를 만드는 비용만 더 든다.

### 출제 의도

Lab#7을 하면서 join 세 줄의 순서가 상관없다는 건 알았는데, join을 **언제** 부르느냐는 생각을 안 해봤다. join은 "여기서 기다린다"는 뜻이라서 위치에 따라 병렬이 순차로 바뀐다. 아기돼지로 치면 엄마가 첫째를 내보내고 돌아올 때까지 문 앞에서 기다렸다가 둘째를 내보내는 셈이다. 실제로 반복문 안에서 start와 join을 같이 쓰는 실수를 하기 쉬울 것 같아서 냈다.

## Quiz #2: 멈추라고 했는데 왜 안 멈출까

자동차 경주를 조금 바꿨다. interrupt 신호가 오면 while 조건에서 걸려서 나가도록 짰다.

```java
Thread car = new Thread(() -> {
	int km = 0;
	while (!Thread.currentThread().isInterrupted()) {   // 신호가 오면 나가려고 했다
		km++;
		System.out.println(km + "km");
		try {
			Thread.sleep(300);
		} catch (InterruptedException e) {
			System.out.println("고장 신호 받음!");
		}
		if (km == 8) break;   // 무한루프 방지용 안전장치
	}
	System.out.println("자동차 종료: " + km + "km");
});
car.start();
Thread.sleep(1000);
car.interrupt();
```

1. 1초 뒤에 interrupt를 보낸다. 자동차는 몇 km에서 종료될까?
2. 의도대로 신호를 받자마자 멈추게 하려면 어떻게 고쳐야 할까? 방법을 두 가지 이상 써보자.

### 풀이

![Quiz #2 실행 결과]({{ '/assets/images/java-programming-2/t4-quiz2-result.png' | relative_url }})

**8km**에서 종료된다. "고장 신호 받음!"은 찍히는데 그 뒤로도 계속 달리다가 안전장치(`km == 8`)에 걸려서야 끝난다.

1초면 300ms씩 세 번 자고 네 번째 sleep 중일 때다. 그래서 4km를 찍고 자다가 신호를 받는다. 여기까지는 의도대로다. 문제는 `sleep()`이 `InterruptedException`을 던지면서 **interrupt 신호를 지운다**는 것이다. catch에서 메시지만 찍고 넘어가니 while 조건에서 `isInterrupted()`를 다시 검사해도 false다. 안전장치가 없었다면 영원히 달렸을 거다.

고치는 방법은 이렇다.

```java
// 방법 1: catch에서 신호를 다시 세운다
} catch (InterruptedException e) {
	System.out.println("고장 신호 받음!");
	Thread.currentThread().interrupt();   // 예외가 지운 신호를 다시 세운다
}

// 방법 2: catch에서 바로 빠져나간다
} catch (InterruptedException e) {
	System.out.println("고장 신호 받음!");
	break;   // 또는 return
}
```

![Quiz #2 고친 뒤 실행 결과]({{ '/assets/images/java-programming-2/t4-quiz2-fix.png' | relative_url }})

방법 1로 고치면 4km에서 바로 종료된다. 방법 1은 while 조건이 신호를 확인하는 원래 구조를 그대로 쓸 수 있다. 이 코드가 다른 메소드 안에 있었다면 신호가 남아 있으니 그 메소드를 부른 쪽에서도 interrupt가 있었다는 걸 알 수 있다. 방법 2는 Lab#4의 `return`과 같다. 간단하지만 반복문 밖에서 정리할 게 있다면 return보다 break가 낫다.

### 출제 의도

Lab#4에서 "catch에서 return을 안 하면?"을 실험하다가 신호 상태가 false로 찍히는 걸 보고 놀랐다. interrupt는 강제 종료가 아니라 신호라서, 받는 쪽이 처리를 안 하면 그냥 사라진다. `catch (InterruptedException e) { e.printStackTrace(); }`로 대충 넘기는 습관이 Thread에서는 "안 멈추는 Thread"라는 버그가 된다.

## Quiz #3: 잠에서 못 깨는 공주

Lab#8 잠자는 숲속의 공주를 짧게 만들었다. 왕자가 오기 전, 공주가 잠드는 부분까지만 있다.

```java
public static void main(String[] args) {
	Object castle = new Object();

	Thread princess = new Thread(() -> {
		System.out.println("공주: Zzz");
		castle.wait();   // 왕자가 깨워줄 때까지 잠든다
		System.out.println("공주: 깨어났어요!");
	});
	princess.start();
}
```

1. 이 코드는 컴파일될까? 안 된다면 이유는?
2. 컴파일 에러를 고쳤더니 이번엔 실행 중에 예외가 난다. 어떤 예외이고 왜 날까?
3. 왕자까지 추가해서 공주가 제대로 잠들고 깨어나게 고쳐보자.

### 풀이

**1. 컴파일 에러**가 난다.

```text
error: unreported exception InterruptedException; must be caught or declared to be thrown
			castle.wait();
			           ^
```

`wait()`도 `sleep()`처럼 checked exception인 `InterruptedException`을 던질 수 있어서 반드시 처리해야 한다. 그런데 람다 안이라 main의 `throws`로 넘길 수도 없다. Runnable의 `run()`이 throws를 선언하지 않았기 때문이다. 그래서 람다 안에서 try-catch로 잡아야 한다.

**2.** try-catch로 감싸면 컴파일은 되는데, 실행하면 공주가 "Zzz"를 말하자마자 쓰러진다.

![Quiz #3 실행 결과]({{ '/assets/images/java-programming-2/t4-quiz3-result.png' | relative_url }})

`IllegalMonitorStateException: current thread is not owner`. 지금 Thread가 castle의 주인(lock을 가진 Thread)이 아니라는 뜻이다. `wait()`는 "이 객체의 lock을 **내려놓고** 기다린다"는 메소드라서, 내려놓을 lock을 먼저 쥐고 있어야 한다. 화장실 열쇠를 반납하려면 먼저 열쇠를 받아야 하는 것처럼. 그래서 `synchronized (castle)` 안에서만 부를 수 있다. 이 에러는 컴파일러가 못 잡고 실행해봐야 안다는 점이 무섭다.

**3.** 공주와 왕자가 같은 castle의 lock을 쓰게 한다.

```java
Thread princess = new Thread(() -> {
	synchronized (castle) {   // 성 열쇠를 들고
		System.out.println("공주: Zzz");
		try {
			castle.wait();     // 열쇠를 내려놓고 잠든다
		} catch (InterruptedException e) {
			return;
		}
		System.out.println("공주: 깨어났어요!");   // 다시 열쇠를 얻어야 여기로 온다
	}
});

Thread prince = new Thread(() -> {
	try {
		Thread.sleep(1000);   // 왕자 등장까지 1초
	} catch (InterruptedException e) {
		return;
	}
	synchronized (castle) {
		System.out.println("왕자: 키스로 깨웁니다!");
		castle.notify();
	}
});
```

![Quiz #3 고친 뒤 실행 결과]({{ '/assets/images/java-programming-2/t4-quiz3-fix.png' | relative_url }})

공주가 wait()로 lock을 내려놓았기 때문에 왕자가 `synchronized (castle)`에 들어갈 수 있다. 만약 공주가 wait() 대신 sleep()으로 잤다면 lock을 쥔 채로 자니까 왕자는 BLOCKED 상태로 문 앞에서 기다리기만 했을 거다. notify()도 lock을 쥔 상태에서만 부를 수 있다.

하나 더. 이 코드는 왕자가 1초 늦게 오니까 잘 되지만, 왕자가 공주보다 먼저 notify()를 해버리면 아무도 안 듣고 있으니 신호가 사라진다. 그러면 공주는 영원히 잔다. 그래서 노션 Lab#10처럼 실제로는 `empty` 같은 상태 변수를 두고 `while (조건) wait();`으로 검사해야 한다.

### 출제 의도

sleep()과 wait()의 차이를 표로 외우는 것보다, wait()를 아무 데서나 불러보면 차이가 확실히 기억에 남는다. wait()가 Thread가 아니라 Object의 메소드인 이유(기다리는 기준이 Thread가 아니라 lock을 가진 객체라서), synchronized 안에서만 불러야 하는 이유가 이 에러 하나로 이어진다. 컴파일 에러(checked exception)와 실행 에러(lock 소유)가 한 문제에 같이 나와서 좋았다.

## Quiz #4: 짱구네 계좌가 마이너스가 됐다

FamilyAccount를 단순하게 줄였다. 잔액 1000원인 계좌에서 짱구와 짱아가 동시에 1000원씩 인출한다. 잔액이 충분한지 확인하고 나서 빼니까 안전하다고 생각했다.

```java
static class Account {
	private int balance = 1000;

	void withdraw(String who, int amount) {
		if (balance >= amount) {           // 1. 확인
			try {
				Thread.sleep(100);         // 확인과 빼기 사이에 잠깐 틈이 생기면?
			} catch (InterruptedException e) {
				return;
			}
			balance -= amount;             // 2. 빼기
			System.out.println(who + " 1000원 인출 성공");
		} else {
			System.out.println(who + " 잔액 부족");
		}
	}
}

public static void main(String[] args) throws InterruptedException {
	Account acc = new Account();
	Thread jjanggu = new Thread(() -> acc.withdraw("짱구", 1000));
	Thread jjanga = new Thread(() -> acc.withdraw("짱아", 1000));
	jjanggu.start();
	jjanga.start();
	jjanggu.join();
	jjanga.join();
	System.out.println("최종 잔액: " + acc.balance);
}
```

1. 최종 잔액은 얼마일까?
2. `if (balance >= amount)`로 확인했는데 왜 이런 일이 생길까?
3. 어떻게 고쳐야 할까? `balance`에 `volatile`을 붙이는 걸로 해결될까?

### 풀이

![Quiz #4 실행 결과]({{ '/assets/images/java-programming-2/t4-quiz4-result.png' | relative_url }})

**-1000원**이다. 둘 다 인출에 성공한다. 여러 번 돌려도 매번 그렇다.

짱구가 잔액을 확인하고(1000원, 충분) 100ms 쉬는 동안, 짱아도 잔액을 확인한다. 아직 짱구가 안 뺐으니 짱아가 봐도 1000원이다. 둘 다 "충분하네"를 보고 들어가서 각자 뺀다. **확인(check)과 실행(act) 사이에 틈**이 있어서 생기는 문제라 check-then-act 경쟁 조건이라고 부른다. sleep은 그 틈을 눈에 보이게 벌려놓은 것뿐이고, sleep이 없어도 타이밍이 맞으면 똑같이 일어난다.

고치려면 확인과 빼기를 하나로 묶어서, 한 사람이 끝날 때까지 다른 사람이 못 들어오게 해야 한다.

```java
synchronized void withdraw(String who, int amount) {
	if (balance >= amount) {
		try {
			Thread.sleep(100);   // 틈이 있어도 다른 쓰레드는 문 밖에서 기다린다
		} catch (InterruptedException e) {
			return;
		}
		balance -= amount;
		System.out.println(who + " 1000원 인출 성공");
	} else {
		System.out.println(who + " 잔액 부족");
	}
}
```

![Quiz #4 고친 뒤 실행 결과]({{ '/assets/images/java-programming-2/t4-quiz4-fix.png' | relative_url }})

한 명은 성공하고 한 명은 "잔액 부족"이 나오고, 최종 잔액은 0원이다.

`volatile`로는 **해결 안 된다.** volatile은 다른 Thread가 쓴 최신 값을 보게 해줄 뿐이다. 짱아가 확인할 때 짱구는 아직 안 뺐으니 최신 값을 봐도 1000원이다. 문제는 값이 오래된 게 아니라 두 동작 사이에 끼어들 수 있다는 것이라서, 끼어들기를 막는 synchronized가 필요하다. [과제 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-01-week4-assignment2 %})의 MyBank 실험에서 volatile을 붙였는데도 오류가 계속 났던 것과 같은 이유다.

설계 측면에서 하나 더. 잔액 확인을 `withdraw()` 밖에서 하는 구조, 예를 들어 `if (acc.getBalance() >= 1000) acc.withdraw(1000);`처럼 쓰면 메소드 두 개에 각각 synchronized를 붙여도 그 사이에 틈이 생긴다. 확인과 실행은 계좌 클래스가 한 메소드 안에서 책임지는 게 맞다. "인출해도 되는지 판단하는 것"도 계좌가 할 일이다.

### 출제 의도

Lab#6 슬라이드에 "잔고를 확인하는 조건 + synchronized 키워드를 붙이기만 하면 해결"이라고 되어 있다. 그 "확인하는 조건"과 "synchronized"가 왜 꼭 같이 있어야 하는지 궁금했다. 확인만 있으면 이 문제처럼 마이너스가 되고, synchronized만 있으면 잔액 부족을 못 막는다. 그리고 OS 시간에 배운 race condition이 count++ 같은 연산 하나에서만 생기는 게 아니라, 확인하고 행동하는 "두 동작 사이"에서도 생긴다는 걸 짚고 싶었다.

## 만들면서 느낀 점

Thread 문제는 정답이 하나로 안 나와서 출제가 어려웠다. 출력 순서를 물으면 매번 달라지니까 문제가 안 된다. 그래서 "순서는 몰라도 확실한 것"을 찾아야 했는데, 생각해보면 MultiThread 코드를 짤 때도 "어떤 순서로 돌든 이건 보장된다"를 먼저 따져야 하니까 같은 연습이었다.
