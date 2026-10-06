---
title: "[자프실2] SOLID 퀴즈: 프린터, 라떼, 타조, 계산대"
date: 2026-10-06 20:30:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - SOLID
  - DI
  - 리팩토링
  - 퀴즈
excerpt: "5주차 SOLID와 DI 범위로 퀴즈 네 문제를 만들었다. 인터페이스를 나눴는데 컴파일이 안 되는 프린터, 순서가 뒤섞이는 라떼 주문서, 못 나는 타조, 네이버페이를 추가해달라는 계산대."
toc: true
toc_sticky: true
---

[제네릭과 컬렉션 퀴즈]({{ site.baseurl }}{% post_url java-programming-2/2026-09-20-generic-collection-quiz %}), [Thread 퀴즈]({{ site.baseurl }}{% post_url java-programming-2/2026-10-01-thread-quiz %})에 이어 5주차 SOLID와 DI 범위로 퀴즈를 만들었다.

SOLID는 원칙이라 "OCP의 뜻은?" 같은 문제를 내기 쉽다. 그런데 그런 건 외우면 끝이라 공부가 안 된다. 그래서 이번 주에 노션 예제를 직접 치거나 커피메이커를 만들면서 실제로 걸려 넘어진 것들을 코드 문제로 바꿨다.

문제 코드는 `JAVA2` 프로젝트 `quiz` 패키지에 넣고 전부 실제로 컴파일하고 실행해서 확인했다. 이론은 [SOLID와 IoC/DI]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-solid-ioc-di %}), 실습은 [커피메이커 V.1]({{ site.baseurl }}{% post_url java-programming-2/2026-10-06-coffee-maker-v1 %})에 있다.

| 번호 | 유형 | 연결되는 것 |
| --- | --- | --- |
| Quiz #1 | 컴파일 에러가 나는 줄과 이유 | ISP, 오버라이딩과 오버로딩, `@Override` |
| Quiz #2 | 실행 결과 예측 + 고치기 | 커피메이커 라떼, 문자열 연결 순서, SRP |
| Quiz #3 | 실행 결과 예측 + 재설계 | LSP, 타조 |
| Quiz #4 | 설계와 리팩토링 | OCP, DIP, 생성자 주입 |

## Quiz #1: 인터페이스를 나눴는데 왜 컴파일이 안 될까

복합기 인터페이스가 너무 커서 ISP에 맞게 Printer, Scanner, Fax로 나눴다. 프린트만 되는 기계와 프린트, 스캔이 되는 기계를 만들었다.

```java
interface Printer { void print(String content); }
interface Scanner { void scan(); }
interface Fax { void fax(String number); }

static class SimplePrinter1 implements Printer {
	public void print(String content) { System.out.println("프린트: " + content); }
}

static class SimplePrinter2 implements Printer, Scanner {                         // (1)
	public void print(String content) { System.out.println("프린트: " + content); }
	public void scan(String content) { System.out.println("스캔: " + content); }   // (2)
}

public static void main(String[] args) {
	Printer p1 = new SimplePrinter1();
	SimplePrinter2 p2 = new SimplePrinter2();
	p1.print("과제");
	p2.print("퀴즈");
	p2.scan();                  // (3)

	Scanner s = p2;             // (4)
	s.scan();                   // (5)
	Scanner s1 = p1;            // (6)
}
```

1. (1)~(6) 중에 컴파일 에러가 나는 줄을 모두 고르고 이유를 설명하시오.
2. (2)번 줄 위에 `@Override`를 붙이면 에러 메시지가 어떻게 달라질까? `@Override`를 붙이는 습관이 왜 도움이 될까?
3. 에러를 고치고 실행하면 무엇이 출력될까?

### 풀이

정답은 **(1)과 (6)**이다. 다만 **(3)은 javac에서만 에러가 하나 더 뜬다.** 이건 문제를 만들고 나서 이클립스와 터미널에서 각각 돌려보다가 알게 됐다.

먼저 이클립스 화면이다.

![Quiz #1 이클립스에서 본 컴파일 에러]({{ '/assets/images/java-programming-2/t5-quiz1-error.png' | relative_url }})

빨간 표시는 16번 줄(1)과 35번 줄(6) 두 군데뿐이다. 30번 줄 `p2.scan()`(3)은 멀쩡하다.

이 상태로도 이클립스는 실행을 시켜준다. 그런데 콘솔에는 아무것도 안 찍히고 바로 `Unresolved compilation problem`이 났다. 스택 트레이스는 35번 줄을 가리키는데, 그보다 위에 있는 `p1.print("과제")`도 찍히지 않았다. 이클립스는 에러가 있는 메소드를 통째로 에러를 던지는 코드로 바꿔서 컴파일하기 때문이라고 한다.

그런데 터미널에서 `javac`로 컴파일하면 에러가 세 개 나온다.

```text
error: SimplePrinter2 is not abstract and does not override abstract method scan() in Scanner
error: method scan in class SimplePrinter2 cannot be applied to given types;
  required: String
  found:    no arguments
error: incompatible types: Printer cannot be converted to Scanner
```

- **(1) 에러.** Scanner가 약속한 건 매개변수 없는 `scan()`인데, SimplePrinter2에는 `scan(String)`만 있다. 이름이 같아도 매개변수가 다르면 다른 메소드다. 그래서 (2)는 오버라이딩이 아니라 **오버로딩**이 됐고, `scan()`은 구현하지 않은 셈이라 "추상 메소드 scan()을 구현하지 않았다"는 에러가 클래스 선언 줄에 뜬다.
- **(2) OK.** 이 줄 자체는 문법상 문제가 없다. 그냥 새 메소드 하나를 추가한 것이다. 그래서 오히려 찾기 어렵다.
- **(3) 컴파일러마다 다르다.** `javac`는 "SimplePrinter2에 있는 scan은 `scan(String)`뿐이라 인자 없이 부를 수 없다"고 본다. 이클립스 컴파일러는 SimplePrinter2가 Scanner를 구현한다고 선언했으니 Scanner의 `scan()`을 물려받은 것으로 보고 넘어간다. 어차피 (1)이 이미 에러라서, (1)을 고치면 (3)은 두 컴파일러 모두에서 정상이 된다. SimplePrinter2를 abstract로 바꿔서 javac로 돌려보니 `p2.scan()`이 그대로 컴파일됐다. (3) 자체는 문법상 문제가 없고, (1) 때문에 javac가 덤으로 띄운 에러로 보는 게 맞는 것 같다. 원인은 하나인데 에러가 몇 군데에 뜨는지는 도구마다 다를 수 있다는 걸 이번에 처음 알았다.
- **(4), (5) OK.** (1)만 고치면 SimplePrinter2는 Scanner이니까 Scanner 변수에 담을 수 있다.
- **(6) 에러.** SimplePrinter1은 Printer만 구현했다. 스캔 못 하는 기계를 Scanner 자리에 넣을 수 없다. 이건 ISP가 제대로 동작하고 있다는 뜻이다. 인터페이스를 나눠둔 덕분에 "이 기계는 스캔이 안 된다"를 컴파일러가 잡아준다.

2번. `@Override`를 붙이면 (2)번 줄에 바로 이 에러가 추가된다.

```text
error: method does not override or implement a method from a supertype
```

`@Override`가 없을 때는 에러가 클래스 선언 줄(1)이나 호출하는 줄(3)에 뜨고, 정작 실수한 (2)번 줄은 멀쩡해 보인다. `@Override`를 붙이면 "나는 부모의 메소드를 재정의하려는 거야"라고 컴파일러에게 의도를 알려주는 거라, 실수한 바로 그 줄에서 잡힌다. 클린코드 시간에 들은 "의도를 드러내라"가 이런 거였다.

3번. (2)를 `@Override public void scan()`으로 고치고 (6)을 지우면 이렇게 나온다.

![Quiz #1 고친 뒤 실행 결과]({{ '/assets/images/java-programming-2/t5-quiz1-result.png' | relative_url }})

네 줄이 정상으로 나왔다.

### 출제 의도

노션의 ISP 개선 예제를 그대로 치다가 실제로 이 에러를 만났다. 처음에는 인터페이스 분리가 뭔가 잘못된 줄 알았는데, 원인은 SOLID가 아니라 자바 기초에서 배운 오버라이딩과 오버로딩의 차이였다. SOLID 문제인 줄 알았는데 오버로딩 문제였다. 하나 더, 인터페이스 이름을 `Scanner`로 지으면 `java.util.Scanner`와 이름이 겹쳐서 import하는 순간 헷갈린다. 이름 짓기가 제일 어렵다는 말이 여기서도 나온다.

## Quiz #2: 라떼 주문서는 어떤 순서로 찍힐까

커피메이커 V.1의 구조를 줄였다. 음료가 준비될 때 "~ 준비 :"를 출력하고, 결과 문자열을 돌려준다.

```java
interface Coffee { String prepare(); }

static class Espresso implements Coffee {
	public String prepare() {
		System.out.print("에스프레소 준비 : ");
		return "Extracting espresso";
	}
}

static class Latte implements Coffee {
	private final Coffee espresso;

	Latte(Coffee espresso) {
		this.espresso = espresso;
	}

	public String prepare() {
		System.out.print("라떼 준비 : ");
		return espresso.prepare() + " + Frothing milk";
	}
}

public static void main(String[] args) {
	Coffee latte = new Latte(new Espresso());
	System.out.println("주문: " + latte.prepare());
	System.out.println("영수증 출력 완료");
}
```

1. 실행 결과를 예측하시오. "주문:"은 줄의 어디에 찍힐까?
2. 이 코드에서 `prepare()`가 맡고 있는 책임은 몇 개일까? SRP 관점에서 어떻게 고치면 좋을까?

### 풀이

![Quiz #2 실행 결과]({{ '/assets/images/java-programming-2/t5-quiz2-result.png' | relative_url }})

"주문:"이 맨 앞이 아니라 **"라떼 준비 : 에스프레소 준비 : " 다음에** 찍힌다.

`System.out.println("주문: " + latte.prepare())`에서 println은 괄호 안을 먼저 다 계산해야 출력할 수 있다. 괄호 안의 문자열 연결을 하려면 `latte.prepare()`를 먼저 실행해야 하고, 그 안에서 `print`가 바로바로 화면에 찍힌다.

1. `latte.prepare()` 실행 → "라떼 준비 : " 출력
2. 그 안에서 `espresso.prepare()` 실행 → "에스프레소 준비 : " 출력, "Extracting espresso" 반환
3. Latte가 " + Frothing milk"를 붙여서 반환
4. 이제야 "주문: " + 결과가 완성되고, println이 출력

"주문: "이 제일 왼쪽에 써 있어도 화면에 나오는 건 가장 늦다. 왼쪽부터 계산하긴 하지만 "주문: "은 println이 불려야 화면에 나가고, prepare() 안의 print는 불리는 즉시 나가기 때문이다.

2번. `prepare()`는 **출력하기**와 **결과 만들기** 두 가지 일을 하고 있다. 그래서 다른 커피를 감싸거나 다른 문자열과 이어 붙이면 화면 순서가 뒤섞인다. 고칠 때는 `prepare()`는 문자열만 돌려주게 하고, 화면에 뭘 어떤 순서로 보여줄지는 출력하는 쪽 한 곳에서만 정한다.

```java
interface Coffee {
	String name();
	String prepare();
}

static class Latte implements Coffee {
	...
	public String name() { return "라떼"; }
	public String prepare() { return espresso.prepare() + " + Frothing milk"; }   // 출력 없음
}

// 화면에 무엇을 어떤 순서로 보여줄지는 출력하는 쪽이 정한다
System.out.println("주문: " + latte.name() + " 준비 : " + latte.prepare());
```

![Quiz #2 고친 뒤 실행 결과]({{ '/assets/images/java-programming-2/t5-quiz2-fix.png' | relative_url }})

이제 "주문:"이 맨 앞에 나오고, 라떼 안에 에스프레소가 몇 겹으로 들어 있든 출력 순서가 흔들리지 않는다. 나중에 콘솔이 아니라 GUI나 웹 화면에 보여주게 바뀌어도 Coffee 클래스들은 고칠 필요가 없다.

### 출제 의도

커피메이커 V.1을 실행했을 때 라떼 줄에 "에스프레소 준비"가 끼어 나와서 한참 들여다봤다. 원인을 따라가 보니 문법(문자열 연결과 메소드 호출 순서) 문제이면서 동시에 설계(SRP) 문제였다. 출력 결과 하나로 둘 다 짚을 수 있어서 문제로 만들었다.

## Quiz #3: 새들아 다 같이 날아라

```java
static class Bird {
	protected String name;
	Bird(String name) { this.name = name; }
	void fly() { System.out.println(name + ": 훨훨~"); }
}

static class Sparrow extends Bird { Sparrow() { super("참새"); } }

static class Ostrich extends Bird {
	Ostrich() { super("타조"); }

	@Override
	void fly() {
		throw new UnsupportedOperationException(name + "는 못 날아요");
	}
}

static class Eagle extends Bird { Eagle() { super("독수리"); } }

// Bird라면 다 날 수 있다고 믿고 짠 메소드
static void flyAll(List<Bird> birds) {
	for (Bird b : birds) {
		b.fly();
	}
	System.out.println("모두 날았다!");
}

public static void main(String[] args) {
	List<Bird> birds = Arrays.asList(new Sparrow(), new Ostrich(), new Eagle());
	try {
		flyAll(birds);
	} catch (UnsupportedOperationException e) {
		System.out.println("비행 중단: " + e.getMessage());
	}
	System.out.println("새는 " + birds.size() + "마리");
}
```

1. 실행 결과를 예측하시오. 독수리는 날까?
2. 컴파일 에러는 없는데 무엇이 문제일까? SOLID의 어떤 원칙을 어겼을까?
3. 타조가 `flyAll()`에 들어오는 것 자체를 막으려면 어떻게 다시 설계해야 할까?

### 풀이

![Quiz #3 실행 결과]({{ '/assets/images/java-programming-2/t5-quiz3-result.png' | relative_url }})

참새만 날고, 타조에서 예외가 터져서 **독수리는 날지 못한다.** "모두 날았다!"도 안 찍힌다. 예외가 for문을 빠져나가 main의 catch로 바로 가기 때문이다. 타조 하나 때문에 독수리는 날아보지도 못했다.

2번. **LSP(리스코프 치환 원칙) 위반**이다. `flyAll()`은 "Bird면 fly()를 부를 수 있다"는 부모의 약속을 믿고 짰다. 그런데 Ostrich는 Bird 자리에 들어가서 그 약속을 깬다. 자식이 부모 자리에서 문제를 일으키면 안 된다는 게 LSP다. 컴파일러는 문법만 보기 때문에 이 문제를 못 잡고, 실행해봐야 터진다. 다형성을 처음 배울 때는 "수술한 강아지는 못 짖어요"처럼 재밌는 예시였는데, 이제는 이런 걸 막아야 하는 입장이 됐다.

3번. "모든 새는 난다"는 가정을 버리고 역할로 나눈다.

```java
interface Bird { String name(); }
interface Flyable extends Bird { void fly(); }
interface Walkable extends Bird { void walk(); }

static class Sparrow implements Flyable { ... }
static class Ostrich implements Walkable { ... }
static class Eagle implements Flyable { ... }

// 나는 새만 받는다. 타조는 처음부터 들어올 수 없다
static void flyAll(List<Flyable> birds) { ... }

flyAll(Arrays.asList(new Sparrow(), new Eagle()));
// flyAll(Arrays.asList(new Sparrow(), new Ostrich()));   // 컴파일 단계에서 막힌다
new Ostrich().walk();
```

![Quiz #3 고친 뒤 실행 결과]({{ '/assets/images/java-programming-2/t5-quiz3-fix.png' | relative_url }})

이제 타조를 `flyAll()`에 넣으려고 하면 실행 전에 컴파일 에러가 난다. 실행해봐야 알던 문제를 컴파일러가 잡아주는 문제로 바꾼 것이다. 타조는 Walkable로서 잘 걷는다. 부모에서 기능을 다 주고 자식에서 예외로 막는 대신, 할 수 있는 것만 인터페이스로 갖게 했다. ISP와도 닮았다.

### 출제 의도

슬라이드의 Bird와 Ostrich 예시는 "타조가 못 난다"에서 끝나는데, 실제로 그게 어떤 피해를 주는지 보고 싶었다. 리스트에 섞어 넣고 돌려보니 아무 잘못 없는 독수리까지 날지 못했다. 한 클래스가 약속을 어기면 그 클래스만이 아니라 같이 쓰이는 전체가 망가진다는 게 LSP가 중요한 이유라고 생각한다.

## Quiz #4: 네이버페이를 추가해달라는 요청이 왔다

```java
static class CreditCard {
	void pay(int amount) { System.out.println("카드로 " + amount + "원 결제 완료!"); }
}

static class KakaoPay {
	void send(int amount) { System.out.println("카카오페이로 " + amount + "원 전송!"); }
}

// 결제 수단을 직접 만들어 들고 있는 계산대
static class Checkout {
	CreditCard card = new CreditCard();
	KakaoPay kakao = new KakaoPay();

	void pay(String type, int amount) {
		if (type.equals("card")) {
			card.pay(amount);
		} else if (type.equals("kakao")) {
			kakao.send(amount);
		} else {
			System.out.println(type + "은(는) 지원하지 않는 결제 수단입니다");
		}
	}
}

public static void main(String[] args) {
	Checkout checkout = new Checkout();
	checkout.pay("card", 10000);
	checkout.pay("kakao", 7000);
	checkout.pay("naver", 5000);
	checkout.pay("Card", 3000);
}
```

1. 실행 결과를 예측하시오. 마지막 `"Card"`는 어떻게 될까?
2. 네이버페이를 추가하려면 Checkout의 어디를 고쳐야 할까? 이 구조가 어기고 있는 SOLID 원칙을 두 개 이상 고르고 이유를 설명하시오.
3. 네이버페이를 추가할 때 Checkout을 한 줄도 고치지 않도록 리팩토링하시오.

### 풀이

![Quiz #4 실행 결과]({{ '/assets/images/java-programming-2/t5-quiz4-result.png' | relative_url }})

카드와 카카오페이는 결제되고, `"naver"`는 지원하지 않는다고 나온다. 함정은 마지막 줄이다. `"Card"`도 **지원하지 않는 결제 수단**이 된다. `equals()`는 대소문자를 구분하니까. 결제 수단을 문자열로 구분하는 코드에서는 오타 하나가 결제 실패로 이어진다. 클린코드 글에서 본 "마법의 문자열" 스멜이다.

2번. 네이버페이를 추가하려면 Checkout에 `NaverPay naver = new NaverPay();` 필드를 추가하고, `pay()`의 if-else에 분기를 하나 더 넣어야 한다. 결제 수단이 늘 때마다 계산대를 뜯어고쳐야 한다.

- **OCP 위반**: 새 결제 수단을 추가(확장)하려면 기존 코드(Checkout)를 수정해야 한다.
- **DIP 위반**: Checkout(상위 모듈)이 CreditCard, KakaoPay(하위 구체 클래스)를 직접 `new`로 만들어서 의존한다. 게다가 결제 메소드 이름도 `pay()`, `send()`로 제각각이라 공통 규격이 없다.
- **SRP 관점**: Checkout이 결제 요청뿐 아니라 결제 수단을 만들고 고르는 일까지 하고 있다.

3번. 공통 규격(PaymentMethod)을 만들고, Checkout은 그 규격에만 의존하게 한다. 어떤 결제 수단을 쓸지는 밖에서 생성자로 넣어준다.

```java
interface PaymentMethod { void pay(int amount); }

static class CreditCard implements PaymentMethod { ... }
static class KakaoPay implements PaymentMethod { ... }

// 새로 추가한 파일은 이것 하나. Checkout은 안 고쳤다
static class NaverPay implements PaymentMethod {
	public void pay(int amount) { System.out.println("네이버페이로 " + amount + "원 결제 완료!"); }
}

// 계산대는 결제 수단이 무엇인지 모르고 pay()만 요청한다
static class Checkout {
	private final PaymentMethod method;

	Checkout(PaymentMethod method) {   // 생성자 주입
		this.method = method;
	}

	void pay(int amount) {
		method.pay(amount);
	}
}

new Checkout(new CreditCard()).pay(10000);
new Checkout(new KakaoPay()).pay(7000);
new Checkout(new NaverPay()).pay(5000);
// 메소드가 pay() 하나뿐이라 테스트용 결제 수단은 람다로 바로 만들 수 있다
new Checkout(amount -> System.out.println("[테스트] " + amount + "원 결제된 셈 치기")).pay(1);
```

![Quiz #4 고친 뒤 실행 결과]({{ '/assets/images/java-programming-2/t5-quiz4-fix.png' | relative_url }})

NaverPay 클래스 하나만 추가했고 Checkout은 그대로다. 문자열로 결제 수단을 고르지 않으니 `"Card"` 같은 오타 문제도 사라졌다. 잘못된 결제 수단을 넣으면 실행 중에 "지원하지 않는다"가 아니라 컴파일 단계에서 막힌다.

마지막 줄의 람다는 덤이다. PaymentMethod가 메소드 하나짜리 인터페이스라서, 진짜 돈이 나가지 않는 테스트용 결제 수단을 한 줄로 만들 수 있다. 커피메이커에서 가짜 머신을 넣어 테스트했던 것과 같은 이야기다.

### 출제 의도

결합도와 응집도 노션 문서의 Checkout 예제를 바탕으로, 수업에서 들은 "결제 수단이 계속 추가되는" 상황을 붙였다. 한 문제 안에서 OCP, DIP, SRP를 다 찾을 수 있고, 고친 코드에서 생성자 주입과 테스트 용이성까지 이어진다. SOLID 원칙들이 따로 노는 게 아니라 서로 연결되어 있다는 걸 보여주는 문제라고 생각해서 마지막에 넣었다.

## 만들면서 느낀 점

Thread 퀴즈는 "결과가 매번 바뀌는데 뭘 물어보지?"가 어려웠다면, SOLID 퀴즈는 "원칙을 코드로 어떻게 물어보지?"가 어려웠다. 원칙 이름을 묻는 순간 암기 문제가 되니까. 그래서 출력 결과나 컴파일 에러처럼 **눈에 보이는 결과에서 시작해서, 왜 그런지 따라가다 보면 원칙에 도착하는** 문제를 만들려고 했다.

만들다 보니 네 문제 중 세 문제가 SOLID 이전의 기본기와 엮여 있었다. 오버라이딩과 오버로딩(Quiz #1), 문자열 연결과 메소드 실행 순서(Quiz #2), 예외가 반복문을 빠져나가는 흐름(Quiz #3). 원칙을 쓰려다 기본 문법에서 걸리는 일이 생각보다 많았다.
