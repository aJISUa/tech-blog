---
title: "[자프실2] 디자인 패턴 기초: 원칙이 요리법이라면 패턴은 레시피"
date: 2026-10-07 20:00:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - 디자인패턴
  - SOLID
  - 데코레이터
  - 전략패턴
excerpt: "생성, 구조, 행위로 나뉘는 디자인 패턴 분류와 전략, 팩토리 메소드, 데코레이터, 싱글톤을 정리하고, 데코레이터는 자바 java.io에서 실제로 어떻게 쓰이는지 따로 파봤다."
toc: true
toc_sticky: true
---

6주차는 SOLID 다음 단계인 디자인 패턴이다. [SOLID와 IoC/DI]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-solid-ioc-di %})가 좋은 설계를 위한 원칙이었다면, 디자인 패턴은 그 원칙을 코드로 녹여낸 검증된 방법이다.

교수님께서 우리 학교에서 SOLID와 패턴을 같이 다루는 수업은 이 수업밖에 없다고 하셨다. 대신 맛보기 수준이라 나중에 꼭 보강하라고. 이 글은 수업에서 다룬 패턴 정리와, 그중 제일 궁금했던 데코레이터를 따로 더 파본 내용이다. 패턴을 커피메이커에 직접 적용한 건 [실습#5 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-13-coffee-maker-v2 %})에 있다.

## 디자인 패턴이란

> 바퀴를 재발명하지 마라.

많은 개발자들이 비슷한 문제를 겪으면서 "이런 문제는 이렇게 풀면 된다"고 정리해둔 해결책이 디자인 패턴이다. 특정 라이브러리나 완성된 코드가 아니라 설계의 청사진이다. 교수님께서 수학으로 치면 공식이고, 건축으로 치면 기법이라고 하셨다.

슬라이드의 비유가 SOLID와 패턴의 관계를 잘 정리해준다.

- SOLID 원칙은 맛있는 요리를 위한 5대 원칙
- 디자인 패턴은 수많은 요리사가 검증한 필승 레시피

패턴을 쓰면 좋은 점이 세 가지 있다.

| 장점 | 설명 |
| --- | --- |
| 재사용성과 유지보수성 | 검증된 구조라 안정적이고 유연하다 |
| 의사소통 | "여기는 전략 패턴 썼어요" 한마디로 설계 의도가 전달된다 |
| 구조화 | 예측 가능한 구조라 새로 온 사람도 빨리 적응한다 |

의사소통 비유가 재밌었다. 머리가 아파서 두통약을 먹었다고 하면 굳이 설명 안 해도 머리가 아팠구나 하고 안다. "전략 패턴 썼어"도 마찬가지로 장황한 설명 없이 통한다. 파일은 많아지지만 구조가 정리되니 오히려 이해하기 쉬워진다고 하셨다.

## 분류: 생성, 구조, 행위

| 분류 | 핵심 질문 | 수업에서 본 패턴 |
| --- | --- | --- |
| 생성(Creational) | 누가, 어떻게 객체를 만들 것인가 | 싱글톤, 팩토리 메소드 |
| 구조(Structural) | 클래스와 객체를 어떻게 조합해 더 큰 구조를 만들 것인가 | 데코레이터 |
| 행위(Behavioral) | 객체들이 어떻게 협력하고 책임을 나눌 것인가 | 전략, 템플릿 메소드, 옵저버 |

교수님께서 이 중에서도 2학년이 충분히 쓰고 이해할 만한 것들로 골랐다고 하셨다. 나머지는 관심 가는 것부터 찾아보라고.

생성 패턴의 비유는 자동차 공장이었다. 자동차가 필요할 때 직접 만들지 않고 공장에 요청해서 산다. 행위 패턴과 구조 패턴의 차이는 관점이다. 행위는 "알고리즘이 어떻게 수행되는가", 구조는 "클래스와 객체가 정적으로 어떻게 연결되어 있는가"에 집중한다.

## 전략 패턴: 알고리즘을 통째로 갈아 끼우기

> 알고리즘(전략)을 통째로 갈아 끼울 수 있게 만드는 패턴

교수님께서 "OCP의 정석"이라고 하셨다. if-else나 switch로 수많은 로직을 분기하고 있을 때 쓴다.

```java
// Before: 결제 수단이 늘 때마다 if문이 늘어난다
public class PaymentService {
    public void processPayment(String paymentMethod, int amount) {
        if ("creditCard".equals(paymentMethod)) {
            // 신용카드 결제 로직
        } else if ("kakaoPay".equals(paymentMethod)) {
            // 카카오페이 결제 로직
        }
        // 새로운 결제 수단이 추가될 때마다 이 클래스를 수정해야 함
    }
}

// After: 전략 인터페이스에만 의존한다
public interface PaymentStrategy {
    void pay(int amount);
}

public class PaymentService {
    public void processPayment(PaymentStrategy strategy, int amount) {
        strategy.pay(amount);
    }
}
```

구조는 Context(PaymentService)가 Strategy 인터페이스를 갖고 있고, 그 아래에 구체 전략들이 매달린 모양이다. 네이버페이가 추가되어도 `NaverPayStrategy` 클래스만 새로 만들면 된다. PaymentService가 구체 클래스가 아니라 인터페이스에 의존하니 DIP도 만족한다.

[SOLID 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-solid-ioc-di %})의 OCP 예제와 코드가 거의 같다. OCP를 설명할 때 이미 전략 패턴을 쓰고 있었던 거다. 교수님께서 OCP 설명 때 "전략 패턴에 딱 들어맞는다"고 하신 이유를 여기서 알았다.

### 템플릿 메소드와 비교하면

같은 행위 패턴인 템플릿 메소드는 **상속**을 쓴다. 부모가 전체 과정의 순서를 고정해두고, 자식은 일부 단계만 오버라이딩한다. 라면 끓이기로 치면 "물 끓이기, 면과 스프 넣기, 토핑, 완성" 순서는 부모가 정하고, 토핑만 신라면은 계란과 파, 불닭은 치즈로 자식이 정한다.

교수님께서 우리가 지금까지 상속으로 짠 코드는 대부분 템플릿 메소드에 가까웠을 거라고 하셨다. 꼬집어서 배운 적은 없지만 늘 써온 패턴이라고. 대신 부모에 의존하고, 자식이 부모의 약속을 깨면 LSP 위반(못 나는 타조)이 생길 수 있다. 그래서 요즘은 인터페이스와 합성을 쓰는 전략 패턴을 훨씬 많이 쓴다고 하셨다.

| | 전략 | 템플릿 메소드 |
| --- | --- | --- |
| 방식 | 인터페이스 + 합성(위임) | 추상 클래스 + 상속 |
| 바꾸는 단위 | 알고리즘 전체를 교체 | 정해진 과정 중 일부 단계만 |
| 교체 시점 | 실행 중에도 바꿔 끼울 수 있다 | 클래스를 정할 때 고정된다 |

## 팩토리 메소드: 공장에 주문하기

> "노트북 만들어줘", "TV 만들어줘", "라떼 만들어줘"

객체를 직접 `new`로 만들지 않고 생성을 맡기는 패턴이다. 생성 로직을 캡슐화하고 싶을 때 쓴다. 슬라이드 예시가 바로 커피메이커였다.

```java
class CoffeeFactory {
    private final CoffeeMachine espressoMachine;
    private final MilkFrother frother;
    ...
    public Coffee createCoffee(String name) {
        switch (name) {
            case "Latte":
                return new Latte(new Espresso(espressoMachine), frother);
            case "Americano":
                return new Americano(new Espresso(espressoMachine));
            default:
                return new Espresso(espressoMachine);
        }
    }
}

// 사용하는 쪽은 이름만 넘긴다
Coffee latte = coffeeFactory.createCoffee("Latte");
```

V1에서는 main이 `new Latte(new Espresso(machine), frother)`처럼 재료와 순서를 다 알아야 했다. 팩토리를 두면 사용하는 쪽은 "Latte 주세요"만 하면 된다. 커피 종류가 늘어도 Factory만 고치면 된다.

엄밀히 말하면 GoF의 팩토리 메소드는 Creator 인터페이스를 두고 생성 방법을 하위 클래스에 맡기는 구조다. 슬라이드의 CoffeeFactory처럼 클래스 하나가 이름에 따라 만들어주는 건 "단순 팩토리"라고 따로 부르기도 한다고 한다. 수업에서는 "생성을 맡긴다"는 핵심에 집중했다.

그런데 이 코드를 보다가 하나 걸렸다. 메뉴가 늘 때마다 switch에 case를 추가해야 하니, Factory 입장에서는 OCP가 지켜지지 않는다. 이건 실습#5에서 직접 고쳐봤다.

## 데코레이터: 겹겹이 꾸미기

> 기존 객체의 기능을 직접 수정하지 않고 겹겹이 꾸미듯 확장하는 패턴

교수님 비유는 케이크였다. 시트를 2단, 3단으로 쌓고, 크림을 바르고, 초콜릿과 과일을 올린다. 시트를 뜯어고치는 게 아니라 위에 덧붙인다.

커피에 대입하면 이렇다. 라떼는 에스프레소에 우유 거품을 꾸민 것이고, 카페모카는 에스프레소 위에 초콜릿 시럽, 그 위에 스팀 밀크를 올린 것이다. 에스프레소 자체는 바뀌지 않는다. 요즘 카페에서 커스텀 메뉴를 "말아서" 먹는 것처럼 위에 올리고 또 올리는 구조에 데코레이터가 딱 어울린다고 하셨다.

```java
Coffee base = coffeeFactory.createCoffee("Latte");
Coffee myDrink1 = new IceDecorator(base);
Coffee myDrink2 = new VanillaDecorator(myDrink1);
Coffee myDrink3 = new ChocolateDecorator(myDrink2);
```

`myDrink3`은 초콜릿이 올라간 바닐라가 올라간 얼음이 들어간 라떼다. 그리고 몇 겹을 감싸도 여전히 `Coffee`다. 그래서 커피를 받는 곳 어디에나 넣을 수 있다.

## 싱글톤: 유일무이한 객체

> 클래스의 인스턴스가 오직 하나만 존재하도록 보장하는 패턴

생성자를 `private`으로 막고, 미리 만들어둔 하나를 `getInstance()`로만 꺼내 쓰게 한다. 2주차에 [하늘 객체 예제]({{ site.baseurl }}{% post_url java-programming-2/2026-09-09-static-singleton-interface-remote %})로 이미 해봤다. 스프링이 관리하는 객체(Bean)도 기본적으로 싱글톤으로 동작한다.

교수님께서 지난 학기 일화를 들려주셨다. 학생들이 소켓으로 게임 채팅을 만들면서 싱글톤 객체가 네트워크 전체에 하나라고 오해했다는 것이다. 싱글톤은 **그 프로그램의 메모리 안에서** 하나다. 컴퓨터 세 대에서 실행하면 세 개가 생긴다. 운영체제 수업에서 배운 대로 프로세스는 각자 메모리 공간을 따로 가지니까 당연한 결과다. 여러 컴퓨터가 정말 하나를 공유하려면 서버가 그 역할을 맡아야 한다.

## MVC

UI, 데이터, 처리 로직을 Model, View, Controller로 나누는 아키텍처 패턴이다. 교수님 비유는 옷과 몸이었다. 옷이 몸에 꿰매져 있지 않듯 화면과 데이터도 붙어 있으면 안 된다. 접시 위의 스테이크(View)가 계산도 하고 데이터도 알고 있으면, 고기가 질길 때 누구 탓을 해야 할지 모른다고 하셨다. Spring MVC는 MVC의 완성형이고, MVC는 다음 주 GUI를 만들면서 직접 경험한다.

## 패턴과 SOLID 연결

교수님께서 "이 패턴을 쓰면 어떤 원칙을 만족할까"와 "이 원칙을 지키려면 어떤 패턴을 쓸까" 두 방향으로 정리해주셨다. 수업에서 본 것 위주로 추리면 이렇다.

| 패턴 | 하는 일 | 주로 만족하는 원칙 |
| --- | --- | --- |
| Strategy | 알고리즘을 캡슐화해서 바꿔 끼움 | OCP, DIP |
| Decorator | 기존 코드를 안 고치고 기능을 덧붙임 | OCP, SRP |
| Template Method | 공통 과정은 부모, 세부는 자식 | LSP, SRP |
| Factory Method | 생성 로직을 맡김 | LSP, DIP |
| Observer | 상태 변화를 여러 객체에 알림 | OCP, DIP |
| Dependency Injection | 의존 객체를 밖에서 넣어줌 | DIP, ISP |

완벽한 1:1 대응은 아니다. 한 패턴이 여러 원칙을 만족하기도 하고, 상황에 따라 여러 패턴을 조합하기도 한다. 교수님께서도 본인 생각으로 정리한 표라 다른 관점이 있는지 찾아보라고 하셨다. 수많은 패턴을 다 외우기보다는, 공식처럼 쓸 일이 생겼을 때 쓰다 보면 익혀진다고.

## 더 파보기: 데코레이터

네 패턴 중에서 데코레이터가 제일 궁금했다. 전략은 OCP 예제로 이미 익숙했고, 팩토리와 싱글톤은 "생성을 막는다"는 점에서 직관적이었다. 그런데 데코레이터는 "감싸도 같은 타입"이라는 게 처음엔 잘 안 그려졌다. 그리고 커피메이커 V2에 바로 쓸 패턴이라 제대로 알고 싶었다.

### 구조

```text
        Component(Coffee)  <──────────┐
         ▲            ▲               │ 감싸고 있음(has-a)
         │            │               │
 ConcreteComponent   Decorator ───────┘
   (Espresso)          ▲
                       │
          ConcreteDecorator(Ice, Syrup, ...)
```

핵심은 Decorator가 Component를 **구현(is-a)하면서 동시에 가지고 있다(has-a)**는 점이다. 그래서 바깥에서 보면 그냥 Component고, 안에서는 감싼 객체에게 일을 넘기고 자기 일을 덧붙인다. 그래서 몇 겹이든 쌓을 수 있다.

### 활용 사례: 자바 java.io

찾아보니 데코레이터의 대표 사례가 우리가 늘 쓰던 자바 입출력이었다. 파일을 읽을 때 `new BufferedReader(new InputStreamReader(...))`처럼 괄호가 겹겹이 들어가는 코드를 많이 봤는데, 그게 데코레이터였다. 직접 확인해봤다.

```java
byte[] data = "첫째 줄\n둘째 줄\n".getBytes(StandardCharsets.UTF_8);

// 바이트를 읽는 스트림
InputStream bytes = new ByteArrayInputStream(data);
// 바이트 스트림을 문자 스트림(Reader)으로 바꿔준다
// 타입을 바꾸는 거라 엄밀히는 데코레이터보다 어댑터에 가깝다
Reader chars = new InputStreamReader(bytes, StandardCharsets.UTF_8);
// Reader를 감싸서 버퍼와 한 줄씩 읽기 기능을 덧씌운다. 감싼 뒤에도 Reader다 -> 데코레이터
BufferedReader lines = new BufferedReader(chars);

System.out.println("lines는 Reader인가? " + (lines instanceof Reader));
System.out.println("chars도 Reader인가? " + (chars instanceof Reader));
String line;
while ((line = lines.readLine()) != null) {
    System.out.println("읽은 줄: " + line);
}
lines.close();   // 바깥 것을 닫으면 안쪽까지 차례로 닫힌다
```

```text
lines는 Reader인가? true
chars도 Reader인가? true
읽은 줄: 첫째 줄
읽은 줄: 둘째 줄
```

`BufferedReader`는 `Reader`를 받아서 감싸고, 자기도 `Reader`다. 원래 Reader에는 없던 `readLine()`과 버퍼 기능을 덧붙인다. 커피로 치면 라떼(Reader)에 샷 추가(Buffered)를 한 셈이다.

조사하다가 헷갈린 게 하나 있었다. 처음엔 `InputStreamReader`도 데코레이터라고 생각했는데, 이건 `InputStream`을 받아서 `Reader`로 **타입을 바꿔준다.** 감싼 뒤에도 같은 타입이어야 데코레이터인데 타입이 달라지니, 이건 서로 다른 인터페이스를 연결하는 어댑터 패턴에 가깝다. 바이트 쪽에서 진짜 데코레이터는 `BufferedInputStream`이나 `DataInputStream`처럼 `InputStream`을 받아서 다시 `InputStream`이 되는 것들이다. 겉모양(괄호로 감싸기)이 같아도 의도와 타입을 봐야 패턴을 구분할 수 있다는 걸 알게 됐다.

그 밖에 `Collections.unmodifiableList(list)`도 원래 리스트를 감싸서 수정 기능만 막은 리스트를 돌려준다. 감싸도 여전히 `List`라는 점에서 같은 구조다.

### 장점과 단점

| 장점 | 단점 |
| --- | --- |
| 기존 클래스를 안 고치고 기능을 추가한다(OCP) | 작은 클래스가 많아지고, 객체도 겹겹이 생긴다 |
| 기능 하나에 클래스 하나라 책임이 명확하다(SRP) | 감싸는 순서에 따라 결과가 달라질 수 있다 |
| 실행 중에 원하는 조합을 골라 만들 수 있다 | 디버깅할 때 몇 겹을 따라 들어가야 한다 |
| 상속으로 모든 조합을 만들 때의 클래스 폭발을 막는다 | 특정 겹만 꺼내거나 빼기가 어렵다 |

"클래스 폭발"이 와닿았다. 상속으로 옵션 조합을 만들면 `IceLatte`, `VanillaLatte`, `IceVanillaLatte`, `IceVanillaShotLatte`처럼 조합마다 클래스가 필요하다. 옵션이 n개면 조합은 2의 n제곱이다. 데코레이터는 옵션마다 클래스 하나씩, n개면 된다.

단점 중 "순서"는 실습에서 실제로 겪었다. 라떼에 샷 추가를 하면 레시피 문자열이 "우유 거품 + 추가 샷" 순서로 찍힌다. 실제로는 샷을 먼저 넣고 우유를 붓는 게 자연스러운데, 데코레이터는 감싼 순서 그대로 쌓이기 때문이다. 순서가 중요하면 감싸는 쪽(팩토리)에서 순서를 책임져야 한다.

## 정리

- 디자인 패턴은 SOLID라는 원칙을 코드로 옮긴 검증된 레시피다
- 생성(누가 만드나), 구조(어떻게 조합하나), 행위(어떻게 협력하나)로 나뉜다
- 전략은 알고리즘을 갈아 끼우고, 팩토리는 생성을 맡기고, 데코레이터는 겹겹이 꾸미고, 싱글톤은 하나만 만든다
- 데코레이터의 핵심은 "감싸도 같은 타입"이다. 겉모양이 비슷해도 타입이 바뀌면 어댑터다

교수님께서 패턴을 다 외우려 하지 말고, 쓸 일이 있을 때 쓰다 보면 저절로 익혀진다고 하셨다. 그래서 바로 커피메이커 V2에 적용해봤다.
