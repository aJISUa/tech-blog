---
title: "[자프실2] 람다식 정리"
date: 2026-09-11 20:10:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - 람다식
  - 함수형인터페이스
  - Stream
excerpt: "이름 없는 함수를 짧게 쓰는 람다식 문법과, 익명 클래스로 쓸 때랑 비교해서 뭐가 편한지 정리했다."
toc: true
toc_sticky: true
---

## 람다식을 왜 지금 배우나

람다식은 알면 편하고 모르면 불편하다고 하셨다. AI한테 코드를 짜달라고 하면 거의 무조건 람다식을 쓰는데, 그 이유를 알아두라고.

지난번 배틀 프로젝트 얘기도 해주셨다. 제출한 코드가 전부 화살표투성이였는데, 본인이 공부해서 쓴 게 아니라 AI 코드를 가져온 거라 뭔지 몰라서 못 고쳤다고. 그러면 잘못 쓴 거라고 하셨다.

AI가 준 코드를 써도 되긴 한다. 다만 내가 무슨 뜻인지 알고 고칠 수 있을 때만. 아직 잘 모르겠으면 "람다식은 쓰지 마, 아직 몰라. 대신 쉬운 건 써도 돼"라고 요청하거나, "이 코드 람다식 빼고 다시 만들어줘"라고 해서 둘을 비교해보면서 익숙해지라고 하셨다.

## 람다식이 뭔가

람다식은 이름 없는 함수(익명 함수)를 짧게 쓰는 문법이다.

```java
(매개변수) -> { 실행문 }
```

함수 이름을 안 쓴다. `요리하기`처럼 이름을 붙이지 않고 괄호만 쓰고, 가운데 화살표 `->`가 들어간다. 처음 보면 좀 이상하게 생겼다. 파이썬의 `lambda`에서 온 개념이라고 한다.

- 함수형 인터페이스랑 같이 쓴다
- 코드가 짧아지고 한눈에 들어온다. 뭔지 모르면 오히려 읽기 어렵다고 느끼겠지만 알고 나면 훨씬 편하다고 하셨다
- 한 번만 쓰는 동작, GUI 이벤트 처리, Stream API에서 거의 항상 쓴다

### 인자 개수에 따라 쓰는 법

| 인자 | 쓰는 법 |
| --- | --- |
| 없을 때 | `() -> { ... }` |
| 1개 | `(x) -> { ... }` 또는 괄호 빼고 `x -> { ... }` |
| 2개 이상 | `(x, y) -> { ... }` (괄호 못 뺀다) |

인자 이름은 `x`든 `a`든 `data`든 상관없다.

## 람다식으로 메소드 만들고 불러보기

순서는 이렇다.

1. 인터페이스에 메소드 틀만 만들어둔다 (추상 메소드)
2. 람다식으로 그 메소드를 구현한다
3. 메소드를 호출한다

```java
interface Hello {
    void sayHello();            // 인자 0개
}
interface Square {
    int calc(int x);            // 인자 1개
}
interface Calculator {
    int operate(int a, int b);  // 인자 2개
}

public class LambdaTest {
    public static void main(String[] args) {
        Hello h = () -> System.out.println("안녕하세요!");   // 구현
        h.sayHello();                                         // 호출하면 "안녕하세요!"

        Square s = x -> x * x;
        System.out.println(s.calc(5));                        // 25

        Calculator add = (a, b) -> a + b;
        Calculator multiply = (a, b) -> a * b;
        System.out.println("3 + 2 = " + add.operate(3, 2));       // 5
        System.out.println("3 * 2 = " + multiply.operate(3, 2));  // 6
    }
}
```

직접 쳐보니까 신기한 점이 몇 개 있었다.

원래라면 `Hello h = new Hello...` 이런 식으로 써야 할 것 같은데, 블록 하나만 붙였는데도 돌아간다. 메소드를 구현하려면 객체가 있어야 하는데, 람다식이 알아서 객체를 만들고 메소드를 구현해서 `h`가 가리키게 해준다.

`x -> x * x`에서 `x`가 int인지 double인지 정하지도 않았다. 인자가 하나인 메소드를 찾아서 그걸 구현하는 코드가 되는 거다.

그리고 `Hello`, `Square`는 인터페이스라서 원래 `new`로 못 만든다. 인터페이스는 만들려고 쓰는 게 아니라 참조하려고 쓰는 거니까. 그래서 이름 없는 객체를 만들고, 그 안의 메소드를 구현하고, 바로 불러서 한 번 쓰는 식이 된다.

메소드 이름(`sayHello`, `calc`, `operate`)은 별로 중요하지 않다. 교수님이 일부러 다 다르게 지었다고 하셨다.

원래는 자바 파일 하나에 클래스 하나가 원칙이라 `Hello.java`, `Square.java`, `Calculator.java`를 따로 만드는 게 맞다. 지금은 연습용이라 한 파일 위쪽에 같이 붙여둔 거다. 이렇게 하면 앞에 접근 제한자가 없어서 같은 패키지에서만 쓸 수 있다.

## 익명 클래스랑 비교

### 익명 클래스로 쓰면

```java
Calculator add = new Calculator() {
    @Override
    public int operate(int a, int b) {
        return a + b;
    }
};
System.out.println("3 + 2 = " + add.operate(3, 2));
```

### 람다식으로 쓰면

```java
Calculator add = (a, b) -> a + b;
System.out.println("3 + 2 = " + add.operate(3, 2));
```

익명 클래스 쪽은 `new`, `@Override`, `public int ...` 같은 걸 다 써야 한다. 이런 코드를 보일러플레이트라고 부른다. 반드시 써야 하긴 하는데 실제로 중요한 로직은 없는, 형식적으로 반복되는 부분이다.

개발자들 사이에서 "절차상 필요한 건 알겠는데 이걸 매번 꼭 써야 해?" 하는 불만이 생겼고, 그래서 중요한 동작만 남기는 람다식이 들어오게 됐다고 한다.

## 실제로 자주 보는 모양

GUI를 해봤으면 이미 알게 모르게 써봤을 거라고 하셨다.

### 버튼 클릭 처리

```java
// 익명 클래스
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("버튼 클릭됨");
    }
});

// 람다식
button.addActionListener(e -> System.out.println("버튼 클릭됨"));
```

한두 줄짜리에 로직도 별로 없는 코드는 이렇게 쓸 수밖에 없다.

### 리스트 출력

```java
List<String> names = List.of("짱구", "유리", "철수");

// for문
for (String n : names) {
    System.out.println(n);
}

// 람다식
names.forEach(n -> System.out.println(n));
```

여기서 `n`은 리스트에 들어 있는 값 하나하나다.

### 데이터 걸러내기 (Stream API)

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// for문으로 하나씩 검사
for (int n : numbers) {
    if (n % 2 == 0) System.out.println(n);
}

// 스트림으로 바꾸고, 짝수만 거르고, 출력
numbers.stream()
       .filter(n -> n % 2 == 0)
       .forEach(n -> System.out.println(n));
```

말로 설명하듯이 읽혀서 파이썬이랑 비슷한 느낌이다. 문제가 복잡해질수록 람다식이 더 편해진다고 하셨다. 정렬 예제는 코드가 길어서 노션에 따로 올려두셨다.

## 정리

- 람다식은 `(매개변수) -> { 실행문 }` 모양의 이름 없는 함수다
- 추상 메소드가 하나인 함수형 인터페이스를 한 줄로 구현할 수 있다
- 익명 클래스의 형식적인 코드를 줄이고 중요한 동작만 남긴다
- 이벤트 처리, 리스트 돌리기, Stream API에서 자주 쓴다
- 기본 문법부터 익숙해지고, 누가 준 람다식 코드는 내가 고칠 수 있을 만큼 이해하고 쓰기

## 다음 주 준비

- 다음 주부터 제네릭이랑 컬렉션이다. 여기서부터 진짜 새로운 내용
- 자프실1에서 `List`, `ArrayList`를 안 배웠으면 노션 자료로 미리 예습해두기
- 다음 주부터 학습 기록 제출 시작. 내면 기본 10점, 부실하면 감점, 잘하면 가산점
- 기술 블로그로 낼 거면 공개로 해둬야 한다
