---
title: "[자프실2] 클린코드와 리팩토링: 방향과 도구, 그리고 둘이 부딪힐 때"
date: 2026-10-02 18:00:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - 클린코드
  - 리팩토링
  - 코드스멜
  - FizzBuzz
excerpt: "5주차는 SOLID에 들어가기 전에 클린코드와 리팩토링이 뭔지, 둘이 어떻게 다르고 언제 충돌하는지부터 정리했다. FizzBuzzRule로 리팩토링 효과도 직접 돌려봤다."
toc: true
toc_sticky: true
---

5주차부터는 Thread를 마무리하고 객체지향 설계로 넘어왔다. 슬라이드 하나(47쪽)로 2주를 한다고 하셨다. 교수님께서 "4주차까지 열심히 달려왔으니 이제는 산에 올라와서 경치 구경하는 마음으로" 보라고 하셨다. 정상에서 라면 먹고 노는 시간이라 정답은 없다고.

수요일에 클린코드 부분을 미리 읽어두고, 금요일에 SOLID와 커피메이커를 했다. 이 글은 SOLID 앞부분인 클린코드와 리팩토링 정리다. SOLID와 IoC/DI는 [다음 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-02-solid-ioc-di %})에, 커피메이커 실습은 [실습#4 글]({{ site.baseurl }}{% post_url java-programming-2/2026-10-06-coffee-maker-v1 %})에 정리했다.

## 내 코드를 다시 열었을 때

슬라이드 첫 질문이 "내가 만든 코드를 수정하려고 열었을 때, 내 표정이 밝을 수 있을까?"였다. 솔직히 1학년 때 과제를 지금 열면 표정이 밝을 자신이 없다.

교수님께서 개발자 유머 이야기를 하셨다. 날개 대신 머리를 돌려서 나는 새, 앞바퀴를 아무거나 달아놓고 굴러가는 차 같은 그림들이다. "이 코드를 짤 땐 신과 나만 알았는데 지금은 신만 안다"는 말도 있다. 옛날엔 몰랐는데 이게 웃기다면 성장한 거라고 하셨다.

결국 "작동은 하는데 엉망인 코드"가 문제다. 의도한 일을 하긴 하는데 중첩된 if가 가득하고, 여기저기 퍼져 있어서 하나 고치려면 여러 군데를 고쳐야 한다. 이런 징후를 **코드 스멜(bad code smell)**이라고 한다. 좋은 코드 스멜이라는 말은 없다. 나쁜 냄새는 코드를 다음처럼 만든다.

- 경직된다(Rigid): 하나 바꾸려면 줄줄이 바꿔야 한다
- 깨지기 쉽다(Fragile): 한 곳을 고쳤는데 엉뚱한 곳이 터진다
- 재사용하기 어렵다(Immobile): 떼어서 다른 데 쓸 수가 없다

## 코드 스멜과 좋은 코드의 향기

슬라이드 8쪽 표가 제일 실용적이었다. 왼쪽이 나쁜 냄새, 오른쪽이 그걸 고친 모습이다.

| 나쁜 코드 스멜 | 좋은 코드의 향기 |
| --- | --- |
| 거대 클래스(God Class): 한 클래스가 책임을 너무 많이 가짐 | 단일 책임 클래스(SRP): 작고 응집도 높음 |
| 긴 메소드: 한 메소드가 수십, 수백 줄 | 작고 목적이 명확한 메소드: 이름만 봐도 뭘 하는지 앎 |
| 중복된 코드: 복붙한 코드가 여러 곳에 | DRY(Don't Repeat Yourself): 공통 로직을 추출해 재사용 |
| 마법의 숫자/문자열: `status == 2` | 의미 있는 상수나 enum: `status == OrderStatus.SHIPPED` |
| 복잡한 조건문(if-else 지옥) | 다형성을 활용한 설계(전략 패턴 등) |
| 지나치게 많은 매개변수 | 연관된 매개변수를 객체로 묶기 |
| '무엇을' 하는지 설명하는 주석: `i++; // i를 1 증가` | '왜' 그렇게 했는지 설명하는 주석 |

마지막 줄이 찔렸다. 지금까지 실습 코드에 "내 말로 설명하는 주석"을 달아왔는데, 돌아보면 "무엇을" 하는지 적은 게 꽤 있다. 수업 정리용이라 괜찮다고 생각하지만, 실제 프로젝트라면 코드만 봐도 아는 내용은 빼고 "왜"를 적어야겠다.

## 클린코드는 방향, 리팩토링은 도구

노션 문서 제목이 이 단원을 한 줄로 요약한다. "클린코드는 방향! 리팩토링은 기술이자 도구! 디자인패턴은 레시피, SOLID는 원칙!!!"

교수님께서 클린코드는 "지켜야 해, 안 하면 안 돼"가 아니라 **방향성**이라고 하셨다. 급하면 우리도 막 짠다. 그래도 여럿이서, 규모 있게, 체계 잡힌 소프트웨어를 만들 때는 누구나 알아보기 쉬운 규칙을 맞추는 게 필요하다.

| | 클린코드 | 리팩토링 |
| --- | --- | --- |
| 정의 | 처음부터 읽기 쉽고 유지보수하기 쉬운 코드를 작성하는 철학 | 기존 코드의 동작은 유지하면서 내부 구조를 개선하는 작업 |
| 목적 | 가독성, 명확성, 의도 전달 | 유지보수성, 확장성, 테스트 용이성 |
| 적용 시점 | 처음 작성할 때부터 | 기능이 동작한 후, 또는 테스트 통과 후 |

리팩토링의 정의에서 중요한 건 "겉보기 동작은 그대로"라는 부분이다. 버그를 잡거나 새 기능을 넣는 건 리팩토링이 아니다. 노션에는 Red-Green-Refactor(실패하는 테스트 작성 → 통과하게 구현 → 리팩토링) 순서로 하라고 되어 있다. 동작이 같은지 확인할 테스트가 먼저 있어야 마음 놓고 구조를 바꿀 수 있다는 뜻이다. 2주차 [Battle 리팩토링]({{ site.baseurl }}{% post_url java-programming-2/2026-09-15-battle-refactoring %}) 때도 원본에는 BugDemo, 고친 쪽에는 RefactorCheck를 붙여서 확인했다. 그때는 버그까지 같이 고쳤으니 엄밀히는 리팩토링만 한 건 아니었지만, 테스트를 먼저 두고 구조를 바꾼 건 같은 생각이었다.

### 순서대로 하라는 뜻이 아니다

슬라이드 7쪽에 "SOLID로 뼈대를 잡고, 디자인 패턴으로 문제를 풀고, 리팩토링으로 다듬고, 클린코드로 완성한다"는 그림이 있다. 교수님께서 이걸 "이 순서대로 해야 한다"로 오해하지 말라고 하셨다. OOD(객체지향 설계)는 설계 자체를 위해 하는 게 아니라 좋은 OOP를 위해 한다. 잘 살자고 먹는 거라고.

외국에서는 기존 코드를 가지고 리팩토링 연습을 많이 한다고 하셨다. 기존 코드를 보고 "지저분하다, 생성에 자율권이 없다, 본드로 붙여놨다" 싶으면 그때 SOLID나 패턴을 적용하는 식이다.

## 왜 처음부터 신경 써야 할까: 기술적 부채

"일단 돌아가게 빨리 만들고 나중에 고치자"는 말을 다들 한다. 그런데 결국 복잡해진 코드를 정리하는 데 엄청난 시간이 든다. 이게 **기술적 부채(Technical Debt)**다. 지금 빨리 가려고 품질을 잠깐 희생한 대가를 나중에 이자까지 쳐서 갚는 것이다.

교수님께서 "초반에 시간을 들이든 뒤에 들이든 어차피 든다"고 하셨다. 수요일에는 기초에 구멍이 있는 것도 부채라서 바쁠 때 대가를 치르게 된다고도 하셨다.

요구사항을 바꾸는 건 주로 고객과 경영자다. 출시 일주일 전에 출력 메시지를 프랑스어로 바꾸라더니, 계약이 틀어져서 다시 중국어로 바꾸라고 한 적도 있다고 하셨다. 벽시계 유머도 들려주셨다. 파이프가 지나가야 할 자리에 시계가 걸려 있는데, 경영자가 시계를 옮기기 싫다고 해서 파이프를 휘어서 돌아가게 만든다는 이야기다. 웃기면서 슬펐다.

그래서 리팩토링의 목적은 **예쁜 코드가 아니라 나중에 바뀔 때 유연하게 대응하는 것**이라고 하셨다.

## FizzBuzzRule로 본 리팩토링 효과

노션에 이걸 체감하게 해주는 예제가 있다. 3의 배수면 "Fizz", 5의 배수면 "Buzz", 15의 배수면 "FizzBuzz", 나머지는 숫자를 출력하는 게임을 만들어 달라는 의뢰다. 369 게임이랑 비슷하다.

처음에는 금방 끝난다.

```java
class FizzBuzzRule1 {
    public List<String> generate(int number) {
        List<String> al = new ArrayList<>();
        for (int i = 1; i <= number; i++) {
            if (i % 15 == 0) al.add("FizzBuzz");
            else if (i % 3 == 0) al.add("Fizz");
            else if (i % 5 == 0) al.add("Buzz");
            else al.add(i + "");
        }
        return al;
    }
}
```

여유가 있어서 조금 모듈화한 게 FizzBuzzRule2다.

```java
class FizzBuzzRule2 {
    public List<String> generate(int number) {
        List<String> al = new ArrayList<>();
        for (int i = 1; i <= number; i++) {
            String s = toWord(i, 3, "Fizz") + toWord(i, 5, "Buzz");
            al.add(s.equals("") ? Integer.toString(i) : s);
        }
        return al;
    }

    public String toWord(int divisor, int value, String word) {
        if (divisor % value == 0) return word;
        return "";
    }
}
```

15를 따로 검사하지 않아도 "Fizz" + "Buzz"가 붙어서 "FizzBuzz"가 된다. 여기서 끝인 줄 알았는데, 고객이 단서를 붙인다. 3이 4가 될 수도 있고, 7이나 9로 나뉘는 규칙이 추가될 수도 있고, 자기가 간단한 테스트 프로그램 정도는 짤 수 있으니 쉽게 바꿔 끼울 수 있게 해달라고.

그래서 규칙 하나하나를 객체로 만들고, FizzBuzz는 규칙 목록을 밖에서 주입받게 바꾼다.

```java
public interface FizzBuzzRule {
    String apply(int number);
}

public class FizzRule implements FizzBuzzRule {
    public String apply(int number) { return number % 3 == 0 ? "Fizz" : ""; }
}
public class BuzzRule implements FizzBuzzRule { ... }   // 5의 배수에 "Buzz"
public class BangRule implements FizzBuzzRule { ... }   // 7의 배수에 "Bang"

public class FizzBuzz {
    private final List<FizzBuzzRule> rules;

    public FizzBuzz(List<FizzBuzzRule> rules) {   // 규칙은 밖에서 받는다
        this.rules = rules;
    }

    public List<String> generate(int limit) {
        List<String> result = new ArrayList<>();
        for (int i = 1; i <= limit; i++) {
            StringBuilder sb = new StringBuilder();
            for (FizzBuzzRule rule : rules) {
                sb.append(rule.apply(i));
            }
            result.add(sb.length() == 0 ? Integer.toString(i) : sb.toString());
        }
        return result;
    }
}
```

노션 코드는 `List`를 raw 타입으로 써서 경고가 나길래, 3주차에 배운 대로 `List<FizzBuzzRule>`로 타입을 붙여서 돌려봤다. 규칙 목록만 바꿔서 세 번 실행한 결과다.

```text
[Fizz, Buzz]        → 1, 2, Fizz, 4, Buzz, Fizz, 7, 8, Fizz, Buzz, 11, Fizz, 13, 14, FizzBuzz
[Fizz, Buzz, Bang]  → ..., Fizz, Bang, 8, ..., 13, Bang, FizzBuzz, ..., Buzz, FizzBang
[4의 배수(람다), Buzz] → 1, 2, 3, Fizz, Buzz, 6, 7, Fizz, ..., 19, FizzBuzz
```

7을 추가할 때도, 3을 4로 바꿀 때도 FizzBuzz 클래스는 한 줄도 안 고쳤다. 마지막은 `FizzBuzzRule`이 메소드 하나짜리 인터페이스라서 `n -> n % 4 == 0 ? "Fizz" : ""`처럼 람다로 바로 넣었다. 21은 3과 7의 배수라 "FizzBang"이 나온다.

파일 개수는 훨씬 많아졌다. 대신 "규칙은 고객 너 마음대로 만들어서 추가해! 나한테 전화하지 말고~"라는 노션 주석처럼, 개발자가 고객 주문에 시달릴 일이 없어진다. 리팩토링은 결국 나를 위한 일이라는 말이 이 예제에서 와닿았다.

하나 더 눈에 띈 게 있다. FizzBuzzRule2의 `toWord(int divisor, int value, ...)`는 `toWord(i, 3, "Fizz")`로 부르니까 실제로는 `divisor`에 나눠지는 수(i)가, `value`에 나누는 수(3)가 들어간다. 이름과 역할이 반대다. 코드는 맞게 돌아가지만 읽는 사람은 헷갈린다. 개발자 설문에서 AI가 있어도 가장 어려운 일이 "이름 짓기"였다고 하셨는데, 이런 걸 두고 하신 말 같다.

## 오버라이딩으로 제어를 단순하게

노션의 "메소드 오버라이딩을 적용한 제어의 단순화" 예제도 같은 흐름이다. Hero를 상속한 Warrior, Archer, Wizard가 있고, 모든 히어로는 주먹을 한 번 지르고 각자 고유 공격을 한다.

```java
// Before: main이 타입을 하나하나 물어보고 캐스팅한다
for (int i = 0; i < heros.length; i++) {
    heros[i].attack();
    if (heros[i] instanceof Warrior) {
        ((Warrior) heros[i]).groundCutting();
    } else if (heros[i] instanceof Archer) {
        ((Archer) heros[i]).fireArrow();
    } else if (heros[i] instanceof Wizard) {
        ((Wizard) heros[i]).freezing();
    }
}
```

각 하위 클래스가 `attack()`을 오버라이드해서 `super.attack()` 다음에 자기 공격을 부르게 하면 main이 이렇게 된다.

```java
// After
for (int i = 0; i < heros.length; i++) {
    heros[i].attack();
}
```

새 히어로를 추가해도 main은 안 고친다(OCP). 각 하위 클래스가 Hero 자리에 문제없이 들어간다(LSP). [Vehicle-Driver 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-04-vehicle-driver %})에서 "instanceof로 타입을 하나하나 비교할 필요가 없었다"고 했던 것과 같은 이야기인데, 이번에는 "누가 판단하느냐"가 더 잘 보였다. main이 판단하던 걸 객체가 스스로 하게 넘겼다. 노션의 "잘 만든 클래스는 사용할 때 좋다"는 말이 이거다.

## if문을 줄이는 다섯 단계

노션의 "클린코드 연습: If문을 줄이자!"가 같은 이야기를 단계별로 정리해둔 문서다. 문서 첫 줄이 "수많은 if문은 개발자의 멘탈을 붕괴시킨다"였다. 처음엔 카카오페이 하나였는데 네이버페이, 토스페이가 붙으면서 else if가 복붙되고, 코드가 오른쪽으로 피라미드처럼 쌓여가는 상황을 고쳐나간다. 1~3단계가 기본이고, 4~5단계는 SOLID와 스프링까지 알아야 이해되는 심화다. SOLID를 정리한 뒤에 다시 읽으니 4~5단계도 무슨 말인지 보였다. 중첩된 if는 "멋진 거 아니에요"라고.

### 1단계: 보호 구문(Early Return)

조건 검사가 `if { if { if { ... } } }`로 겹겹이 쌓여서 진짜 하고 싶은 일이 맨 안쪽에 숨어 있을 때 쓴다. 실패 조건을 메소드 입구에서 먼저 검사하고 바로 빠져나간다.

```java
// Before: 사용자 null, 휴면 계정, 금액, 잔액 검사가 네 겹으로 중첩
// After: 부정 조건부터 먼저 검사하고 바로 던진다(Fail Fast)
public void validate(User user, int amount) {
    if (user == null) throw new IllegalArgumentException("사용자 정보가 없습니다.");
    if (!user.isActive()) throw new IllegalStateException("휴면 계정입니다.");
    if (amount <= 0) throw new IllegalArgumentException("결제 금액은 0원보다 커야 합니다.");
    if (user.getBalance() < amount) throw new IllegalStateException("잔액이 부족합니다.");

    System.out.println("결제 검증 성공!");   // 본문이 들여쓰기 없이 맨 아래에 드러난다
}
```

Before에서는 각 else가 어느 if의 짝인지 찾으려면 괄호를 세야 했다. After는 위에서 아래로 읽으면 끝난다. 노션에 "1학년이라면 봐줌"이라고 적혀 있어서 웃었다.

### 2단계: Enum에 값과 행위를 넣기

등급에 따라 할인율을 돌려주는 `if ("VIP"...) return 0.20; else if ("GOLD"...)` 같은 코드는 enum으로 옮긴다.

```java
public enum UserGrade {
    VIP(0.20), GOLD(0.10), SILVER(0.05), BASIC(0.0);

    private final double discountRate;
    UserGrade(double discountRate) { this.discountRate = discountRate; }
    public double getDiscountRate() { return discountRate; }
}

// 호출부는 if 없이 바로 꺼낸다
double rate = grade.getDiscountRate();
```

할인율을 바꿀 때는 enum만 고치면 된다. 그리고 `"GLOD"`처럼 문자열을 잘못 쳐도 컴파일러가 못 잡던 문제가 없어진다. enum 이름을 틀리면 바로 컴파일 에러가 난다. 앞에서 본 마법의 숫자/문자열 스멜을 같이 없앤 셈이다.

### 3단계: Map에 동작을 등록하기

문자열 키에 따라 서로 다른 메소드를 실행하는 if-else if 사슬은 Map으로 바꾼다. 키와 실행할 동작을 미리 넣어두고 꺼내 쓰는 방식이다. 노션에 "2학년이라면 최소한 이렇게"라고 적혀 있다.

```java
// Ver.1: 인터페이스 + 익명 클래스
public interface PayAction { void execute(int amount); }

payActions.put("KAKAO", new PayAction() {
    @Override
    public void execute(int amount) { payWithKakao(amount); }
});

// Ver.2: 람다와 메소드 참조
private final Map<String, Consumer<Integer>> payActions = Map.of(
        "KAKAO", this::payWithKakao,
        "NAVER", this::payWithNaver,
        "TOSS", this::payWithToss
);

public void execute(String payType, int amount) {
    Consumer<Integer> action = payActions.get(payType.toUpperCase());
    if (action == null) throw new IllegalArgumentException("지원하지 않는 결제 수단: " + payType);
    action.accept(amount);   // if-else 사슬 없이 꺼내서 실행
}
```

Ver.1에서 Ver.2로 가는 과정이 2주차 [람다식 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-11-lambda-expression %})에서 익명 클래스를 람다로 줄였던 것과 비슷했다. `Consumer<Integer>`는 2주차 Battle 리팩토링에서 BattleLog 리스너로 직접 써봤던 함수형 인터페이스고, forEach가 받는 것도 사실 Consumer다. 배운 것들이 이렇게 쓰이는구나 싶었다.

### 4단계: 전략 패턴

분기마다 실행할 로직이 100줄짜리 API 통신처럼 크고 복잡하면 Map에 메소드 하나 넣는 걸로는 부족하다. 그럴 때는 `PaymentStrategy` 인터페이스를 만들고 결제사마다 구현 클래스를 따로 둔다. 서비스는 `strategy.pay(amount)`만 부른다. 다음 글의 OCP 예제와 같은 구조이고, 다음 주에 배울 전략 패턴이다.

### 5단계: 스프링이 전략을 모아서 넣어주기

전략 패턴을 써도 "어떤 전략 객체를 만들지" 고르는 팩토리에 다시 if나 switch가 생긴다. 스프링에서는 구현체마다 `@Component("kakaoPaymentStrategy")`처럼 이름을 붙여두면, 서비스 생성자의 `Map<String, PaymentStrategy>`에 스프링이 모든 구현체를 알아서 모아 넣어준다. 애플페이가 생기면 `ApplePaymentStrategy` 클래스 하나만 추가하면 되고, 서비스 코드는 0줄 수정이다. 교수님께서 실무에서는 거의 이 스타일을 쓴다고 하셨다.

다섯 단계를 다 써야 하는 게 아니라, 상황에 맞게 고르는 거다. 바로 다음에 나오는 "충돌" 이야기와도 이어진다.

## 클린코드와 리팩토링이 부딪힐 때

여기서부터가 제일 재밌었다. 둘 다 품질을 높이자는 건데, 클린코드는 한눈에 읽히는 단순함을 보고, 리팩토링에서 쓰는 설계 기법은 나중에 바꾸기 쉽게 추상화를 넣는다. 그런데 추상화를 넣으면 구조가 복잡해진다. 그래서 둘이 부딪힐 수밖에 없다고 하셨다.

| 상황 | 클린코드 쪽 | 리팩토링/설계 쪽 | 딜레마 |
| --- | --- | --- | --- |
| 분기문 | if-else 3~4줄로 한눈에 | 전략 패턴으로 인터페이스와 구현 클래스 분리 | 파일이 늘어서 여러 클래스를 점프하며 읽어야 함 |
| 메소드 분리 | 긴 메소드 하나에 절차가 다 보임 | 3~5줄 단위로 쪼개서 SRP | 호출 깊이가 깊어져 전체 맥락 파악이 어려움 |
| 추상화 수준 | `ArrayList`, `MySQLRepository`를 바로 써서 실체가 명확 | 인터페이스를 두고 DI로 주입(DIP) | 실제로 뭐가 실행되는지 IDE로 추적해야 함 |

노션 충돌사례도 비슷했다. 중복을 없애려고 `if (user.isAdmin() || user.isGuest())`로 합쳤더니 조건의 의도가 흐려지는 경우, 클래스를 OrderValidator, PaymentService, EmailNotifier로 나눴더니 이걸 묶어주는 OrderService가 또 필요해지는 경우 같은 것들이다. 판단 기준은 "이 코드를 읽는 사람이 명확하게 이해할 수 있는가?"였다.

교수님 비유가 와닿았다. 생일파티 같은 특별한 날이면 촛불 켜고 그릇도 예쁘게 담는다. 그런데 둘이서 간식 먹는데 접시 50개를 꺼내서 예쁘게 닦을 필요는 없다. 혼자 밥 먹으면서 그릇 100개 깔아놓고 회전초밥처럼 먹지도 않는다. 큰 접시 하나에 담아 먹어도 괜찮다. 한 번 쓰고 버릴 코드에 패턴을 잔뜩 넣는 건 **오버 엔지니어링**이고, 그 자체가 안티패턴이다.

### 현업에서는 어떻게 판단할까

현업은 **변경 가능성**과 **협업**을 기준으로 삼는다고 하셨다.

- 변경 가능성: 이 부분이 정말 자주 바뀌는가. 계약서를 써도 위약금 물고 바꾸는 게 현실이라고 하셨다
- 협업: 사람 수가 아니라 팀원들의 이해도가 기준이다. 다 주니어면 패턴부터 공부해야 하고, 다 시니어면 바로 간다. 그래서 리드가 균형을 잡아야 한다

리팩토링한 코드가 오히려 더 많은 설명을 필요로 하면, 짧고 명확한 코드로 되돌리는 경우도 있다고 한다.

노션에 정리된 실무 기준 세 가지는 이렇다.

1. **Rule of Three**: 두 번 반복되거나 분기가 2~3개일 때는 단순한 코드(if-else, 약간의 중복)를 유지한다. 같은 패턴이 세 번째 나오거나 실제로 변경 요구가 생겼을 때 비로소 패턴을 적용한다
2. **YAGNI(You Aren't Gonna Need It)**: "나중에 필요할지도 몰라서" 미리 추상화 계층을 만들지 않는다. 막연한 확장성보다 지금 당장의 가독성이 더 가치 있다
3. **도메인 복잡도에 맞추기**: 단순 CRUD나 거의 안 바뀌는 설정 로직은 직관적인 코드로, 결제 수단이나 할인율, 등급 판정처럼 정책이 수시로 바뀌는 핵심 로직은 패턴과 추상화로

FizzBuzz 예제에 대입해보니 이해가 됐다. 의뢰가 "3, 5로 끝"이었다면 Rule1으로 충분하다. 고객이 규칙을 계속 바꾸겠다고 한 순간부터 Rule 인터페이스가 의미를 가진다.

교수님께서 "SOLID끼리도 충돌한다. 다 적용하려는 강박을 버리고 상황에 맞는 것부터 하나씩"이라고 하셨다. 리팩토링은 정답이 없는 예술이고, 철학끼리 충돌할 수도 있다고.

## 코드 컨벤션과 도구

클린코드의 가장 기본은 팀이 같은 스타일로 쓰는 것이다. 노션 코드 컨벤션 문서에서 기억해둘 것만 적는다.

- 클래스는 PascalCase(`KeyHandler`), 메소드와 변수는 camelCase(`startGameThread()`), 상수는 SNAKE_CASE(`SCREEN_COLUMN`)
- `gp`, `keyH`, `tileM` 같은 의미 없는 축약은 쓰지 않는다. `gamePanel`, `keyHandler`처럼 풀어 쓴다
- 한 줄짜리 if라도 중괄호를 생략하지 않는다
- `for(int i=0;i<4;i++)` 대신 `for (int i = 0; i < 4; i++)`처럼 연산자와 쉼표 뒤에 공백

코드 스멜을 자동으로 찾아주는 도구도 있다. 이클립스용으로는 JDeodorant(구조적 스멜 탐지와 리팩토링 제안)와 SonarLint(실시간 품질 피드백), IntelliJ는 SonarQube와 붙여서 프로젝트 전체 품질까지 본다고 노션에 정리돼 있다. 현업에서는 내 코드가 몇 점인지까지 다 나온다고 하셨다. 커피메이커 V.2나 Battle을 리팩토링할 때 설치해서 내 코드가 어떻게 나오는지 돌려볼 생각이다.

## 정리

- 코드 스멜은 코드를 경직되고, 깨지기 쉽고, 재사용하기 어렵게 만든다
- 클린코드는 처음부터 잘 쓰자는 방향이고, 리팩토링은 동작을 유지한 채 구조를 고치는 도구다
- 리팩토링의 목적은 예쁜 코드가 아니라 미래의 변경에 대응하는 것이다
- 둘은 자주 부딪힌다. 변경 가능성과 팀의 이해도를 보고 판단하고, 필요할 때까지는 단순하게 둔다

"코드는 한 번 작성하고 여러 번 본다"는 말을 수요일 수업에서 두 번 들었고, 슬라이드와 노션에도 계속 나온다. 이번 주에 본 건 결국 나중에 이 코드를 다시 열 사람, 그러니까 미래의 나를 위한 거였다. 다음 글에서는 이 방향을 구체적인 원칙으로 정리한 SOLID를 다룬다.
