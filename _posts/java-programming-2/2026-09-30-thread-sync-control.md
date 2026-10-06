---
title: "[자프실2] Thread 상태제어와 동기화: interrupt, synchronized, join, wait"
date: 2026-09-30 20:00:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - Thread
  - MultiThread
  - 동기화
  - synchronized
  - join
excerpt: "Thread 상태도를 다시 정리하고, stop() 대신 interrupt()로 멈추는 법, race condition과 synchronized, join(), sleep()과 wait()의 차이까지 MultiThread 상태제어를 묶었다."
toc: true
toc_sticky: true
---

추석 연휴 뒤 첫 수업에서 Thread를 마무리했다. [MultiThread 기초]({{ site.baseurl }}{% post_url java-programming-2/2026-09-23-thread-concepts %})와 [Thread 생성과 실행]({{ site.baseurl }}{% post_url java-programming-2/2026-09-23-thread-create-run %})에서 Thread를 만드는 법까지 했다면, 이번에는 여러 Thread가 같이 돌 때 순서와 상태를 어떻게 제어하는지다.

교수님께서 Thread의 학습 목표는 완벽한 동작 제어가 아니라고 하셨다. 공유 자원을 쓸 때 Thread의 순서와 상태를 원하는 대로 제어하려면 뭐가 필요한지 체감하는 게 목표라고. 어려운 건 3학년 운영체제를 위해 남겨두겠다고 하셨다. "반복문도 없는데 돈다", "반복문이 있는데 멈춰 있다" 같은 현상이 왜 생기는지 메커니즘만 이해하면 된다고.

나는 운영체제를 먼저 들어서 race condition, 임계 영역, 모니터 같은 말이 익숙하다. 교수님께서 운영체제를 먼저 들은 학생한테 "이걸 하고 나서 운영체제를 들었으면 어땠을까?" 하고 물어보셨는데, 3학년 때는 이론만 하니까 여기서 미리 맛보게 하려는 의도라고 하셨다. 나는 거꾸로 OS를 듣고 자바로 직접 해보니 이론이 코드로 보이는 느낌이었다.

실습(Lab#4~#7)은 [4주차 과제 2]({{ site.baseurl }}{% post_url java-programming-2/2026-10-01-week4-assignment2 %})에, 퀴즈는 [Thread 퀴즈]({{ site.baseurl }}{% post_url java-programming-2/2026-10-01-thread-quiz %})에 정리했다.

## Thread 상태도 다시 보기

교수님께서 정리한 걸 보니 잘못 쓴 사람이 있다고 짚어주셨다.

> start()를 부르면 Thread가 생성된다. → 틀림

생성은 `new`다. `start()`는 "OS한테 내 몸을 맡기는 것"이라고 하셨다. 그래서 상태로 보면 이렇다.

| 상태 | 언제 | 비유 |
| --- | --- | --- |
| NEW | `new`로 객체만 만든 상태. 아직 start() 전 | |
| RUNNABLE | start()가 호출됨. 실제로 실행 중일 수도, CPU를 기다리는 중일 수도 있다 | 그네 대기줄에 서 있는 것과 그네를 타는 순간이 둘 다 RUNNABLE |
| TIMED_WAITING | `sleep(ms)`, `join(ms)`처럼 시간을 정해두고 쉼 | "10초만 아이스크림 먹고 올게, 너 대신 타고 있어". 알람 맞춰놓고 알아서 깨는 스타일 |
| WAITING | `wait()`, `join()`. 누가 깨워주거나 끝날 때까지 무한 대기 | 얼음땡의 얼음. 누가 와서 땡(notify) 해주기 전까지 못 움직인다 |
| BLOCKED | 내 차례인데 다른 Thread가 lock을 쥐고 있어서 못 들어감 | 내 차례인데 앞에서 lock이 걸려 있다 |
| TERMINATED | run()이 끝남 | 그네 다 타고 집에 감 |

전에 정리할 때는 RUNNABLE을 "실행 대기"로만 생각했다. 그런데 자바의 RUNNABLE은 OS 관점의 Ready와 Running을 합친 거다. 그네 대기줄에 서 있어도, 그네를 타고 있어도 RUNNABLE이라는 비유가 딱 이 얘기였다. 자바는 지금 CPU를 잡고 있는지까지는 구분해주지 않는다. 그건 OS 스케줄러 몫이니까.

`stop()`, `suspend()`, `resume()`은 안전성 문제로 쓰지 않는다. 그럼 Thread는 어떻게 멈출까.

## stop() 대신 interrupt()

`interrupt()`는 Thread를 강제로 죽이는 게 아니라 **"멈춰줘"라는 신호만 보내는 것**이다. 슬라이드 설명대로 신호를 보내도 무시하고 안 멈출 수 있어서, 신호를 받았을 때 어떻게 할지는 프로그래머가 코드로 써야 한다.

신호를 받는 방법은 두 가지다.

```java
// 1. 일하는 중이면 직접 확인한다
if (Thread.interrupted()) {   // 신호 확인. 확인하면서 신호를 지운다
    return;                   // run()을 빠져나가면 Thread가 끝난다
}

// 2. sleep() 같은 대기 중이면 예외로 받는다
try {
    Thread.sleep(300);
} catch (InterruptedException e) {
    return;                   // 여기서 return을 안 하면 계속 달린다
}
```

`stop()`이 위험한 이유도 이해가 됐다. 외부에서 아무 때나 죽여버리면 계좌에서 돈을 빼고 아직 기록은 안 한 중간 상태로 끝날 수 있다. `interrupt()`는 신호만 주고, 멈출 타이밍은 Thread가 정리할 거 정리하고 스스로 고른다. 그래서 더 안전하다.

함정이 하나 있다. sleep 중에 interrupt를 받아서 `InterruptedException`이 나면, **신호가 지워진다.** catch에서 return이나 break를 안 하면 신호는 사라지고 Thread는 아무 일 없었다는 듯 계속 돈다. 실습과 퀴즈에서 직접 확인했다.

## 공유 자원과 race condition

MultiThread는 메모리를 공유한다. 여러 Thread가 같은 데이터에 동시에 접근하면 예상하지 못한 결과가 나올 수 있다. 은행 계좌에 만 원이 있는데 A와 B가 동시에 만 원을 인출하면 -10000원이 될 수 있다.

```java
this.balance += amount;
```

이 한 줄이 문제다. 코드로는 한 줄이지만 실제로는 "읽기 → 더하기 → 쓰기" 세 단계다. 두 Thread가 같은 값을 읽고 각자 더해서 쓰면, 먼저 쓴 쪽의 결과가 덮여서 사라진다. 이렇게 실행 순서(타이밍)에 따라 결과가 달라지는 상황을 **race condition(경쟁 조건)**이라고 한다. 슬라이드에 "더 찾아볼 키워드"로 적혀 있었다.

### 출력문이 버그를 가린다

Lab#5 MyBank에서 제일 신기한 부분이었다. 입출금할 때마다 출력문을 넣으면 오류가 안 보이는데, 출력문을 다 빼고 반복 횟수를 늘리면 "오류발생!"이 뜬다.

이유는 `System.out.println`이 내부적으로 동기화되어 있어서다. 출력하는 동안 Thread들이 줄을 서게 되고, 그게 실행 흐름을 늦추고 정렬하는 효과를 낸다. 교수님 표현으로는 출력문이 **타이밍 완충 장치** 역할을 한다. 버그가 없어진 게 아니라 눈에 안 띄게 가려진 것뿐이다. 그래서 출력문을 빼서 완충 장치를 없애고, sleep을 줄이고, 반복을 늘리면 드러난다.

디버깅하려고 출력문을 넣었더니 버그가 사라지는 상황이라니. 실제로 겪으면 정말 당황스러울 것 같다.

## synchronized: 화장실 열쇠

해결책은 공유 데이터에 접근하는 Thread들을 한 줄로 세우는 것이다. 한 Thread가 작업을 끝낼 때까지 다른 Thread는 기다린다. 이걸 **Thread 동기화**라고 하고, 자바에서는 `synchronized`로 한다.

```java
public synchronized void deposit(int amount) {
    balance += amount;
}

public synchronized void withdraw(int amount) {
    if (balance >= amount) {
        balance -= amount;
    }
}
```

`synchronized`는 "여기는 한 번에 한 Thread만"이라고 표시하는 키워드다. 이렇게 표시한 영역을 **임계 영역(critical section)**이라고 한다.

교수님 비유는 역시 화장실이었다. "한 놈이 들어가면 다른 놈은 못 들어온다." 카페 화장실에 가면 열쇠를 받아서 문을 따고, 쓰고 나와서 다시 걸어놓는다. 그 시스템 그대로다. 그 열쇠가 **lock**이다. lock을 가진 Thread만 실행할 수 있고, 없으면 BLOCKED 상태로 문 앞에서 기다린다.

메소드 전체에 붙일 수도 있고, `synchronized (객체) { ... }`처럼 필요한 블록에만 붙일 수도 있다. 메소드에 붙이면 그 객체(this)의 lock을 쓴다.

Vector 이야기도 하셨다. Vector는 메소드마다 동기화가 되어 있어서 MultiThread에서 안전하지만, 혼자 쓸 때도 매번 lock을 잡는다. "혼자 쓰는데 화장실 문을 열쇠로 잠그고 다니는 비효율성" 때문에 지금은 많이 안 쓴다고 하셨다. 3주차 컬렉션에서 Vector를 잠깐 봤던 게 여기서 이어졌다.

### synchronized만으로는 "웬만큼"

교수님께서 synchronized를 붙이면 "웬만큼" 잡힌다고 하셨다. 웬만큼인 이유가 Lab#6 FamilyAccount에서 나온다. 예금자와 인출자가 한 계좌를 쓸 때 synchronized를 붙이면 잔액이 깨지지는 않는다. 그런데 **누가 먼저 할지**는 여전히 스케줄러 마음이다. 슬라이드 28쪽 표처럼 최종 잔고가 같게 나와도 거래 과정은 다를 수 있다.

"저축 한 번, 인출 한 번씩 번갈아"처럼 순서까지 맞춰야 한다면 synchronized로는 부족하고 상태를 제어해야 한다. 슬라이드에는 wait(), notify(), join()을 적절히 쓰라고 되어 있는데, 번갈아 하게 만드는 핵심은 wait()와 notify()다.

## join(): 끝날 때까지 기다리기

`join()`은 해당 Thread가 끝날 때까지 지금 Thread를 기다리게 한다.

```java
pig1.start();
pig2.start();
pig3.start();

pig1.join();   // 첫째가 올 때까지 대기
pig2.join();   // 둘째가 올 때까지 대기
pig3.join();   // 셋째가 올 때까지 대기
System.out.println("모두 왔구나! 문 열어줄게~");
```

Lab#7 아기돼지삼형제다. 교수님 설명이 와닿았다. join은 내가 다 끝났다고 종료하는 게 아니라, **"쟤 끝날 때까지 기다릴 거야"라고 기다리는 쪽(main)이 부르는 것**이다. main에는 반복문이 없는데도 셋이 다 올 때까지 마지막 줄로 안 넘어간다. 교수님께서 main에 while 하나 없는데 세 마리가 다 끝날 때까지 기다린다고 짚어주신 게 이 부분이다.

수업 때 한 학생이 실행 순서가 이해가 안 된다고 질문했다. 교수님 답은 "위에서 아래로 흐른다는 개념을 버려라"였다. pig1.join()이 먼저 써 있다고 첫째가 먼저 온다는 뜻은 아니다. join 세 줄은 "1, 2, 3이 다 끝나면 알려줘"라는 뜻이다. 계산 Thread들이 다 끝난 뒤 결과를 모아 써야 할 때 주로 쓴다.

교수님께서 어릴 때 혼자 노는 걸 좋아해서 "혼자 오지 말고 같이 오라"는 말을 들었고, 아이를 키울 때도 "동생 챙겨서 같이 와야 문 열어준다"고 하셨다는 이야기를 해주셨다. 엄마 돼지가 딱 그거다.

## sleep()과 wait()

둘 다 Thread를 멈추지만 성격이 완전히 다르다.

| | sleep() | wait() |
| --- | --- | --- |
| 비유 | 알람 맞춰두고 눈 감고 쉬다가 정해진 시간에 깸 | 누군가 깨워줄 때까지 잠자기 |
| 소속 | Thread 클래스(static) | Object 클래스 |
| lock | 쥔 채로 잔다 | 내려놓고 잔다 |
| 깨어나는 조건 | 시간이 지나면 자동 | 누가 notify(), notifyAll() 해줘야 |
| 상태 | TIMED_WAITING | WAITING |

lock을 쥐고 자느냐 내려놓고 자느냐가 핵심이다. 화장실 안에서 sleep하면 문을 잠근 채로 자는 거라 밖에서 다들 기다린다. wait는 열쇠를 걸어두고 나와서 벤치에서 기다리는 것이다. 그래서 다른 Thread가 들어가서 일을 하고 "다 했어"(notify)라고 깨워줄 수 있다.

wait()가 Object 소속인 이유도 여기서 이해됐다. 모든 객체가 각자 lock(모니터)과 대기실을 갖고 있고, wait()는 **그 객체의 대기실**에서 기다리는 것이기 때문이다. 같은 이유로 wait()와 notify()는 그 객체의 lock을 쥔 상태, 즉 `synchronized` 안에서만 부를 수 있다. 밖에서 부르면 `IllegalMonitorStateException`이 난다. 퀴즈에서 직접 확인했다.

### 잠자는 숲속의 공주와 얼음땡

노션 Lab#8이 이걸 그대로 시나리오로 만든 것이다. 공주는 `synchronized (lock)` 안에서 `lock.wait()`로 잠들고, 3초 뒤 왕자가 `synchronized (lock)` 안에서 `lock.notify()`로 깨운다.

얼음땡 예제도 있었다. 짱구, 맹구, 훈이가 놀이터(공유 객체)에서 각자 `freeze()`로 얼음(wait)을 외치고, 철수가 `unfreezeAll()`에서 `notifyAll()`로 단체 땡을 외친다. 중간에 `getState()`로 찍어보면 셋 다 WAITING이 나온다. notify()는 하나만, notifyAll()은 기다리는 전부를 깨운다는 차이가 얼음땡으로 보니 바로 와닿았다.

### 빵집: 생산자와 소비자

마지막은 생산자-소비자 문제다. 교수님께서 빵집에 비유하셨다. 생산자는 빵을 만들고 소비자는 빵을 산다. 접시가 비어 있으면 소비자가 기다려야 하고, 생산자는 만들었으면 "다 만들었어"라고 알려야 하고, 소비자는 "잘 먹었어"라고 알려야 한다.

```java
public synchronized int get() {      // 소비자
    while (empty) {                  // 접시가 비었으면
        wait();                      // lock 내려놓고 대기
    }
    empty = true;
    notifyAll();                     // 생산자 깨우기
    return data;
}

public synchronized void put(int data) {   // 생산자
    while (!empty) {                       // 접시가 차 있으면
        wait();
    }
    this.data = data;
    empty = false;
    notifyAll();                           // 소비자 깨우기
}
```

(예외 처리는 생략했다.) `empty`라는 깃발을 세워두고, 상태가 안 맞으면 wait로 기다리고, 상태를 바꾼 뒤에는 notifyAll로 깨운다. 이렇게 하면 생산과 소비가 "쿵짝쿵짝" 번갈아 맞게 나온다고 하셨다. 임계 영역의 안전뿐 아니라 처리 순서까지 맞춘 것이다.

운영체제 시간에 배운 모니터가 바로 이 구조였다. synchronized가 상호 배제를, wait()와 notifyAll()이 조건 변수 역할을 한다. 그리고 `if`가 아니라 `while`로 상태를 다시 검사하는 이유도 OS에서 배웠다. 깨어났다고 해서 내가 원하는 상태라는 보장이 없다. notifyAll()로 여러 Thread가 같이 깨면 다른 Thread가 먼저 lock을 잡고 상태를 바꿔버릴 수 있기 때문이다.

교수님께서는 여기까지만 알면 된다고, 이걸 완벽하게 코딩할 수 있어야 하냐면 "아니에요, 3학년 때 가서 하세요"라고 하셨다. 필요성만 느끼면 된다고. 수업에서는 Lab#5, #6은 설명만 듣고 Lab#7 아기돼지를 직접 실습했다. Lab#8~#10은 고학년이나 궁금한 사람이 해보라는 보충이다.

## 정리

- 생성은 new, start()는 OS에 맡기는 것. RUNNABLE에는 대기와 실행이 같이 들어 있다
- interrupt()는 신호일 뿐이다. 받는 쪽이 확인하고 스스로 끝내야 한다. 예외로 받으면 신호가 지워진다
- 공유 자원에 동시에 접근하면 race condition이 생긴다. 출력문은 그걸 가릴 뿐이다
- synchronized는 화장실 열쇠다. 한 번에 한 Thread만 들어간다
- join()은 기다리는 쪽이 부른다
- sleep()은 lock을 쥐고 자고, wait()는 내려놓고 잔다. wait/notify는 synchronized 안에서만

결국 남는 건 누가 화장실을 쓰고 있으면 문을 억지로 열고 들어가면 안 되고, 그걸 코드로 막아줘야 한다는 것이다. 더 깊은 부분은 운영체제에서 배운 것과 연결해서 천천히 다시 보려고 한다. 이번 주 금요일부터는 [SOLID]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-solid-ioc-di %})로 넘어간다.
