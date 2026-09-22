---
title: "[자프실2] 와일드카드와 PECS: 사과 바구니는 과일 바구니가 아니다"
date: 2026-09-18 20:00:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - 제네릭
  - 와일드카드
  - PECS
  - 불공변성
excerpt: "와일드카드(?)가 왜 필요한지 불공변성과 과일 바구니 사고 실험으로 정리하고, ? extends는 꺼내기, ? super는 넣기라는 PECS 원칙까지 직접 컴파일해보며 확인했다."
toc: true
toc_sticky: true
---

[제네릭 기초 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-16-generic-basics %})에서 이어지는 내용이다. 이번 주에 새로 배우는 건 사실상 와일드카드 하나라고 하셨다. 앞의 제네릭 클래스와 메소드는 지난 학기에 이미 했던 내용이라서.

공지에도 모든 학생이 와일드카드 타입이 왜 나왔는지, 어떻게 쓰는지는 반드시 복습하라고 적혀 있었다. 시험 범위는 와일드카드까지고, PECS는 24학번 이상에게 실습을 권장하되 중간고사 범위에서는 빠진다. 그래도 "PECS 원칙이 있다는 건 알아두라"고 하셔서 같이 정리했다.

교수님께서 시중에 파는 자바 기초 책에는 이 부분이 거의 안 나온다고 하셨다. 현업 백엔드 자바에서는 많이 쓰는 부분인데 책은 기본 문법에서 끝나서, 노션의 "Generic의 와일드카드 사용하기(중요!)" 페이지에 따로 정리해두셨다. 이 글은 슬라이드랑 그 노션 페이지를 같이 보면서 정리했다.

노션 페이지는 `<T>`, `<T extends OOO>` 복습으로 시작한다. `<T extends OOO>`를 쓰는 이유가 한 줄 더 적혀 있었는데, 아무 타입이나 들어오는 걸 막으면 코드 안에서 OOO이 가진 메소드와 필드를 마음 놓고 호출할 수 있어서다. 이건 [제네릭 기초 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-16-generic-basics %})에 같이 적어뒀다.

## 먼저 문제부터: 불공변성

질문 하나로 시작한다.

> 사과가 과일의 자식이면, 사과 바구니도 과일 바구니의 자식일까?

당연히 그럴 것 같은데 아니다. `Apple`이 `Fruit`의 자식이어도 `List<Apple>`은 `List<Fruit>`의 자식이 아니다. 둘은 아무 관계가 없는 다른 타입이다. 이걸 불공변성(Invariance)이라고 한다.

처음엔 단어부터 어려웠는데, 노션에 한자로 풀어둔 게 있었다.

| 글자 | 뜻 |
| --- | --- |
| 不 (아니 불) | 부정 |
| 共 (함께 공) | 같이, 함께 |
| 變 (변할 변) | 변하다 |
| 性 (성품 성) | 성질 |

"함께 변하지 않는 성질"이다. 사과가 과일 쪽으로 올라가도(업캐스팅) 사과 바구니는 과일 바구니 쪽으로 같이 따라 올라가지 않는다는 뜻으로 이해했다.

```java
List<Apple> apples = new ArrayList<>();
List<Fruit> fruits = apples;   // 컴파일 에러
```

이클립스에서는 `Type mismatch: cannot convert from List<Apple> to List<Fruit>`라고 빨간 줄이 생긴다. 이건 [퀴즈 1번]({{ site.baseurl }}{% post_url java-programming-2/2026-09-20-generic-collection-quiz %})에서 직접 캡처해봤다.

동물 리스트랑 강아지 리스트로 설명하셨을 때도 똑같다. 강아지 리스트를 동물 리스트에 그냥 넣을 수 있을 것 같은데 안 된다.

리스트, 셋, 맵은 쓰기 쉬워서 코드를 막 짜다가 상속 관계가 얽히면 "이거 문법적으로 맞는 것 같은데 왜 컴파일이 안 되지?" 하는 상황이 꼭 온다고 하셨다. 문제는 거의 여기서 시작된다.

### 허용하면 무슨 일이 생길까: 과일 바구니 사고 실험

수업 때 한 친구가 "그거 허용하면 안 되나요?"라고 물었다. 허용했다고 가정해보면 바로 문제가 보인다.

```java
List<Apple> appleBasket = new ArrayList<Apple>();  // 1. 사과만 담는 바구니
List<Fruit> fruitBasket = appleBasket;             // 2. 만약 이게 허용된다면? (실제로는 컴파일 에러)
fruitBasket.add(new Pear());                       // 3. 과일 바구니 타입이니까 컴파일러는 배 넣기를 못 막는다
Apple apple = appleBasket.get(0);                  // 4. 사과인 줄 알고 꺼냈는데 배. ClassCastException
```

`fruitBasket`이라는 참조 타입으로 보면 배를 넣는 게 합법이다. 그런데 실제로 가리키는 메모리 객체는 사과만 들어가야 하는 `appleBasket`이다. 과일 밑에 사과도 있고 배도 있으니까 이런 일이 생긴다.

2번이 허용되면 제네릭의 가장 큰 목적인 "컴파일 시점에 타입 오류를 잡아낸다"는 약속이 무너진다. 결국 4번에서 실행 중에 터진다. [제네릭 기초 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-16-generic-basics %})에서 본 Object 박스 문제로 다시 돌아가는 거다. 그래서 자바는 사과 바구니에 배가 섞이는 참사를 막으려고, 아예 2번 줄 자체를 문법 에러로 막는 불공변성을 선택했다.

### 그런데 이러면 불편하다: 딜레마

안전해지긴 했는데 이번엔 이런 메소드를 못 만든다.

```java
public static void print(List<Fruit> basket) {
    for (Fruit f : basket) System.out.println(f);
}

print(appleBasket);   // 에러! List<Apple>은 List<Fruit>가 아니니까
```

사과 바구니든 배 바구니든 과일 바구니면 다 출력해주고 싶은데, 과일을 꺼내서 보여주기만 하는 메소드에 사과 바구니조차 못 넘긴다. 이 유연성의 한계를 풀려고 나온 게 와일드카드다. 자바가 추론할 수 있는 데까지 추론하고, 허용할 수 있는 데까지만 허용하려고 위쪽 경계(extends)랑 아래쪽 경계(super)를 쓴다.

## 와일드카드 `?`의 세 가지 모양

와일드카드는 물음표 `?`로 쓴다. 다른 언어에서 `*`로 "뭐든지"를 나타내는 것과 비슷하다. 제네릭 타입을 매개변수나 리턴 타입으로 쓸 때, 범위에 있는 모든 타입으로 대체할 수 있다는 표시다.

```java
리턴타입 메소드명(제네릭타입<?> 변수) { ... }                  // 제한 없음
리턴타입 메소드명(제네릭타입<? extends Student> 변수) { ... }  // 상위 제한
리턴타입 메소드명(제네릭타입<? super Worker> 변수) { ... }     // 하위 제한
```

| 모양 | 이름 | 올 수 있는 타입 |
| --- | --- | --- |
| `<?>` | Unbounded Wildcards | 모든 클래스, 인터페이스 |
| `<? extends 상위타입>` | Upper Bounded Wildcards | 상위 타입 포함, 그 아래 |
| `<? super 하위타입>` | Lower Bounded Wildcards | 하위 타입 포함, 그 위 |

슬라이드 그림은 Person 밑에 Worker, Student가 있고 Student 밑에 HighStudent, MiddleStudent가 있는 구조였다.

```text
Person
 ├ Worker                 ← ? super Worker   : Worker, Person
 └ Student                ← ? extends Student : Student, HighStudent, MiddleStudent
     ├ HighStudent
     └ MiddleStudent
```

`? super Worker`는 Worker를 포함해서 Person까지, `? extends Student`는 Student를 포함해서 HighStudent, MiddleStudent까지다.

한 친구가 와일드카드가 여러 개를 덮는 용도냐고 물어봤는데, 그게 아니라 상위, 하위, 전부 이 세 가지를 나타내는 거라고 하셨다.

그리고 앞 글에서 본 것처럼 `T extends`는 되는데 `T super`는 없다. 아래쪽을 막고 싶으면 와일드카드를 써야 한다.

## `? extends`는 꺼내기만, `? super`는 넣기만

여기가 제일 헷갈렸던 부분이다.

### `? extends Number`: 넣기가 안 된다

```java
// Number 포함 하위(Double, Integer, Float) 리스트를 참조할 수 있다. 실제로는 Double 리스트
List<? extends Number> list1 = new ArrayList<Double>();

list1.add(10);        // 에러
list1.add(10.123);    // 심지어 Double을 넣어도 에러!
list1.add(null);      // 유일하게 허용되는 건 null

List<? extends Number> list2 = Arrays.asList(1.2, 1.3, 1.4, 10, 20, 30);
System.out.println(list2.get(1));   // 꺼내는 건 된다
```

실제로 만든 게 Double 리스트니까 Double 하나 정도는 넣어줘야 하는 거 아닌가 싶었다. 안 된다.

컴파일러는 참조 변수의 타입(`List<? extends Number>`)만 보고 판단하고, 실제 메모리에 무엇이 만들어졌는지(`new ArrayList<Double>`)는 보지 않는다. `List<? extends Number>`만 보면 이게 Double 리스트인지 Integer 리스트인지 알 수가 없다. 그러니까 뭘 넣든 안 받아준다.

교수님 비유가 딱 와닿았다.

> 문을 열었는데 도둑인지 착한 사람인지 모르겠다. 그러면 안 연다.

애매할 때 자바는 안전하려고 금지를 건다. `null`만 되는 건 null이 "비어 있음"이라서 어떤 타입에도 들어갈 수 있기 때문이다.

대신 꺼내는 건 된다. 뭐가 나오든 Number인 건 확실하니까 `Number n = list2.get(1)`로 받을 수 있다.

### `? super Integer`: 꺼내기가 애매하다

```java
List<? super Integer> list = new ArrayList<Object>();   // Integer의 조상인 Object 리스트 가능
list.add(10);                     // 10은 Integer니까 넣기 가능
Integer num = list.get(0);        // 에러! 꺼낸 값이 Integer라는 보장이 없다
Object obj = list.get(0);         // 오직 Object로만 받을 수 있다
```

이번엔 반대다. Integer의 조상 리스트라는 건 확실하니까 Integer는 어느 쪽이든 넣을 수 있다. 그런데 꺼낼 때는 이게 Integer 리스트인지 Number 리스트인지 Object 리스트인지 모르니까 Object로밖에 못 받는다. 그래서 읽기에는 활용도가 매우 떨어진다.

### 과일 바구니에 적용하면

아까 딜레마였던 출력 메소드를 노션에서는 이렇게 풀었다.

```java
// 조회용: ? extends Fruit. 사과 바구니든 배 바구니든 받는다
public static void printFruits(List<? extends Fruit> basket) {
    for (Fruit f : basket) {         // 꺼내서 과일로 다루는 건 100% 안전
        System.out.println(f);
    }
    // basket.add(new Apple());      // 컴파일 에러! 사과 바구니인지 배 바구니인지 몰라서 쓰기 금지
}

// 추가용: ? super Fruit. 과일 바구니든 더 넓은 Object 바구니든 받는다
public static void addFruits(List<? super Fruit> basket) {
    basket.add(new Apple());         // 안전하게 추가 가능
    basket.add(new Pear());          // 안전하게 추가 가능
    // Fruit f = basket.get(0);      // 컴파일 에러! 꺼낸 게 Fruit인지 그냥 Object인지 모른다
}

List<Fruit> fruitBasket = new ArrayList<>();
addFruits(fruitBasket);
printFruits(fruitBasket);

List<Apple> appleBasket = new ArrayList<>();
appleBasket.add(new Apple());
printFruits(appleBasket);            // 이제 된다
```

`printFruits` 안에서 `(Fruit)`나 `(Apple)`로 위험하게 형변환할 필요가 없다. `?` 자리에 뭐가 들어왔는지는 몰라도 최소한 Fruit이거나 Fruit의 자식이니까 Fruit로 업캐스팅해서 받으면 된다. `addFruits`에서 사과랑 배를 넣을 때도 알아서 업캐스팅된다. 조회용 메소드에는 `? extends`, 추가용 메소드에는 `? super`를 걸면 된다.

## PECS 원칙

이걸 한 줄로 정리한 게 PECS다.

> **P**roducer **E**xtends, **C**onsumer **S**uper

| | `? extends T` | `? super T` |
| --- | --- | --- |
| 역할 | Producer (생산자) | Consumer (소비자) |
| 용도 | 읽기 전용 | 쓰기 전용 |
| 예시 | `List<? extends Number>` | `List<? super Integer>` |
| 읽기(get) | T 타입으로 안전하게 읽기 가능 | Object로만 가능 |
| 쓰기(add) | 불가능 (null만 가능) | T 타입 데이터 쓰기 가능 |
| 입장 | 만들어둔 걸 꺼내 쓰는 쪽 | 값을 받아서 담는 쪽 |

교수님께서는 더 외우기 쉽게 **PEG_CSA**라고 외우라고 하셨다. Producer는 Extends로 Get, Consumer는 Super로 Add.

용어가 좀 헷갈렸는데, "리스트 입장에서" 생각하니까 이해가 됐다. `? extends` 리스트는 나한테 데이터를 생산해서 주는 쪽이라 꺼내기만 하고, `? super` 리스트는 내가 주는 데이터를 소비하는 쪽이라 넣기만 한다.

컬렉션 API가 전부 이 원칙을 따른다. Collection의 `addAll(Collection<? extends E> c)`가 딱 그 모양이다. c에서 꺼내서(Producer) 나한테 넣으니까 extends다.

### Lab: PECS로 과일 복사하기

과일 계층을 `Object > Fruit > Apple > GoldenApple`로 만들고, 한 바구니에서 다른 바구니로 옮기는 `copy` 메소드를 만드는 실습이었다.

```java
// src: 데이터를 꺼내 주는 원본 리스트 (Producer -> Extends)
// dest: 데이터를 받는 목적지 리스트 (Consumer -> Super)
public static <T> void copy(List<? extends T> src, List<? super T> dest) {
    for (T item : src) {     // src에서는 T(또는 그 자식)를 안전하게 꺼낼 수 있다 (get)
        dest.add(item);      // dest에는 T를 안전하게 넣을 수 있다 (add)
    }
}
```

```java
List<GoldenApple> goldenApples = new ArrayList<>();   // 황금사과 바구니
List<Apple> apples = new ArrayList<>();               // 사과 바구니
List<Fruit> fruits = new ArrayList<>();               // 과일 바구니
List<Object> objects = new ArrayList<>();             // 잡동사니 바구니

copy(goldenApples, fruits);   // 황금사과 -> 과일 바구니: OK
copy(apples, objects);        // 사과 -> 최상위(Object) 바구니: OK
// copy(fruits, goldenApples); // 컴파일 에러
```

마지막 줄은 과일(Fruit)을 황금사과 바구니에 담으려는 거라 안 된다. src의 요소는 T이거나 T의 자식이어야 하고 dest의 요소는 T이거나 T의 조상이어야 하는데, 여기서는 오히려 src(과일)가 dest(황금사과)보다 위에 있다. 과일 중에는 배도 있으니까 황금사과 바구니에 넣으면 안 되는 게 맞다. 주석을 풀고 컴파일해보니까 `inference variable T has incompatible bounds`라는 에러가 났다. T가 Fruit보다 아래이면서 동시에 GoldenApple보다 위여야 하는데, 그런 T는 없다는 뜻이다.

## 냉장고 사고 실험

과일 바구니까지 이해했으면 다행이고, 아직 잘 모르겠으면 냉장고 사고 실험을 보라고 노션에 하나 더 있었다. 교수님도 금요일 수업 끝에 일반 냉장고, 김치냉장고 같은 걸 만들어놓고 "이게 왜 안 되지? 이게 왜 되지?"를 마음껏 연습해보라고 하셨다.

```text
Food
 ├ Fruit
 │   ├ Apple
 │   └ Orange
 └ Kimchi
```

사과냉장고, 오렌지냉장고, 과일냉장고, 김치냉장고, 아무거나 다 들어가는 일반냉장고가 있고, 냉장고는 리스트로 만들었다고 가정한다. `Apple`은 `Fruit`의 하위 타입이지만 `List<Apple>`은 `List<Fruit>`나 `List<Food>`의 하위 타입이 아니다. 그래서 사과냉장고의 사과를 과일냉장고나 일반냉장고로 옮기려면 한정적 와일드카드가 필수다.

```java
// src 공급처: 데이터를 꺼내므로 Producer -> extends
// tar 소비처: 데이터를 담으므로 Consumer -> super
public static <T> void moveItems(List<? extends T> src, List<? super T> tar) {
    for (T item : src) {
        tar.add(item);   // 안전하게 꺼내어 안전하게 추가
    }
}

List<Apple> appleBox = ...;       // 사과 1개
List<Orange> orangeBox = ...;     // 오렌지 1개
List<Kimchi> kimchiFridge = ...;  // 김치 1개

// 1) 사과, 오렌지 -> 과일 바구니
List<Fruit> fruitBasket = new ArrayList<>();
moveItems(appleBox, fruitBasket);
moveItems(orangeBox, fruitBasket);
System.out.println("과일 바구니 개수: " + fruitBasket.size());        // 2

// 2) 과일 바구니, 김치 -> 일반 대형 냉장고
List<Food> generalFridge = new ArrayList<>();
moveItems(fruitBasket, generalFridge);
moveItems(kimchiFridge, generalFridge);
System.out.println("일반 냉장고 총 음식 개수: " + generalFridge.size()); // 3

// moveItems(kimchiFridge, fruitBasket);   // 김치를 과일냉장고에? 컴파일 에러
```

노션 코드를 그대로 쳐서 돌려봤더니 2, 3이 나왔다.

`moveItems(appleBox, fruitBasket)`를 T = Fruit로 놓고 보면 이렇게 설명된다.

- 사과냉장고 `appleBox`는 `List<? extends Fruit>`에 맞으니까, 원소를 꺼낼 때 Fruit 타입이라는 게 보장된다.
- 과일냉장고 `fruitBasket`은 `List<? super Fruit>`에 맞으니까, 꺼낸 Fruit를 문제없이 넣을 수 있다.

마지막 줄 주석을 풀어보니 과일 바구니 때랑 같은 `inference variable T has incompatible bounds` 에러가 났다. 김치는 Fruit와 상속 관계가 없으니까 "Kimchi보다 위이면서 Fruit보다 아래"인 T가 없다.

노션에 이런 말이 있었다. 상속 관계에 있는 것들을 컬렉션에 담고 add나 get을 하는데 문법은 맞는 것 같은데 컴파일 에러가 나면, 대부분 이 원칙을 놓친 거라고.

### T는 정확히 뭘로 정해질까

슬라이드 과일 복사 주석에는 `copy(goldenApples, fruits)`에서 T가 Apple로 추론된다고 되어 있고, 노션 냉장고 설명에는 `moveItems(appleBox, fruitBasket)`에서 T가 Fruit로 결정된다고 되어 있다. 퀴즈 예시로도 "컴파일러가 T를 Apple로 추론하는 이유"가 나왔다고 해서, 실제로 컴파일러가 뭘 고르는지 궁금해서 확인해봤다.

메소드가 마지막으로 옮긴 값을 T로 돌려주게 바꾸고, 그 결과를 제일 구체적인 타입 변수에 넣어봤다.

```java
static <T> T moveLast(List<? extends T> src, List<? super T> tar) { ... return last; }

var x = moveLast(appleBox, fruitBasket);
Apple a = x;                                           // 컴파일된다 -> T는 Apple
RefrigeratorExample.<Fruit>moveItems(appleBox, fruitBasket);   // T를 Fruit로 직접 정해줘도 된다
```

내가 쓰는 JDK에서는 T가 Apple로 추론됐다. 황금사과 복사에서도 GoldenApple로 추론됐다. 그런데 T를 Fruit로 직접 정해줘도 컴파일이 된다. 조건이 "Apple ≤ T ≤ Fruit"이라서 그 사이에 있는 타입이면 다 맞고, 따로 정해주지 않으면 컴파일러가 제일 구체적인 쪽(src 쪽)을 고르는 것 같다.

그래서 노션처럼 T를 Fruit로 놓고 설명해도 틀린 건 아니라고 이해했다. 중요한 건 T가 딱 하나로 정해지는 게 아니라 "src 요소 타입보다 위, dest 요소 타입보다 아래"라는 범위만 맞으면 된다는 점이다. 이 부분은 교수님께 한번 여쭤보려고 한다.

### 헤더만 써보기: moveBasket

노션 마지막에 PECS를 쓴 copy 예시가 하나 더 있었다. 이번엔 인자 순서가 `copy(dest, src)`이고, add 대신 `dest.set(i, src.get(i))`로 자리를 채운다. src는 dest로 넘겨줄 데이터를 꺼내니까(get) `? extends T`, dest는 받아서 채우니까(set/add) `? super T`다.

그리고 문제가 하나 있었다. src 바구니에 담긴 과일을 dest 바구니로 모두 옮겨 담는 정적 메소드 `moveBasket`의 헤더만 써보라는 것. 지금까지 한 걸 그대로 적용하면 이렇게 된다.

```java
public static <T> void moveBasket(List<? extends T> src, List<? super T> dest)
```

꺼내는 쪽은 extends, 담는 쪽은 super다.

## Lab#4: 와일드카드로 Box의 put 제한하기

Lab#3에서 만든 `Box<T>`(안에 `ArrayList<T>`를 가진 박스)에 와일드카드로 받는 메소드를 넣는 실습이다.

```java
// T 또는 T의 상위 클래스만 허용
public void put_super(ArrayList<? super T> t) { ... }
// T 또는 T의 하위 클래스만 허용
public void put_sub(ArrayList<? extends T> t) { ... }
```

뱀 박스(`Box<뱀>`)에서 부르면 이렇게 된다. 계층은 `동물 > 뱀 > 방울달린뱀`이고 자동차는 상관없는 클래스다.

| 인자 | `box3.put_super(...)` | `box3.put_sub(...)` |
| --- | --- | --- |
| `ArrayList<뱀>` | O | O |
| `ArrayList<동물>` | O | X |
| `ArrayList<자동차>` | X | X |
| `ArrayList<방울달린뱀>` | X | O |

자동차는 어느 쪽도 안 된다. 상하 관계 자체가 없으니까.

## 정리

- `List<Apple>`은 `List<Fruit>`의 자식이 아니다(불공변성, 함께 변하지 않는 성질). 허용하면 사과 바구니에 배가 들어가서 실행 중에 ClassCastException이 난다
- 그 대신 과일 바구니면 다 받는 출력 메소드를 못 만드는 딜레마가 생긴다. 이걸 풀려고 와일드카드 `?`를 쓴다. `<?>`, `<? extends 상위>`, `<? super 하위>` 세 가지
- 조회용 메소드에는 `? extends`, 추가용 메소드에는 `? super`. 냉장고끼리 음식을 옮길 때도 꺼내는 쪽은 extends, 담는 쪽은 super
- 컴파일러는 참조 변수의 타입만 보고 판단한다. 실제로 뭘 new 했는지는 안 본다
- `? extends`는 꺼내기(get)만, `? super`는 넣기(add)만 안전하다. PECS, 또는 PEG_CSA
- `T extends`는 되지만 `T super`는 없다. 아래쪽 제한은 와일드카드로
- 퀴즈로도 만들어봤다: [3주차 퀴즈 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-20-generic-collection-quiz %})
