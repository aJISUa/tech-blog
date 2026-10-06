---
title: "[자프실2] 4주차 과제 2: 자동차 경주, MyBank, FamilyAccount, 아기돼지삼형제"
date: 2026-10-01 20:00:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - Thread
  - MultiThread
  - synchronized
  - join
  - 과제
excerpt: "Lab#4~#7을 직접 돌려봤다. interrupt로 자동차 멈추기, 출력문을 빼야 보이는 race condition, synchronized와 join으로 가족 계좌 지키기, 아기돼지 join 실험까지."
toc: true
toc_sticky: true
---

[4주차 과제 1]({{ site.baseurl }}{% post_url java-programming-2/2026-09-26-week4-assignment %})에서 Lab#1~#3으로 Thread를 만들고 돌려봤다면, 이번엔 Lab#4~#7로 Thread를 멈추고, 기다리고, 공유 자원을 지키는 실습이다. 운영체제에서 이론으로만 배운 race condition과 동기화를 자바로 직접 재현해볼 수 있는 부분이라 제일 기대했던 실습이다.

이론은 [Thread 상태제어와 동기화]({{ site.baseurl }}{% post_url java-programming-2/2026-09-30-thread-sync-control %}) 글에 정리했다.

실습 전에 Lab마다 궁금한 걸 먼저 적어뒀다.

- Lab#4: Thread를 어떻게 멈춰야 안전할까? interrupt(), return, break는 각각 어디서 쓰일까?
- Lab#5: 출력문을 넣으면 왜 괜찮아 보일까? 정말 괜찮은 걸까?
- Lab#6: 인출자와 예금자의 속도는 왜 다르게 했을까? 잔액 조회는 왜 동기화가 필요 없을까?
- Lab#7: join()은 언제 꼭 필요할까?

코드는 `JAVA2` 프로젝트의 `thread` 패키지에 이어서 넣었다. Lab#7은 수업 시간에 직접 만든 `pig3` 패키지를 그대로 썼다.

| 실습 | 내용 | 파일 |
| --- | --- | --- |
| Lab#4 | 자동차 경주: interrupt로 멈추기 | Car, Lab4_CarRace, Lab4_NoReturn |
| Lab#5 | MyBank: 출력문과 race condition, synchronized | BankAccount, SyncBankAccount, User, Lab5_MyBank |
| Lab#6 | FamilyAccount: synchronized + join | FamilyAccount, Depositor, Withdrawer, Lab6_FamilyAccount |
| Lab#7 | 아기돼지삼형제: join | pig3.Piglet, pig3.LittlePig, Lab7_JoinTest |

## 1. Lab#4: 자동차 경주

### 실습 목표

자동차들이 goal 지점까지 달리다가, 특정 확률로 고장이 나면 경기를 중단한다. `stop()` 대신 `interrupt()`와 `return`으로 Thread를 안전하게 멈춘다. 밖에서 보내는 신호와 스스로 보내는 신호를 둘 다 써본다.

### 코드

```java
// Lab#4: 자동차 한 대가 쓰레드 하나. goal까지 1km씩 달린다
public class Car extends Thread {
	private int speed;   // 1km 달리는 데 걸리는 시간(ms). 클수록 느리다

	public Car(String name, int speed) {
		super(name);
		this.speed = speed;
		System.out.println(name + " 생성");
	}

	@Override
	public void run() {
		for (int i = 0; i <= Lab4_CarRace.GOAL; i++) {
			System.out.println(getName() + ": " + i + "km ..");

			// 5% 확률로 고장. 스스로에게 interrupt 신호를 보낸다
			if ((int) (Math.random() * 1000) % 100 < 5) {
				System.out.println(getName() + " 고장고장고장!");
				this.interrupt();
			}

			// interrupt 신호가 와 있는지 확인. interrupted()는 확인하면서 신호를 지운다
			if (Thread.interrupted()) {
				System.out.println(getName() + ": " + i + "km 지점에서 중단 (고장 신호 감지) -> 쓰레드 종료");
				return;   // run()이 끝나면 쓰레드도 끝난다. stop() 대신 스스로 빠져나가는 것
			}

			try {
				Thread.sleep(speed);
			} catch (InterruptedException e) {
				// 자고 있을 때 밖에서 interrupt가 오면 여기로 온다
				System.out.println(getName() + ": " + i + "km 지점에서 sleep 도중 interrupt 받음 -> 쓰레드 종료");
				return;   // 여기서 return을 안 하면? 아래 실험 참고
			}
		}
		System.out.println(getName() + " 도착!!");
	}
}
```

```java
// Lab#4: 자동차 경주. stop() 대신 interrupt()로 안전하게 멈추기
public class Lab4_CarRace {
	static final int GOAL = 10;   // 목표 지점(km)

	public static void main(String[] args) {
		Thread car1 = new Car("붕붕카", 300);
		Thread car2 = new Car("스포츠카", 100);
		Thread car3 = new Car("세발자전거", 800);

		System.out.println("------------- 자동차 경주 시작! -------------");
		car1.start();
		car2.start();
		car3.start();

		// 3초 뒤에 심판이 붕붕카를 멈춘다(밖에서 보내는 interrupt)
		try {
			Thread.sleep(3000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("[심판] 붕붕카 경기 중단 요청!");
		car1.interrupt();
	}
}
```

슬라이드 코드를 치다가 세 가지가 눈에 띄었다. 슬라이드 main에는 `int goal = 100;`이라는 지역 변수가 있는데, Car는 `자동차경주.goal`(static, 30)을 쓴다. main의 100은 아무도 안 쓰는 변수다. 그리고 Car의 `volatile boolean stop=false;` 필드도 선언만 되어 있고 쓰이지 않는다. 그리고 Car가 speed를 받아놓고 sleep은 300으로 고정이라 차마다 속도가 같았다. 헷갈려서 나는 `GOAL` 상수 하나로 정리하고, 안 쓰는 필드는 빼고, `sleep(speed)`로 바꿔서 차마다 속도가 다르게 했다. 아무도 안 쓰는 변수와 필드가 남아 있으면 읽는 사람만 헷갈린다.

#### 실행 전에 예상한 것

- 스포츠카(100ms)가 제일 빠르고 세발자전거(800ms)가 제일 느리다. 0km부터 10km까지 11번 반복하니까 스포츠카는 1.1초 남짓이면 도착한다
- 5% 확률이라 대부분 완주하겠지만, 실행할 때마다 누군가 고장 날 수 있다
- 3초 뒤 심판의 interrupt는 붕붕카가 sleep 중일 때 받을 확률이 높으니 catch 쪽 메시지가 나올 것 같다

#### 실제 결과

![Lab#4 자동차 경주 실행 결과]({{ '/assets/images/java-programming-2/t4-lab4-race1.png' | relative_url }})

![Lab#4 자동차 경주 다시 실행한 결과]({{ '/assets/images/java-programming-2/t4-lab4-race2.png' | relative_url }})

고장 확률이 5%라서 고장은 드물 줄 알았는데, km마다 한 번씩 굴리니까 11번 중에 한 번이라도 걸릴 확률은 생각보다 높다(1 - 0.95^11, 약 43%). 실제로 여러 번 돌리면 고장 나는 차가 꽤 자주 나왔다.

재밌는 건 심판의 interrupt였다. 붕붕카가 이미 고장으로 끝나버린 경우에는 `[심판] 붕붕카 경기 중단 요청!`만 찍히고 아무 일도 안 일어난다. 이미 TERMINATED된 Thread에 interrupt를 보내도 에러 없이 아무 일도 안 일어난다. 붕붕카가 살아 있을 때는 대부분 sleep 중에 신호를 받아서 catch 쪽 메시지가 나왔다. 300ms 중 대부분을 자고 있으니까 당연하다.

신호를 받는 길이 두 개라는 게 이번 실습의 핵심이었다. 달리는 중이면 `Thread.interrupted()`로 직접 확인하고, 자는 중이면 `InterruptedException`으로 받는다. 둘 다 `return`으로 run()을 빠져나가서 끝낸다.

### 추가 실험: catch에서 return을 안 하면?

Car의 catch 블록 주석에 "return을 안 하면?"이라고 써놨는데 직접 해봤다.

```java
// Lab#4 추가 실험: sleep 중에 interrupt를 받고 catch에서 return을 안 하면?
Thread runner = new Thread(() -> {
	for (int i = 1; i <= 5; i++) {
		System.out.println("달리는 중 " + i + "km");
		try {
			Thread.sleep(500);
		} catch (InterruptedException e) {
			System.out.println("  interrupt 받음! 그런데 return 안 함. 지금 신호 상태 = "
					+ Thread.currentThread().isInterrupted());
		}
	}
	System.out.println("결국 끝까지 달렸다");
});
runner.start();
Thread.sleep(700);    // 두 번째 sleep 중일 때
runner.interrupt();   // 멈추라고 신호
```

#### 실행 전에 예상한 것

catch에서 메시지만 찍고 끝내지 않으니 계속 달릴 것 같다. 신호 상태는 interrupt를 받았으니 true일 것 같았다.

#### 실제 결과

![Lab#4 return 없는 catch 실험 결과]({{ '/assets/images/java-programming-2/t4-lab4-noreturn.png' | relative_url }})

계속 달린 건 예상대로인데, 신호 상태가 **false**로 나왔다. `InterruptedException`이 던져지는 순간 신호가 지워진 거다. 그래서 catch에서 아무것도 안 하면 "멈춰"라는 요청이 흔적도 없이 사라진다. 나중에 반복문 조건으로 `isInterrupted()`를 검사해도 못 알아챈다. 이건 [Thread 퀴즈]({{ site.baseurl }}{% post_url java-programming-2/2026-10-01-thread-quiz %})의 한 문제로 만들었다.

### 실습 후기

- interrupt는 "멈춰"가 아니라 "멈춰줄래?"였다. 받는 쪽이 성실하게 확인하고 정리해야 한다. `stop()`처럼 아무 때나 죽이면 정리할 기회가 없으니 위험하다는 게 이해됐다.
- 예외가 신호를 지운다는 건 직접 안 해봤으면 몰랐을 거다. catch 블록을 `e.printStackTrace()`로 대충 넘기는 습관이 Thread에서는 버그가 될 수 있다.
- 운영체제에서 배운 시그널과 비슷하다고 느꼈다. 프로세스에 시그널을 보내도 핸들러가 무시하면 안 죽는 것처럼(SIGKILL 같은 건 빼고), interrupt도 받는 쪽 코드가 결정한다.

## 2. Lab#5: MyBank

### 실습 목표

두 사람이 한 계좌로 만 원 넣고 만 원 빼기를 반복한다. 넣은 만큼 빼니까 잔고가 0 밑으로 내려가면 안 되고, 끝나면 0이어야 한다. 출력문이 있을 때와 없을 때, `synchronized`가 있을 때와 없을 때를 비교한다.

### 코드

슬라이드는 한 클래스에서 출력문을 주석 처리했다 풀었다 하면서 비교하는데, 세 가지 조건을 한 번에 비교하려고 조금 바꿨다. 출력 여부는 생성자로 받고, synchronized 버전은 상속해서 메소드만 덮었다. 슬라이드 변수 이름 `balane`은 오타 같아서 `balance`로 고쳤다.

```java
// Lab#5: 두 사람이 같이 쓰는 계좌. 동기화를 안 한 버전
public class BankAccount {
	// volatile: 값을 캐시에 들고 있지 말고 매번 메모리에서 읽고 쓰라는 표시
	// 처음엔 없이 돌렸는데 오류가 거의 안 보여서 붙였다. 그래도 += 자체가 한 번에 되는 건 아니다
	protected volatile int balance;
	protected boolean printOn;   // 입출금할 때마다 출력할지

	public BankAccount(boolean printOn) {
		this.printOn = printOn;
	}

	public void deposit(int amount) {
		if (printOn) System.out.println("+" + amount);
		// balance += amount 는 한 줄이지만 실제로는 읽기 -> 더하기 -> 쓰기 세 단계다
		// 두 쓰레드가 같은 값을 읽고 각자 쓰면 한쪽 결과가 사라진다
		this.balance += amount;
	}

	public void withdraw(int amount) { ... }   // deposit과 같고 -=
	public int getBalance() { ... }            // printOn이면 "잔고 : " 출력 후 반환
}

// Lab#5 수정: 메소드에 synchronized를 붙인 버전
// 한 쓰레드가 이 계좌의 synchronized 메소드 안에 있으면, 다른 쓰레드는 문 앞에서 기다린다(화장실 열쇠)
public class SyncBankAccount extends BankAccount {
	@Override
	public synchronized void deposit(int amount) {
		super.deposit(amount);
	}
	// withdraw, getBalance도 똑같이 synchronized로 덮었다
}
```

```java
// Lab#5: 계좌를 쓰는 사람. 넣었다 뺐다를 반복한다
public class User implements Runnable {
	...
	@Override
	public void run() {
		for (int i = 0; i < repeat; i++) {
			account.deposit(10000);
			if (sleepMs > 0) {
				try {
					Thread.sleep(sleepMs);
				} catch (InterruptedException e) {
					return;
				}
			}
			account.withdraw(10000);
			// 내가 넣은 만큼 뺐으니 0 밑으로 내려갈 수 없어야 정상
			if (account.getBalance() < 0) {
				errorCount++;
			}
		}
	}
}
```

```java
// 두 사람이 같은 계좌로 입출금하고, 끝나면 결과를 본다
static void test(String title, BankAccount account, int repeat, int sleepMs) throws InterruptedException {
	User a = new User(account, repeat, sleepMs);
	User b = new User(account, repeat, sleepMs);
	Thread one = new Thread(a);
	Thread two = new Thread(b);
	one.start();
	two.start();
	one.join();   // 둘 다 끝날 때까지 기다려야 최종 잔고를 볼 수 있다
	two.join();
	... // 최종 잔고와 마이너스 잔고 발견 횟수 출력. 둘 중 하나라도 이상하면 "오류발생!"
}

public static void main(String[] args) throws InterruptedException {
	// 1. 슬라이드 설정: 10번, 20ms 쉬기, 출력문 있음
	test("1. 출력 O, 10회", new BankAccount(true), 10, 20);
	// 2. 출력문을 빼고, 쉬지 않고, 100만 번
	test("2. 출력 X, 100만회", new BankAccount(false), 1000000, 0);
	// 3. 2번과 같은 조건에 synchronized만 붙인 계좌
	test("3. 출력 X, 100만회, synchronized", new SyncBankAccount(false), 1000000, 0);
}
```

마이너스 횟수는 각 User가 자기 것만 세고 main에서 join 뒤에 더했다. 횟수를 세는 변수까지 공유하면 그것도 race condition이 생기니까.

#### 실행 전에 예상한 것

- 1번: 슬라이드 말대로 출력문이 완충 장치가 돼서 문제없어 보일 것 같다
- 2번: 출력문을 빼고 100만 번이면 최종 잔고가 0이 아니고, 마이너스도 보일 것 같다
- 3번: synchronized를 붙였으니 항상 0

#### 실제 결과

![Lab#5 MyBank 실행 결과]({{ '/assets/images/java-programming-2/t4-lab5-mybank.png' | relative_url }})

![Lab#5 MyBank 다시 실행한 결과]({{ '/assets/images/java-programming-2/t4-lab5-mybank2.png' | relative_url }})

1번과 3번은 예상대로 정상이었다. 1번은 출력문이 수십 줄 찍히는 동안 잔고가 10000과 0을 오가다가 최종 0으로 끝났다. 2번은 두 번 다 "오류발생!"이었는데 숫자가 완전히 달랐다. 첫 번째는 최종 잔고가 약 -12억 9천만 원에 마이너스 잔고를 85만 번 넘게 봤고, 두 번째는 약 -3억 원에 168만 번 넘게 봤다. 터미널에서 더 돌려보면 멀쩡하게 0이 나오는 회차도 있었다. **한 번 잘 나왔다고 맞는 코드가 아니다**라는 걸 이번에도 확인했다.

캡처 두 장은 둘 다 마이너스였지만, 터미널에서 돌렸을 때는 잔고가 수천만 원 **늘어난** 회차도 있었다. 처음엔 이상했는데 생각해보면 당연하다. "더하기"가 덮여서 사라지면 잔고가 줄고, "빼기"가 덮여서 사라지면 잔고가 는다. 두 Thread가 같은 잔고를 읽고 각자 계산해서 쓰면 한쪽 계산이 증발하는 거다. 돈이 늘든 줄든 둘 다 lost update다.

예상과 달랐던 건 `volatile`을 붙이기 전이다. 처음에는 volatile 없이 2번을 돌렸는데, 그때는 다섯 번 중 한 번 정도만 오류가 났다. 나중에 다시 돌려보니 매번 오류가 날 때도 있어서 실행할 때마다 들쭉날쭉했다. 찾아보니 JIT 컴파일러가 반복문 안의 필드 값을 레지스터에 들고 계산하는 식으로 최적화할 수 있다고 한다. 그러면 메모리에 쓰는 순간이 줄어서 충돌이 덜 보인다. volatile을 붙이면 매번 메모리에서 읽고 쓰게 되니 충돌이 잘 드러났다.

다만 volatile은 "최신 값을 보게" 해줄 뿐이지 `+=`의 세 단계를 한 번에 해주지는 않는다. 그래서 volatile을 붙였는데도 오류가 나고, synchronized를 붙여야 해결된다. 버그가 안 보이는 이유가 출력문 말고도 있을 수 있다는 걸 알게 됐다.

### 실습 후기

- 디버깅하려고 넣은 출력문이 버그를 가린다는 게 제일 충격이었다. MultiThread 버그는 관찰하는 행위 자체가 타이밍을 바꿔버린다. 다만 내 1번 조건은 출력문만 있는 게 아니라 sleep 20ms에 10회뿐이라, 출력문 하나만의 효과라고 하기는 어렵다. 출력문만 켜고 100만 번 돌리는 비교도 해볼 걸 그랬다.
- synchronized를 붙이니 100만 번을 돌려도 매번 0이었다. 대신 터미널에서 2번과 3번 조건만 따로 시간을 재보니, synchronized 쪽이 두 배 정도 오래 걸렸다(약 0.1초 대 0.2초). 화장실 열쇠를 매번 받고 반납하는 비용이다. 교수님께서 Vector가 혼자 쓸 때 비효율적이라고 하신 게 이런 비용 얘기였던 것 같다.
- 운영체제에서 배운 생산자-소비자의 `count++` 예제가 바로 이 `balance += amount`였다. 어셈블리 세 줄로 쪼개서 설명하던 그림이 자바 코드에서 그대로 재현됐다.

## 3. Lab#6: FamilyAccount

### 실습 목표

온 가족이 함께 쓰는 계좌다. 예금자는 1000원씩 0.5초 간격으로 5번, 인출자는 1500원씩 0.8초 간격으로 4번. 한 계좌에 여러 Thread가 동시에 접근하지만 한 번에 하나씩 처리되게 하고, 모든 거래가 끝난 뒤 최종 잔고를 출력한다. 확장 시나리오로 엄마, 아빠는 입금하고 짱구, 짱아, 흰둥이는 출금한다.

### 코드

```java
// Lab#6: 온 가족이 함께 쓰는 계좌 ver.1
public class FamilyAccount {
	private int balance = 0;
	private int depositAll = 0;    // 총 저축금액
	private int withdrawAll = 0;   // 총 인출금액
	private int failCount = 0;     // 잔액 부족으로 실패한 횟수

	// 예금. 한 번에 한 사람만
	public synchronized void deposit(String who, int amount) {
		balance += amount;
		depositAll += amount;
		System.out.println("$$ " + who + " 예금: " + amount + "원 → 현재 잔액: " + balance + "원");
	}

	// 인출. 잔액 확인과 빼기가 한 덩어리로 실행돼야 한다
	// synchronized가 없으면 둘이 동시에 "잔액 충분하네" 하고 같이 빼버릴 수 있다
	public synchronized void withdraw(String who, int amount) {
		if (balance >= amount) {
			balance -= amount;
			withdrawAll += amount;
			System.out.println("-- " + who + " 인출: " + amount + "원 → 현재 잔액: " + balance + "원");
		} else {
			failCount++;
			System.out.println("!! " + who + " 인출 실패: 잔액 부족 (" + balance + "원)");
		}
	}

	// 잔액 조회. 모든 쓰레드가 끝난 뒤(join 후)에만 부르니까 동기화를 안 했다
	public int getBalance() { ... }
}
```

Depositor와 Withdrawer는 이름, 계좌, 금액, 횟수, 간격을 생성자로 받아서 반복하는 Thread다. 노션 코드는 주석에 "5회 반복"이라고 써 있는데 반복문이 `i < 10`이고, 슬라이드 코드는 예금 1초, 인출 1.5초 간격으로 각각 10회라서 셋이 조금씩 달랐다. 나는 슬라이드 26쪽 시나리오 ver.1(5회, 4회)에 맞췄다. 횟수와 간격을 생성자로 받게 만든 것도 이것저것 바꿔보기 위해서였다.

```java
// ver.1
FamilyAccount account = new FamilyAccount();
Thread depositor = new Depositor("예금자", account, 1000, 5, 500);
Thread withdrawer = new Withdrawer("인출자", account, 1500, 4, 800);
depositor.start();
withdrawer.start();
depositor.join();    // 예금자 끝날 때까지 대기
withdrawer.join();   // 인출자 끝날 때까지 대기
System.out.println("거래 종료 → 최종 잔액: " + account.getBalance() + "원");

// 확장: 엄마, 아빠는 입금 / 짱구, 짱아, 흰둥이는 출금
FamilyAccount family = new FamilyAccount();
Thread[] members = {
		new Depositor("엄마", family, 1000, 5, 500),
		new Depositor("아빠", family, 1000, 5, 500),
		new Withdrawer("짱구", family, 1500, 4, 800),
		new Withdrawer("짱아", family, 1500, 4, 800),
		new Withdrawer("흰둥이", family, 1500, 4, 800)
};
for (Thread t : members) t.start();
for (Thread t : members) t.join();   // 다섯 명 다 끝나야 잔액을 본다
```

#### 실행 전에 예상한 것

- ver.1: 총 예금 5000원, 인출 시도 6000원이니 적어도 한 번은 실패한다. 처음에 잔액이 0일 때 인출자가 먼저 오면 바로 실패할 것 같다
- 확장: 들어오는 돈은 0.5초에 2000원, 나가려는 돈은 0.8초에 4500원이라 금방 바닥난다. 실패가 많이 나올 것 같다
- synchronized 덕분에 잔액이 마이너스가 되는 일은 없다

#### 실제 결과

![Lab#6 FamilyAccount 실행 결과]({{ '/assets/images/java-programming-2/t4-lab6-family.png' | relative_url }})

위쪽이 ver.1, 아래쪽이 확장 시나리오다. ver.1은 첫 인출 시도 때 잔액이 1000원이라 실패했고, 최종 잔액 500원으로 끝났다. 확장 시나리오는 시작하자마자 흰둥이, 짱아, 짱구가 줄줄이 실패했고, 인출 실패 6번에 최종 잔액 1000원이었다. 잔액이 마이너스가 된 적은 한 번도 없었다. 확인과 빼기가 synchronized로 묶여 있으니까.

처음에 적어둔 "인출자와 예금자의 속도는 왜 다르게 했을까?"는 결과를 보고 내 나름대로 답을 찾았다. 둘의 간격이 같으면 늘 비슷한 타이밍에 번갈아 와서 결과가 단조로울 것 같다. 0.5초와 0.8초로 어긋나게 해두면 어떨 때는 예금이 두 번 연속, 어떨 때는 인출이 먼저 오면서 다양한 순서가 생긴다. 잔액이 모자란 상황(인출 실패)이 자연스럽게 생기게 만든 설정이다.

"잔액 조회는 왜 동기화가 필요 없을까?"는 `getBalance()`를 부르는 시점 때문이다. main이 두 Thread를 `join()`으로 다 기다린 다음에 부르니까, 그때는 계좌를 건드리는 Thread가 아무도 없다. 다만 거래 도중에 잔액을 조회하는 Thread가 있다면 얘기가 달라진다. 그때는 조회도 synchronized로 묶어야 중간 값을 안 본다. 노션 주석의 "동기화 필요 없음"은 어디서나 맞는 말이 아니라, join 다음에 부르는 이 코드에서만 맞는 말이었다.

슬라이드 28쪽에 교수님께서 엑셀로 예상한 표가 있다. 최종 잔고는 같지만 예금과 인출의 과정이 다르다는 내용이다. 내 결과도 실행할 때마다 순서가 달랐다. synchronized는 "한 번에 한 명"까지만 보장하고, "누가 먼저"는 보장하지 않는다. 저축과 인출을 꼭 번갈아 해야 한다면 노션의 Version2처럼 `empty` 깃발과 `wait()`, `notifyAll()`로 상태를 제어해야 한다. 교수님께서 synchronized만으로는 "웬만큼"이라고 하신 이유다.

확장 시나리오는 언제 파산하는지 보는 거였는데, 인출 시도 12번 중 절반이 잔액 부족으로 막혔다. 파산 방지는 `if (balance >= amount)` 확인 하나로 됐다. 단, 그 확인이 synchronized 안에 있어야 한다. 밖에 있으면 둘이 동시에 "잔액 충분하네"를 보고 같이 빼버릴 수 있다. 이것도 퀴즈로 만들었다.

### 실습 후기

- synchronized는 안전(마이너스 방지)을 주고, join은 기다림(모두 끝난 뒤 정산)을 준다. 둘이 맡은 일이 달랐다.
- 슬라이드, 노션, 시나리오 설명의 숫자가 조금씩 달라서 처음엔 헷갈렸다. 반복 횟수와 간격을 생성자로 빼니까 이것저것 바꿔보기가 편했다.

## 4. Lab#7: 아기돼지삼형제

### 실습 목표

아기돼지 삼형제는 각자 자유롭게 놀다가도 집에 들어올 때는 서로를 기다렸다 함께 들어온다. 엄마 돼지(main)가 `join()`으로 셋이 다 올 때까지 기다렸다가 문을 열어준다.

### 코드

수업 시간에 직접 친 `pig3` 패키지 코드다. 슬라이드는 "밖에서 놀다 올게요"인데, 나는 아기돼지 이야기답게 집을 짓는 걸로 바꿨다.

```java
public class Piglet extends Thread {
	private final String name;
	int time;

	public Piglet(String name) {
		this.name = name;
		this.time = (int)(Math.random() * 7000) + 3000; // 3~10s
	}

	public void run() {
		System.out.println(name + " 집을 " + time/1000 + "초 동안 짓는 중 ...");

		try {
			Thread.sleep(time);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println(name + " 다 하고 집 도착~");
	}
}
```

```java
public class LittlePig {
	public static void main(String[] args) {
		Piglet pig1 = new Piglet("돼지1");
		Piglet pig2 = new Piglet("돼지2");
		Piglet pig3 = new Piglet("돼지3");

		pig1.start();
		pig2.start();
		pig3.start();

		try {
			pig1.join(); //첫째가 집에 올 때까지 대기
			pig2.join(); //둘째가 집에 올 때까지 대기
			pig3.join(); //셋째가 집에 올 때까지 대기
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		System.out.println("모두 도착!");
	}
}
```

`(int)(Math.random() * 7000) + 3000`은 3000~9999ms라서 최소 3초, 최대 10초 가까이다. 교수님께서 Piglet의 `name` 필드는 Thread 이름이 아니라 "첫째, 둘째, 셋째" 같은 돼지 이름을 넣으려는 의도라고 하셨다. `super(name)`이나 `setName()`으로 Thread 이름을 줄 수도 있어서 꼭 필요한 건 아니라고.

#### 실행 전에 예상한 것

- 짓는 시간이 짧은 돼지부터 도착한다
- join 순서가 pig1, pig2, pig3이지만 도착 순서와는 상관없다
- "모두 도착!"은 무조건 맨 마지막이다

#### 실제 결과

![Lab#7 아기돼지삼형제 실행 결과]({{ '/assets/images/java-programming-2/t4-lab7-pig3.png' | relative_url }})

예상대로 시간이 짧은 돼지부터 도착했고, "모두 도착!"은 셋이 다 온 뒤에 나왔다. main에는 반복문이 하나도 없는데 몇 초씩 멈춰서 기다린다. 교수님께서 main에 while 하나 없는데 셋이 다 끝날 때까지 기다린다고 하신 게 이거다.

### 추가 실험: join이 없으면, join 순서를 바꾸면

두 가지가 궁금해서 `thread` 패키지에 실험용 main을 따로 만들었다. pig3의 Piglet을 import해서 썼다.

```java
// 1. join 없이
Piglet a1 = new Piglet("돼지1");
...
a1.start(); a2.start(); a3.start();
// 기다리지 않고 바로 문을 열어버린다
System.out.println("엄마: 모두 도착! (" + (System.currentTimeMillis() - start) + "ms) ← 아무도 안 왔는데?");

// 2. join 순서를 거꾸로(셋째, 둘째, 첫째)
b1.start(); b2.start(); b3.start();
b3.join();   // 셋째부터 기다린다. 이미 끝난 쓰레드를 join하면 바로 통과한다
b2.join();
b1.join();
// 가장 오래 걸린 돼지 시간만큼만 기다린다. 세 시간을 더한 만큼이 아니다
System.out.println("엄마: 모두 도착! (" + (System.currentTimeMillis() - start) + "ms)");
```

#### 실행 전에 예상한 것

- 1번: 엄마가 돼지들이 출발 인사도 하기 전에 "모두 도착!"을 외칠 것 같다
- 2번: 순서를 거꾸로 해도 결과는 같고, 걸린 시간은 가장 오래 걸린 돼지 시간과 비슷할 것 같다

#### 실제 결과

![Lab#7 join 실험 결과]({{ '/assets/images/java-programming-2/t4-lab7-jointest.png' | relative_url }})

join이 없으면 엄마가 몇 ms 만에 "모두 도착!"을 외친다. 돼지들이 "집 짓는 중" 인사를 하기도 전이다. main은 start()로 일을 맡겨두기만 하고 자기 갈 길을 가니까. Lab#1에서 본 "start()는 부탁만 하고 끝난다"가 여기서 문제가 된다.

join 순서를 거꾸로 해도 걸린 시간은 가장 오래 걸린 돼지의 시간과 거의 같았다. 셋째를 기다리는 동안 첫째와 둘째도 같이 집을 짓고 있으니까. 셋째가 오면 둘째와 첫째는 이미 와 있거나 곧 오고, 이미 끝난 Thread의 join()은 기다리지 않고 바로 통과한다. 그래서 join 순서는 결과에 영향이 없다. 교수님께서 join 세 줄은 "1, 2, 3이 다 끝나면 알려줘"라는 뜻이라고 하신 게 이 말이었다.

### 실습 후기

- join은 기다리는 쪽이 부른다. 처음엔 `pig1.join()`이 "pig1아, 끝나면 합류해"처럼 읽혀서 pig1이 뭔가 하는 줄 알았다. 실제로는 main이 "pig1 끝날 때까지 나는 멈춰 있을게"라고 하는 거다.
- join을 어디서 부르느냐에 따라 결과가 달라질 것 같아서, start 바로 뒤에 join을 부르면 어떻게 되는지를 퀴즈로 만들었다.
- 돌아보니 Lab#6에서도 join을 쓰고 있었다. FamilyAccount에서 join이 없었다면 최종 잔액이 거래 도중에 찍혔을 거다. 계산을 다 끝낸 뒤 결과를 모아야 할 때 join이 필요하다.

## 마치며

Lab#1~#3이 "Thread는 순서를 보장하지 않는다"를 보여줬다면, Lab#4~#7은 "그래서 뭘 해야 하는가"였다. 멈출 때는 interrupt로 부탁하고, 공유 자원은 synchronized로 지키고, 결과를 모을 때는 join으로 기다린다. 그래도 순서까지 맞추려면 wait/notify가 필요하다.

보통은 이 실습을 먼저 하고 나중에 운영체제에서 "아, 그때 그거였구나" 하게 된다는데, 나는 반대로 운영체제에서 배운 race condition, 임계 구역, 시그널이 실습 속에서 하나씩 보였다. 순서는 달라도 연결되는 경험은 같은 것 같다.
