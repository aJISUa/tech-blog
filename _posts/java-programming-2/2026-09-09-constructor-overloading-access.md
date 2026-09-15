---
title: "[자프실2] 생성자 호출 순서, 오버로딩과 오버라이딩, 접근 지정자"
date: 2026-09-09 20:10:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - OOP
  - 생성자
  - 오버로딩
  - 오버라이딩
  - 접근지정자
excerpt: "생성자가 왜 부모부터 실행되는지, 오버로딩에서 리턴 타입이 왜 상관없는지, 접근 지정자 4가지를 정리했다."
toc: true
toc_sticky: true
---

## 생성자

사람은 태어날 때 아무것도 없이 태어나지만, 태어날 때부터 이름이나 핸드폰을 갖고 태어나게 할 수 있다면? 이렇게 객체를 만들 때 처음 값을 넣어주는 게 생성자다.

- 이름이 클래스 이름이랑 같고, 리턴 타입이 없다
- `new`로 객체를 만들 때 딱 한 번 반드시 호출된다
- 파라미터를 다르게 해서 여러 개 만들 수 있다 (생성자 오버로딩)
- 생성자는 상속되지 않는다

### 기본 생성자

생성자를 하나도 안 만들면 컴파일러가 `public Circle() {}` 같은 기본 생성자를 알아서 넣어준다. 근데 내가 생성자를 하나라도 만들면 기본 생성자는 자동으로 안 생긴다. 그래서 기본 생성자는 그냥 써두는 게 좋다고 하셨다.

```java
public class Book {
    String title;
    String author;

    public Book() { }                        // 기본 생성자는 써두기
    public Book(String t) {                  // 제목만 받을 때
        this.title = t;
        this.author = "작자미상";
    }
    public Book(String t, String a) {        // 제목이랑 저자 둘 다 받을 때
        this.title = t;
        this.author = a;
    }
}
```

생성자도 이클립스가 만들어준다. `Source → Generate Constructor using Fields`에서 필요한 필드만 체크하면 된다. 자동으로 `super()`가 같이 들어가는데 필요 없으면 지우면 된다.

### 호출 순서와 실행 순서

이게 이번 파트에서 제일 중요했다. A를 상속한 B, B를 상속한 C가 있을 때 `new C()`를 하면 뭐가 먼저 찍힐까?

```java
class A { A() { System.out.println("생성자 A"); } }
class B extends A { B() { System.out.println("생성자 B"); } }
class C extends B { C() { System.out.println("생성자 C"); } }

new C();
```

```text
생성자 A
생성자 B
생성자 C
```

C, B, A 순서가 아니다.

호출은 C부터 한다. C를 만들려고 하니까. 그런데 C를 만들려면 B가 있어야 하고, B를 만들려면 A가 있어야 한다. 생성자 사이사이에 `super()`가 생략돼 있어서 계속 위로 올라가는 거다. 더 올라갈 데가 없으면 A부터 실행하고, 그다음 B, 그다음 C가 실행된다.

사실 부모가 없는 클래스도 생성자 첫 줄에는 `super()`가 숨어 있다. 모든 클래스는 `Object`를 상속하고 있어서 지금까지 에러가 안 났던 것뿐이다.

### 이건 왜 에러일까

```java
class A {
    A(int x) { System.out.println("생성자 A"); }   // 기본 생성자가 없다
}
class B extends A {
    B() { System.out.println("생성자 B"); }       // 여기 super()가 숨어 있다
}
```

`B()` 안에 숨어 있는 `super()`는 A의 기본 생성자를 찾으러 간다. 근데 A에는 `A(int x)`밖에 없어서 에러가 난다.

```text
Implicit super constructor A() is undefined. Must explicitly invoke another constructor
```

고치는 방법은 두 가지다.

1. A에 기본 생성자를 추가한다.
2. B에서 `super(10);`처럼 인자가 있는 생성자를 직접 부른다. 이때 `super(...)`는 꼭 생성자 첫 줄에 있어야 한다.

## 메소드 오버로딩

같은 클래스 안에 이름이 같은 메소드를 여러 개 두는 거다. `getSum1`, `getSum2`처럼 이름을 나누지 않고 그냥 `getSum`으로 쓴다. 대신 컴파일러가 구분할 수는 있어야 한다.

구분하는 기준은 파라미터의 개수, 타입, 순서다. 리턴 타입은 기준에 안 들어간다.

```java
// 오버로딩 실패: 파라미터가 똑같다
public int getSum(int i, int j) { return i + j; }
public double getSum(int i, int j) { return (double)(i + j); }
```

"리턴 타입이 다르니까 구분할 수 있지 않나?" 싶었는데 안 된다. `getSum(1, 2)`라고 부르는 쪽만 보면 둘 중 뭘 불러야 할지 알 수가 없으니까.

### 퀴즈: plus(10, 3.3)의 결과는?

```java
static int plus(int x, int y) { return x + y; }
static double plus(double x, double y) { return x + y; }

public static void main(String[] args) {
    System.out.println(plus(10, 3.3));
}
```

1. `plus(정수, 정수)`로 계산해서 13
2. `plus(실수, 실수)`로 계산해서 13.3
3. 딱 맞는 메소드가 없어서 에러

정답은 2번, 13.3이다.

요즘 언어들은 유연하게 동작하려고 자동 형변환이 가능하면 최대한 해준다. 3.3을 int로 바꾸면 3이 돼서 값이 날아가니까 안 되고, 10을 double로 바꾸면 10.0이라 손실이 없으니까 된다. 그래서 `plus(double, double)`로 찾아간다.

두 번째 문제는 여기에 `double plus(int x, int b)`를 추가할 수 있냐는 거였다. 이것도 안 된다. 이미 `int plus(int, int)`가 있고 리턴 타입은 구분 기준이 아니니까.

교수님이 이 문제를 GPT한테 풀게 했더니 1번에서 "컴파일 에러가 난다"고 답했다고 한다. 잘 실행된다고 알려주니까 그제야 "죄송합니다" 하더라고. 내가 모르는 상태에서 물어보면 이런 틀린 답을 걸러낼 수가 없으니까, 따로 찾아보고 직접 돌려보거나 교수님한테 물어보라고 하셨다.

지난 학기 DB 수업에서도 AI가 추천한 프레임워크를 쓰겠다고 한 팀이 있었는데, 알고 보니 Spring Boot가 그걸 더 편하게 만든 거였다고 한다. 잘못된 방향으로 가서 며칠을 날렸다고.

## 메소드 오버라이딩

부모한테 물려받은 메소드를 자식이 다시 정의하는 거다. 아예 무시할 수도 있고, 덮어쓸 수도 있고, 기능을 더 붙일 수도 있다.

### 조건

- 부모 메소드랑 이름, 파라미터(개수, 타입, 순서), 리턴 타입까지 전부 같아야 한다. 하나라도 다르면 오버라이딩이 아니라 그냥 오버로딩이 된다.
- 접근 제한을 부모보다 더 좁힐 수 없다. 부모가 `public`인데 자식이 `private`으로 바꾸는 건 안 되고, 반대로 부모가 default인데 자식이 `public`으로 넓히는 건 된다.
- 부모에 없던 예외를 `throws`로 새로 붙이는 것도 안 된다.

### 오버라이딩하면

부모 메소드는 가려지고 자식이 다시 만든 메소드가 실행된다. 부모 원래 메소드를 쓰고 싶으면 `super.work()`처럼 `super`를 붙인다. `super`는 부모, `this`는 나 자신이다.

## 접근 지정자 4가지

밖에서 함부로 못 쓰게 막아주는 장치다. 특별한 이유 없으면 `private`을 쓰면 된다.

| 지정자 | 같은 클래스 | 같은 패키지 | 다른 패키지의 자식 | 어디서나 |
| --- | :---: | :---: | :---: | :---: |
| `private` | O | X | X | X |
| default (안 씀) | O | O | X | X |
| `protected` | O | O | O | X |
| `public` | O | O | O | O |

- private: 같은 클래스 안에서만 쓸 수 있다. 자식도 못 본다.
- default: 아무것도 안 붙였을 때다. 같은 패키지 안에서만 public처럼 열린다. 편해서 쓰긴 하는데 되도록 `private`을 붙이자.
- protected: 다른 패키지에 있어도 상속 관계면 쓸 수 있다. 상속 전용이라고 생각하면 된다.
- public: 어디서든 쓸 수 있다.

패키지 3개쯤 만들어두고 왔다 갔다 하면서 직접 접근해본 적이 없으면 꼭 해보라고 하셨다. 표로 외우는 거랑 직접 해보는 건 다르다고.

### getter, setter

내 필드라도 직접 건드리기보다 "이거 넣어줘", "이거 가져다 줘"처럼 메소드로 부탁하는 방식을 쓰라고 하셨다.

```java
private int money;

public int getMoney() { return money; }            // getter: 값 가져오기
public void setMoney(int value) { money = value; } // setter: 값 넣기
```

앞으로 중요한 변수는 `private`으로 두고 getter, setter는 이클립스로 자동 생성해서 쓰자.

## 정리

- 생성자는 호출은 자식부터, 실행은 부모부터. 숨어 있는 `super()`가 부모의 기본 생성자를 찾으니까 기본 생성자는 써두자
- 오버로딩은 파라미터로만 구분하고 리턴 타입은 상관없다. 값이 안 날아가는 방향으로 자동 형변환이 된다
- 오버라이딩은 시그니처가 완전히 같아야 하고, 접근 범위는 넓히는 것만 된다
- 접근 범위는 private, default, protected, public 순으로 넓어진다
- 답이 헷갈리면 직접 돌려보는 게 제일 확실하다
