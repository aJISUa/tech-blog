---
title: "[자프실2] 실습#5: 커피메이커 V2, 데코레이터와 팩토리로 다시 짜기"
date: 2026-10-08 00:30:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - SOLID
  - 디자인패턴
  - 데코레이터
  - 리팩토링
  - 과제
excerpt: "V1에서 불편했던 점을 코드 스멜로 정리하고, 데코레이터, 팩토리, 전략 패턴으로 주문과 결제가 있는 V2를 만들었다. 기존 파일을 하나도 고치지 않고 새 메뉴, 옵션, 결제 방식을 붙여서 OCP를 확인했다."
toc: true
toc_sticky: true
---

[커피메이커 V1]({{ site.baseurl }}{% post_url java-programming-2/2026-10-06-coffee-maker-v1 %})을 만들고 나서 "다음에 고치고 싶은 것" 세 가지를 적어뒀다. `prepare()`에서 출력 분리하기, 메뉴 이름으로 커피를 만들어주는 쪽 따로 두기, 공통 속성이 생기면 구조 바꾸기. 마침 6주차에 [디자인 패턴]({{ site.baseurl }}{% post_url java-programming-2/2026-10-07-design-pattern-basics %})을 배워서, 그때 적어둔 걸 패턴으로 고쳐보기로 했다.

수업에서 교수님께서 V1을 너무 자연스럽게 받아들이고 넘어간 학생이 많다고 하셨다. "이건 왜 이렇게 됐지? 이러면 뭐가 불편하지?"를 의심해보라고. 노션 문서 맨 아래에 흰 글씨로 숨겨둔 힌트도 있었다고 하셨는데, 네이밍과 주입 방식이었다. 그래서 V1을 다시 열어서 불편한 점부터 찾았다.

코드는 `JAVA2` 프로젝트에 `coffee2` 패키지를 새로 만들어 넣었다. V1(`coffee` 패키지)은 비교하려고 그대로 뒀다.

| 순서 | 내용 |
| --- | --- |
| 1 | V1 다시 보기: 무엇이 불편한가 |
| 2 | V2 설계: 어떤 원칙과 패턴을 왜 썼나 |
| 3 | Before / After 핵심 코드 비교 |
| 4 | V2 실행: 주문, 결제, 복잡한 음료 |
| 5 | OCP 검증: 기존 코드를 안 고치고 확장하기 |

## 1. V1 다시 보기: 무엇이 불편한가

V1을 다시 읽으면서 걸리는 곳을 코드 스멜 관점으로 적어봤다.

| 번호 | V1의 문제 | 어떤 냄새인가 |
| --- | --- | --- |
| ① | `prepare()`가 출력도 하고 문자열도 돌려준다. 그래서 라떼 줄 중간에 "에스프레소 준비"가 끼어들었다 | 한 메소드에 책임 두 개(SRP 위반) |
| ② | Americano의 필드 이름은 `espressoMachine`인데 타입은 `CoffeeMachine`이라 드립 머신도 들어간다. 실제로 필요한 건 "에스프레소"인데 "머신"을 받고 있다 | 모호한 네이밍, 의도와 다른 의존 |
| ③ | CoffeeMaker가 setter 주입만 있어서 `setCoffee()`를 깜빡하면 NullPointerException이 난다 | 필수 의존성을 선택처럼 받음 |
| ④ | main이 `new Latte(new Espresso(machine), frother)`처럼 모든 조립 순서를 알아야 한다 | 생성 책임이 한 곳에 몰림 |
| ⑤ | 샷 추가, 바닐라 시럽, 얼음 같은 옵션을 넣으려면 `IceVanillaLatte` 같은 조합별 클래스가 필요하다 | 클래스 폭발 |
| ⑥ | 메뉴, 가격, 주문, 결제가 없다. 카페라고 하기엔 커피만 만든다 | 기능 부족 |

②는 V1 실험에서 드립 머신으로 아메리카노를 만들었을 때 발견했던 거다. 그때는 "조립 책임이 main으로 넘어갔다" 정도로 정리했는데, 다시 보니 근본 원인은 Americano가 받는 것의 이름과 타입이 실제 의도와 맞지 않는 데 있었다. 아메리카노에 필요한 건 머신이 아니라 이미 뽑아둔 에스프레소다.

## 2. V2 설계

### 적용한 패턴과 원칙

V1의 문제를 하나씩 보면서 어떤 패턴이 맞을지 골랐다.

| 문제 | 해결 | 패턴 / 원칙 |
| --- | --- | --- |
| ② ⑤ 음료와 옵션 조합 | 에스프레소를 아메리카노, 라떼, 샷, 시럽, 얼음이 한 겹씩 감싼다 | **데코레이터**, OCP |
| ④ 조립 순서를 main이 다 앎 | "Latte"라는 이름만 주면 조립된 커피를 돌려주는 공장 | **팩토리**, SRP |
| ⑥ 결제 수단이 여러 개 | 카드, 현금, 포인트가 Payment 인터페이스를 구현 | **전략**, OCP, DIP |
| ① 출력과 반환이 섞임 | Coffee는 이름과 레시피 문자열만 돌려주고 출력은 CoffeeMaker만 한다 | SRP |
| ③ setter 주입 | Manager, Factory 등 필요한 것은 전부 생성자로 받는다 | DI, DIP |

슬라이드의 V2 예시도 팩토리와 데코레이터를 썼다. 다만 예시의 CoffeeFactory는 switch문이라 메뉴가 늘면 Factory를 고쳐야 했다. [디자인 패턴 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-07-design-pattern-basics %})을 정리하면서 이게 걸려서, V2에서는 레시피를 밖에서 등록받는 방식으로 바꿨다. 교수님께서도 노션 코드는 "솔루션 중 하나"일 뿐이라고 하셨다.

### 클래스와 책임

| 클래스 | 책임 | 원칙 |
| --- | --- | --- |
| Coffee | 음료의 규격. 이름과 레시피만 돌려준다 | SRP, ISP |
| Espresso, DripCoffee | 기본 음료. 머신으로 추출 | DI |
| CoffeeDecorator | 커피 하나를 품고 한 겹을 덧붙이는 공통 부모 | 데코레이터, OCP |
| Americano, Latte | 에스프레소를 감싸는 음료 | 데코레이터 |
| ExtraShot, Syrup, Ice | 어떤 커피든 감싸는 옵션 | 데코레이터 |
| NamedCoffee | 부품을 조합한 시그니처 메뉴에 이름만 붙인다 | |
| CoffeeFactory | 이름으로 레시피를 찾아 조립하고 옵션을 감싼다 | 팩토리, OCP |
| Menu | 음료와 옵션의 가격표 | SRP |
| Order | 무엇을, 어떤 옵션으로 주문했는지 | SRP |
| Payment | 결제 방식의 규격 | 전략, DIP |
| CardPayment, CashPayment, PointPayment | 각 결제 방식 | 전략 |
| CoffeeMaker | 완성된 커피를 내보낸다(출력 담당) | SRP |
| Manager | 메뉴 확인, 가격 계산, 결제, 제조 순서 지휘 | SRP, DI |

가격을 어디에 둘지 고민했다. 데코레이터마다 가격을 갖게 하면(샷 +500원) 깔끔해 보이지만, 가격이 바뀔 때마다 커피 클래스를 열어야 한다. 가격은 카페 정책이고 레시피와는 바뀌는 이유가 다르다. 그래서 가격은 Menu 한 곳에만 두고, 커피 클래스들은 가격을 모르게 했다. 바뀌는 이유가 다르면 나눈다는 SRP를 그대로 따른 셈이다.

### Before / After 클래스 다이어그램

Before는 V1이다.

![V1 클래스 다이어그램]({{ '/assets/images/java-programming-2/coffee1-design-full.png' | relative_url }})

After는 V2다.

![V2 클래스 다이어그램]({{ '/assets/images/java-programming-2/coffee2-after-uml.png' | relative_url }})

클래스는 9개에서 23개로 늘었다. 대신 화살표 방향이 정리됐다. Manager는 Menu, CoffeeFactory, CoffeeMaker와 Payment 인터페이스만 알고, 음료 쪽은 모두 Coffee 인터페이스로 모인다. 구체 클래스끼리 직접 연결된 곳은 머신과 거품기를 쓰는 부분 정도다.

## 3. Before / After 핵심 코드

### Coffee: 출력 빼기(①)

```java
// Before (V1)
public interface Coffee {
	String prepare();   // 구현체들이 안에서 System.out.print까지 했다
}

// After (V2)
// V2의 Coffee는 "이름"과 "레시피"만 돌려준다
// V1에서는 prepare()가 화면 출력까지 해서 라떼 줄이 뒤섞였다 -> 출력은 CoffeeMaker가 맡는다(SRP)
public interface Coffee {
	String name();
	String prepare();
}
```

### Americano: 머신 말고 에스프레소를 받기(②)

```java
// Before (V1): 이름은 espressoMachine인데 어떤 머신이든 들어간다
public class Americano implements Coffee {
	private final CoffeeMachine espressoMachine;
	public Americano(CoffeeMachine espressoMachine) { ... }
	public String prepare() {
		System.out.print("아메리카노 준비 : ");
		return espressoMachine.brew() + " + hot water";
	}
}
```

```java
// After (V2): 데코레이터의 공통 부모
// "안에 커피 하나를 품고 있고, 자기도 커피다"
public abstract class CoffeeDecorator implements Coffee {
	protected final Coffee base;

	protected CoffeeDecorator(Coffee base) {
		this.base = base;
	}

	// 이름과 레시피는 안쪽 것에 내 것을 덧붙인다. 하위 클래스는 덧붙일 부분만 정하면 된다
	@Override
	public String name() {
		return base.name() + addName();
	}

	@Override
	public String prepare() {
		return base.prepare() + " + " + addStep();
	}

	// 이름에 붙일 말. 기본은 없음
	protected String addName() {
		return "";
	}

	protected abstract String addStep();
}

// 아메리카노 = 에스프레소 + 물. 에스프레소를 감싸는 데코레이터
public class Americano extends CoffeeDecorator {
	public Americano(Coffee base) {
		super(base);
	}

	// 이름은 "Espresso + ..."가 아니라 그냥 Americano로 바꾼다
	@Override
	public String name() {
		return "Americano";
	}

	@Override
	protected String addStep() {
		return "hot water";
	}
}

// [옵션] 샷 추가. 어떤 커피든 감쌀 수 있다. 샷 두 번이면 두 번 감싸면 된다
public class ExtraShot extends CoffeeDecorator {
	private final CoffeeMachine machine;
	...
	@Override
	protected String addName() {
		return " +shot";
	}

	@Override
	protected String addStep() {
		return machine.brew() + "(extra)";
	}
}
```

Americano가 이제 머신이 아니라 `Coffee base`를 받는다. 이름과 실제로 필요한 것이 맞아졌다. 시럽(`Syrup`)과 얼음(`Ice`)도 `addStep()`만 다르게 구현했다.

만들고 나서 알았는데, CoffeeDecorator는 템플릿 메소드 모양이기도 하다. `name()`과 `prepare()`가 "안쪽 결과 + 내 것"이라는 순서를 고정하고, 자식은 `addStep()`만 채운다. 데코레이터 안에 템플릿 메소드가 들어간 셈이다. 패턴을 섞어 쓴다는 게 이런 거구나 싶었다.

### CoffeeFactory: switch 대신 등록(④)

```java
// Before (슬라이드 V2 예시): 메뉴가 늘면 case를 추가해야 한다
public Coffee createCoffee(String name) {
	switch (name) {
		case "Latte": return new Latte(new Espresso(espressoMachine), frother);
		case "Americano": return new Americano(new Espresso(espressoMachine));
		default: return new Espresso(espressoMachine);
	}
}
```

```java
// After (V2)
// 커피 공장. "Latte 만들어줘"라고 이름만 주면 조립된 커피를 돌려준다(팩토리)
// 레시피를 밖에서 등록받는다. 메뉴가 늘어도 이 클래스는 안 고친다(OCP)
public class CoffeeFactory {
	private final Map<String, Supplier<Coffee>> recipes = new HashMap<>();
	private final Map<String, UnaryOperator<Coffee>> toppings = new HashMap<>();

	// 음료 레시피 등록: 이름 -> 만드는 방법
	public void register(String name, Supplier<Coffee> recipe) {
		recipes.put(name, recipe);
	}

	// 옵션 등록: 이름 -> 커피를 한 겹 감싸는 방법(데코레이터)
	public void registerOption(String name, UnaryOperator<Coffee> topping) {
		toppings.put(name, topping);
	}

	public Coffee create(Order order) {
		Coffee coffee = recipes.get(order.getDrink()).get();
		// 옵션 순서대로 한 겹씩 감싼다
		for (String option : order.getOptions()) {
			coffee = toppings.get(option).apply(coffee);
		}
		return coffee;
	}
}
```

`Supplier<Coffee>`는 "부르면 커피를 하나 만들어주는 것", `UnaryOperator<Coffee>`는 "커피를 받아서 커피를 돌려주는 것"이다. 둘 다 메소드 하나짜리 함수형 인터페이스라 [람다식]({{ site.baseurl }}{% post_url java-programming-2/2026-09-11-lambda-expression %})으로 등록할 수 있다. 특히 `UnaryOperator<Coffee>`는 "커피를 받아 커피를 돌려준다"는 점에서 데코레이터 자체를 함수로 표현한 것과 같다. [클린코드 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-clean-code-refactoring %})의 "if문 줄이기 3단계: Map에 동작 등록하기"를 팩토리에 적용했다.

### CoffeeMaker와 Manager: setter 대신 생성자(③)

```java
// Before (V1): 커피가 없어도 만들어진다
public class CoffeeMaker {
	private Coffee coffee;
	public void setCoffee(Coffee coffee) { this.coffee = coffee; }
	public void makeCoffee() { System.out.println(coffee.prepare()); }
}
```

```java
// After (V2)
// 커피를 "내보내는" 담당. 화면 출력은 여기서만 한다
public class CoffeeMaker {
	public void make(Coffee coffee) {
		System.out.println("  [제조] " + coffee.name() + " : " + coffee.prepare());
	}
}

// 주문 흐름 담당. 메뉴 확인 -> 가격 계산 -> 결제 -> 제조 순서만 지휘한다
// 필요한 것은 전부 생성자로 받는다. 스스로 new 하는 것은 없다(DI)
public class Manager {
	private final Menu menu;
	private final CoffeeFactory factory;
	private final CoffeeMaker maker;

	public Manager(Menu menu, CoffeeFactory factory, CoffeeMaker maker) { ... }

	public void takeOrder(Order order, Payment payment) {
		System.out.println("주문: " + order);
		if (!menu.hasDrink(order.getDrink())) {
			System.out.println("  죄송합니다. " + order.getDrink() + "는 없는 메뉴입니다");
			return;
		}
		... // 옵션도 메뉴에 있는지 확인
		int price = menu.priceOf(order);
		if (!payment.pay(price)) {
			System.out.println("  결제 실패로 주문이 취소되었습니다");
			return;   // 돈을 못 받았으면 만들지 않는다
		}
		maker.make(factory.create(order));
	}
}
```

CoffeeMaker는 이제 커피를 필드로 들고 있지 않고, 만들 커피를 매번 매개변수로 받는다. 그러니 "커피를 안 넣고 만들기"가 아예 불가능하다. Manager가 쓰는 세 객체는 `final`로 생성자에서만 받는다. Manager는 결제 수단이 카드인지 현금인지 모르고 `pay()`만 부른다.

결제는 전략 패턴이다. Payment 인터페이스에 `boolean pay(int amount)` 하나만 두고, 현금은 낸 돈이 모자라면 실패하고 거스름돈을 주고, 포인트는 잔액이 줄어드는 식으로 각자 구현했다. 결제에 실패하면 커피를 만들지 않도록 성공 여부를 돌려받게 했다.

## 4. V2 실행

### 실습 목표

메뉴와 옵션을 등록해서 카페를 차리고, 기본 주문, 옵션을 겹겹이 얹은 복잡한 음료, 안 되는 주문까지 돌려본다.

### 코드

조립은 전부 `openCafe()`에서 한다. 확장 테스트에서도 같은 카페를 다시 차리려고 메소드로 뺐다.

```java
// 카페 하나를 차리는 과정. 확장 테스트에서도 그대로 다시 쓴다
static Manager openCafe(Menu menu, CoffeeFactory factory) {
	CoffeeMachine espressoMachine = new EspressoMachine();
	CoffeeMachine dripMachine = new DripCoffeeMachine();
	MilkFrother frother = new MilkFrother();

	// 1. 레시피 등록: 이름 -> 만드는 방법
	factory.register("Espresso", () -> new Espresso(espressoMachine));
	factory.register("Americano", () -> new Americano(new Espresso(espressoMachine)));
	factory.register("Latte", () -> new Latte(new Espresso(espressoMachine), frother));
	factory.register("DripCoffee", () -> new DripCoffee(dripMachine));
	// 시그니처 메뉴: 새 클래스 없이 기존 부품을 조합하고 이름만 붙인다
	factory.register("VanillaLatte", () -> new NamedCoffee("VanillaLatte",
			new Syrup(new Latte(new Espresso(espressoMachine), frother), "vanilla")));

	// 2. 옵션 등록: 이름 -> 한 겹 감싸는 방법
	factory.registerOption("샷추가", c -> new ExtraShot(c, espressoMachine));
	factory.registerOption("바닐라", c -> new Syrup(c, "vanilla"));
	factory.registerOption("얼음", c -> new Ice(c));

	// 3. 가격표
	menu.addDrink("Espresso", 2000);
	menu.addDrink("Americano", 3000);
	menu.addDrink("Latte", 4000);
	menu.addDrink("DripCoffee", 3500);
	menu.addDrink("VanillaLatte", 4500);
	menu.addOption("샷추가", 500);
	menu.addOption("바닐라", 500);
	menu.addOption("얼음", 0);

	return new Manager(menu, factory, new CoffeeMaker());
}
```

```java
Manager manager = openCafe(new Menu(), new CoffeeFactory());

System.out.println("===== 1. 기본 주문, 결제 방식은 제각각 =====");
manager.takeOrder(new Order("Americano"), new CardPayment());
manager.takeOrder(new Order("Latte"), new CashPayment(5000));
manager.takeOrder(new Order("DripCoffee"), new PointPayment(10000));

System.out.println("===== 2. 복잡한 음료: 옵션을 겹겹이 =====");
manager.takeOrder(new Order("Latte", "샷추가", "바닐라", "얼음"), new CardPayment());
manager.takeOrder(new Order("Americano", "샷추가", "샷추가"), new CardPayment());
manager.takeOrder(new Order("VanillaLatte", "얼음"), new CardPayment());

System.out.println("===== 3. 안 되는 주문 =====");
manager.takeOrder(new Order("Mocha"), new CardPayment());
manager.takeOrder(new Order("Latte", "휘핑"), new CardPayment());
manager.takeOrder(new Order("Latte", "샷추가"), new PointPayment(3000));
```

#### 실행 전에 예상한 것

- 1번: 아메리카노 3000원 카드, 라떼는 현금 5000원을 내서 거스름돈 1000원, 드립 커피는 포인트 3500P를 쓰고 6500P가 남는다
- 2번: 라떼에 샷, 바닐라, 얼음을 얹으면 4000 + 500 + 500 + 0 = 5000원. 레시피는 감싼 순서대로 이어 붙는다. 샷 두 번 아메리카노는 4000원
- 3번: 모카는 등록 안 했으니 없는 메뉴, 휘핑은 없는 옵션, 포인트 3000P로 4500원짜리는 결제 실패라서 만들지 않는다

#### 실제 결과

![커피메이커 V2 실행 결과]({{ '/assets/images/java-programming-2/coffee2-run.png' | relative_url }})

예상대로였다. V1처럼 출력이 뒤섞이는 일도 없다. 출력은 Manager와 CoffeeMaker, 그리고 결제 클래스에서 각자 한 줄씩만 하기 때문이다.

예상과 조금 달랐던 건 레시피 순서였다. "샷추가, 바닐라, 얼음" 라떼의 레시피를 보면 우유 거품 다음에 추가 샷이 들어간다. 실제 카페라면 샷을 먼저 넣고 우유를 부을 텐데, 데코레이터는 감싼 순서 그대로 쌓인다. 라떼가 먼저 만들어지고 그 위에 샷을 감쌌으니 당연한 결과다. [디자인 패턴 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-07-design-pattern-basics %})에 데코레이터의 단점으로 적은 "순서에 따라 결과가 달라진다"를 직접 본 셈이다. 순서가 정말 중요해지면 Factory가 옵션 순서를 정리해서 감싸면 된다.

VanillaLatte는 새 클래스 없이 `Syrup(Latte(Espresso))`를 조합하고 NamedCoffee로 이름만 붙였는데, 거기에 얼음 옵션을 또 감싸도 잘 동작했다. 시그니처 메뉴도 결국 커피라 옵션을 얹을 수 있다.

### 실습 후기

- V1의 문제 ①~⑥이 다 해결됐다. 특히 ⑤ 클래스 폭발이 사라진 게 컸다. 샷, 바닐라, 얼음 세 클래스로 조합 8가지(2의 3제곱)를 다 만들 수 있다.
- 결제 실패 시 커피를 안 만드는 흐름을 넣으면서, Manager가 "순서를 지휘하는 역할"이라는 게 실감 났다. 결제 방식이 뭐든 Manager의 흐름은 똑같다.

## 5. OCP 검증: 기존 코드를 안 고치고 확장하기

### 실습 목표

"구조가 좋아졌으니 새 요구사항도 쉽게 붙일 수 있다"를 직접 증명한다. 고객이 이런 걸 요청했다고 가정했다.

1. 새 메뉴: 모카(라떼 + 초콜릿 시럽)
2. 새 옵션: 휘핑크림
3. 새 결제 방식: 할인 쿠폰(쿠폰만큼 빼고 나머지는 다른 방법으로 결제)

조건은 **V2의 기존 파일은 하나도 고치지 않는 것**이다.

### 코드

새로 만든 파일은 세 개다.

```java
// [확장] 새 옵션: 휘핑크림. 기존 파일은 하나도 안 고치고 이 파일만 추가했다
public class WhippedCream extends CoffeeDecorator {
	public WhippedCream(Coffee base) {
		super(base);
	}

	@Override
	protected String addName() {
		return " +whip";
	}

	@Override
	protected String addStep() {
		return "whipped cream";
	}
}
```

```java
// [확장] 새 결제 방식: 할인 쿠폰. 쿠폰만큼 빼고 나머지는 다른 결제 방식에 맡긴다
// Payment를 구현하면서 Payment를 하나 품고 있다. 결제 쪽에도 데코레이터 같은 구조가 생겼다
public class CouponPayment implements Payment {
	private final int discount;
	private final Payment rest;

	public CouponPayment(int discount, Payment rest) {
		this.discount = discount;
		this.rest = rest;
	}

	@Override
	public boolean pay(int amount) {
		int left = Math.max(0, amount - discount);
		System.out.println("  [결제] 쿠폰 " + discount + "원 할인 -> 남은 금액 " + left + "원");
		if (left == 0) {
			return true;
		}
		return rest.pay(left);
	}
}
```

```java
// OCP 검증: 기존 클래스는 하나도 고치지 않고 새 메뉴, 새 옵션, 새 결제 방식을 붙인다
// 새로 만든 파일은 WhippedCream, CouponPayment, 그리고 이 테스트뿐이다
public class CafeExtensionTest {
	public static void main(String[] args) {
		Menu menu = new Menu();
		CoffeeFactory factory = new CoffeeFactory();
		Manager manager = CafeV2Test.openCafe(menu, factory);   // 원래 카페 그대로 차리고

		CoffeeMachine espressoMachine = new EspressoMachine();
		MilkFrother frother = new MilkFrother();

		// 새 메뉴: 모카 = 라떼 + 초콜릿 시럽. 새 클래스 없이 기존 부품을 조합해서 등록만 한다
		factory.register("Mocha", () -> new NamedCoffee("Mocha",
				new Syrup(new Latte(new Espresso(espressoMachine), frother), "chocolate")));
		menu.addDrink("Mocha", 4500);

		// 새 옵션: 휘핑크림. 새 데코레이터 클래스 하나 + 등록
		factory.registerOption("휘핑", c -> new WhippedCream(c));
		menu.addOption("휘핑", 700);

		System.out.println("===== 4. 확장: 새 메뉴, 새 옵션, 새 결제 =====");
		manager.takeOrder(new Order("Mocha", "휘핑"), new CardPayment());
		manager.takeOrder(new Order("Latte", "휘핑"), new CouponPayment(1000, new CashPayment(5000)));
		manager.takeOrder(new Order("Espresso"), new CouponPayment(3000, new CardPayment()));
	}
}
```

#### 실행 전에 예상한 것

- 모카 + 휘핑: 4500 + 700 = 5200원. 이름은 Mocha +whip
- 라떼 + 휘핑을 1000원 쿠폰과 현금 5000원으로: 4700 - 1000 = 3700원을 현금으로 내고 거스름돈 1300원
- 에스프레소 2000원을 3000원 쿠폰으로: 남은 금액이 0원이라 카드 결제 없이 끝난다

#### 실제 결과

![OCP 검증 실행 결과]({{ '/assets/images/java-programming-2/coffee2-extension.png' | relative_url }})

예상대로 세 주문이 다 됐다. 처음 V2 테스트에서 "휘핑 옵션은 없습니다"로 거절됐던 주문이, 파일 하나 추가하고 등록 한 줄을 넣자 바로 됐다. 바뀐 파일을 정리하면 이렇다.

| 요구사항 | 새로 만든 것 | 고친 기존 파일 |
| --- | --- | --- |
| 모카 메뉴 | 없음(기존 부품 조합 + 등록) | 없음 |
| 휘핑크림 옵션 | WhippedCream | 없음 |
| 할인 쿠폰 결제 | CouponPayment | 없음 |

Manager, CoffeeFactory, Menu, Coffee 쪽 어느 파일도 열지 않았다. V1이었다면 모카를 추가하려면 main에서 조립 코드를 직접 짜야 했고, 휘핑을 얹은 모카는 또 다른 클래스가 필요했을 거다.

확장까지 끝낸 `coffee2` 패키지 전체를 AmaterasUML로 그려봤다.

![AmaterasUML로 그린 확장 후 coffee2 패키지]({{ '/assets/images/java-programming-2/coffee2-amateras.png' | relative_url }})

WhippedCream은 CoffeeDecorator 아래에, CouponPayment는 Payment 아래에 붙었다. 2절의 After 그림과 비교하면 새 상자 두 개가 인터페이스 쪽에 매달렸을 뿐이고, Manager, CoffeeFactory, Menu로 가는 선은 그대로다. CouponPayment에서 Payment로 가는 선이 두 개(구현, 그리고 품고 있음)인 것도 보인다.

CouponPayment를 만들다가 재밌는 걸 발견했다. 쿠폰은 Payment를 구현하면서 다른 Payment를 품고 있다. 커피에서 쓴 데코레이터와 똑같은 모양이 결제 쪽에도 생긴 것이다. 그래서 "쿠폰 + 현금", "쿠폰 + 카드"처럼 조합이 자유롭다. 쿠폰 두 장을 겹쳐 쓰는 것도 `new CouponPayment(1000, new CouponPayment(500, new CardPayment()))`로 가능하다. 패턴을 한 번 이해하니 다른 곳에서도 같은 모양이 보였다.

### 실습 후기

- "확장에는 열려 있고 수정에는 닫혀 있다"를 표 하나로 확인할 수 있었다. 고친 기존 파일 칸이 전부 "없음"이다.
- 새 메뉴는 새 클래스조차 필요 없었다. 데코레이터로 부품을 작게 나눠두니, 메뉴 추가는 "부품 조합 + 등록"이 됐다.

## 마치며

V1에서 V2로 오면서 클래스가 9개에서 23개로 늘었다. [클린코드 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-clean-code-refactoring %})에서 정리한 오버 엔지니어링이 계속 마음에 걸렸다. 커피 세 잔만 만드는 거라면 V1으로 충분하다. 그런데 이번처럼 옵션, 시그니처 메뉴, 결제 방식이 계속 늘어나는 카페라면 이야기가 다르다. 실제로 확장 요구사항 세 개를 붙일 때 기존 파일을 한 번도 안 열었다. 현업 판단 기준에서 본 "정책이 수시로 바뀌는 비즈니스 코어는 유연하게"가 이런 경우였다.

알게 된 점과 느낀 점을 정리하면 이렇다.

- 패턴은 문제에서 출발해야 한다. 처음부터 "데코레이터를 써야지"가 아니라, V1의 불편한 점(조합별 클래스, 모호한 Americano)을 적고 나니 데코레이터가 답으로 보였다
- 한 패턴이 여러 원칙을 만족한다. 데코레이터 하나로 OCP와 SRP가 같이 해결됐다
- 패턴끼리 섞인다. CoffeeDecorator 안에 템플릿 메소드가 있고, 쿠폰 결제는 전략이면서 데코레이터 모양이다
- 데코레이터의 순서 문제처럼, 패턴에도 비용이 있다. 장점만 보고 쓰면 안 된다
- 네이밍이 설계를 드러낸다. 교수님 힌트대로 Americano가 받는 것의 이름(espressoMachine)과 실제 타입(CoffeeMachine)이 어긋난 곳이 설계 문제의 신호였다

다음 주에는 전략 패턴과 템플릿 메소드를 직접 구현하는 실습을 한다. 이번에 V2를 만들면서 결제(전략)와 CoffeeDecorator(템플릿 메소드)에서 두 패턴을 이미 조금씩 써봤으니, 실습에서 제대로 비교해볼 생각이다.
