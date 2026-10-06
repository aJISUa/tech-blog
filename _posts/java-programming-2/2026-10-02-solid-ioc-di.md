---
title: "[자프실2] SOLID 원칙과 IoC/DI: 본드 결합을 C타입으로"
date: 2026-10-02 20:00:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - SOLID
  - 결합도
  - 응집도
  - IoC
  - DI
excerpt: "결합도는 낮게, 응집도는 높게에서 출발해 SOLID 다섯 원칙을 before/after로 정리하고, 헷갈리는 DIP, IoC, DI의 관계와 주입 방식 세 가지까지 묶었다."
toc: true
toc_sticky: true
---

[클린코드와 리팩토링]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-clean-code-refactoring %})이 "어느 쪽으로 가야 하나"였다면, SOLID는 그 방향을 다섯 개의 원칙으로 정리한 것이다. 슬라이드에 "실제로 현업 개발자가 다시 공부하는 영역 중 하나"라고 적혀 있었다. 교수님께서 기업 강의를 나가보면 7년 차도 까먹는다고 하셨다.

이 원칙들을 직접 적용해본 커피메이커 실습은 [실습#4 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-06-coffee-maker-v1 %})에 있다.

## SOLID란

| | 원칙 | 한 줄 |
| --- | --- | --- |
| S | Single Responsibility(단일 책임) | 책임을 분리하라 |
| O | Open/Closed(개방-폐쇄) | 수정 없이 확장하라 |
| L | Liskov Substitution(리스코프 치환) | 일관되게 상속하라 |
| I | Interface Segregation(인터페이스 분리) | 관련 없는 인터페이스는 분리하라 |
| D | Dependency Inversion(의존성 역전) | 추상화에 의존하라 |

S가 앞에 있다고 제일 중요한 게 아니다. 다섯 원칙의 앞 글자로 말이 되는 단어를 만들다 보니 SOLID가 된 거다. 그런데 우연히도 S가 가장 쉽고 D가 조금 어렵다고 하셨다. 실제로 정리해보니 정말 그랬다.

SOLID는 새 코드를 잘 짜기 위한 규칙이라기보다, 문서도 없는 **레거시 코드를 변화에 강한 코드로 바꾸는 기술**이라고 하셨다. 전제는 "소프트웨어는 계속 바뀐다"는 것이다. 세상도 바뀌고 고객 마음도 바뀐다. 그래서 목표는 세 가지다.

- 유연성: 새 기능을 쉽게 추가할 수 있다
- 유지보수성: 버그를 고치거나 코드를 이해하기 쉽다
- 재사용성: 다른 곳에서 다시 쓰기 쉽다

여기에 테스트하기 쉬워지는 효과가 따라온다. 그리고 다섯 원칙은 서로 연결되어 있고, 모두 적용할 필요는 없다. SOLID끼리 충돌하기도 하니 상황에 맞는 것부터 하나씩 하면 된다.

## 출발점: 결합도는 낮게, 응집도는 높게

SOLID 전에 교수님께서 "출발점은 이거"라고 하신 게 결합도와 응집도다. 앞으로 계속 나온다고 하셨다.

- **응집도(Cohesion)**: 한 모듈 안의 구성 요소들이 얼마나 관련 있는가. 높을수록 좋다
- **결합도(Coupling)**: 모듈과 모듈이 서로 얼마나 의존하는가. 낮을수록 좋다

응집도가 낮은 예는 **잡동사니 박스**다. 관련 없는 게 우당탕탕 다 들어 있는 상자. 교수님께서 짱구 클래스를 예로 드셨다. 놀기, 유치원 가기, 공부하기, 책 읽기, 밥 먹기를 전부 짱구 클래스에 넣으면 응집도가 낮다. 분류할 수 있으면 나누면 된다.

이 부분에서 하신 말씀이 이번 주에 제일 크게 남았다.

> 객체를 데이터 컨테이너가 아니라 기능의 꾸러미로 봐라.

지금까지는 `짱구.setName()`, `짱구.getName()`처럼 객체를 데이터 담는 상자로 연습했다. 이제는 거기서 벗어나야 한다. 책임은 곧 기능이다.

결합도가 높은 상태는 **본드로 붙여놓은 것**에 비유하셨다. 뗄 수가 없다.

```java
// Before: Player가 무기를 직접 만들어서 들고 있다
class Player {
    Sword sword = new Sword();
    Gun gun = new Gun();

    void attack() {
        sword.slash();
        gun.shoot();
    }
}
```

오른손에 검, 왼손에 총을 본드로 붙인 모습이다. 총알이 떨어져도, 칼이 상해도 못 바꾼다. 폭탄이나 마법 도구를 추가하려면 Player를 뜯어고쳐야 한다. Player가 무기를 만들고 관리하는 일까지 하니 응집도도 낮다.

```java
// After: Weapon이라는 추상에만 의존하고, 무기는 밖에서 받는다
interface Weapon { void use(); }

class Sword implements Weapon {
    public void use() { System.out.println("칼로 베었다!"); }
}
class Gun implements Weapon {
    public void use() { System.out.println("총을 발사했다!"); }
}

class Player {
    private Weapon weapon;            // 무기만 알고 있음(추상)

    public Player(Weapon weapon) {    // 생성 시 주입받음
        this.weapon = weapon;
    }

    public void attack() {
        weapon.use();                 // 어떤 무기든 같은 방식으로 공격
    }
}
```

Player는 `attack()` 하나에 집중하니 응집도가 높아졌고, 무기가 바뀌어도 Player 코드는 그대로라 결합도가 낮아졌다. 2주차 [Battle 리팩토링]({{ site.baseurl }}{% post_url java-programming-2/2026-09-15-battle-refactoring %})에서 캐릭터가 안에서 무기를 따로 만들지 않고 생성자로 받은 전용무기를 쓰게 고쳤던 게 바로 이거였구나 싶었다. 그때는 "이렇게 하면 깔끔하다" 정도로 알았는데, 이름이 붙으니 왜 좋은지 설명할 수 있게 됐다.

## S: 단일 책임 원칙(SRP)

> 클래스는 단 하나의 기능(책임)에 집중하도록 한다.

책임이 여러 개면 한 책임이 바뀔 때 다른 책임까지 흔들린다.

```java
// Before: 보고서 생성과 이메일 전송이 한 클래스에
class ReportService {
    public void generateReport() { /* 보고서 생성 */ }
    public void sendEmail() { /* 이메일 전송 */ }
}

// After
class ReportGenerator {
    public void generateReport() { /* 보고서 생성 */ }
}
class EmailSender {
    public void sendEmail() { /* 이메일 전송 */ }
}
```

슬라이드에는 UserService(회원가입 + 이메일)와 ReportService(문서 작성 + 이메일)가 둘 다 이메일 기능을 들고 있는 예가 나온다. 이메일 전송 방법이 바뀌면 두 클래스를 다 고쳐야 했는데, EmailService로 빼면 거기만 고치면 된다.

교수님께서 "1인 1클래스는 가혹해 보이지만 응집도를 높이기 위해 나누자"고 하셨다. 요리하기도 재료 썰기, 볶기, 상차리기로 나눌 수 있는 것처럼.

## O: 개방-폐쇄 원칙(OCP)

> 확장에는 열려 있고, 수정에는 닫혀 있어야 한다.

교수님 비유가 정확했다. **수정**은 납땜을 풀고 본드를 떼서 내부를 바느질하고 꿰매는 것이다. **확장**은 파일 하나 붙이는 정도로, 새 코드를 추가하고 연결고리만 맺는 것이다.

```java
// Before: 결제 수단이 늘 때마다 이 if문을 열어서 고쳐야 한다
public class PaymentProcessor {
    public void process(String paymentType) {
        if ("creditCard".equals(paymentType)) {
            // 신용카드 결제
        } else if ("kakaoPay".equals(paymentType)) {
            // 카카오페이 결제
        }
        // 네이버페이, 토스페이가 생기면? 또 여기를 고친다
    }
}
```

```java
// After: PaymentMethod 인터페이스를 두고 각 결제 수단이 구현한다
public interface PaymentMethod {
    void pay();
}

public class CreditCard implements PaymentMethod { ... }
public class KakaoPay implements PaymentMethod { ... }

public class PaymentProcessor {
    public void process(PaymentMethod paymentMethod) {
        paymentMethod.pay();   // 어떤 결제 수단이 와도 코드는 같다
    }
}
```

네이버페이를 추가하려면 `NaverPay implements PaymentMethod` 클래스를 하나 만들면 끝이다. PaymentProcessor는 그대로다. 회원 등급별 할인율도 `DiscountPolicy` 인터페이스로 같은 방식을 쓸 수 있다.

핵심은 "인터페이스나 추상 클래스에 의존하라"다. 여기서 교수님께서 **의존 = 사용한다**라고 짚어주셨다. A가 B를 쓰면 A는 B에 의존한다. 그리고 OCP는 다음 주에 배울 전략 패턴에 딱 맞아떨어진다고 하셨다.

## L: 리스코프 치환 원칙(LSP)

> 하위 타입은 언제나 상위 타입으로 대체될 수 있어야 한다.

"치환에 집중하라"고 하셨다. 자식은 부모가 쓰이던 자리에 들어가서 문제를 일으키면 안 된다. 그래서 상속은 "OO는 OO이다(IS-A)"가 말이 될 때만 쓴다.

```java
// Before: 모든 새는 난다는 가정
class Bird {
    public void fly() { /* 날기 */ }
}
class Ostrich extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("타조는 못 날아요");
    }
}
```

`Bird`를 받아서 `fly()`를 부르는 코드에 타조가 들어가면 예외가 터진다. 부모 자리에 자식을 넣었더니 깨진 거다.

```java
// After: 나는 새와 걷는 새를 나눈다
interface Bird { void move(); }

class FlyingBird implements Bird {
    public void move() { System.out.println("날아요"); }
}
class WalkingBird implements Bird {
    public void move() { System.out.println("걸어요"); }
}
```

교수님께서 예전 다형성 수업의 "수술한 강아지는 못 짖어요" 예시를 다시 꺼내셨다. 다형성 배울 때는 "재밌죠?" 했지만 이제부터는 그런 걸 막는 거라고. 상위에서 기능을 다 주고 하위에서 막으면 또 다른 예외가 생긴다.

## I: 인터페이스 분리 원칙(ISP)

> 클라이언트는 자신이 사용하지 않는 메소드에 의존해서는 안 된다.

하나의 거대한 인터페이스보다 구체적인 인터페이스 여러 개가 낫다.

```java
// Before: Worker에 eat()까지 들어 있어서 Robot도 구현해야 한다
interface Worker {
    void work();
    void eat();
}
class Robot implements Worker {
    public void work() { /* 작업 */ }
    public void eat() { throw new UnsupportedOperationException(); }
}

// After: 역할별로 나눈다
interface Workable { void work(); }
interface Eatable { void eat(); }

class Robot implements Workable {
    public void work() { /* 작업 */ }
}
```

수업 때 "로봇도 전기를 먹는다", "로봇도 열받으면 냉각기가 필요하니 쉬어야 한다"는 반론이 나왔다고 하셨다. 교수님은 일단 순수하게 햄버거 먹는 행위로 보자고 하셨다.

노션에는 복합기 예제가 하나 더 있다. `MultiFunctionDevice`에 `print()`, `scan()`, `fax()`가 다 있으면, 프린트만 되는 `SimplePrinter`도 `scan()`과 `fax()`를 구현해서 예외를 던져야 한다. 이건 ISP 위반이면서 LSP 위반이기도 하다. 상위 타입으로 쓰다가 `scan()`을 부르면 터지니까. 하나의 문제가 여러 원칙에 같이 걸린다는 게 "원칙들이 서로 연결되어 있다"는 말의 의미 같다.

분리한 버전을 직접 쳐보다가 하나 발견했다.

```java
interface Printer { void print(String content); }
interface Scanner { void scan(); }

class SimplePrinter2 implements Printer, Scanner {
    public void print(String content) { System.out.println("프린트: " + content); }
    public void scan(String content) { System.out.println("스캔: " + content); }
}
```

```text
error: SimplePrinter2 is not abstract and does not override abstract method scan() in Scanner
```

인터페이스는 `scan()`인데 구현은 `scan(String content)`이라서, 오버라이딩이 아니라 오버로딩이 됐다. 그러니 `scan()`은 구현 안 한 셈이다. `@Override`를 붙였다면 그 줄에서 바로 잡혔을 거다. 교수님께서 커피메이커 노션 주석을 두고 AI 점검을 받은 것도 있고 안 받은 것도 있다고 하셨는데, 이 코드도 그대로 복붙했으면 몰랐을 거다. 하나 더, `Scanner`라는 이름은 `java.util.Scanner`와 겹쳐서 import하면 헷갈리겠다.

## D: 의존성 역전 원칙(DIP)

> 상위 모듈은 하위 모듈에 의존해서는 안 된다. 둘 다 추상화에 의존해야 한다.

변하기 쉬운 구체 클래스 말고, 잘 안 변하는 인터페이스나 추상 클래스에 의존하라는 뜻이다. 스프링의 DI(의존성 주입)와 이름이 비슷해서 헷갈리지 말라고 하셨다. SOLID의 D는 DIP다.

```java
// Before
class SmartHomeSwitch {
    private Light light;

    public SmartHomeSwitch() {
        this.light = new Light();   // 구체 클래스에 직접 의존
    }

    public void operate(String command) {
        if (command.equals("on")) light.turnOn();
        else light.turnOff();
    }
}
```

이 스위치는 전등만 켤 수 있다. TV, 에어컨, 보일러를 붙이려면 스위치를 고쳐야 한다. 교수님께서 "불 끄는 스위치, 보일러 켜는 스위치, 냉장고 켜는 스위치를 따로 사야 한다는 것, 말이 안 된다"고 하셨다.

```java
// After: Device라는 추상화 계층을 사이에 둔다
interface Device {
    void turnOn();
    void turnOff();
}
class Light implements Device { ... }
class AirConditioner implements Device { ... }
class CoffeeMachine implements Device { ... }

public class SmartHomeSwitch {
    private final List<Device> devices;

    public SmartHomeSwitch(List<Device> devices) {   // 밖에서 주입
        this.devices = devices;
    }

    public void operateAll(String command) {
        for (Device device : devices) {
            if ("on".equals(command)) device.turnOn();
            else device.turnOff();
        }
    }
}

// main
SmartHomeSwitch switchPanel = new SmartHomeSwitch(Arrays.asList(light, ac, cm));
switchPanel.operateAll("on");
```

아파트 벽에 붙은 통합 패널을 생각하면 된다고 하셨다. 새 기기가 생기면 스위치는 안 고치고 플러그 꽂듯이 연결만 하면 된다. 테스트할 때 가짜 기기(MockDevice)를 넣기도 쉽다.

"역전"이 뭘 뒤집는 건지 처음엔 헷갈렸다. Before에서는 화살표가 `Switch → Light`로 상위가 하위를 향한다. After에서는 `Switch → Device ← Light`가 되어서, 하위 모듈(Light)의 화살표가 위쪽의 추상화를 향한다. 하위가 상위가 정한 규격에 맞추도록 방향이 뒤집힌 거다. 그리고 상위, 하위는 그림의 위치가 아니라 의미로 정해진다. 스위치가 전등을 사용하니까 스위치가 상위다.

### 구형 폰 배터리와 C타입

DIP 비유는 듣자마자 이해됐다. 옛날 핸드폰은 기종마다 전용 배터리와 전용 충전선이 있었다. 친구 집에 가면 충전을 못 했고, 집마다 가족 수보다 배터리와 선이 더 많았다. 폰을 바꾸면 멀쩡한 걸 다 버려야 했다. 가방 하나가 배터리로 찼다고 하셨다. 이게 DIP가 없던 시절이다.

지금은 C타입 하나로 드라이어든 선풍기든 다 연결된다. 폰(상위)도 충전기(하위)도 "C타입"이라는 규격(추상화)에 맞춘다. DIP의 철학은 **상위 모듈이 하위 모듈에 끌려다니지 말라**는 것이다. 삼성이 갤럭시를 만들 때 충전기 회사 사정에 끌려다니면 안 되는 것처럼.

운영체제 수업에서 배운 장치 드라이버가 떠올랐다. OS는 프린터 회사마다 다른 코드를 직접 알지 않고, 정해진 드라이버 인터페이스만 부른다. 제조사가 그 규격에 맞춰 드라이버를 만든다. OS(상위)가 장치(하위)에 끌려다니지 않으려고 둘 다 규격에 의존하게 만든 것이니, 같은 구조다.

교수님께서 "SOLID는 D 하나 빼고 그렇게 힘든 개념이 아니다. 언제, 어디에 적용할지가 문제"라고 하셨다.

### 다섯 원칙 정리

| 원칙 | 목표 | 해결한 문제 | 결과 |
| --- | --- | --- | --- |
| SRP | 책임 분리 | 한 클래스가 여러 역할을 맡음 | 응집도↑, 이해도↑ |
| OCP | 수정 없이 확장 | 새 정책마다 메소드를 일일이 수정 | 유연성↑, 유지보수 비용↓ |
| LSP | 일관된 상속 | 잘못된 상속으로 자식이 부모 자리에서 깨짐 | 신뢰성↑, 안정성↑ |
| ISP | 인터페이스 분리 | 거대 인터페이스가 안 쓰는 메소드를 강요 | 결합도↓, 불필요한 의존 제거 |
| DIP | 추상화에 의존 | 상위 모듈이 하위 구현에 묶임 | 유연성↑, 재사용성↑, 테스트 용이성↑ |

이론이 아니라 실제 개발 현장의 문제를 푸는 실용적인 도구라고 하셨다.

## IoC와 DI

수업에서는 IoC/DI를 "고학년 참고"로 짧게 다루고 SOLID에 집중했다. 그런데 DIP를 정리하다 보니 "그래서 구현체는 누가 넣어주지?"가 계속 걸렸고, 스프링을 공부하려면 어차피 넘어야 할 부분이라 노션 문서를 여러 번 읽으면서 따로 정리했다. 먼저 헷갈리는 용어부터 구분했다.

### DIP, IoC, DI는 다른 것이다

슬라이드 22쪽도 같은 질문으로 시작한다. **"그래서 구체적인 구현체는 누가, 어떻게 넣어주는가?"** 여기서 IoC와 DI가 나온다.

| 용어 | 성격 | 한 줄 설명 | 비유 |
| --- | --- | --- | --- |
| 의존(Dependency) | 관계 | A가 동작하려면 B가 필요하다 | 폰을 쓰려면 배터리가 필요한 상태 |
| DIP | 설계 원칙 | 특정 부품 말고 표준 규격(인터페이스)에 맞춰라 | 특정 제조사 말고 C타입 규격에 맞추라는 규칙 |
| IoC | 구조적 패러다임 | 객체 생성과 실행 흐름의 주도권을 프레임워크에 넘겨라 | 직접 운전하지 않고 자율주행에 맡기기 |
| DI | 구현 패턴 | 필요한 부품을 안에서 만들지 않고 밖에서 꽂아준다 | 배터리를 직접 만들지 않고 끼워주는 행위 |

노션에 정리된 흐름이 이해하기 좋았다. 부품이 필요한 **문제 상황**(Dependency) → 부품 대신 규격을 보라는 **설계 가이드**(DIP) → 개발자 대신 프레임워크가 부품을 관리하겠다는 **운영 체제**(IoC) → 프레임워크가 부품을 실제로 꽂아주는 **실행 도구**(DI).

```java
// DI 미적용: Car가 엔진을 직접 만든다. 강한 결합
public class Car {
    private Engine engine;
    public Car() {
        this.engine = new GasolineEngine();   // 전기 엔진으로 바꾸려면 Car를 고쳐야 한다
    }
}

// DI 적용: Car는 Engine 인터페이스만 보고, 구현체는 밖에서 받는다
public class Car {
    private final Engine engine;
    public Car(Engine engine) {
        this.engine = engine;
    }
    public void drive() {
        engine.start();   // 어떤 엔진이 들어와도 Car는 수정 없음
    }
}

// 조립은 main(나중에는 스프링 컨테이너)이 한다
Car car1 = new Car(new GasolineEngine());
Car car2 = new Car(new ElectricEngine());
```

### IoC: 내가 전문가를 찾아가던 걸, 전문가가 나를 찾아온다

전통적인 방식에서는 개발자가 필요한 객체를 직접 `new`로 만들고, 필드에 연결하고, 실행 흐름까지 다 관리했다. 슬라이드 표현으로는 "내가 전문가를 찾아다니는" 방식이다.

IoC는 이 제어의 흐름을 뒤집는다. 객체의 생성부터 생명주기 관리까지 개발자가 아니라 프레임워크(전문가)가 한다. "이거 어떻게 써요?" 하면 "이리 와, 내가 해줄게!" 하는 느낌이다. 스위치 입장에서는 "나는 그저 스위치일 뿐! Device는 외부에서 연결해줘!"가 된다.

스프링에서는 `@Component`로 클래스를 등록하면 스프링 컨테이너가 객체를 만들어서 관리하고, `@Autowired`(생성자가 하나면 생략 가능)를 붙인 곳에 알아서 넣어준다. 슬라이드 21쪽에도 "Device를 @Component로 등록하고 @Autowired로 주입"이라고 되어 있다. 우리가 main에서 손으로 하던 생성과 연결을 어노테이션 한 줄로 프레임워크가 대신하는 거다.

DI는 IoC를 구현하는 대표적인 방법이지만 유일한 방법은 아니다. 이벤트 처리 방식이나 팩토리 패턴으로도 IoC를 구현할 수 있다고 슬라이드에 나와 있다. GUI에서 버튼을 누르면 내가 등록한 리스너를 시스템이 불러주는 것도 제어가 뒤집힌 구조다. 운영체제의 인터럽트 핸들러도 비슷하다고 느꼈다. 내가 부르는 게 아니라, 일이 생기면 OS가 등록된 핸들러를 부른다.

### 주입 방법 세 가지

| 방식 | 특징 |
| --- | --- |
| 생성자 주입 | 스프링이 가장 권장. `final`을 쓸 수 있어 한 번 받은 의존성이 바뀌지 않는다. 생성할 때 다 받아야 하니 누락이 없다. 프레임워크 없이도 가짜 객체를 넣어 테스트하기 쉽다 |
| Setter 주입 | 의존성이 선택 사항일 때 유용하다. 대신 setter를 부르기 전까지는 null이다 |
| 필드 주입 | 코드는 제일 짧지만 의존성이 숨겨지고 테스트가 어렵다. 권장하지 않는다 |

Setter 주입의 "부르기 전까지는 null"이 커피메이커에서 실제로 문제가 됐다. [실습#4 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-06-coffee-maker-v1 %})에서 직접 확인했다.

수요일에 교수님께서 MyView2처럼 JFrame을 상속하지 않고 필드로 갖는 상속 회피를 보여주셨고, 이어서 버튼 예제를 MVC로 나눈 main에서 View와 Model을 만들어 Controller에게 넘기는 것을 "주입"이라고 하셨다. 웨이터가 메뉴판을 직접 만드는 게 아니라 만들어진 걸 받아서 쓰는 형태라고. 그때는 GUI 이야기로만 들었는데, 금요일 수업을 듣고 나니 같은 이야기였다.

## 정리

- 결합도는 낮게, 응집도는 높게가 SOLID의 출발점이다
- 객체는 데이터 상자가 아니라 기능의 꾸러미다. 책임은 곧 기능이다
- SRP는 나누고, OCP는 붙이고, LSP는 치환해도 안 깨지게, ISP는 안 쓰는 걸 강요하지 않게, DIP는 규격에 의존하게 한다
- DIP는 설계 원칙, IoC는 제어 흐름을 넘기는 패러다임, DI는 그걸 구현하는 방법이다
- 필수 의존성은 생성자로, 선택 의존성은 setter로 받는다

SOLID 자체는 생각보다 어렵지 않았다. 교수님 말씀대로 어려운 건 "언제, 어디에" 적용하느냐다. 그 감을 잡으려고 커피메이커를 직접 만들어봤다.
