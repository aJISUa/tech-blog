---
title: "[자프실2] 제네릭과 컬렉션 퀴즈: 과일 바구니부터 PrettyPrinter까지"
date: 2026-09-20 20:10:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - 제네릭
  - 와일드카드
  - 컬렉션
  - 퀴즈
excerpt: "3주차부터 시작한 퀴즈 만들기 과제. 와일드카드 컴파일 에러, Arrays.asList의 정체, HashSet 중복 판단, PrettyPrinter 리팩토링까지 네 문제를 만들고 직접 돌려서 풀었다."
toc: true
toc_sticky: true
---

3주차부터 새 내용이 시작되면서 퀴즈 만들기 활동이 생겼다. 중요하다고 생각하는 개념이랑 실습을 엮어서 문제를 내고 직접 풀어보는 거다. 우수 문항은 중간고사에 반영된다고 하셨다.

조건이 있었다.

- 괄호 채우기나 단답형은 안 된다. "여기 들어갈 타입은?" 같은 건 누구한테도 도움이 안 된다고 하셨다
- [짱구 퀴즈]({{ site.baseurl }}{% post_url java-programming-2/2026-09-11-polymorphism-jjanggu %})처럼 이론과 실습을 연결하는 문제
- 예를 들면 코드 실행 결과 예측, 여기서 왜 컴파일 에러가 나는지(또는 왜 안 나는지), 설계나 리팩토링 방안

그래서 이번 주 실습하면서 헷갈렸던 걸 위주로 네 문제를 만들었다. 문제 코드는 전부 `JAVA2` 프로젝트 `quiz` 패키지에 넣고 실제로 컴파일, 실행해서 확인했다. 이론은 [제네릭 기초]({{ site.baseurl }}{% post_url java-programming-2/2026-09-16-generic-basics %}), [와일드카드와 PECS]({{ site.baseurl }}{% post_url java-programming-2/2026-09-18-wildcard-pecs %}), [컬렉션 프레임워크]({{ site.baseurl }}{% post_url java-programming-2/2026-09-18-collection-framework %}) 글에 있다.

| 번호 | 유형 | 연결되는 것 |
| --- | --- | --- |
| Quiz #1 | 컴파일 에러가 나는 줄 찾기 | 불공변성, 와일드카드, PECS |
| Quiz #2 | 실행 결과 예측 | Arrays.asList, 다운캐스팅, 수업 때 난 에러 |
| Quiz #3 | 실행 결과 예측 | HashSet 중복 판단, hashCode/equals, Map put |
| Quiz #4 | 설계와 리팩토링 | Lab#7/#8 PrettyPrinter, 오버로딩 |

## Quiz #1: 과일 바구니에서 컴파일 에러가 나는 줄은?

계층은 `Fruit > Apple > GoldenApple`이고, `Pear`도 `Fruit`의 자식이다.

```java
List<Apple> apples = new ArrayList<>();
apples.add(new Apple());
apples.add(new GoldenApple());          // (A)
List<Fruit> fruits = new ArrayList<>();
fruits.add(new Pear());

List<Fruit> f1 = apples;                // (B)
List<? extends Fruit> f2 = apples;      // (C)
Fruit first = f2.get(0);                // (D)
f2.add(new Apple());                    // (E)
List<? super Apple> f3 = fruits;        // (F)
f3.add(new GoldenApple());              // (G)
Apple a = f3.get(0);                    // (H)
Object o = f3.get(0);                   // (I)
```

1. (A)부터 (I) 중에 컴파일 에러가 나는 줄을 모두 고르고, 이유를 설명하시오.
2. 에러 나는 줄을 지우고 실행하면 `apples`, `fruits`, `first`, `o`는 각각 무엇이 출력될까?

### 풀이

정답은 **(B), (E), (H)** 세 줄이다.

![Quiz #1 컴파일 에러]({{ '/assets/images/java-programming-2/g3-quiz1-error.png' | relative_url }})

세 줄의 주석을 풀면 이클립스에서 16, 19, 22번 줄에 빨간 줄이 생긴다. 그래도 실행은 시켜주는데, 출력은 하나도 없이 `Unresolved compilation problems` Error와 함께 에러 세 개가 한꺼번에 나열된다. 한 줄씩 보면 이렇다.

- **(A) OK.** 황금사과는 사과니까 사과 바구니에 넣을 수 있다. 업캐스팅이다. 요소 하나하나는 상속 관계가 그대로 통한다.
- **(B) 에러.** 사과는 과일의 자식이지만 사과 바구니(`List<Apple>`)는 과일 바구니(`List<Fruit>`)의 자식이 아니다. 불공변성이다. 이게 허용되면 `f1.add(new Pear())`로 사과 바구니에 배가 들어간다.
- **(C) OK.** `? extends Fruit`는 "과일이나 그 자식이 담긴 바구니"라서 사과 바구니를 가리킬 수 있다.
- **(D) OK.** f2에 뭐가 들었는지는 몰라도 전부 과일인 건 확실하니까 Fruit로 꺼낼 수 있다. Producer Extends.
- **(E) 에러.** f2가 사과 바구니인지 배 바구니인지 컴파일러는 모른다. 참조 변수 타입만 보니까. 배 바구니일 수도 있으니 사과도 못 넣는다. 에러 메시지에 `add(capture#2-of ? extends Fruit)`라고 나오는데, capture는 "? extends Fruit인 어떤 한 타입"이라는 뜻이다. 그 어떤 타입이 뭔지 모르니까 Apple을 넣어줄 수 없다.
- **(F) OK.** `? super Apple`은 "사과나 사과의 조상이 담긴 바구니"라서 과일 바구니를 가리킬 수 있다.
- **(G) OK.** f3이 사과 바구니든 과일 바구니든 Object 바구니든, 황금사과는 전부에 들어갈 수 있다. Consumer Super.
- **(H) 에러.** f3에서 꺼낸 게 사과라는 보장이 없다. 에러 메시지도 `cannot convert from capture#4-of ? super Apple to Apple`이다. 실제로 지금 f3 안의 첫 번째는 배다.
- **(I) OK.** 뭐가 나오든 Object인 건 확실하다.

에러 나는 세 줄을 다시 주석 처리하고 실행한 결과는 이렇다.

![Quiz #1 실행 결과]({{ '/assets/images/java-programming-2/g3-quiz1-result.png' | relative_url }})

`fruits`는 [배, 황금사과]다. f3에 넣은 황금사과가 fruits에 들어가 있다. f3과 fruits는 같은 바구니를 가리키고 있었으니까. `o`는 f3의 첫 번째, 즉 배다.

(H)가 왜 막혀야 하는지가 여기서 보인다. `Apple a = f3.get(0)`이 허용됐다면 a에 배가 들어갔을 거다.

### 출제 의도

와일드카드를 공부하면서 제일 헷갈렸던 게 "Double 리스트로 만들었는데 왜 Double도 못 넣냐"였다. 이 문제는 그걸 과일로 바꾼 거다. 핵심은 컴파일러가 오른쪽(`new ...`)이 아니라 왼쪽(참조 변수 타입)만 보고 판단한다는 것. 그리고 요소 하나하나에는 상속이 통하지만(A, G) 바구니끼리는 안 통한다(B).

## Quiz #2: Arrays.asList로 만든 리스트의 정체

수업 때 `(ArrayList<String>) Arrays.asList(str)`로 썼다가 에러가 났던 걸 문제로 만들었다.

```java
String[] str = {"aa", "bd"};

List<String> fixed = Arrays.asList(str);
System.out.println("1) " + fixed + " / " + fixed.getClass().getName());
fixed.set(0, "zz");
System.out.println("2) fixed = " + fixed + ", str[0] = " + str[0]);

try {
    fixed.add("cc");
} catch (UnsupportedOperationException e) {
    System.out.println("3) add 실패: " + e.getClass().getSimpleName());
}

try {
    ArrayList<String> casted = (ArrayList<String>) fixed;
    System.out.println("4) 캐스팅 성공 " + casted);
} catch (ClassCastException e) {
    System.out.println("4) 캐스팅 실패: " + e.getClass().getSimpleName());
}

List<String> copy = new ArrayList<>(Arrays.asList(str));
copy.add("cc");
System.out.println("5) copy = " + copy + " / " + copy.getClass().getName());
```

1. 1)~5) 출력을 예측하시오.
2. 4)번 캐스팅은 컴파일은 될까? 된다면 왜 컴파일러가 못 잡을까?
3. 배열을 "마음대로 늘리고 줄일 수 있는" ArrayList로 바꾸려면 어떻게 써야 할까?

### 풀이

![Quiz #2 실행 결과]({{ '/assets/images/java-programming-2/g3-quiz2-result.png' | relative_url }})

**1) 클래스 이름이 `java.util.Arrays$ArrayList`다.** `java.util.ArrayList`가 아니다. `Arrays` 클래스 안에 들어 있는, 이름만 같은 내부 클래스다. 여기서부터 함정이다.

**2) `str[0]`도 zz로 바뀐다.** 이 리스트는 원래 배열을 복사한 게 아니라 배열을 리스트 모양으로 보여주는 거라서, set으로 바꾸면 원본 배열도 같이 바뀐다. 나도 이건 예상 못 했다.

**3) add는 `UnsupportedOperationException`.** 배열을 그대로 쓰고 있으니까 크기를 바꿀 수 없다. set(바꾸기)은 되고 add, remove(크기 바꾸기)는 안 된다. "고정 크기 리스트"다.

**4) 캐스팅 실패, `ClassCastException`.** 1)에서 봤듯이 진짜 ArrayList가 아니니까.

**5) copy는 `[zz, bd, cc]`이고 진짜 `java.util.ArrayList`다.** `new ArrayList<>(...)`는 내용을 복사해서 새 ArrayList를 만든다. 그래서 add가 된다. 그리고 zz가 들어 있는 건 2)에서 str[0]이 이미 바뀌었기 때문이다.

2번 질문의 답: 컴파일은 된다. `List`를 `ArrayList`로 바꾸는 건 부모 타입에서 자식 타입으로 다운캐스팅하는 거라서, 컴파일러 입장에서는 "실제 객체가 ArrayList일 수도 있겠지" 하고 통과시켜준다. 실제 객체가 뭔지는 실행해봐야 안다. 제네릭을 쓰는 이유가 이런 형변환 에러를 컴파일 단계로 끌어오려는 건데, 내가 괄호로 캐스팅을 써버리면 컴파일러가 그 말을 믿어버린다.

3번 답: `new ArrayList<>(Arrays.asList(str))`. 공지에 올라온 예제가 이 모양이었던 이유다. 그리고 변수 타입도 `ArrayList`보다는 `List`로 받는 게 좋다. 그러면 애초에 캐스팅할 이유가 없다.

### 출제 의도

실행 결과가 에러로 끝나는 코드를 직접 겪어봐서, 왜 그런지 끝까지 확인하고 싶었다. "컴파일러가 잡는 에러"와 "실행해야 나오는 에러"를 구분하는 게 제네릭 단원의 핵심이라고 생각했다.

## Quiz #3: 짱구를 Set에 두 번 넣으면?

```java
class Student {                        // equals, hashCode 없음
    String name;
    Student(String name) { this.name = name; }
    public String toString() { return name; }
}

class Student2 {                       // 이름이 같으면 같은 학생
    String name;
    Student2(String name) { this.name = name; }
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Student2)) return false;
        return name.equals(((Student2) o).name);
    }
    public int hashCode() { return Objects.hash(name); }
    public String toString() { return name; }
}
```

```java
Set<String> names = new HashSet<>();
names.add("짱구");
names.add(new String("짱구"));
System.out.println("1) String Set 크기 = " + names.size());

Set<Student> s1 = new HashSet<>();
s1.add(new Student("짱구"));
s1.add(new Student("짱구"));
System.out.println("2) Student Set 크기 = " + s1.size() + " " + s1);

Set<Student2> s2 = new HashSet<>();
s2.add(new Student2("짱구"));
s2.add(new Student2("짱구"));
System.out.println("3) Student2 Set 크기 = " + s2.size() + " " + s2);

Map<String, Integer> score = new HashMap<>();
score.put("짱구", 30);
score.put("철수", 100);
Integer old = score.put("짱구", 90);
System.out.println("4) Map 크기 = " + score.size() + ", 짱구 = " + score.get("짱구") + ", put이 돌려준 값 = " + old);
```

1. 1)~4)의 출력을 예측하시오.
2. Set은 중복을 허용하지 않는다고 했는데, 2)는 왜 그렇게 나올까?

### 풀이

![Quiz #3 실행 결과]({{ '/assets/images/java-programming-2/g3-quiz3-result.png' | relative_url }})

**1) 크기 1.** `new String("짱구")`는 새 객체라서 `==`로 비교하면 다르지만, String은 문자열이 같으면 같은 객체로 보도록 `hashCode()`와 `equals()`가 재정의돼 있다. 그래서 HashSet은 같은 걸로 보고 안 넣는다.

**2) 크기 2, [짱구, 짱구].** 중복이 들어갔다. Student는 equals와 hashCode를 재정의하지 않았으니까 Object의 기본 버전을 쓴다. 기본 버전은 "같은 메모리에 있는 같은 객체인가"만 본다. `new`를 두 번 했으니 이름이 같아도 다른 객체다. HashSet 입장에서는 짱구가 두 명이다.

**3) 크기 1, [짱구].** 이름 기준으로 equals와 hashCode를 오버라이딩했으니까 같은 학생으로 본다.

**4) 크기 2, 짱구 = 90, put이 돌려준 값 = 30.** 같은 키로 put하면 칸이 늘어나는 게 아니라 값이 대체된다. 그리고 put은 원래 있던 값(30)을 돌려준다. 처음 넣을 때는 원래 값이 없으니 null을 돌려준다.

2번 질문의 답: Set의 "중복"은 내가 생각하는 중복이 아니라 `hashCode()`와 `equals()`가 판단하는 중복이다. 슬라이드 HashSet 쪽에 "hashcode로 동등 객체 여부를 판단한다", HashMap 쪽에 "다른 기준을 사용할 경우 hashCode()와 equals()를 재정의해 동등 객체가 될 조건을 정해야 한다"고 나온 게 이거다. 내가 만든 클래스를 Set에 넣거나 Map의 키로 쓸 거면 둘 다 오버라이딩해야 한다.

### 출제 의도

Set은 중복이 안 된다고 외우기만 하면 2)번을 1개라고 답하기 쉽다. 이론(Set은 중복 불가)이 실제로는 어떤 규칙(hashCode, equals)으로 돌아가는지를 확인하는 문제다.

## Quiz #4: PrettyPrinter 리팩토링

Lab#7 PrettyPrinter를 줄여서 이런 출력 클래스를 만들었다.

```java
class OldPrinter {
    public <E> void show(Set<E> set) { System.out.println("Set " + set); }
    public <E> void show(ArrayList<E> list) { System.out.println("ArrayList " + list); }
}

OldPrinter p = new OldPrinter();
ArrayList<Integer> a = new ArrayList<>(List.of(1, 2));
List<Integer> b = new ArrayList<>(List.of(3, 4));
LinkedList<Integer> c = new LinkedList<>(List.of(5, 6));

p.show(a);      // (1)
p.show(b);      // (2)
p.show(c);      // (3)
p.show(null);   // (4)
```

1. (1)~(4) 중 컴파일되는 것과 안 되는 것을 고르고 이유를 설명하시오.
2. 리스트라면 뭐든 받을 수 있게 OldPrinter를 고쳐보시오.
3. 고친 다음에도 (4)는 여전히 문제일까? 어떻게 해결할 수 있을까?

### 풀이

(1)만 컴파일되고 **(2), (3), (4)는 에러**다.

![Quiz #4 컴파일 에러]({{ '/assets/images/java-programming-2/g3-quiz4-error.png' | relative_url }})

- **(1) OK.** a는 ArrayList 타입이니까 딱 맞는다.
- **(2) 에러.** b도 실제로는 `new ArrayList`로 만들었다. 그런데 변수 타입이 `List`다. 컴파일러는 변수 타입만 보니까 "List를 ArrayList 자리에 넣을 수 없다"고 한다. Quiz #1이랑 같은 원리다.
- **(3) 에러.** LinkedList는 List를 구현했지만 ArrayList의 자식이 아니다. 형제 관계다.
- **(4) 에러, 모호함(ambiguous).** null은 Set 자리에도, ArrayList 자리에도 들어갈 수 있으니까 컴파일러가 둘 중 뭘 불러야 할지 못 고른다.

이클립스는 (2), (3)에 `The method show(Set<E>) in the type OldPrinter is not applicable for the arguments`, (4)에 `The method show(Set<Object>) is ambiguous`라고 알려준다. 같은 코드를 javac로 컴파일해보면 `no suitable method found for show(List<Integer>)` 밑으로 "Set으로도 못 바꾸고 ArrayList로도 못 바꾼다"는 이유가 하나씩 적혀 있어서, 오버로딩된 show를 하나씩 대보고 맞는 게 없다고 하는 과정이 더 잘 보였다.

그리고 이 상태로 실행하면 에러가 없는 (1)도 출력되지 않았다. 콘솔에는 Error만 찍혔다. 에러 줄 세 개를 주석 처리하고 다시 실행해야 (1)의 결과가 나온다.

![Quiz #4 (1)만 남기고 실행한 결과]({{ '/assets/images/java-programming-2/g3-quiz4-result.png' | relative_url }})

ArrayList 버전의 show가 불려서 `ArrayList [1, 2]`가 나온다. b와 c는 이제 안 쓰는 변수라서 노란 경고가 떴다.

**2번 리팩토링:** 파라미터를 구현 클래스가 아니라 인터페이스로 받는다.

```java
class NewPrinter {
    public <E> void show(Set<E> set) { ... }
    public <E> void show(List<E> list) { ... }      // ArrayList<E> -> List<E>
    public <K, V> void show(Map<K, V> map) { ... }
}
```

이러면 (1), (2), (3) 전부 된다. ArrayList, LinkedList, Arrays.asList 결과까지 다 List니까. 컬렉션 글에서 배운 "인터페이스로 참조하고 구현 클래스로 생성한다"를 파라미터에도 적용한 거다. 받는 쪽은 넓게 받아야 쓰는 쪽이 편하다. 내 [Lab#8]({{ site.baseurl }}{% post_url java-programming-2/2026-09-20-week3-assignment %})이 이렇게 만든 버전이다.

**3번:** 여전히 문제다. Lab#8에서 직접 해봤더니 show가 Set, Map, List 세 개로 늘었을 뿐, 똑같이 `show(Set<Object>) is ambiguous` 에러가 났다. null은 세 자리 어디에나 들어갈 수 있으니까. 해결 방법은 몇 가지가 있다.

- 타입이 정해진 변수로 넘긴다: `List<Integer> none = null; p.show(none);`
- 캐스팅으로 어느 버전인지 알려준다: `p.show((List<Integer>) null);`
- 오버로딩을 포기하고 이름을 나눈다: `showSet()`, `showList()`, `showMap()`

나는 오버로딩을 쓰는 게 과제 조건이라 첫 번째 방법으로 테스트했다. 대신 메소드 안에서 null을 받았을 때 "(null 컬렉션)"이라고 출력하게 만들어서 Null-Safe 조건을 맞췄다.

### 출제 의도

Lab#7 코드를 읽으면서 "LinkedList를 넣으면 어떻게 되지?"가 궁금해서 만든 문제다. 오버로딩이 편하긴 한데, 파라미터 타입을 좁게 잡으면 못 받는 게 생기고, 너무 비슷한 걸 여러 개 만들면 null 같은 값에서 모호해진다. 코드가 잘 돌아가는 것과 설계가 좋은 건 다르다는 걸 보여주고 싶었다.

## 정리

- 요소에는 상속이 통하지만 바구니끼리는 안 통한다. 넓게 받고 싶으면 와일드카드. 꺼낼 땐 extends, 넣을 땐 super
- `Arrays.asList`는 ArrayList가 아니라 배열을 감싼 고정 크기 리스트다. 늘리고 싶으면 `new ArrayList<>(...)`
- 컴파일러는 변수 타입만 본다. 내가 캐스팅을 쓰면 그 말을 믿고 넘어가서 실행할 때 터진다
- Set과 Map의 "같다"는 `hashCode()`와 `equals()`가 정한다
- 파라미터는 인터페이스 타입으로 넓게 받자. 오버로딩은 null 앞에서 모호해질 수 있다
