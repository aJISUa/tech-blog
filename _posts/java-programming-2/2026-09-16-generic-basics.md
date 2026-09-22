---
title: "[자프실2] 제네릭 기초: 타입을 파라미터로 받는 클래스와 메소드"
date: 2026-09-16 20:00:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - 제네릭
  - Generic
  - 타입안정성
  - 제네릭메소드
excerpt: "3주차부터 새로운 내용이다. Object로 뭐든 담는 박스가 왜 위험한지, 제네릭 클래스와 제네릭 메소드, 제한된 타입 파라미터까지 정리했다."
toc: true
toc_sticky: true
---

3주차부터는 복습이 끝나고 새로운 내용이다. 이번 주 주제는 제네릭(Generic)과 컬렉션(Collection)인데, 이 글은 제네릭 부분이다. 와일드카드는 [와일드카드와 PECS 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-18-wildcard-pecs %})에, List, Set, Map은 [컬렉션 프레임워크 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-18-collection-framework %})에 따로 정리했다.

수업 시작하면서 교수님께서 "제네릭과 컬렉션을 해본 적 없다, 배웠는데 까먹었다" 하는 사람 손 들어보라고 하셨다. 생각보다 많이 들었다. 자바에서 ArrayList를 써봤으면 제네릭을 안 썼을 리가 없는데, 기억이 안 나는 거다. 나도 `List<String>`은 많이 써봤지만 `<T>`가 정확히 뭘 하는 건지 설명하라고 하면 막혔을 것 같다.

이번 주 목표는 제네릭을 완벽하게 이해하는 게 아니라 이 두 가지라고 하셨다.

- 제네릭으로 만든 라이브러리(컬렉션)를 잘 쓰는 것
- API 문서에 `<T>`, `<? extends T>` 같은 게 나와도 쫄지 않고 무슨 뜻인지 읽을 수 있는 것

## 제네릭은 다이소 수납함이다

제네릭(generic)을 우리말로 하면 "일반적인, 포괄적인"이다.

교수님 비유는 다이소 수납함이었다. 다이소에서 수납함을 팔 때 "여기는 볼펜만 넣으세요, 여기는 신발만 넣으세요"라고 써서 팔지 않는다. 그냥 수납함을 팔고, 뭘 넣을지는 사는 사람이 정한다.

제네릭도 똑같다. 개발자는 제일 일반적인 박스를 만들어두고, 쓰는 사람이 "이 박스에는 String을 넣을래"라고 정해서 쓴다. 그렇게 약속해놓고 숫자나 짱구를 넣으려고 하면 컴파일러가 바로 잡아준다.

```java
public class Box<T> {    // 이 박스에는 T 타입을 담아서 써
    private T t;
    public T get() { return t; }
    public void set(T t) { this.t = t; }
}
```

처음 보면 세모 괄호 `<>`가 낯설다. 소괄호는 메소드, 중괄호는 블록인데, 여기에 세모 괄호가 하나 더 붙고 그 안에 `T`가 들어간다.

`T`는 Type의 T다. `A`, `B`, `ABC`로 써도 에러는 안 나지만 개발자들끼리 약속한 이름이 있다.

| 이름 | 뜻 | 어디서 많이 보나 |
| --- | --- | --- |
| `T` | Type | 제네릭 클래스, 메소드 |
| `E` | Element | List, Set 같은 컬렉션 |
| `K`, `V` | Key, Value | Map |

## 왜 쓰나: Object 박스의 문제

제네릭 없이 뭐든 담으려면 최상위 클래스인 `Object`를 쓰면 된다.

```java
class Box {
    private Object data;
    public void set(Object data) { this.data = data; }
    public Object get() { return this.data; }
}

public class Main_Generic {
    public static void main(String[] args) {
        Box s = new Box();          // 어떤 객체든 저장할 수 있는 s
        s.set(1234);                // 정수는 Integer로 저장된다
        s.set("abcde");             // 이번에는 문자열. 제한이 없다

        String p = (String) s.get();    // 꺼낼 때는 형변환 코드가 반드시 필요
        s.set(5678);
        Integer a = (Integer) s.get();
        p = (String) s.get();           // 컴파일은 통과한다. 그러나 곧 죽는다
    }
}
```

Integer, Double, String, 강아지, 고양이, 자동차까지 다 넣을 수 있으니까 좋아 보이는데, 교수님께서 "좋아할 게 아니라니까"라고 하셨다. 문자열 넣었다가 숫자 넣었다가, 꺼내서 Integer로 바꿨다가 다시 String으로 바꿨다가. 이런 동작을 허용하는 것 자체가 정신없는 거다.

마지막 줄이 핵심이다. 박스에는 5678(Integer)이 들어 있는데 String으로 형변환해서 꺼낸다. 문법은 맞으니까 컴파일러는 통과시켜주는데, 실행하다가 `ClassCastException`으로 죽는다. 이런 일이 비일비재하다고 하셨다.

뭐든지 다 넣을 수 있는 박스는 좋은 박스가 아니라 잡동사니 박스다. 신발 바구니, 음식 바구니가 따로 있어야 한다. 냉장고에 신발을 넣으려고 하면 아예 문을 못 열게 막고 에러로 알려주는 게 제네릭이다.

정리하면 제네릭을 쓰는 이유는 두 가지다.

1. **컴파일할 때 타입을 강하게 체크한다.** 클래스 구조가 복잡해질수록 타입을 잘못 캐스팅하는 실수가 생기는데, 이걸 실행 중에 터지게 두지 않고 컴파일 단계에서 에러로 잡는다.
2. **형변환 코드가 없어진다.**

```java
// 제네릭 없이
List list = new ArrayList();
list.add("hello");
String str = (String) list.get(0);   // 형변환 필요

// 제네릭으로
List<String> list = new ArrayList<String>();
list.add("hello");
String str = list.get(0);            // 필요 없다. String만 들어 있다는 게 보장되니까
```

Object를 쓰면 저장된 타입에 따라 형변환이 자주 일어나서 성능도 떨어진다.

## 제네릭 클래스 만들어보기

```java
class BoxG<T> {
    private T data;
    public void set(T data) { this.data = data; }
    public T get() { return this.data; }
}

public class Main_Generic {
    public static void main(String[] args) {
        BoxG<String> s1 = new BoxG<String>();    // 문자열만 저장한다
        BoxG<Integer> s2 = new BoxG<Integer>();  // 정수만 저장한다

        s1.set(123);      // 에러!
        s2.set("abc");    // 에러!
    }
}
```

`BoxG<String>`으로 만들면 클래스 안의 `T`가 전부 String으로 바뀐다고 생각하면 된다. `get()`의 리턴 타입도 String, `set()`의 파라미터 타입도 String이 된다. 그래서 `s1.set(123)`은 빨간 줄이 생긴다.

### int는 못 넣는다

제네릭 타입 자리에는 클래스만 올 수 있다. `int`, `double`, `boolean` 같은 기본 타입은 안 된다. 그래서 `BoxG<int>`가 아니라 래퍼 클래스인 `BoxG<Integer>`로 쓴다.

대신 넣고 꺼낼 때는 자동으로 박싱, 언박싱이 되니까 `Box<Integer> box`에 `box.set(6)`으로 넣고 `int value = box.get()`으로 꺼내는 것처럼 편하게 쓸 수 있다.

### 자바 7부터는 뒤쪽을 비워도 된다

```java
Box<String> box = new Box<String>();   // 옛날에는 앞뒤 다 써야 했다
Box<String> box = new Box<>();         // 자바 7부터는 뒤를 생략 가능
```

앞은 반드시 써야 하고, 뒤는 생략할 수 있다. 대신 `<>` 괄호 자체는 빼면 안 된다. 뒤를 비워두면 컴파일러가 앞을 보고 추론해서 채워준다.

### 제네릭 클래스 안에서 배열은 안 된다

색깔 주사위랑 숫자 주사위를 만든다고 하면 제네릭 없이는 클래스가 두 개 필요하다.

```java
public class ColorDice  { String[] value = new String[10]; }
public class NumberDice { int[] value = new int[10]; }
```

제네릭을 쓰면 `MyDice<T>` 하나면 된다. 그런데 여기서 교수님께서 일부러 주석 처리해두신 줄이 있었다.

```java
public class MyDice<T> {
    // T[] value = new T[10];                 // 안 된다
    T result;                                  // 변수 하나는 괜찮다
    ArrayList<T> value = new ArrayList<T>();   // 그래서 ArrayList를 쓴다
}
```

`T[]`로 선언까지는 되는데 `new T[10]`으로 만들 수가 없다. 직접 해보니까 `generic array creation`이라는 에러가 났다. 그래서 제네릭은 거의 항상 컬렉션이랑 같이 쓰인다고 하셨다. 배열이 익숙하고 쉽다고 제네릭 안에서 배열을 쓰면 안 된다.

주사위는 직접 만들어봤고, [3주차 과제 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-20-week3-assignment %})에 정리했다.

## 멀티 타입 파라미터

타입 파라미터는 두 개 이상도 된다. 콤마로 구분한다.

```java
public class Product<T, M> {
    private T kind;
    private M model;

    public T getKind() { return this.kind; }
    public M getModel() { return this.model; }
    public void setKind(T kind) { this.kind = kind; }
    public void setModel(M model) { this.model = model; }
}

Product<Tv, String> product = new Product<>();   // 앞은 Tv, 뒤는 모델명(String)
```

`T`는 종류, `M`은 모델이다. 이름은 역시 자유다. 나중에 Map을 쓸 때 `Map<K, V>`로 키랑 값의 타입을 따로 정하는 게 바로 이거다.

## 제네릭 메소드

여기서부터 처음 보는 모양이 나온다. 클래스는 제네릭이 아닌데 메소드에만 타입 파라미터가 붙는다.

```java
public <A, B, ...> 리턴타입 메소드명(매개변수, ...) { ... }
//      ↑ 타입 파라미터 정의
```

리턴 타입 앞에 `<>`를 한 번 더 써서 "이 메소드에서는 T를 타입으로 쓸 거다"라고 먼저 알려준다. 제네릭 클래스라면 클래스 이름 옆에 이미 `<T>`가 있으니까 필요 없는데, 제네릭 메소드는 그게 없으니까 여기서 선언해줘야 한다. 이건 문법이라 외워야 한다고 하셨다.

```java
public class Array {
    public static <T> T getLast(T[] a) {
        return a[a.length - 1];
    }
}

String[] language = {"C++", "C#", "JAVA"};
String last = Array.<String>getLast(language);   // last에는 "JAVA"가 저장됨
```

수업 때 한 친구가 "T가 여러 번 나오는데 각각 뭐가 다르냐"고 물어봤다. 뒤에서부터 읽으면 된다고 하셨다.

```java
public static <T> T getLast(T[] a)
//            ①   ② ③       ④
// ④ 인자, ③ 메소드 이름, ② 리턴 타입, ① 이 메소드가 T를 쓰겠다는 타입 파라미터 선언
```

`T`의 범위는 이 메소드 안으로 제한된다.

### T는 컴파일러가 추정한다

```java
public <T> Box<T> boxing(T t) { ... }

Box<Integer> box1 = boxing(100);           // 100을 보고 T를 Integer로
Box<String> box2 = boxing("안녕하세요");     // "안녕하세요"를 보고 T를 String으로
```

`<Integer>`라고 따로 안 써줬는데도 된다. 넘겨준 값을 보고 컴파일러가 컴파일 과정에서 T를 구체적인 타입으로 추정해서 바꿔준다. 앞에서 `new Box<>()`의 뒤쪽을 비워도 되는 것도 같은 원리다.

교수님 말로는 자바는 안 되는 건 딱 막고, 되는 범위 안에서는 최선을 다해서 추정한다고 하셨다.

`Arrays.`까지 치고 자동완성 목록을 보면 `asList(T... a)`처럼 T가 들어간 메소드가 잔뜩 나온다. 제네릭 메소드는 생긴 게 좀 기괴하지만 실제로는 엄청 많이 쓰이고, 자꾸 보면 익숙해진다고 하셨다.

## 제한된 타입 파라미터

`T`에 아무 타입이나 오는 게 아니라, 상속 관계를 이용해서 범위를 제한할 수 있다.

```java
public class Box<T extends 동물> { ... }

Box<동물> box1 = new Box<>();
Box<강아지> box2 = new Box<>();
Box<오리> box3 = new Box<>();
Box<자동차> box4 = new Box<>();   // 불가! 자동차는 동물이 아니다
```

`T extends 동물`은 "동물을 포함해서 동물의 하위 타입만 된다"는 뜻이다. 위쪽을 동물로 막아둔 거라서 상위 타입 제한이라고 한다. 상위 타입 자리에는 클래스뿐 아니라 인터페이스도 올 수 있다.

왜 막을까? 노션 와일드카드 페이지 첫머리에 복습으로 정리돼 있었다. 아무 타입이나 들어오는 걸 막고 특정 클래스의 자식(또는 인터페이스 구현체)으로 제한하면, 코드 안에서 그 클래스가 가진 메소드와 필드를 마음 놓고 안전하게 호출할 수 있다. 그냥 `<T>`면 T가 뭔지 모르니까 Object의 메소드밖에 못 부르는데, `<T extends 동물>`이면 T는 적어도 동물이니까 동물의 메소드를 부를 수 있다.

메소드에도 걸 수 있다. Lab#3 코드에 이런 게 있었다.

```java
public static <T extends 동물> T getAnimal() {
    자동차 a = new 자동차();
    강아지 b = new 강아지();
    뱀 c = new 뱀();
    방울달린뱀 d = new 방울달린뱀();
    return (T) c;    // b, c, d만 가능. a는 문법 에러
}
```

그럼 반대로 "이 타입 위로만" 하는 하위 제한도 될까? 된다. 그런데 그때는 `super`를 써야 하고, `T`로는 못 하고 와일드카드 `?`를 써야 한다. `T super 뭐` 같은 문법은 없다. 이게 다음 글 내용이다.

## 정리

- 제네릭은 타입을 파라미터로 받아서, 일반적인 코드를 여러 타입에 재사용하는 방법이다
- 약속한 타입이 아니면 실행 중에 죽는 게 아니라 컴파일할 때 에러로 잡힌다
- 꺼낼 때 형변환이 필요 없어진다
- 타입 자리에는 클래스만 온다. int 대신 Integer
- 제네릭 타입으로 배열은 만들 수 없어서 ArrayList 같은 컬렉션을 쓴다
- 제네릭 메소드는 리턴 타입 앞에 `<T>`를 붙이고, T는 넘긴 값을 보고 컴파일러가 추정한다
- `T extends 상위타입`으로 위쪽을 제한할 수 있다. 아래쪽 제한은 와일드카드로
