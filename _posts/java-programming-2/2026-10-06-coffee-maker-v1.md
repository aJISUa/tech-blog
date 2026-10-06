---
title: "[자프실2] 실습#4: 커피메이커 V.1로 느껴본 DI와 IoC"
date: 2026-10-06 20:00:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - SOLID
  - DI
  - IoC
  - 리팩토링
  - 과제
excerpt: "콤비 커피머신을 상위 역할부터 하위 구체 클래스까지 설계하고, main에서 머신 바꿔 끼우기, 메뉴 추가, 가짜 객체 테스트, setter 주입 실수까지 직접 돌려보며 DI와 IoC의 효과를 확인했다."
toc: true
toc_sticky: true
---

SOLID와 DI, IoC를 이론으로 정리하고 나니, 직접 짜보지 않으면 "그래서 뭐가 좋은데?"에 답을 못 할 것 같았다. 그래서 5주차 실습인 커피메이커 V.1을 상위 역할부터 하위 구체 클래스 순서로 설계하고, 다 만든 뒤에는 main에서 이것저것 바꿔 끼워보면서 DI와 IoC가 실제로 뭘 해주는지 확인했다.

이론은 [클린코드와 리팩토링]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-clean-code-refactoring %}), [SOLID와 IoC/DI]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-solid-ioc-di %}) 글에 정리했다.

코드는 이클립스 `JAVA2` 프로젝트에 `coffee` 패키지를 만들어 넣었다.

| 순서 | 내용 |
| --- | --- |
| 1 | 요구사항 읽고 역할부터 설계하기 |
| 2 | 커피메이커 V.1 기본 코드와 실행 |
| 3 | main에서 DI, IoC 효과 테스트하기 |
| 4 | 돌아보며 정리한 질문 네 개 |

## 1. 요구사항과 설계

### 요구사항

실제 모델은 드립과 에스프레소를 같이 하는 콤비 커피머신이다. 교수님께서 이런 일체형 머신이 진짜 있길래 넣어봤다고 하셨다.

- 커피를 추출하는 머신은 EspressoMachine과 DripCoffeeMachine이 있다
- 음료는 에스프레소, 아메리카노, 라떼가 있다
- 각 음료는 자기 준비 방법을 알고 스스로 만들어진다(아메리카노 = 에스프레소 + 물, 라떼 = 에스프레소 + 우유)
- DI/IoC는 생성자 주입(노션에는 생성자/Setter 주입)으로 구성한다
- SOLID는 SRP, OCP, DIP를 중심으로 분리한다

노션 첫머리에 이런 문장이 있었다.

> 이제부터 우리가 만들 객체는 데이터 집합이 아니라, 기능(행동, 역할)의 집합입니다.

교수님께서도 데이터 컨테이너로 보는 순간 원두 종류, 모델명 같은 필드를 막 넣게 된다고 하셨다. 기능 중심으로만 보라고. 에스프레소 머신과 드립 머신은 "추출(brew)한다"는 기능은 같고 방법만 다른 애들이다.

### 상위 수준부터: 역할 세 개

구체 클래스를 생각하기 전에 역할부터 뽑았다. 노션에 "배치가 자유로운 그림으로 간단히 설명하면"이라는 문장이 있다.

> CoffeeMaker는 Coffee를 제작하도록 주문하고, Coffee는 CoffeeMachine에 의존(사용)한다.

이걸 그대로 그리면 세 칸이다.

![커피메이커 상위 역할 설계]({{ '/assets/images/java-programming-2/coffee1-design-top.png' | relative_url }})

- **CoffeeMaker**: 어떤 커피를 만들지 받아서 "만들어줘"라고 요청한다. 직접 재료를 조합하거나 기계를 다루지 않는다
- **Coffee**: 스스로 준비 방법을 안다. 교수님 표현으로는 "각 커피 안에 전용 바리스타가 있다"
- **CoffeeMachine**: 추출만 한다

여기서 Coffee를 인터페이스로 할지 추상 클래스로 할지 고민했다. 노션 답은 "Coffee를 행동 중심으로 볼 거면 인터페이스, 데이터 중심으로 볼 거면 추상 클래스"였다. 커피가 스스로 제조 방법을 아는 행동 중심이라 인터페이스로 했다. 나중에 가격이나 이름 같은 공통 속성이 늘어나면 추상 클래스로 바꿀 수 있다고 노션 코드 주석에 적혀 있다.

MilkFrother(우유 거품기)도 고민거리였다. CoffeeMachine을 상속하게 할지, 머신의 필드로 넣을지, 따로 둘지. 거품기는 커피를 추출하지 않으니 `brew()`를 구현하는 게 어색하다. 억지로 상속하면 [지난 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-solid-ioc-di %})의 타조처럼 될 것 같아서 노션과 같이 독립시켰다.

### 하위 구체 클래스 배치

역할 아래에 구체 클래스를 매달면 이렇게 된다.

![커피메이커 V.1 전체 클래스 다이어그램]({{ '/assets/images/java-programming-2/coffee1-design-full.png' | relative_url }})

수업 시간에 AmaterasUML로 다이어그램을 그리고 Java 뼈대 코드까지 export하는 걸 봤다. Amateras는 처음 설계할 때, PlantUML은 이미 있는 코드를 한눈에 볼 때 좋다고 한다. 나는 PlantUML로 위처럼 역할 세 칸을 먼저 그리고, 그 아래에 구체 클래스를 붙여가면서 설계했다. 코드를 다 짠 뒤에는 Amateras와 이클립스 PlantUML 플러그인으로 패키지 전체를 다시 그려봤고, 두 그림은 3번 실험 뒤에 붙였다.

다이어그램을 보면서 클래스마다 책임을 표로 정리했다. 슬라이드 28쪽 표의 "사용목적"을 내 말로 바꾼 것이다.

| 클래스 | 책임 | 관련 원칙 |
| --- | --- | --- |
| CoffeeMaker | 주문받은 커피에게 만들라고 요청 | DIP(Coffee 인터페이스만 앎), setter DI |
| CoffeeMachine | 추출 기능의 규격 | ISP, 추상화 |
| EspressoMachine, DripCoffeeMachine | 각자 방식으로 추출 | OCP(기계 추가 시 기존 코드 그대로) |
| MilkFrother | 우유 거품 | SRP(추출과 분리) |
| Coffee | 음료 준비의 규격 | OCP 기반 다형성 |
| Espresso, Americano | 주입받은 머신으로 음료 준비 | 생성자 DI, SRP |
| Latte | 완성된 에스프레소 + 거품기 조합 | 합성, OCP |

## 2. 커피메이커 V.1 기본 코드

### 실습 목표

노션 주석을 보면서 V.1을 직접 치고, 슬라이드 29쪽의 테스트를 돌려서 객체가 어떻게 연결되는지 확인한다.

### 코드

머신 쪽부터.

```java
// 커피를 "추출"하는 기계의 역할만 정해둔 인터페이스
// 에스프레소 머신이든 드립 머신이든 brew()만 할 줄 알면 된다
public interface CoffeeMachine {
	String brew();
}

// 콤비 머신의 왼쪽: 에스프레소 추출
public class EspressoMachine implements CoffeeMachine {
	@Override
	public String brew() {
		return "Extracting espresso";
	}
}

// 콤비 머신의 오른쪽: 드립 커피 추출
public class DripCoffeeMachine implements CoffeeMachine {
	@Override
	public String brew() {
		return "Dripping coffee";
	}
}

// 우유 거품기. 커피를 추출하는 기계가 아니라서 CoffeeMachine을 구현하지 않고 따로 뒀다
public class MilkFrother {
	public String frothMilk() {
		return "Frothing milk";
	}
}
```

음료 쪽. 핵심은 어떤 머신을 쓸지 음료가 직접 `new`로 고르지 않는다는 점이다.

```java
// 커피 메뉴의 역할. 데이터(이름, 가격)가 아니라 "스스로 준비할 줄 안다"는 기능만 약속한다
public interface Coffee {
	String prepare();
}

// 에스프레소: 주입받은 머신으로 추출만 하면 끝
public class Espresso implements Coffee {
	private final CoffeeMachine espressoMachine;

	// 어떤 머신을 쓸지 내가 new로 고르지 않고 생성자로 받는다(생성자 주입)
	public Espresso(CoffeeMachine espressoMachine) {
		this.espressoMachine = espressoMachine;
	}

	@Override
	public String prepare() {
		System.out.print("에스프레소 준비 : ");
		return espressoMachine.brew();
	}
}

// 아메리카노: 에스프레소 추출 + 물
public class Americano implements Coffee {
	private final CoffeeMachine espressoMachine;

	public Americano(CoffeeMachine espressoMachine) {
		this.espressoMachine = espressoMachine;
	}

	@Override
	public String prepare() {
		System.out.print("아메리카노 준비 : ");
		return espressoMachine.brew() + " + hot water";
	}
}

// 라떼: 머신이 아니라 "완성된 에스프레소(Coffee)"와 거품기를 받아서 조합한다(합성)
public class Latte implements Coffee {
	private final Coffee espresso;          // 에스프레소 구성
	private final MilkFrother milkFrother;  // 우유 구성

	public Latte(Coffee espresso, MilkFrother milkFrother) {
		this.espresso = espresso;
		this.milkFrother = milkFrother;
	}

	@Override
	public String prepare() {
		System.out.print("라떼 준비 : ");
		// 에스프레소를 어떻게 뽑는지는 모른다. espresso에게 준비해달라고 요청만 한다
		return espresso.prepare() + " + " + milkFrother.frothMilk();
	}
}
```

그리고 주문을 받는 CoffeeMaker.

```java
// 커피 제작(의뢰)기. 어떤 커피를 만들지 스스로 정하지 않고 setter로 받는다
// 구체 클래스(Espresso, Latte)는 하나도 모르고 Coffee 인터페이스만 안다 -> DIP
public class CoffeeMaker {
	private Coffee coffee;

	public void setCoffee(Coffee coffee) {   // setter 주입
		this.coffee = coffee;
	}

	// 만드는 방법은 커피가 안다. 나는 "네 방식대로 만들어줘"라고 요청만 한다
	public void makeCoffee() {
		System.out.println(coffee.prepare());
	}
}
```

조립은 main이 한다.

```java
// 모든 객체를 여기(main)에서 만들고 연결한다
// main이 조립을 맡는 공장(IoC 컨테이너) 역할이다
public class CoffeeMakerTest {
	public static void main(String[] args) {
		// 머신과 거품기 생성
		CoffeeMachine espressoMachine = new EspressoMachine();
		MilkFrother milkFrother = new MilkFrother();

		// 테스트메뉴로 에스프레소 준비생성.
		Coffee espresso = new Espresso(espressoMachine);
		System.out.println(espresso.prepare());

		// 이번에는 라떼생성
		Coffee latte = new Latte(espresso, milkFrother);
		System.out.println(latte.prepare());

		// 이번에는 DI를 적용하여 라떼를 만들자.
		CoffeeMaker maker = new CoffeeMaker();
		maker.setCoffee(latte);
		maker.makeCoffee();
	}
}
```

슬라이드 주석에는 라떼 결과가 "Extracting espresso + Frothing milk"라고 적혀 있다.

#### 실행 전에 예상한 것

- 에스프레소는 "에스프레소 준비 : Extracting espresso" 한 줄이 나온다
- 라떼는 슬라이드 주석처럼 "라떼 준비 : Extracting espresso + Frothing milk"가 나온다
- CoffeeMaker로 만든 라떼도 바로 위 줄과 똑같이 나온다

#### 실제 결과

![커피메이커 V.1 기본 테스트 실행 결과]({{ '/assets/images/java-programming-2/coffee1-basic.png' | relative_url }})

라떼 줄이 예상과 달랐다. 라떼 줄 중간에 에스프레소 준비 문구가 끼어 있다.

이유를 따라가보니 Latte가 `espresso.prepare()`를 부르기 때문이었다. 순서는 이렇다.

1. `System.out.println(latte.prepare())`에서 println은 괄호 안을 먼저 계산한다
2. Latte의 `prepare()`가 "라떼 준비 : "를 **print로 바로** 출력한다
3. 이어서 `espresso.prepare()`를 부르면 Espresso가 "에스프레소 준비 : "를 또 바로 출력하고, "Extracting espresso"를 **돌려준다**
4. Latte가 받은 문자열에 " + Frothing milk"를 붙여서 돌려준다
5. 그제야 println이 그 문자열을 출력한다

`prepare()`가 "출력하기"와 "결과 문자열 돌려주기"라는 두 가지 일을 같이 하고 있어서, 라떼처럼 다른 커피를 감싸면 중간 출력이 섞인다. 알고 보니 슬라이드 29쪽 코드에는 `prepare()` 안에 print가 없어서 주석대로 나오는 거였고, "에스프레소 준비 : " 같은 print는 노션 코드에만 있었다. 나는 노션 쪽을 따라 쳐서 결과가 달랐다. 결과가 틀린 건 아니지만, 출력은 화면 담당이 하고 `prepare()`는 문자열만 돌려주게 나누는 게 SRP에 더 맞겠다는 생각이 들었다.

CoffeeMaker로 만든 마지막 줄은 바로 위 줄과 똑같다. CoffeeMaker는 라떼가 뭔지 모르고 `prepare()`만 불렀을 뿐인데, 라떼가 자기 방식대로 만들었다.

## 3. main에서 DI, IoC 효과 테스트하기

### 실습 목표

DI를 하면 유연해진다는 말이 정말인지, 기본 코드를 하나도 안 고치고 main에서만 실험해봤다. 확인하고 싶었던 건 네 가지다.

1. 같은 클래스에 다른 객체를 주입하면 어떻게 되는가
2. 새 메뉴와 새 기계를 추가할 때 기존 코드를 정말 안 고쳐도 되는가(OCP)
3. 진짜 머신 대신 가짜를 넣어서 테스트할 수 있는가
4. setter 주입을 깜빡하면 어떻게 되는가

### 코드

2번을 위해 클래스 두 개를 추가했다. 콤비 머신에 드립 머신이 있는데 정작 드립 머신을 쓰는 메뉴가 없다는 게 마음에 걸려서 드립 커피를 만들었다. 수업 때 바리스타가 손으로 드립을 내릴 수도 있다는 이야기가 나와서, 핸드드립 기계도 하나 만들어 꽂아봤다.

```java
// [추가 메뉴] 드립 커피
// Coffee만 구현하면 끝. CoffeeMaker는 한 줄도 안 고쳤다 -> OCP
public class DripCoffee implements Coffee {
	private final CoffeeMachine dripMachine;

	public DripCoffee(CoffeeMachine dripMachine) {
		this.dripMachine = dripMachine;
	}

	@Override
	public String prepare() {
		System.out.print("드립 커피 준비 : ");
		return dripMachine.brew();
	}
}

// [추가 기계] 바리스타가 손으로 내리는 핸드드립
// CoffeeMachine만 구현하면 DripCoffee에 그대로 꽂을 수 있다. DripCoffee도 안 고친다
public class HandDripper implements CoffeeMachine {
	@Override
	public String brew() {
		return "Hand dripping by barista";
	}
}
```

3번을 위한 가짜 머신이다. 추출은 안 하고 몇 번 불렸는지만 센다.

```java
// 테스트용 가짜 머신. 진짜로 추출하지 않고, 몇 번 불렸는지만 센다
// 인터페이스에 의존하니까 진짜 머신 자리에 이런 것도 끼울 수 있다
public class FakeMachine implements CoffeeMachine {
	private int brewCount = 0;

	@Override
	public String brew() {
		brewCount++;
		return "[fake brew]";
	}

	public int getBrewCount() {
		return brewCount;
	}
}
```

4번은 CoffeeMaker를 생성자 주입으로 바꾼 버전을 따로 만들어서 비교했다. 원래 CoffeeMaker는 노션 코드 그대로 두고 싶어서 새 클래스로 만들었다.

```java
// CoffeeMaker를 생성자 주입으로 바꿔본 버전
// 커피 없이 만들어질 수 없으니 makeCoffee()에서 null 때문에 터질 일이 없다
public class SafeCoffeeMaker {
	private Coffee coffee;

	public SafeCoffeeMaker(Coffee coffee) {   // 필수 의존성은 생성자로
		if (coffee == null) {
			throw new IllegalArgumentException("커피 없이 커피메이커를 만들 수 없습니다");
		}
		this.coffee = coffee;
	}

	// 메뉴 변경은 선택 사항이라 setter도 남겨뒀다
	public void setCoffee(Coffee coffee) {
		if (coffee != null) {
			this.coffee = coffee;
		}
	}

	public void makeCoffee() {
		System.out.println(coffee.prepare());
	}
}
```

테스트 main이다.

```java
public class DiIocTest {
	public static void main(String[] args) {
		CoffeeMachine espressoMachine = new EspressoMachine();
		CoffeeMachine dripMachine = new DripCoffeeMachine();
		MilkFrother milkFrother = new MilkFrother();
		CoffeeMaker maker = new CoffeeMaker();

		System.out.println("===== 1. 같은 Americano에 머신만 바꿔 끼우기 =====");
		// Americano 코드는 그대로인데, 무엇을 주입하느냐에 따라 결과가 달라진다
		maker.setCoffee(new Americano(espressoMachine));
		maker.makeCoffee();
		maker.setCoffee(new Americano(dripMachine));   // 드립 머신을 넣어도 컴파일이 된다
		maker.makeCoffee();

		System.out.println("===== 2. 새 메뉴, 새 기계 추가 (기존 코드 수정 없음) =====");
		maker.setCoffee(new DripCoffee(dripMachine));
		maker.makeCoffee();
		maker.setCoffee(new DripCoffee(new HandDripper()));
		maker.makeCoffee();

		System.out.println("===== 3. 가짜 객체로 테스트하기 =====");
		// 라떼를 만들면 에스프레소 추출이 정확히 한 번 일어나야 한다
		FakeMachine fake = new FakeMachine();
		Coffee testLatte = new Latte(new Espresso(fake), milkFrother);
		String result = testLatte.prepare();
		System.out.println(result);
		System.out.println("brew 호출 횟수 = " + fake.getBrewCount()
				+ (fake.getBrewCount() == 1 ? "  -> PASS" : "  -> FAIL"));
		// Coffee는 메소드가 prepare() 하나뿐이라 람다로 가짜 커피를 바로 만들 수 있다
		maker.setCoffee(() -> "람다로 만든 테스트 음료");
		maker.makeCoffee();

		System.out.println("===== 4. setter 주입을 깜빡하면? =====");
		CoffeeMaker forgotMaker = new CoffeeMaker();
		try {
			forgotMaker.makeCoffee();   // setCoffee()를 안 불렀다
		} catch (NullPointerException e) {
			System.out.println("NullPointerException 발생! 커피가 주입되지 않았다");
		}
		SafeCoffeeMaker safeMaker = new SafeCoffeeMaker(new Espresso(espressoMachine));
		safeMaker.makeCoffee();
		try {
			new SafeCoffeeMaker(null);
		} catch (IllegalArgumentException e) {
			System.out.println("생성 시점에 막힘: " + e.getMessage());
		}
	}
}
```

#### 실행 전에 예상한 것

- 1번: Americano에 드립 머신을 넣으면 컴파일 에러가 날 것 같았다. 필드 이름이 `espressoMachine`이니까
- 2번: 드립 커피 두 잔이 각각 다른 기계 이름으로 나온다
- 3번: 라떼가 에스프레소를 한 번 부르니 호출 횟수 1로 PASS가 나온다. 람다로 만든 커피도 CoffeeMaker가 받아줄 것 같다
- 4번: setCoffee()를 안 부르면 coffee가 null이라 `coffee.prepare()`에서 NullPointerException이 난다. SafeCoffeeMaker는 null을 넣는 순간 막힌다

#### 실제 결과

![DI, IoC 효과 테스트 실행 결과]({{ '/assets/images/java-programming-2/coffee1-diioc.png' | relative_url }})

1번이 예상과 달랐다. 컴파일 에러 없이 드립 커피에 물을 탄 아메리카노가 나왔다. 생각해보니 당연했다. `espressoMachine`은 그냥 변수 이름이고, 타입은 `CoffeeMachine`이다. Americano가 약속받은 건 "`brew()`를 할 줄 아는 무언가"뿐이라 드립 머신도 자격이 된다. DI 덕분에 바꿔 끼우기는 자유로워졌지만, **올바르게 조립할 책임은 main으로 넘어갔다.** 이게 제어가 역전됐다는 말의 다른 면 같다. Americano가 스스로 고르던 걸 이제 조립하는 쪽이 고르니까, 조립하는 쪽이 실수하면 이상한 커피가 나온다. 막으려면 생성자 타입을 `EspressoMachine`으로 좁힐 수도 있지만, 그러면 구체 클래스에 의존하게 되어 DIP가 깨진다. 이런 게 교수님께서 말씀하신 원칙끼리의 충돌인가 싶었다.

2번은 예상대로였다. 드립 커피와 핸드드립 기계를 추가하면서 기존 파일은 하나도 열지 않았다. CoffeeMaker도, DripCoffee도 그대로다. OCP를 "확장에는 파일 하나 붙이는 정도"라고 설명하신 게 이런 느낌이었다.

3번도 예상대로 PASS가 나왔다. 가짜 머신 덕분에 "라떼를 만들면 에스프레소 추출이 한 번 일어난다"는 걸 숫자로 확인할 수 있었다. 진짜 머신이 결과를 화면에 찍기만 했다면 이런 확인은 못 했을 거다. 슬라이드에서 생성자 주입의 장점으로 "Mock 객체를 주입해서 테스트하기 쉽다"고 한 게 이거였다. 람다 커피도 잘 들어갔다. Coffee가 메소드 하나짜리 인터페이스라 [람다식 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-11-lambda-expression %})에서 배운 대로 함수형 인터페이스처럼 쓸 수 있었다.

4번도 예상대로였다. setter 주입은 setter를 부르기 전까지 null이라는 게 실제로 문제가 됐다. 이번 테스트는 내가 일부러 catch로 잡았지만, 실제로는 `makeCoffee()`를 부르는 순간 프로그램이 죽는다. 그리고 에러가 나는 곳이 실수한 곳(setCoffee를 빼먹은 곳)이 아니라 `makeCoffee()` 안이라 원인을 찾기도 어렵다. SafeCoffeeMaker는 만드는 순간 막히니까 실수한 바로 그 줄에서 알 수 있다.

실험을 다 하고 나서 `coffee` 패키지 전체를 두 가지 도구로 그려봤다. 먼저 교수님께서 보여주신 AmaterasUML이다.

![AmaterasUML로 그린 coffee 패키지 클래스 다이어그램]({{ '/assets/images/java-programming-2/coffee1-amateras.png' | relative_url }})

같은 패키지를 이클립스 PlantUML 플러그인으로 그린 그림이다.

![PlantUML 플러그인으로 그린 coffee 패키지 클래스 다이어그램]({{ '/assets/images/java-programming-2/coffee1-plantuml.png' | relative_url }})

Amateras도 기존 코드를 불러와 그릴 수 있고 저장하면 알아서 갱신되지만, 자동 배치는 교수님 말씀대로 좀 제멋대로라 상자를 손으로 잡아주게 된다. PlantUML은 코드를 읽어서 배치까지 균형 있게 해준다. 처음 설계할 때는 Amateras처럼 손으로 놓아보는 게 생각 정리에 좋고, 클래스가 많아진 뒤 한눈에 볼 때는 PlantUML이 편하다는 교수님 말씀이 두 그림을 나란히 놓으니 이해됐다.

1번 설계 그림과 비교하면 DripCoffee는 Coffee를, HandDripper와 FakeMachine은 CoffeeMachine을 구현하는 선이 하나씩 늘었고, DripCoffee에서 CoffeeMachine으로 가는 선이 하나 생겼을 뿐이다. 원래 있던 화살표는 그대로다. SafeCoffeeMaker도 CoffeeMaker처럼 Coffee 하나만 가리킨다. 재밌는 건 모든 new를 하는 CoffeeMakerTest와 DiIocTest가 두 그림 모두에서 아무 선 없이 떨어져 있다는 거다. main 안에서 지역 변수로만 만들고 연결하니 필드 관계로 안 잡히는 것 같다. 조립을 제일 많이 하는 클래스가 그림에서는 제일 안 보인다.

### 실습 후기

- DI는 "new를 밖으로 빼는 것"이라고만 생각했는데, 빼고 나니 바꿔 끼우기, 추가하기, 가짜로 테스트하기를 전부 main만 고쳐서 할 수 있었다.
- 대신 조립 책임이 main으로 모인다. 1번처럼 이상하게 조립해도 컴파일러가 안 막아준다. 스프링 같은 프레임워크가 이 조립을 대신 해주는 게 IoC라는 게 이제 이해된다.
- 요구사항은 생성자 주입인데 CoffeeMaker는 setter 주입이었다. 노션 주석에는 "선택적 의존성"이라고 되어 있다. 어떤 커피를 만들지는 주문마다 바뀌니까 setter가 자연스럽긴 하다. 그런데 커피 없는 커피메이커는 쓸 데가 없으니, 처음 하나는 생성자로 받고 바꾸는 건 setter로 하는 SafeCoffeeMaker 방식이 더 안전하다고 생각했다.

## 4. 돌아보며 정리한 질문 네 개

다 만들고 나서 "왜 이렇게 짰지?"를 스스로 설명할 수 있는지 질문 네 개로 점검해봤다.

### CoffeeMachine과 Coffee를 인터페이스로 정의한 이유

둘 다 "무엇을 할 수 있는가"만 정하면 되는 역할이기 때문이다. CoffeeMachine은 추출할 줄 알면 되고, Coffee는 스스로 준비할 줄 알면 된다. 공유할 데이터가 없으니 추상 클래스보다 인터페이스가 맞다.

그리고 이렇게 해두면 위쪽(CoffeeMaker, Coffee 구현체)이 아래쪽 구체 클래스를 몰라도 된다. 실제로 DripCoffee, HandDripper, FakeMachine, 람다 커피까지 추가했는데 위쪽 코드는 한 줄도 안 바뀌었다. 인터페이스가 [C타입 규격]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-solid-ioc-di %}) 역할을 한 거다.

### Espresso 같은 음료가 머신을 직접 new하지 않고 외부에서 받는 이유

음료 안에서 `new EspressoMachine()`을 하면 그 음료는 그 기계와 본드로 붙어버린다. 기계를 바꾸려면 음료 코드를 열어야 한다.

밖에서 받으면 음료는 "어떤 기계든 brew()만 해주면 된다"가 된다. 그래서 테스트할 때 가짜 머신을 넣을 수 있었다. 노션 주석에 "보통은 머신이 커피를 준비한다로 표현하겠지만 여기서는 커피가 머신을 결정한다. 그것도 주입으로"라고 적혀 있다. 음료가 어느 머신을 쓸지 스스로 정하지 않고 외부에서 받으니 제어가 역전된 것이다.

### setCoffee()로 DI가 적용되는 방식

CoffeeMaker는 `Coffee` 타입의 필드 하나만 갖고, `setCoffee()`로 들어온 객체를 거기 담는다. `makeCoffee()`는 그 필드에 `prepare()`를 요청할 뿐이다. 라떼가 들어오면 라떼가, 드립 커피가 들어오면 드립 커피가 자기 방식대로 만든다.

교수님께서 앞으로 `.뭐뭐`를 "~에게 ~를 요청한다"로 읽으라고 하셨다. `coffee.prepare()`는 "커피에게 준비를 요청한다"다. 노션 주석에도 "makeCoffee를 보면 제어권이 커피메이커에 있는 듯하지만, 결국 coffee.prepare()를 호출하고 제어권은 커피 구현체로 넘어간다"고 되어 있다. CoffeeMaker가 주문을 의뢰하고 Coffee가 스스로 만든다는 게 이 구조에서의 IoC다.

setter 방식이라 메뉴를 바꾸기 쉽다는 장점이 있지만, 4번 실험에서 본 것처럼 안 부르면 null이 된다는 단점도 있다.

### main의 실행 흐름 따라가기

`maker.makeCoffee()` 한 줄이 실행될 때 매개변수와 인자로 오가는 객체를 따라가봤다.

| 단계 | 누가 | 누구에게 요청 | 받은/돌려준 것 |
| --- | --- | --- | --- |
| 조립 | main | `new Espresso(espressoMachine)` | Espresso가 EspressoMachine을 받음 |
| 조립 | main | `new Latte(espresso, milkFrother)` | Latte가 Espresso와 MilkFrother를 받음 |
| 조립 | main | `maker.setCoffee(latte)` | CoffeeMaker가 Latte를 Coffee로 받음 |
| 1 | CoffeeMaker | coffee(실제로는 Latte)에게 `prepare()` | |
| 2 | Latte | espresso(실제로는 Espresso)에게 `prepare()` | |
| 3 | Espresso | espressoMachine에게 `brew()` | "Extracting espresso" 돌려받음 |
| 4 | Latte | milkFrother에게 `frothMilk()` | "Frothing milk" 돌려받음 |
| 5 | Latte | CoffeeMaker에게 | 둘을 합친 문자열을 돌려줌 |

`new`는 전부 main에만 있다. 나머지 클래스들은 받은 걸 쓰기만 한다. CoffeeMaker는 자기가 라떼를 만든다는 것도, 그 안에 에스프레소가 있다는 것도 모른다. 각자 자기 바로 아래에게만 요청한다. 이렇게 표로 그려보니 "결합도를 낮춘다"는 게 눈에 보였다.

## 마치며

처음에는 클래스가 10개 가까이 되는 게 커피 세 잔 만드는 것치고 과하다고 생각했다. [클린코드 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-clean-code-refactoring %})의 접시 50개 비유처럼. 그런데 main에서 이것저것 바꿔보니, 변경이 생길 때마다 기존 파일을 안 열어도 된다는 게 생각보다 편했다. 메뉴가 계속 늘어날 카페라면 이 정도 구조는 의미가 있다.

V.1을 하면서 다음에 고치고 싶은 게 생겼다.

- `prepare()`에서 출력과 문자열 반환을 분리하기(SRP)
- 노션 주석에 있던 것처럼 이름, 가격 같은 공통 속성이 생기면 Coffee를 추상 클래스나 레시피 객체(컴포지션)로 바꾸기
- 메뉴 이름으로 커피를 만들어주는 쪽을 따로 두기. 지금은 main이 모든 `new`를 직접 하고 있다

슬라이드 뒤쪽 V.2가 마침 팩토리, 메뉴, 결제를 추가하는 내용이고, 다음 주에는 디자인 패턴을 배운다. V.1에서 아쉬웠던 걸 V.2에서 패턴으로 고쳐볼 생각이다.
