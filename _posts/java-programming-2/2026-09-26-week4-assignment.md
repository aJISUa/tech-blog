---
title: "[자프실2] 4주차 과제: Thread 만들기 Lab#1~#3과 우선순위 테스트"
date: 2026-09-26 20:00:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - Thread
  - Runnable
  - MultiThread
  - 과제
excerpt: "Thread를 만드는 세 가지 방법(Lab#1), 말 달리기(Lab#2), 짱구 맹구 흰둥이(Lab#3)를 직접 돌려보면서 실행 순서가 매번 달라지는 걸 확인했다."
toc: true
toc_sticky: true
---

이번 주 과제는 Thread 핵심 이론 정리와 Lab#1~#3 복기다. 금요일부터 추석 연휴라서 교수님께서 부담 없는 범위로 정해주셨다. 퀴즈 만들기는 4장을 끝내고 다음 주에 한다.

이론은 [MultiThread 기초]({{ site.baseurl }}{% post_url java-programming-2/2026-09-23-thread-concepts %})와 [Thread 생성과 실행]({{ site.baseurl }}{% post_url java-programming-2/2026-09-23-thread-create-run %})에 정리했고, 이 글은 실습 부분이다.

코드는 이클립스 `JAVA2` 프로젝트에 `thread` 패키지를 만들어서 넣었다. 노션에 코드가 있어도 복붙하지 말고 직접 쳐보라고 하셔서 손으로 치면서 주석을 달았다.

| 실습 | 내용 |
| --- | --- |
| Lab#1 | Thread 생성 방법 세 가지 |
| Lab#1 추가 | start()와 run()의 차이 확인 |
| Lab#2 | 이름 붙은 Thread와 말 달리기(sleep) |
| Lab#3 | 짱구 맹구 흰둥이를 두 가지 방식으로 |
| 추가 | 우선순위 테스트 |

## 1. Lab#1: Thread 만드는 세 가지 방법

### 실습 목표

Thread 상속, Runnable 구현, 익명 클래스, 람다식으로 Thread를 만들어서 한꺼번에 돌려본다. 네 개가 어떻게 섞여서 실행되는지 본다.

### 코드

```java
// 방법 1: Thread 클래스를 상속
public class MyThread extends Thread {
    @Override
    public void run() {
        work();     // run()에 다 쓰지 않고 할 일을 메소드로 빼두면 읽기 좋다
    }

    private void work() {
        for (int i = 1; i <= 5; i++) {
            // currentThread(): 지금 이 코드를 실행 중인 쓰레드 객체를 가져온다
            System.out.println("[상속] " + Thread.currentThread().getName() + " " + i + "번째");
            try {
                Thread.sleep(500);   // 0.5초 쉬기. 안 쉬면 너무 빨리 끝나서 섞이는 게 안 보인다
            } catch (InterruptedException e) {
                return;
            }
        }
    }
}
```

```java
// 방법 2: Runnable 인터페이스를 구현
public class MyRunnable implements Runnable {
    private String myName;   // Thread의 name과는 별개로 내가 직접 가진 이름

    public MyRunnable(String myName) {
        this.myName = myName;
    }

    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
            // Runnable 자체는 쓰레드가 아니라서 getName()이 없다. currentThread()로 가져온다
            System.out.println("[구현] " + myName + "(" + Thread.currentThread().getName() + ") " + i + "번째");
            ...
        }
    }
}
```

```java
public class Lab1_CreateThread {
    public static void main(String[] args) {
        Thread t1 = new MyThread();                        // 1. 상속
        Thread t2 = new Thread(new MyRunnable("러너블이"));  // 2. 구현

        // 3-1. 익명 클래스: 이름 없는 Runnable을 그 자리에서 만들어 넘긴다
        Thread t3 = new Thread(new Runnable() {
            @Override
            public void run() { ... }
        });

        // 3-2. 람다식: Runnable은 추상 메소드가 run() 하나뿐이라 화살표로 줄일 수 있다
        Thread t4 = new Thread(() -> { ... });

        System.out.println("t1 이름: " + t1.getName() + ", t2 이름: " + t2.getName());
        t1.setName("상속쓰레드");

        // run()이 아니라 start()!
        t1.start();
        t2.start();
        t3.start();
        t4.start();

        System.out.println("메인 쓰레드(" + Thread.currentThread().getName() + ") 할 일 끝. 살아있는 쓰레드 수 = "
                + Thread.activeCount());
    }
}
```

익명 클래스와 람다식은 sleep을 0.3초로 줘서 다른 둘(0.5초)보다 빨리 끝나게 했다.

#### 실행 전에 예상한 것

- 이름을 안 주면 `Thread-0`, `Thread-1`처럼 자동으로 붙는다.
- 네 Thread가 번갈아 출력될 것 같다. 누가 먼저 나올지는 모른다.
- 0.3초로 쉬는 익명, 람다가 먼저 끝나고 0.5초인 상속, 구현이 나중에 끝날 것 같다.
- 메인은 start()만 시켜놓고 자기 줄을 먼저 출력하고 끝날 것 같다. 그래도 작업 Thread가 살아 있으니 프로그램은 안 끝난다.

#### 실제 결과

![Lab#1 세 가지 생성 방법 실행 결과]({{ '/assets/images/java-programming-2/t4-lab1-create.png' | relative_url }})

예상대로 네 Thread가 섞여서 나왔다. 메인의 "할 일 끝" 줄은 제일 먼저가 아니라 작업 Thread 두어 줄 뒤에 나왔다. `start()`를 부른 뒤 메인이 다음 줄로 가는 사이에 이미 다른 Thread가 출력을 시작한 거다. 그래도 메인은 네 번째 줄에서 할 일을 다 끝냈고, 그 뒤로도 작업 Thread들이 계속 돌았다. 메인이 끝나도 프로그램이 안 끝나는 걸 눈으로 봤다.

신기했던 건 `Thread.activeCount()`가 5로 나온 것이다. 내가 만든 건 4개인데, 메인 Thread까지 세서 5다. 메인도 Thread라는 게 숫자로 확인됐다.

시작은 t1부터 시켰는데 출력은 익명이나 람다가 먼저 나올 때가 많았다. 여러 번 돌려보니 매번 달랐다. 끝나는 건 쉬는 시간이 짧은 익명, 람다가 먼저였다.

### start()와 run()의 차이

같은 코드를 `run()`으로도 불러봤다.

```java
Thread a = new Thread(() -> System.out.println("  실행한 쓰레드: " + Thread.currentThread().getName()));
a.run();     // 새 쓰레드가 안 생기고 main이 그냥 실행한다

Thread b = new Thread(() -> System.out.println("  실행한 쓰레드: " + Thread.currentThread().getName()));
b.start();   // 새 쓰레드가 생겨서 그 쓰레드가 실행한다

try {
    b.start();   // 한 번 끝난 쓰레드를 다시 start() 하면?
} catch (IllegalThreadStateException e) {
    System.out.println("=== 끝난 쓰레드를 다시 start()하면: " + e.getClass().getSimpleName());
}
```

#### 실행 전에 예상한 것

- `run()`으로 부르면 실행한 Thread가 `main`으로 찍힌다.
- `start()`로 부르면 `Thread-1` 같은 이름이 찍힌다.
- 끝난 Thread를 다시 start()하면 예외가 날 것 같다. 슬라이드에 "한 번 종료한 Thread는 다시 시작시킬 수 없다"고 나와 있었다.

#### 실제 결과

![start()와 run() 비교 실행 결과]({{ '/assets/images/java-programming-2/t4-lab1-startvsrun.png' | relative_url }})

`run()`은 `main`, `start()`는 `Thread-1`로 찍혔다. 다시 start()하면 `IllegalThreadStateException`이 난다.

한 가지 더 보였다. `start()`로 만든 Thread의 출력이 예외 메시지보다 늦게 나왔다. 메인이 `b.start()`를 부르고 바로 다음 줄로 넘어가버려서, 새 Thread가 출력할 틈도 없이 메인이 먼저 진행한 거다. `start()`는 "실행해줘"라고 부탁만 하고 끝난다는 걸 알 수 있었다.

### 실습 후기

- `run()`과 `start()`의 차이가 제일 중요했다. 겉보기에는 똑같이 실행되니까 `run()`을 불러도 모르고 지나갈 뻔했다.
- 람다식이 확실히 짧다. 대신 이 Thread를 다른 데서도 쓰려면 이름 있는 클래스로 빼는 게 맞다는 것도 알겠다.
- Runnable로 만들면 `getName()`이 없어서 `Thread.currentThread().getName()`을 써야 했다. Runnable은 Thread가 아니라 "할 일"이라는 게 이 차이에서 느껴졌다.

## 2. Lab#2: 이름 붙이기와 말 달리기

### 실습 목표

이름을 가진 Runnable Thread를 만들고, `sleep()`을 랜덤하게 줘서 실행 순서가 매번 바뀌는 걸 체감한다.

### 코드

```java
public class Horse implements Runnable {
    private String name;
    private int sleepTime;
    // static으로 하나만 두고 말들이 같이 쓴다
    private final static Random generator = new Random();

    public Horse(String name) {
        this.name = name;
        this.sleepTime = generator.nextInt(3000);   // 0~3초 사이에서 각자 다르게
    }

    @Override
    public void run() {
        try {
            Thread.sleep(sleepTime);   // 달리는 데 걸리는 시간이라고 생각하면 된다
        } catch (InterruptedException e) {
            System.out.println(name + " 말이 경주를 중단했습니다");
            return;
        }
        System.out.println(name + "말이 경주를 완료하였습니다 (" + sleepTime + "ms)");
    }
}
```

```java
// 1. 이름만 다르게 준 쓰레드 두 개
Thread t1 = new Thread(new MyRunnable("First"));
Thread t2 = new Thread(new MyRunnable("Second"));
t1.start();
t2.start();

// 2. 말 세 마리. 쉬는 시간이 랜덤이라 도착 순서도 매번 달라진다
Thread h1 = new Thread(new Horse("질풍"));
Thread h2 = new Thread(new Horse("번개"));
Thread h3 = new Thread(new Horse("적토마"));
h1.start();
h2.start();
h3.start();
```

얼마나 쉬었는지 보려고 출력에 `sleepTime`을 같이 찍었다.

#### 실행 전에 예상한 것

- First와 Second는 0.5초씩 쉬니까 번갈아가며 나올 것 같다.
- 말은 출발 순서(질풍, 번개, 적토마)와 상관없이 `sleepTime`이 짧은 말이 먼저 도착할 것 같다.
- 실행할 때마다 도착 순서가 바뀔 것 같다.

#### 실제 결과

![Lab#2 말 달리기 실행 결과]({{ '/assets/images/java-programming-2/t4-lab2-horse.png' | relative_url }})

예상대로였다. 같이 찍은 밀리초를 보니 도착 순서가 정확히 sleepTime이 짧은 순서였다. 출발은 질풍이 먼저 했는데 번개가 먼저 들어오는 식이다.

First와 Second는 한 줄씩 번갈아 나왔다. 둘 다 0.5초씩 쉬니까 깨어나는 타이밍이 비슷해서 그런 것 같다.

### 실습 후기

- `sleep()`을 걸면 순서가 눈에 보인다. 슬립이 없으면 너무 빨리 끝나서 순서를 확인하기 어렵다고 하신 이유를 알겠다.
- 랜덤한 sleep은 "각자 속도가 다른 작업"을 흉내 내는 방법이었다. 실제로 네트워크 응답이나 파일 읽기도 걸리는 시간이 제각각일 테니 비슷한 상황이겠다.

## 3. Lab#3: 짱구 맹구 흰둥이

### 실습 목표

같은 동작을 Thread 상속(Player1)과 Runnable 구현(Player2) 두 가지로 만들어서, 이름이 어떻게 붙는지와 실행 순서가 매번 어떻게 달라지는지 확인한다.

### 코드

```java
// 방법 1(Thread 상속)으로 만든 짱구네 친구들
public class Player1 extends Thread {
    private String sound;
    private int time;

    public Player1(String name, String sound, int time) {
        // 이름은 내가 따로 필드로 갖지 않고 부모(Thread)가 가진 name에 넣는다
        super(name);
        this.sound = sound;
        this.time = time;
    }

    @Override
    public void run() {
        int i = 0;
        while (true) {
            if (i == time) break;              // 정해진 횟수를 다 하면 멈춘다
            System.out.printf("%d %s %s %n", (i + 1), getName(), sound);
            i++;
        }
        System.out.println(getName() + "끝");
    }
}
```

```java
// 방법 2(Runnable 구현). Runnable에는 name이 없어서 이름을 직접 필드로 가져야 한다
public class Player2 implements Runnable {
    private String name;
    private String sound;
    private int time;
    ...
}
```

```java
public class Lab3_Jjanggu {
    public static void main(String[] args) {
        Thread p1 = new Player1("짱구", "신난다신난다~", 20);
        Thread p2 = new Player1("맹구", "훌쩍~", 20);
        Thread p3 = new Player1("흰둥이", "멍멍멍멍~", 20);

        Thread th1 = new Thread(new Player2("**짱구", "신난다신난다~", 10));
        Thread th2 = new Thread(new Player2("**맹구", "맹맹~", 10));
        Thread th3 = new Thread(new Player2("**흰둥이", "멍멍멍~", 10));

        System.out.println("p1: " + p1.getName() + ...);     // 내가 준 이름
        System.out.println("th1: " + th1.getName() + ...);   // 자동 이름
        th1.setName("**짱구");

        p1.start(); p2.start(); p3.start();
        th1.start(); th2.start(); th3.start();
    }
}
```

`Player1`은 `super(name)`으로 부모(Thread)의 이름에 넣었고, `Player2`는 Thread가 아니라서 이름을 직접 필드로 가졌다.

#### 실행 전에 예상한 것

- `p1.getName()`은 짱구, 맹구, 흰둥이가 나오고, `th1.getName()`은 `Thread-0` 같은 자동 이름이 나올 것 같다.
- 여섯 Thread가 뒤섞여서 출력될 것 같다.
- 끝나는 순서는 실행할 때마다 달라질 것 같다.

#### 실제 결과

![Lab#3 짱구 맹구 흰둥이 실행 결과]({{ '/assets/images/java-programming-2/t4-lab3-run1.png' | relative_url }})

이름은 예상대로였다. Thread를 상속한 쪽은 내가 준 이름이 그대로 나오고, Runnable로 만든 쪽은 `Thread-0`, `Thread-1`, `Thread-2`가 나왔다. `setName()`으로 바꾸니 `**짱구`로 바뀌었다.

그런데 출력이 예상과 달랐다. 여섯이 골고루 섞일 줄 알았는데, 짱구가 20줄을 다 찍고 나서 다음 Thread로 넘어가는 식으로 덩어리째 나왔다. Lab#1에서는 잘만 섞였는데.

차이는 `sleep()`이었다. Lab#1은 한 줄 찍고 0.5초씩 쉬니까 그사이 다른 Thread가 끼어들 수 있는데, Lab#3은 쉬지 않고 20줄을 찍는다. 한 번 CPU를 잡으면 20줄 정도는 순식간에 끝나버려서 중간에 끼어들 틈이 없었던 것 같다.

그래서 여러 번 실행해봤다. 아래는 다시 실행한 화면인데, 끝나는 순서가 첫 번째와 다르다.

![Lab#3 다시 실행한 결과]({{ '/assets/images/java-programming-2/t4-lab3-run2.png' | relative_url }})

끝나는 순서가 매번 달랐다. 터미널에서 다섯 번 더 돌린 결과를 적어보면 이렇다.

| 실행 | 끝난 순서 |
| --- | --- |
| 1회 | 맹구, \*\*짱구, \*\*맹구, 짱구, \*\*흰둥이, 흰둥이 |
| 2회 | 흰둥이, 짱구, 맹구, \*\*짱구, \*\*맹구, \*\*흰둥이 |
| 3회 | 맹구, 흰둥이, 짱구, \*\*짱구, \*\*맹구, \*\*흰둥이 |
| 4회 | 짱구, \*\*짱구, 흰둥이, \*\*흰둥이, 맹구, \*\*맹구 |
| 5회 | 흰둥이, 맹구, 짱구, \*\*짱구, \*\*맹구, \*\*흰둥이 |

`p1.start()`를 제일 먼저 불렀는데 짱구가 꼴찌로 끝난 적도 있다. 교수님 말씀대로 먼저 start했다고 먼저 끝난다는 보장이 없었다.

### 실습 후기

- 같은 동작인데 만드는 방법만 다른 두 클래스를 나란히 두니까 차이가 확실했다. Thread를 상속하면 이름을 물려받고, Runnable은 직접 들고 있어야 한다.
- 실행 순서가 매번 다른 걸 직접 보니까 "결과가 한 번 맞게 나왔다고 맞는 코드가 아니다"라는 말이 이해됐다. 2주차 리팩토링 때와 비슷한 교훈인데 이유는 완전히 다르다.
- 출력이 섞이냐 덩어리로 나오냐가 sleep 하나로 갈렸다. 순서를 확인하고 싶을 때는 일부러 sleep을 넣어보는 게 방법이겠다.
- 궁금한 점: 덩어리로 나오는 게 정말 "너무 빨리 끝나서"인지, 아니면 출력 자체가 한 번에 밀려 나오는 건지 확실하지 않다. 반복 횟수를 아주 크게 하면 중간에 섞일지 다음에 실험해보고 싶다.

## 4. 우선순위 테스트

슬라이드에서 "우선순위를 믿지 말라"고 한 부분을 직접 확인해봤다.

```java
Thread th1 = new Thread(new PriorityRunner("Thread-1"));
Thread th2 = new Thread(new PriorityRunner("Thread-2"));
Thread th3 = new Thread(new PriorityRunner("Thread-3"));

th1.setPriority(Thread.MAX_PRIORITY);   // 10
// th2는 그대로 5 (메인의 우선순위가 5라서 자식도 5)
th3.setPriority(Thread.MIN_PRIORITY);   // 1

th1.start(); th2.start(); th3.start();
```

각 Thread는 자기 이름과 우선순위를 10번 출력하고 끝난다.

#### 실행 전에 예상한 것

우선순위대로라면 Thread-1(10)이 먼저 다 끝나고, Thread-2(5), Thread-3(1) 순서일 것 같다. 슬라이드에서 그렇지 않다고 했으니 어긋나긴 할 텐데 얼마나 어긋날지가 궁금했다.

#### 실제 결과

![우선순위 테스트 실행 결과]({{ '/assets/images/java-programming-2/t4-priority.png' | relative_url }})

이클립스에서 한 번 돌렸더니 오히려 우선순위대로 10, 5, 1 순서로 깔끔하게 나왔다. 여기까지만 보면 우선순위가 잘 지켜지는 것처럼 보인다.

그런데 같은 코드를 다시 돌렸더니 결과가 완전히 달랐다.

![우선순위 테스트 다시 실행한 결과]({{ '/assets/images/java-programming-2/t4-priority2.png' | relative_url }})

이번에는 우선순위가 5인 Thread-2가 먼저 두 줄을 출력하고 시작했다. 우선순위 10인 Thread-1이 그다음에 몰아서 찍고, 우선순위 1인 Thread-3이 중간에 끼어든다. 끝나는 순서는 Thread-1, Thread-2, Thread-3이었지만 실행 과정은 우선순위 순서가 전혀 아니었다.

터미널에서 여러 번 더 돌려봐도 매번 달랐다. 한 Thread가 10줄만 도는 짧은 작업이라, 한 번 CPU를 잡으면 그대로 끝나버리는 경우가 많은 것 같다.

결국 슬라이드에 나온 대로다. 우선순위대로 나오는 것처럼 보일 때도 있지만 그게 보장은 아니다. JVM이 우선순위를 OS 스케줄러에 전달하긴 해도 OS가 무시하거나 약하게 반영하기 때문이다. 우선순위를 높였으니 먼저 실행되겠지 하고 코드를 짜면 안 된다는 걸, 같은 코드의 두 실행 결과가 갈리는 걸로 확인했다.

## 마치며

이번 주에 제일 크게 바뀐 생각은 "지금까지 짠 코드가 전부 SingleThread였다"는 것이다. main이 Thread라는 걸 몰랐는데, `activeCount()`가 5로 나오는 걸 보고 확실히 알았다.

그리고 MultiThread에서는 같은 코드를 돌려도 결과가 매번 다르다. 테스트로 한 번 맞게 나왔다고 안심하면 안 되고, 순서를 맞춰야 한다면 별도로 제어해야 한다. 그 제어 방법(`synchronized`, `wait()`, `notify()`, `join()`)이 연휴 뒤에 배울 Lab#4부터의 내용이다.

다음 주 Lab#4는 자동차 경주인데, Thread를 안전하게 멈추는 방법을 `stop()` 대신 `interrupt()`, `break`, `return`으로 다룬다고 하셨다. 슬라이드를 미리 보고 오라고 하셔서 코드는 읽어뒀다.
