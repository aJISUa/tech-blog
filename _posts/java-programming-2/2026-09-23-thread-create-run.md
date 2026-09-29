---
title: "[자프실2] Thread 생성과 실행: 세 가지 방법과 주요 메소드"
date: 2026-09-23 20:10:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - Thread
  - Runnable
  - 람다식
  - 우선순위
excerpt: "Thread 상속, Runnable 구현, 익명 클래스와 람다식으로 Thread를 만드는 방법과 start()와 run()의 차이, 우선순위를 믿으면 안 되는 이유를 정리했다."
toc: true
toc_sticky: true
---

[MultiThread 기초 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-23-thread-concepts %})에서 이어지는 내용이다. 이번엔 자바에서 Thread를 실제로 만들고 실행하는 방법이다.

## 만드는 방법 세 가지

Thread를 만드는 방법은 세 가지인데, 공통점이 있다. 전부 **`run()`에 할 일을 쓰고 `start()`로 실행시킨다.**

### 방법 1: Thread 클래스를 상속

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread 실행 중: " + Thread.currentThread().getName());
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        t1.start();   // run()이 아닌 start() 호출!
    }
}
```

제일 쉬운 방법이다. 이미 있는 `Thread` 클래스를 상속받고 `run()`만 오버라이딩하면 된다. 반드시 구현해야 하는 메소드는 `run()` 하나뿐이다.

언제 쓸까? 지금 아무것도 상속받고 있지 않을 때.

### 방법 2: Runnable 인터페이스를 구현

```java
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Runnable 실행 중: " + Thread.currentThread().getName());
    }
}

public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(new MyRunnable());   // Runnable을 Thread에 넘긴다
        t1.start();
    }
}
```

자바는 다중 상속이 안 되니까, 이미 다른 클래스를 상속받고 있으면 `extends Thread`를 더 붙일 수 없다. 그때는 인터페이스로 구현한다. `class MyClass extends SomeThing`이면 뒤에 `implements Runnable`을 붙이는 식이다.

여기서 헷갈렸던 부분. `MyRunnable`은 Thread가 아니라 "할 일"이다. 그래서 `new Thread(new MyRunnable())`처럼 Thread의 생성자에 넘겨줘야 한다. 교수님께서 이걸 주입이라고 표현하셨다.

### 방법 3: 익명 클래스나 람다식

```java
// 익명 클래스: 이름 없는 Runnable을 그 자리에서 만든다
Thread t = new Thread(new Runnable() {
    public void run() {
        System.out.println("익명 클래스 쓰레드 실행!");
    }
});
t.start();

// 람다식: Runnable은 추상 메소드가 run() 하나뿐이라 이렇게 줄일 수 있다
Thread t = new Thread(() -> {
    System.out.println("람다식 쓰레드 실행!");
});
t.start();
```

[람다식 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-11-lambda-expression %})에서 배운 게 여기서 나왔다. Runnable은 구현할 메소드가 `run()` 하나뿐이라 함수형 인터페이스고, 그래서 `() -> { ... }`로 쓸 수 있다.

교수님께서 괄호 짝 맞추는 순서를 알려주셨다. 무작정 따라 치지 말고, `new Thread()` 쓰고 괄호 안으로 커서를 옮겨서 그 안에 `new Runnable()`을 만들고, 또 그 안에서 `run()`을 구현하는 식으로 안쪽으로 들어가야 한다. 세미콜론까지 먼저 찍어두고 안으로 들어가면 괄호가 안 꼬인다.

### 어떤 방법을 쓸까

| 상황 | 방법 |
| --- | --- |
| 여기서 한 번만 쓰는 일회성 Thread | 람다식이 제일 편하다 |
| 여러 곳에서 공유해야 하는 Thread | 파일을 따로 만들어서 이름 있는 클래스로 |
| 이미 다른 클래스를 상속받고 있다 | Runnable 구현 |

중간 정리 슬라이드에서는 다중 상속이 안 되는 걸 생각하면 Runnable을 쓰는 게 좋다고 했다. 고수준의 Thread 관리 API도 Runnable 쪽을 쓴다고 한다.

## start()와 run()의 차이

제일 많이 하는 실수라고 해서 직접 확인해봤다.

```java
Thread a = new Thread(() -> System.out.println(Thread.currentThread().getName()));
a.run();     // main이 출력된다. 새 스레드가 안 생긴다
a.start();   // Thread-0이 출력된다. 새 스레드가 생겨서 그 스레드가 실행한다
```

`run()`을 직접 부르면 그냥 평범한 메소드 호출이라 메인 Thread가 실행한다. `start()`를 불러야 새 Thread가 생기고, 그 Thread가 `run()`을 실행한다.

## 주요 메소드

슬라이드에 메소드가 20개쯤 나오는데, 교수님께서 체크해주신 것 위주로 정리했다.

| 메소드 | 설명 |
| --- | --- |
| `Thread()` / `Thread(String name)` | 기본 생성자 / 이름을 가진 Thread |
| `Thread(Runnable target)` | Runnable 구현 객체를 받아서 그 객체의 run()을 실행하는 Thread |
| `Thread(Runnable target, String name)` | 위에 이름까지 같이 |
| `void start()` | Thread 시작. run()을 호출해준다 |
| `void run()` | Thread에서 실행될 코드 |
| `static void sleep(long millis)` | 현재 실행 중인 Thread를 밀리초 단위로 일시 정지 |
| `void join()` | 다른 Thread가 끝날 때까지 현재 Thread를 기다리게 한다 |
| `void interrupt()` | Thread에 중단하라는 신호를 보낸다 |
| `boolean isInterrupted()` | 인터럽트 신호가 왔는지 검사 |
| `Thread currentThread()` | 지금 실행 중인 Thread 객체를 가져온다 |
| `getName()` / `setName()` | Thread 이름 가져오기 / 정하기 |
| `getPriority()` / `setPriority()` | 우선순위 가져오기 / 정하기 (1~10) |
| `stop()`, `resume()`, `suspend()` | 안전성 문제로 사용하지 않는다 |

### sleep은 예외 처리가 필요하다

```java
try {
    Thread.sleep(500);   // 1000이 1초니까 0.5초
} catch (InterruptedException e) {
    // 자는 도중에 누가 깨우면(interrupt) 여기로 온다
}
```

`Thread.sleep(500)`만 쓰면 예외 처리를 하라고 에러가 뜬다. try-catch로 막아주면 된다.

### 이름

- 메인 Thread 이름은 `main`
- 작업 Thread는 이름을 안 주면 `Thread-0`, `Thread-1`처럼 자동으로 붙는다
- `setName("이름")`으로 바꿀 수 있고, `getName()`으로 가져온다

여기서 재밌는 걸 배웠다. Thread를 상속한 클래스에서 이름 필드를 또 만들 필요가 없다. 부모인 Thread에 이미 `name` 속성이 있으니까 `setName()`으로 넣거나 `super(name)`으로 넘기면 된다. `getName()`을 구현하지 않아도 부모 것이 불린다.

반대로 Runnable을 구현한 클래스는 Thread를 물려받은 게 아니라서 name이 없다. 그래서 이름을 직접 필드로 가져야 한다. 배틀 리팩토링에서 본 "부모에 있는데 자식이 또 선언한" 문제가 여기서도 나올 뻔했다.

`this.`을 찍어보면 name, priority, state 같은 속성이 get/set으로 보이는데, 보인다는 건 그 속성이 Thread에 있다는 뜻이라고 하셨다. 안 보인다고 없는 게 아니다.

## 우선순위는 믿으면 안 된다

`setPriority()`로 1(가장 낮음)부터 10(가장 높음)까지 줄 수 있다. 상수로는 `MIN_PRIORITY`, `NORM_PRIORITY`, `MAX_PRIORITY`다. 메인 Thread의 우선순위가 5라서, 따로 안 주면 자식 Thread도 5다.

```java
th1.setPriority(Thread.MAX_PRIORITY);   // 10
// th2는 그대로 5
th3.setPriority(Thread.MIN_PRIORITY);   // 1
th1.start(); th2.start(); th3.start();
```

우선순위 10짜리가 먼저 다 끝날 것 같은데 그렇지 않다. 직접 돌려봤더니 실행할 때마다 순서가 달랐고, 우선순위 1인 Thread가 중간에 몰아서 출력되기도 했다. 결과는 [4주차 과제 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-26-week4-assignment %})에 정리했다.

이유는 제어권이 자바한테 100% 있는 게 아니라서다. JVM은 OS의 스케줄러에 의존한다. 우선순위 값은 OS의 Thread 스케줄러에게 전달되지만, OS가 이 값을 무시하거나 약하게 반영할 수 있다. 우선순위가 최대여도 다른 Thread보다 먼저 실행된다는 보장은 없다.

> 우선순위라는 개념은 존재하지만 기대하는 대로 항상 반영되지 않는다

## 기억할 주의사항

중간 정리 슬라이드에 있던 내용이다.

- `run()` 메소드가 끝나면 Thread도 끝난다. 계속 살아있게 하려면 `run()` 안에 반복문을 써야 한다
- **한 번 종료한 Thread는 다시 시작시킬 수 없다.** 다시 쓰려면 객체를 새로 만들고 `start()`를 불러야 한다
- 한 Thread에서 다른 Thread를 강제 종료할 수 있는데, 이건 다음 주 Lab#4(자동차 경주)에서 `interrupt()`로 다룬다
- MultiThread를 만들기 전에 몇 개의 작업을 병렬로 실행할지부터 정한다

## 정리

- Thread를 만드는 방법은 Thread 상속, Runnable 구현, 익명 클래스/람다식 세 가지다
- 공통은 `run()`에 할 일을 쓰고 `start()`로 실행하는 것. `run()`을 직접 부르면 새 Thread가 안 생긴다
- 다중 상속이 안 되니까 Runnable 구현이 더 유연하다
- Thread를 상속하면 name을 물려받고, Runnable을 구현하면 이름을 직접 가져야 한다
- 우선순위는 줄 수 있지만 OS 스케줄러에 달려 있어서 믿으면 안 된다
- 끝난 Thread는 다시 start()할 수 없다
