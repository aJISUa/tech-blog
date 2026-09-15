---
title: "[자프실2] 2주차 과제: Vehicle-Driver 리팩토링, 짱구네 집, 하늘 싱글톤"
date: 2026-09-15 20:00:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - OOP
  - 리팩토링
  - 다형성
  - 싱글톤
  - 과제
excerpt: "결과는 잘 나오는 Vehicle-Driver 코드에서 문제 4개를 찾아서 고쳐봤다. 짱구네 집 다형성이랑 하늘 싱글톤도 직접 돌려서 확인했다."
toc: true
toc_sticky: true
---

이번 과제는 짱구, 리모컨, 리팩토링 중에 하나 이상을 하고, 거기에 나한테 필요한 걸 더하는 거였다. 나는 이렇게 했다.

- 리팩토링: Vehicle-Driver 코드를 분석하고, 문제를 직접 재현해본 다음 고쳤다
- 짱구: 퀴즈 1~4번을 코드로 확인했다
- 나한테 필요한 것: 수업 때 컴파일이 안 됐던 하늘 싱글톤을 다시 만들었다

## 1. Vehicle-Driver 리팩토링

### 지금 코드 상태

돌려보면 결과는 퀴즈에서 원하는 대로 잘 나온다.

```text
총 주행거리 : 1200   ← 타요버스 (연료 100 × 연비 12)
총 주행거리 : 1210   ← 쑝비행기 10km 추가 (100 × 0.1)
총 주행거리 : 1410   ← 탱크탱크 200km 추가 (50 × 4)
```

![원본 Vehicle-Driver 실행 결과]({{ '/assets/images/java-programming-2/vd-original-console.png' | relative_url }})

원본 코드를 이클립스에서 돌린 결과다. 버스 1200km, 비행기 10km, 탱크 200km를 달리고 총 주행거리가 1410까지 잘 나온다.

![원본 Vehicle-Driver 클래스 다이어그램]({{ '/assets/images/java-programming-2/vd-original-uml.png' | relative_url }})

PlantUML로 원본 구조를 뽑아봤다. 여기서부터 이상한 부분이 조금씩 보였다. Driveable에는 `drive()` 하나만 있고, Driver에는 `input: Scanner`랑 drive 메소드가 5개나 있다. Vehicle에는 빈 생성자 `Vehicle()`와 setter 3개가 있고, 생성자는 연료량을 int로 받는데 필드는 double이다. 소리 내는 메소드도 비행기는 `fly()`, 버스는 `운행하기()`로 이름이 다르다.

교수님이 배틀 코드 보여주실 때 "실행은 너무 잘 되는데 열어보면 심각한 문제가 있다"고 하셨던 게 생각나서, 결과가 맞게 나오는 이 코드도 한번 의심해보기로 했다. 코드를 읽으면서 이상한 부분을 적어두고, 진짜 문제가 되는지 테스트 코드를 따로 만들어서 확인했다.

### 코드 읽으면서 이상했던 부분

**Vehicle의 move()**
- `return this.주행거리;`로 이번에 달린 거리가 아니라 지금까지 달린 전체 거리를 돌려준다. 그런데 Driver는 이 값을 `distance +=`로 계속 더한다.
- `if (this.연료량 <= 0.01) break;`로 멈추는데, 요구사항은 "10km 갈 연료가 없으면 멈춘다"였다. 이대로면 연료가 0.01보다만 많아도 출발한다.

**생성자**
- `Vehicle()`, `버스()` 같은 빈 생성자가 있어서 이름도 연비도 없는 차를 만들 수 있다. 연비가 0이면 `10/연비`가 무한대가 된다.
- 생성자는 `int 연료량`으로 받는데 필드는 `double 연료량`이다.

**Driveable 인터페이스**
- 인자 없는 `drive()` 하나만 있다. Driver에 있는 `drive(Moveable m)`은 인터페이스에 없다.

**Driver**
- `Scanner input` 필드가 있고 `input.nextLine()`을 부른다. 운전자가 키보드를 갖고 있는 것도 이상하고, 차 한 대 운전할 때마다 엔터를 쳐야 넘어간다.
- `drive(Vehicle)`, `drive(버스)`, `drive(탱크)`, `drive(Moveable)`이 다 "문구 출력하고 move() 부르기"로 하는 일이 같다. `drive(Moveable)` 하나면 충분해 보였다.
- `java.lang.reflect.Type`, `javax.naming.Name`을 import했는데 안 쓴다.

**그 밖에**
- `show(Vehicle[] vs)` 안에 `show()`랑 똑같은 printf를 복사해서 넣어뒀다.
- 버스는 `운행하기()`, 비행기는 `fly()`, 탱크는 println을 바로 써서 소리 내는 방식이 제각각이다. 그리고 하위 클래스에서 `super.move()`를 깜빡해도 컴파일러가 모른다.

### 진짜 문제가 되는지 확인

원본 클래스를 그대로 가져다 쓰는 테스트 코드를 만들었다.

```java
public class BugDemo {
    public static void main(String[] args) {
        System.out.println("=== 문제1: 같은 차를 두 번 drive하면 총 주행거리가 중복 누적된다 ===");
        Driver d = new Driver("미미");
        탱크 t1 = new 탱크("탱크A", 5, 4);   // 10km에 2.5L -> 딱 20km
        d.drive((Moveable) t1);
        d.drive((Moveable) t1);            // 연료 0이라 한 칸도 못 가는데
        System.out.println("탱크A 실제 주행거리 = " + t1.get주행거리() + "km, 드라이버 총 주행거리 = " + d.distance + "km");

        System.out.println("\n=== 문제2: 10km 갈 연료가 없어도 출발해서 연료가 마이너스가 된다 ===");
        탱크 t2 = new 탱크("탱크B", 3, 4);   // 3L -> 10km(2.5L)만 가능해야 함
        t2.move();

        System.out.println("\n=== 문제3: 기본 생성자로 이름·연비가 없는 차를 만들 수 있다 ===");
        버스 empty = new 버스();
        empty.show();
        System.out.println("연비 = " + empty.get연비() + " -> 10/연비 = " + (10 / empty.get연비()));
    }
}
```

```text
=== 문제1: 같은 차를 두 번 drive하면 총 주행거리가 중복 누적된다 ===
Moveable한 것을 운전합니다.
클래스 이름 :탱크
탱크탱크탱크탱크~~
             탱크A |         10               2.50              4.00
             탱크A |         20               0.00              4.00
연료가 부족합니다. 이 차는 더 이상 움직일 수 없습니다.
Moveable한 것을 운전합니다.
클래스 이름 :탱크
탱크탱크탱크탱크~~
연료가 부족합니다. 이 차는 더 이상 움직일 수 없습니다.
탱크A 실제 주행거리 = 20km, 드라이버 총 주행거리 = 40km

=== 문제2: 10km 갈 연료가 없어도 출발해서 연료가 마이너스가 된다 ===
탱크탱크탱크탱크~~
             탱크B |         10               0.50              4.00
             탱크B |         20              -2.00              4.00
연료가 부족합니다. 이 차는 더 이상 움직일 수 없습니다.

=== 문제3: 기본 생성자로 이름·연비가 없는 차를 만들 수 있다 ===
            null |          0               0.00              0.00
연비 = 0.0 -> 10/연비 = Infinity
```

생각했던 문제가 전부 그대로 나왔다.

문제1은 탱크A가 실제로는 20km만 갔는데 미미는 40km를 운전한 걸로 기록된 것이다. 두 번째 운전 때는 연료가 없어서 한 칸도 못 갔는데, move()가 누적 거리 20을 또 돌려줘서 그게 더해졌다.

문제2는 3L로는 10km만 가야 하는데 20km를 가고 연료가 -2.00L가 된 것이다. 퀴즈에 나온 값(100, 100, 50)은 연료가 딱 나누어떨어져서 이 문제가 안 보였던 거였다.

문제3은 이름이 `null`인 차가 그냥 만들어진다.

그리고 퀴즈 6번 답이었던 `((Driveable) d).drive(m)`을 원본 코드에서 쳐보니까 컴파일이 안 됐다.

```java
Driver d = new Driver("미미");
Moveable m = new 버스("타요버스", 100, 12);
((Driveable) d).drive(m);
```

```text
Exception in thread "main" java.lang.Error: Unresolved compilation problem:
	The method drive() in the type Driveable is not applicable for the arguments (Moveable)

	at BugDemo.main(BugDemo.java:23)
```

이클립스에서는 코드를 치자마자 `drive`에 빨간 줄이 생기고, 그래도 실행하면 위처럼 컴파일 문제가 해결되지 않았다는 에러가 난다.

이게 문제4다. Driveable 인터페이스에는 인자 없는 `drive()`만 있어서, Driveable로 바라보면 `drive(m)`이 안 보인다. 짱구 퀴즈에서 배운 "어떤 타입으로 보느냐에 따라 보이는 게 다르다"가 여기서 그대로 나와서 좀 신기했다.

![BugDemo 실행 결과]({{ '/assets/images/java-programming-2/vd-bugdemo-console.png' | relative_url }})

원본 클래스로 BugDemo를 돌린 결과다. 탱크A는 20km만 갔는데 드라이버 기록은 40km고, 탱크B는 연료가 -2.00까지 내려가고, 기본 생성자로 만든 차는 이름이 null에 연비가 0이라 Infinity가 나온다.

![퀴즈 6번 코드 컴파일 에러]({{ '/assets/images/java-programming-2/vd-bugdemo-error.png' | relative_url }})

BugDemo 맨 아래 주석을 풀어서 퀴즈 6번 코드를 넣어본 화면이다. `drive`에 빨간 줄이 생기고, 실행하면 Driveable의 `drive()`는 Moveable을 인자로 받을 수 없다는 에러가 난다.

### 어떻게 고쳤나

원본은 그대로 두고 `VehicleDriverRefactor`라는 프로젝트를 새로 만들어서 고쳤다.

#### move()는 이번에 달린 거리만 돌려주고, 10km 갈 연료가 있을 때만 출발하게

```java
@Override
public final int move() {
    소리내기();

    double 소요량 = MOVE_UNIT_KM / 연비;   // 10km 가는 데 필요한 연료
    int 이번이동거리 = 0;

    while (연료량 + EPSILON >= 소요량) {    // 갈 수 있을 때만 출발
        연료량 -= 소요량;
        if (연료량 < EPSILON) 연료량 = 0;
        주행거리 += MOVE_UNIT_KM;
        이번이동거리 += MOVE_UNIT_KM;
        show();
    }

    System.out.println("연료가 부족합니다. 이 차는 더 이상 움직일 수 없습니다.");
    return 이번이동거리;                     // 전체 거리가 아니라 이번에 간 거리
}
```

코드에 그냥 적혀 있던 10은 `MOVE_UNIT_KM`이라는 상수로 뺐다.

`EPSILON`(1e-9)은 double 계산 오차 때문에 넣었다. 버스는 10/12 = 0.8333...을 120번 빼는데, 실제로 돌려보니까 1200km를 다 가고 남은 연료가 딱 0이 아니라 `1.3e-13`이었다. 이런 자잘한 값을 0으로 정리하고, 오차가 반대로 생겨서 마지막 10km를 못 가는 경우도 막으려고 비교할 때 여유를 조금 뒀다.

#### 차마다 다른 부분만 하위 클래스에 맡기기

퀴즈 5번 답은 "하위 클래스의 move()에서 자기 동작을 하고 super.move()를 부른다"였다. 근데 이렇게 하면 하위 클래스에서 `super.move()`를 깜빡해도 에러가 안 난다.

그래서 순서는 부모가 정하고, 하위 클래스는 소리만 만들게 바꿨다.

```java
public abstract class Vehicle implements Moveable {
    protected abstract void 소리내기();   // 하위 클래스는 이것만 만든다
    public final int move() { 소리내기(); /* 공통 동작 */ }   // 순서는 부모가 정한다
}

public class 버스 extends Vehicle {
    @Override
    protected void 소리내기() {
        System.out.println("부릉부릉부르릉~~");
    }
}
```

`move()`에 final을 붙여서 하위 클래스가 공통 동작을 건드리지 못하게 했다. `소리내기()`는 abstract라서 새 차를 추가할 때 이걸 안 만들면 컴파일 에러가 난다. 운행하기, fly, println으로 제각각이던 이름도 자연스럽게 하나로 맞춰졌다.

`Vehicle`도 abstract로 바꿨다. 세상에 "탈것" 자체인 물건은 없으니까. 찾아보니까 이런 구조를 템플릿 메소드 패턴이라고 부른다.

#### 이상한 차는 아예 못 만들게

```java
protected Vehicle(String name, double 연료량, double 연비) {
    if (연비 <= 0) throw new IllegalArgumentException("연비는 0보다 커야 합니다: " + 연비);
    if (연료량 < 0) throw new IllegalArgumentException("연료량은 음수일 수 없습니다: " + 연료량);
    ...
}
```

빈 생성자(`Vehicle()`, `버스()` 등)는 지웠고, 연료량 파라미터는 필드랑 맞춰서 double로 바꿨다. 필드는 전부 private으로 바꾸고 `set연료`, `set연비`, `set주행거리`도 지웠다. 밖에서 연료나 주행거리를 마음대로 바꿀 일이 없어서다.

#### Driveable에 할 일을 적어두고, Driver는 Moveable로만 받기

```java
public interface Driveable {
    default void drive() {
        System.out.println("모든 Vehicle을 운전할 수 있습니다. 움직일 객체를 인자로 넣어주세요");
    }
    void drive(Moveable m);
    void drive(Moveable[] ms);
}
```

```java
public class Driver implements Driveable {
    private final String name;
    private int distance = 0;

    public Driver(String name) {
        this.name = name;
    }

    @Override
    public void drive(Moveable m) {
        System.out.println(name + "이(가) " + m.getClass().getSimpleName() + "을(를) 운전합니다.");
        distance += m.move();
        System.out.println("총 주행거리 : " + distance);
    }

    @Override
    public void drive(Moveable[] ms) {
        for (Moveable m : ms) {
            drive(m);
        }
    }
}
```

`drive(Vehicle)`, `drive(버스)`, `drive(탱크)`를 지웠다. 이제 오토바이 같은 새 차가 생겨도 Driver는 안 고쳐도 된다. 리모컨 예제에서 배운 OCP랑 같은 얘기다.

Scanner랑 `nextLine()`도 지워서 엔터 안 쳐도 끝까지 돌아간다. 안 쓰는 import도 지우고 필드는 private으로 바꿨다.

### 고친 후

원래 Main 시나리오를 그대로 돌려봤다. 표에 찍히는 숫자는 원본이랑 한 줄도 안 다르다. (두 출력 결과를 diff로 비교했다.)

```text
모든 Vehicle을 운전할 수 있습니다. 움직일 객체를 인자로 넣어주세요
미미이(가) 3대를 순서대로 운전합니다.
미미이(가) 버스을(를) 운전합니다.
부릉부릉부르릉~~
            타요버스 |         10              99.17             12.00
            ...
            타요버스 |       1200               0.00             12.00
연료가 부족합니다. 이 차는 더 이상 움직일 수 없습니다.
총 주행거리 : 1200

미미이(가) 비행기을(를) 운전합니다.
슈우우우웅~~
            쑝비행기 |         10               0.00              0.10
연료가 부족합니다. 이 차는 더 이상 움직일 수 없습니다.
총 주행거리 : 1210

미미이(가) 탱크을(를) 운전합니다.
탱크탱크탱크탱크~~
            탱크탱크 |         10              47.50              4.00
            ...
            탱크탱크 |        200               0.00              4.00
연료가 부족합니다. 이 차는 더 이상 움직일 수 없습니다.
총 주행거리 : 1410
```

문제들이 고쳐졌는지도 확인했다.

```text
=== 확인1: 같은 차를 두 번 drive해도 총 주행거리가 중복 누적되지 않는다 ===
...
탱크A 실제 주행거리 = 20km, 드라이버 총 주행거리 = 20km

=== 확인2: 10km 갈 연료가 없으면 출발하지 않는다 ===
탱크탱크탱크탱크~~
             탱크B |         10               0.50              4.00
연료가 부족합니다. 이 차는 더 이상 움직일 수 없습니다.
남은 연료 = 0.50L

=== 확인3: 연비가 0인 차는 만들 수 없다 ===
생성 실패: 연비는 0보다 커야 합니다: 0.0

=== 확인4: 퀴즈 6번 ((Driveable) d).drive(m) 이 컴파일되고 동작한다 ===
미미이(가) 비행기을(를) 운전합니다.
슈우우우웅~~
            쑝비행기 |         10               0.00              0.10
연료가 부족합니다. 이 차는 더 이상 움직일 수 없습니다.
총 주행거리 : 30
```

![리팩토링 후 Vehicle-Driver 실행 결과]({{ '/assets/images/java-programming-2/vd-refactor-console.png' | relative_url }})

리팩토링한 프로젝트에서 같은 Main을 돌린 결과다. 운전할 때 나오는 문구만 "미미이(가) 비행기을(를) 운전합니다."로 바뀌었고, 주행거리와 연료는 원본이랑 똑같다.

![리팩토링 후 Vehicle-Driver 클래스 다이어그램]({{ '/assets/images/java-programming-2/vd-refactor-uml.png' | relative_url }})

원본 다이어그램이랑 비교해보면 차이가 바로 보인다. Driveable에 `drive(m: Moveable)`, `drive(ms: Moveable[])`가 추가됐고, Driver는 Scanner 없이 drive 메소드가 2개로 줄었다. Vehicle은 빈 생성자와 setter가 사라지고 `MOVE_UNIT_KM`, `EPSILON` 상수가 생겼다. 버스, 비행기, 탱크는 생성자랑 `소리내기()`만 남았다.

![RefactorCheck 실행 결과]({{ '/assets/images/java-programming-2/vd-refactor-check-console.png' | relative_url }})

BugDemo에서 확인했던 문제들을 리팩토링한 코드로 다시 확인했다. 같은 탱크를 두 번 운전해도 20km로 기록되고, 3L 탱크는 10km만 가고 0.50L가 남고, 연비 0인 차는 만들어지지 않고, 퀴즈 6번 코드도 잘 돌아간다.

| 확인한 것 | 고치기 전 | 고친 후 |
| --- | --- | --- |
| 같은 차 두 번 운전 | 드라이버 기록 40km (실제는 20km) | 20km |
| 연료 3L, 연비 4 | 20km 가고 연료 -2.00 | 10km 가고 연료 0.50 |
| 연비 0인 차 | 만들어짐 (`null`, Infinity) | 못 만듦 |
| `((Driveable) d).drive(m)` | 컴파일 에러 | 잘 됨 |
| 여러 대 운전 | 한 대마다 엔터 | 끝까지 자동으로 |
| Driver의 drive 메소드 | 5개 | 2개 (+ default 1개) |

### 알게 된 점

결과가 맞게 나온다고 코드가 맞는 건 아니었다. 퀴즈 값이 딱 나누어떨어지는 숫자라서 연료가 마이너스가 되는 문제가 안 보였다. 연료 3L처럼 애매한 값을 넣어보거나 같은 걸 두 번 불러보는 식으로 경계를 건드려봐야 문제가 드러난다.

인터페이스에 메소드를 안 적어두면 인터페이스 타입으로는 그 메소드가 안 보인다. 짱구 퀴즈에서 배운 "보는 타입에 따라 보이는 게 다르다"가 인터페이스에서도 똑같았다.

오버로딩을 여러 개 만드는 것보다 부모 타입 하나로 받는 게 나중에 뭔가 추가될 때 훨씬 편하다.

"하위 클래스에서 super.move()를 꼭 불러라" 같은 건 약속일 뿐이라 안 지켜도 컴파일이 된다. final이랑 abstract를 쓰면 이런 약속을 아예 문법으로 강제할 수 있었다.

필드는 double인데 생성자는 int로 받으면, 지금은 괜찮아도 `new 버스("x", 12.5, 12)`처럼 소수로 넣는 걸 막아버린다.

### 궁금한 점

- 퀴즈 5번 답은 "하위 move() + super.move()"였는데, 이번에 한 것처럼 부모가 순서를 정하는 방식으로 바꾸는 것도 좋은 설계라고 볼 수 있을까? 대신 하위 클래스가 move() 전체를 바꿀 수는 없게 된다.
- 차 종류를 출력할 때 `m.getClass().getSimpleName()`으로 클래스 이름을 가져왔는데, 클래스 이름에 기대는 것도 결합이라고 봐야 할까? 차마다 종류를 돌려주는 메소드를 두는 게 나을까?
- 이상한 값이 들어왔을 때 지금처럼 예외를 던지는 게 나을지, 기본값으로 바꾸고 경고만 띄우는 게 나을지 궁금하다.

## 2. 짱구네 집 퀴즈를 코드로 확인

`JAVA1review/src/test`에 할머니, 엄마, 짱구, 핸드폰 클래스를 만들었다. 수업 때 만든 `테스트.java`에 더해서, 퀴즈를 확인하려고 `다형성테스트.java`를 추가했다.

```java
public class 짱구 extends 엄마 {
	private 핸드폰 phone = new 핸드폰();
	public 핸드폰 getPhone() { return phone; }
	public void setPhone(핸드폰 phone) { this.phone = phone; }

	public void 요리하기() { System.out.println("컵라면 먹기"); }
	public void 청소하기() { System.out.println("더럽히기"); }
	public void 공부하기() { System.out.println("열심히 공부중..."); }

	public void 엄마처럼요리하기() { super.요리하기(); }
	public void 할머니처럼요리하기() { super.할머니처럼요리하기(); }

	public void 딴짓하기() {
		if (phone == null) 공부하기();
		else {
			for (int i = 0; i < 5; i++) System.out.println("공부하는 척 ...");
		}
	}

	public void 핸드폰압수(엄마 엄) {
		System.out.println("헤헤 몰래 다시 되찾기~");
		this.setPhone(엄.핸드폰압수bag);
		엄.핸드폰압수bag = null;
	}
}
```

```java
public class 엄마 extends 할머니 {
	private String 일기장;
	public 핸드폰 핸드폰압수bag;

	public void 요리하기() { System.out.println("스파게티 요리하기"); }
	public void 청소하기() { System.out.println("깨끗하게 청소하기"); }
	public void 할머니처럼요리하기() { super.요리하기(); }

	public void 핸드폰압수(짱구 짱) {
		System.out.println("너!!! 압수!!!");
		this.핸드폰압수bag = 짱.getPhone();
		짱.setPhone(null);
	}
}
```

### 수업 때 만든 테스트.java

![테스트.java 실행 결과]({{ '/assets/images/java-programming-2/jjanggu-test.png' | relative_url }})

수업 시간에 만든 테스트 코드다. 할머니, 엄마, 짱구를 각자 자기 타입으로 만들어서 요리하기, 청소하기를 불러봤다. 짱구가 할머니처럼 요리하면 된장과 고추장이 나오고, 엄마가 핸드폰을 압수한 다음 짱구가 바로 되찾아서 마지막에도 공부하는 척만 한다.

다만 이 코드는 전부 자기 타입으로 만들었기 때문에 퀴즈의 핵심인 업캐스팅(`엄마 mom = new 짱구()`)은 확인이 안 된다. 그리고 압수하자마자 되찾아서 압수당한 동안 열심히 공부하는 모습도 안 나온다. 그래서 퀴즈 확인용으로 다형성테스트.java를 따로 만들었다.

### 다형성테스트.java 실행 결과

```text
===== Quiz #1 : 엄마 mom = new 짱구(); mom.요리하기() =====
컵라면 먹기

===== Quiz #2 : a, b, c 모두 짱구 객체 =====
a.요리하기() -> 컵라면 먹기
b.요리하기() -> 컵라면 먹기
c.요리하기() -> 컵라면 먹기
b.청소하기() -> 더럽히기
c.청소하기() -> 더럽히기

===== 짱구가 다른 요리를 하려면? =====
c.엄마처럼요리하기() -> 스파게티 요리하기
c.할머니처럼요리하기() -> 된장과 고추장을 만들기

===== Quiz #4 : 핸드폰 압수와 되찾기 =====
[1] 핸드폰이 있을 때
공부하는 척 ...
(5번 반복)
[2] 엄마가 압수한 뒤
너!!! 압수!!!
짱구 폰: null / 엄마 bag: test.핸드폰@7e6cbb7a
열심히 공부중...
[3] 짱구가 몰래 되찾은 뒤
헤헤 몰래 다시 되찾기~
짱구 폰: test.핸드폰@7e6cbb7a / 엄마 bag: null
공부하는 척 ...
(5번 반복)
```

![다형성테스트.java 실행 결과]({{ '/assets/images/java-programming-2/jjanggu-polymorphism-console.png' | relative_url }})

`mom`은 엄마 타입인데도 컵라면이 나왔다. 부르는 건 엄마 타입으로 불렀지만 실행된 건 짱구 메소드다.

`할머니 a`로 `a.청소하기()`를 쓰면 컴파일 에러가 난다. 할머니 설계도에는 청소하기가 없으니까.

핸드폰을 되찾은 뒤에 짱구 폰이랑 아까 엄마 bag에 있던 폰이 같은 해시코드(@7e6cbb7a)라서 같은 핸드폰이라는 걸 확인했다. 엄마 bag은 null이 됐다. 옮긴 다음에 원래 자리를 비워야 핸드폰이 두 개가 되는 일이 없다.

`엄마처럼요리하기()`는 `super.요리하기()`로 한 단계 위인 엄마 요리(스파게티)가 나오고, `할머니처럼요리하기()`는 엄마한테 있는 `할머니처럼요리하기()`를 거쳐서 된장, 고추장까지 간다. `super.super`는 안 되니까 이렇게 한 단계씩 올라가야 한다.

하나 아쉬운 건, 짱구가 핸드폰을 되찾는 메소드 이름도 `핸드폰압수(엄마 엄)`로 지었다는 거다. 엄마의 `핸드폰압수(짱구 짱)`랑 이름은 같고 파라미터 타입만 달라서 오버로딩이 되긴 하는데, 읽을 때는 `핸드폰되찾기`가 더 나았을 것 같다.

## 3. 하늘 싱글톤 다시 만들기

수업 슬라이드처럼 `하늘`이라는 이름으로 싱글톤을 만들고 싶었다. 그런데 이클립스에서 클래스를 만들 때 이름을 `Sky`로 해놓고, 안에 코드는 슬라이드 보면서 `하늘`로 쳐버려서 컴파일이 안 됐다.

```java
// 파일: Sky.java
public class Sky {                  // 클래스 이름은 Sky
	static private 하늘 instance = null;   // (1) 하늘이라는 타입이 없다
	private 하늘() { ... }                 // (2) error: invalid method declaration; return type required
	public static 하늘 getInstance() { ... }
}
```

### 왜 안 됐을까

처음엔 한글로 클래스 이름을 써서 안 되는 줄 알았다. 근데 한글은 문제가 아니었고, 이름이 서로 달랐던 게 문제였다.

생성자는 이름이 클래스 이름이랑 같아야 한다. `Sky` 클래스 안에 있는 `하늘()`은 생성자가 아니라서, 컴파일러는 이걸 리턴 타입을 빼먹은 메소드로 보고 `return type required` 에러를 낸다. 그게 (2)번 에러다.

(1)번도 같은 이유다. `하늘`이라는 클래스가 없으니까 `하늘 instance`, `하늘 getInstance()`에 나오는 `하늘`이 뭔지 모른다.

그리고 `public class` 이름은 파일 이름이랑 같아야 한다. 그래서 `Sky.java` 파일 안에서 클래스 이름만 `public class 하늘`로 바꿔도 또 에러가 난다. 직접 해봤더니 `class 하늘 is public, should be declared in a file named 하늘.java`라고 나왔다.

결국 파일 이름, 클래스 이름, 생성자 이름, 타입 이름이 전부 하나로 맞아야 한다.

### 하늘로 돌아가게 고치기

파일 이름부터 `하늘.java`로 바꿨다. 이클립스에서 `Sky.java`를 우클릭하고 `Refactor → Rename`으로 `하늘`로 바꾸면 파일 이름, 클래스 이름, 생성자, 쓰는 곳까지 한 번에 바뀐다.

```java
// 파일: 하늘.java
package singleton;

// public class 이름 = 파일 이름(하늘.java) = 생성자 이름 = 타입 이름
public class 하늘 {

	// 하늘 객체가 만들어졌는지 저장하는 변수 (null이면 아직 없음)
	static private 하늘 instance = null;

	// private 생성자라서 밖에서 new 하늘() 못 함
	private 하늘() {
		System.out.println("하늘 객체를 만들어요");
	}

	// 없으면 만들고, 있으면 있는 거 돌려주기
	public static 하늘 getInstance() {
		if (instance == null) instance = new 하늘();
		return instance;
	}
}
```

```java
// 파일: Main.java
package singleton;

public class Main {
	public static void main(String[] args) {
		//하늘 sky1 = new 하늘(); - 생성자가 private이라서 불가능
		하늘 sky1 = 하늘.getInstance();   // 부를 때도 클래스 이름 하늘로
		하늘 sky2 = 하늘.getInstance();
		하늘 sky3 = 하늘.getInstance();

		System.out.println(sky1);
		System.out.println(sky2);
		System.out.println(sky3);
		System.out.println("sky1 == sky2 == sky3 ? " + (sky1 == sky2 && sky2 == sky3));
	}
}
```

```text
하늘 객체를 만들어요
singleton.하늘@5305068a
singleton.하늘@5305068a
singleton.하늘@5305068a
sky1 == sky2 == sky3 ? true
```

![하늘 싱글톤 실행 결과]({{ '/assets/images/java-programming-2/singleton-console.png' | relative_url }})

한글 클래스 이름으로도 잘 돌아간다. 프로젝트 인코딩이 UTF-8이라 한글 파일 이름이랑 클래스 이름을 문제없이 읽는다.

`getInstance()`를 세 번 불렀는데 "하늘 객체를 만들어요"는 한 번만 찍혔다. 해시코드도 셋 다 같고 `==` 비교도 true라서, 세 변수가 전부 같은 객체 하나를 가리키고 있다는 걸 확인했다.

## 마치며

리팩토링은 코드를 짧게 만드는 게 아니라, 결과는 그대로 두고 숨어 있는 문제랑 나중에 고치기 힘든 구조를 바꾸는 거라는 걸 이번에 느꼈다. 그래서 고치기 전이랑 후의 출력을 비교해보는 게 중요했다.

짱구 퀴즈에서 배운 "보는 타입에 따라 보이는 게 다르다"가 Vehicle-Driver의 Driveable 문제를 이해하는 데 그대로 도움이 됐다.

싱글톤은 `하늘`로 쓰고 싶었는데 클래스를 `Sky`로 만들어서 에러가 났던 덕분에, 생성자는 리턴 타입이 없고 클래스 이름이랑 같아야 한다는 것, public class 이름은 파일 이름이랑 같아야 한다는 걸 확실히 기억하게 됐다. 한글이 문제가 아니라 이름이 서로 달랐던 게 문제였다.
