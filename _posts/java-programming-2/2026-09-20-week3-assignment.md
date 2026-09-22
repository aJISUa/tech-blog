---
title: "[자프실2] 3주차 과제: 제네릭 주사위, HashSet, HashMap, PrettyPrinter"
date: 2026-09-20 20:00:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - 제네릭
  - 컬렉션
  - 다형성
  - 과제
excerpt: "Lab#1 제네릭 주사위, Lab#5 HashSet, Lab#6 HashMap, Lab#8 PrettyPrinter를 직접 만들고 돌려봤다. 수업 때 ArrayList 이름을 잘못 지어서 난 에러도 같이 정리했다."
toc: true
toc_sticky: true
---

이번 주 과제는 세 가지였다.

1. Generic + Collection 이론 정리
2. 실습: Lab#1~#8 중 3개 이상. 실습마다 목표, 코드, 결과 캡처, 후기
3. 퀴즈 만들고 풀어보기 3개 이상

이론은 주제별로 나눠서 [제네릭 기초]({{ site.baseurl }}{% post_url java-programming-2/2026-09-16-generic-basics %}), [와일드카드와 PECS]({{ site.baseurl }}{% post_url java-programming-2/2026-09-18-wildcard-pecs %}), [컬렉션 프레임워크]({{ site.baseurl }}{% post_url java-programming-2/2026-09-18-collection-framework %})에 정리했고, 퀴즈는 [3주차 퀴즈 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-20-generic-collection-quiz %})에 따로 썼다. 이 글은 실습 부분이다.

실습은 이렇게 골랐다.

| 실습 | 내용 | 고른 이유 |
| --- | --- | --- |
| 수업 때 한 것 | ArrayList, 순회 방법, 배열 → 리스트 | 수업 때 에러가 나서 다시 해봐야 했다 |
| Lab#1 | 색깔 주사위와 숫자 주사위 | 제네릭 클래스 하나로 여러 타입을 다루는 걸 직접 해보고 싶었다 |
| Lab#5 | HashSet 집합 연산 | 수업 때 한 게 결과가 이상하게 나왔다 |
| Lab#6 | HashMap | keySet, values, entrySet 연습 |
| Lab#8 | PrettyPrinter 만들기 | 제네릭, 컬렉션, 다형성을 한 번에 써볼 수 있다 |

코드는 전부 이클립스 `JAVA2` 프로젝트에 패키지를 나눠서 만들었다. 노션에 코드가 있어도 복붙하지 말고 손으로 쳐보라고 하셔서 직접 쳤고, 로직은 주석으로 내 말로 설명해뒀다.

## 0. 수업 때 한 것: ArrayList와 순회

### 클래스 이름을 ArrayList로 지었더니

수업 때 `generic` 패키지에 ArrayList 연습용 클래스를 만들면서 이름을 그냥 `ArrayList`로 지었다.

```java
package generic;

import java.util.List;
import java.util.ArrayList;   // 여기서 에러

public class ArrayList {      // 내가 만든 클래스 이름도 ArrayList
    public static void main(String[] args) {
        List<String> list1 = new ArrayList<>();
        ...
    }
}
```

![클래스 이름 ArrayList 에러]({{ '/assets/images/java-programming-2/g3-classname-error.png' | relative_url }})

import 줄과 `new ArrayList<>()`가 있는 9번 줄에 바로 빨간 줄이 생겼다. `java.util.ArrayList`를 가져오겠다고 했는데 이 파일 안에 이미 `ArrayList`라는 클래스(내가 만든 것)가 있으니까, 컴파일러 입장에서는 `ArrayList`가 둘 중 뭔지 알 수가 없다. javac로 따로 컴파일해보니까 `ArrayList is already defined in this compilation unit`이라고 나왔다.

import를 지워도 문제다. 그러면 `new ArrayList<>()`가 내가 만든 `generic.ArrayList`를 가리키는데, 이건 제네릭 클래스도 아니고 List를 구현하지도 않는다. 자바가 이미 쓰는 이름(ArrayList, List, String 같은)은 클래스 이름으로 쓰지 말아야 한다는 걸 이번에 확실히 알았다.

### (ArrayList) Arrays.asList(...)로 캐스팅했더니

같은 날 만든 `ListTest`에서는 배열을 리스트로 바꾸려고 이렇게 썼다.

```java
String[] str = {"aa", "bd"};
Integer[] ar = {1, 2, 3, 4};

ArrayList<String> l3 = (ArrayList<String>) Arrays.asList(str);
ArrayList<Integer> l4 = (ArrayList<Integer>) Arrays.asList(ar);
```

![ListTest 에러]({{ '/assets/images/java-programming-2/g3-listtest-error.png' | relative_url }})

처음 에러는 단순했다. `Arrays`를 import하지 않았다. 그런데 import를 넣고 실행하면 이번에는 `ClassCastException`으로 죽는다.

`Arrays.asList`가 돌려주는 건 `java.util.ArrayList`가 아니라 이름만 같은 `Arrays` 안의 내부 클래스(`java.util.Arrays$ArrayList`)였다. 둘 다 List를 구현하긴 하지만 서로 다른 클래스라서 캐스팅이 안 된다. 컴파일러는 "List를 ArrayList로 다운캐스팅하는 거니까 될 수도 있겠지" 하고 통과시켜주고, 실제 객체를 보는 실행 때 터진다. 제네릭 기초에서 본 Object 박스 문제랑 똑같은 모양이다.

그래서 공지에 올라온 예제는 `new ArrayList<>(Arrays.asList(str))`로 한 번 더 감싸서 진짜 ArrayList를 새로 만든다. 이게 왜 필요한지 궁금해서 퀴즈 2번으로 더 파봤다.

### 순회 방법 연습 (ListLoop)

공지의 ListTest를 참고해서 순회 연습용 코드를 `collection` 패키지에 새로 만들었다.

```java
// 1. 참조는 인터페이스(List)로, 생성은 구현 클래스(ArrayList)로
List<Integer> list1 = new ArrayList<>();
List<String> list2 = new ArrayList<>();

// 2. 배열을 리스트로 바꾸기. Arrays.asList 결과를 new ArrayList<>(...)에 한 번 더 담는다
String[] str = {"사과", "포도", "오렌지"};
Integer[] ar = {1, 2, 3, 4};
List<String> list3 = new ArrayList<>(Arrays.asList(str));
List<Integer> list4 = new ArrayList<>(Arrays.asList(ar));

// 3. 추가, 끼워넣기, 대체, 삭제
list3.add("바나나");        // 맨 뒤에 추가
list3.add(1, "딸기");       // 1번 자리에 끼워 넣기 (뒤는 한 칸씩 밀린다)
list3.set(2, "키위");       // 2번 자리를 키위로 바꾸기
list3.remove(3);            // 3번 자리 삭제

// 4. 순회 방법 네 가지
for (int i = 0; i < list3.size(); i++) System.out.print(list3.get(i) + " ");
for (String s : list3) System.out.print(s + " ");
list3.forEach(s -> System.out.print(s + " "));
list3.forEach(System.out::println);          // 제일 간편

// 5. Iterator: 꺼내면서 지우고 싶을 때 (짝수만 지우기)
Iterator<Integer> it = list4.iterator();
while (it.hasNext()) {
    if (it.next() % 2 == 0) it.remove();
}

// 6. int로 받아도 된다. Integer가 int로 자동 언박싱
int sum = 0;
for (int i : list4) sum += i;
```

#### 실행 전에 예상한 것

- `list3`는 [사과, 포도, 오렌지]에 바나나를 붙이고, 1번에 딸기를 끼우면 [사과, 딸기, 포도, 오렌지, 바나나]가 된다. 2번(포도)을 키위로 바꾸고 3번(오렌지)을 지우면 [사과, 딸기, 키위, 바나나], 크기 4.
- 순회 네 가지는 모양만 다르고 결과는 똑같이 나올 것 같다. 4번은 `println`이라 한 줄에 하나씩.
- `list4`는 [1, 2, 3, 4]에서 짝수를 지우니까 [1, 3], 합은 4.

#### 실제 결과

![ListLoop 실행 결과]({{ '/assets/images/java-programming-2/g3-listloop.png' | relative_url }})

예상대로 나왔다. `add(1, ...)`로 끼워 넣으면 뒤에 있던 것들 번호가 하나씩 밀려서, 그다음 `set(2, ...)`는 원래 1번이었던 포도를 가리킨다. 순서대로 번호를 따라가야 헷갈리지 않는다.

Iterator로 지운 이유는 for-each 안에서 `list4.remove()`를 부르면 `ConcurrentModificationException`이 날 수 있어서다. 도는 중에 지울 때는 Iterator의 `remove()`를 써야 한다.

## 1. Lab#1: 색깔 주사위와 숫자 주사위

### 실습 목표

제네릭 클래스 `MyDice<T>` 하나로 색깔 주사위(`String`)와 숫자 주사위(`Integer`)를 둘 다 만든다. 제네릭 없이 하면 ColorDice, NumberDice 두 개를 만들어야 하는 걸 하나로 줄이는 게 목표다.

### 코드

```java
public class MyDice<T> {

    // T[] value = new T[6]; 은 컴파일 에러. 제네릭 타입으로는 배열을 new 할 수 없다
    // 그래서 면(face)들은 ArrayList<T>에 담는다
    private ArrayList<T> faces = new ArrayList<>();
    private T result;   // 마지막으로 굴려서 나온 면 하나. 변수 하나는 T로 선언해도 된다
    private Random random = new Random();

    // 면 하나 추가. T로 약속한 타입만 들어온다
    public void addFace(T face) {
        faces.add(face);
    }

    // 면 여러 개를 한 번에 세팅
    // ? extends T: T이거나 T의 자식 타입 리스트면 다 받는다 (꺼내서 넣기만 하니까 extends)
    public void setFaces(List<? extends T> newFaces) {
        faces.clear();
        faces.addAll(newFaces);
    }

    // 주사위 굴리기: 면들 중 하나를 랜덤으로 골라 result에 보관하고 돌려준다
    public T roll() {
        if (faces.isEmpty()) {
            result = null;   // 면이 없으면 굴릴 수 없으니 null
            return null;
        }
        result = faces.get(random.nextInt(faces.size()));
        return result;
    }
    ...
}
```

```java
public class DiceTest {
    public static void main(String[] args) {
        // 1. 타입만 바꿔서 주사위 두 종류를 만든다
        MyDice<String> colorDice = new MyDice<>();
        MyDice<Integer> numberDice = new MyDice<>();

        // 2. 면 세팅. 색깔은 addFace로 하나씩, 숫자는 setFaces로 한 번에
        colorDice.addFace("빨강");
        ... // 주황, 노랑, 초록, 파랑, 보라
        numberDice.setFaces(Arrays.asList(1, 2, 3, 4, 5, 6));

        // colorDice.addFace(3);           // 에러! String 주사위에 Integer
        // numberDice.addFace("여섯");      // 에러! Integer 주사위에 String
        // MyDice<int> d = new MyDice<>();  // 에러! 기본 타입은 타입 인자로 못 쓴다

        // 3. 어떤 주사위를 굴릴지 입력받아서 5번 굴린다
        Scanner in = new Scanner(System.in);
        System.out.print("주사위 타입을 선택하세요 (1: 색깔, 2: 숫자) > ");
        int type = in.nextInt();
        for (int i = 1; i <= 5; i++) {
            if (type == 1) {
                String c = colorDice.roll();     // 형변환 없이 바로 String
                System.out.println(i + "번째: " + c);
            } else {
                int n = numberDice.roll();       // Integer가 int로 자동 언박싱
                System.out.println(i + "번째: " + n + (n % 2 == 0 ? " (짝수)" : " (홀수)"));
            }
        }

        // 4. 면이 하나도 없는 주사위를 굴리면?
        MyDice<Double> emptyDice = new MyDice<>();
        System.out.println("빈 주사위 굴리기: " + emptyDice.roll());
    }
}
```

`setFaces`의 파라미터를 `List<T>`가 아니라 `List<? extends T>`로 한 건 와일드카드를 배운 김에 써본 거다. 이 리스트에서는 꺼내서 내 ArrayList에 넣기만 하니까 Producer다. 그래서 extends. 예를 들어 `MyDice<Number>`에 `List<Integer>`를 넘겨도 받아준다.

#### 실행 전에 예상한 것

- 주사위 면 목록은 넣은 순서대로 [빨강, 주황, 노랑, 초록, 파랑, 보라], [1, 2, 3, 4, 5, 6]으로 나올 것 같다. ArrayList는 순서를 유지하니까.
- 굴린 결과는 랜덤이라 매번 다르다. 1을 넣으면 색깔 5개, 2를 넣으면 숫자 5개와 짝수/홀수가 나온다.
- 빈 주사위는 null이 나올 것 같다.
- 주석 친 세 줄은 풀면 컴파일 에러가 날 것 같다.

#### 실제 결과

![Lab#1 색깔 주사위 실행 결과]({{ '/assets/images/java-programming-2/g3-lab1-dice-color.png' | relative_url }})

![Lab#1 숫자 주사위 실행 결과]({{ '/assets/images/java-programming-2/g3-lab1-dice-number.png' | relative_url }})

같은 `MyDice` 클래스인데 타입만 다르게 줬더니 색깔 주사위, 숫자 주사위가 둘 다 잘 돌아간다. `colorDice.roll()`은 바로 String으로, `numberDice.roll()`은 바로 int로 받았다. Object로 만들었으면 `(String)`, `(Integer)` 형변환을 매번 해야 했을 거다.

주석 세 줄은 풀어서 javac로 컴파일해봤는데 예상대로 셋 다 에러였다. `addFace(3)`은 `int cannot be converted to String`, `MyDice<int>`는 `unexpected type`이었다. `new T[6]`도 시험 삼아 넣어봤는데 `generic array creation` 에러가 났다.

### 노션 참고 코드와 비교

다 만들고 나서 노션의 "Generic&Collection : MyDice" 코드랑 비교해봤다. 노션 코드는 이렇게 생겼다.

```java
public class MyDice<T> {
    T result = null;          // 구르기 결과
    ArrayList<T> list;        // 주사위 값 리스트

    public MyDice(T t) {      // 초기값을 받아서
        this();
        result = t;
        showType();           // "이 주사위의 요소의 타입은 Integer입니다."
    }
    public void showType() {
        System.out.println("이 주사위의 요소의 타입은 " + result.getClass().getSimpleName() + "입니다.");
    }
    public void setDiceToColor() {
        ...
        for (String s : ar) list.add((T) s);       // String을 T로 형변환
    }
    public void setDiceToNumber(int size) {
        for (Integer i = 0; i < size; i++) {
            Integer num = i + 1;
            list.add((T) num);                       // Integer를 T로 형변환
        }
    }
    ...
}
```

main에서는 `dice = new MyDice<String>("")`, `dice = new MyDice<Integer>(0)`처럼 빈 문자열이나 0을 "초기화를 위해서" 넘긴다.

비교하면서 알게 된 것이 두 가지 있다.

**1. 왜 굳이 ""나 0을 넘길까.** `showType()`이 `result.getClass()`로 타입 이름을 알아내기 때문이다. 실행 중에는 `T`가 String인지 Integer인지 코드로 바로 물어볼 방법이 없어서, 실제 값을 하나 넣어두고 그 값의 클래스를 보는 거다. 값 없이 `new MyDice<String>()`으로 만들고 `showType()`을 부르면 result가 null이라 NullPointerException이 났다.

**2. `(T) s` 형변환은 컴파일러가 못 잡는다.** 궁금해서 `MyDice<Integer>`로 만들고 `setDiceToColor()`를 불러봤다.

```java
MyDice<Integer> d = new MyDice<>(0);
d.setDiceToColor();        // Integer 주사위에 색깔을 넣는다. 컴파일 에러 없음
System.out.println(d.list);   // [빨강색, 주황색, ...] 그대로 들어가 있다
d.구르기();
Integer n = d.result;      // 여기서 ClassCastException
```

컴파일할 때는 `unchecked cast` 경고만 나오고 에러는 없다. 실행하면 색깔이 Integer 리스트에 그대로 들어가고, 꺼내서 Integer로 받는 순간 `String cannot be cast to Integer`로 죽었다. 컴파일할 때 T가 지워져서 `(T)` 형변환은 실제로는 아무 검사도 안 하기 때문이라고 한다.

노션 코드는 main에서 타입에 맞는 세팅 메소드만 부르니까 문제가 없지만, 실수로 반대로 부르면 제네릭의 장점인 "컴파일할 때 잡기"가 사라진다. 내 버전은 세팅 값을 밖에서 `addFace(T)`, `setFaces(List<? extends T>)`로 받게 해서 이런 형변환이 없고, 잘못 넣으면 컴파일 에러가 난다. 대신 노션 코드처럼 "색깔 주사위로 세팅해줘" 한 번에 끝나는 편리함은 없어서, 쓰는 쪽(main)이 면을 직접 넣어줘야 한다.

### 실습 후기

- 제네릭을 쓰니까 클래스를 타입 수만큼 만들 필요가 없었다. 나중에 TV 주사위나 짱구 주사위를 만들고 싶어도 `MyDice<Tv>`처럼 타입만 바꾸면 된다.
- 약속을 어기면 실행해보기 전에 에디터에서 바로 빨간 줄이 생긴다. "컴파일러가 딱 잡는다"는 게 이런 거였다.
- 슬라이드에서 `T[] value`가 왜 주석 처리돼 있었는지 `new T[6]`을 직접 넣어보고 알았다. 제네릭이 왜 컬렉션이랑 같이 다니는지 이해됐다.
- 궁금한 점: `roll()`에서 면이 없을 때 null을 돌려줬는데, 그러면 숫자 주사위에서 `int n = numberDice.roll()`로 받을 때 언박싱하다가 NullPointerException이 날 것 같다. 예외를 던지는 게 나을지 고민이다.

## 2. Lab#5: HashSet

### 실습 목표

HashSet으로 합집합, 교집합, 차집합, 부분집합을 해보고, Iterator로 출력하는 제네릭 메소드를 만든다. 같은 데이터를 HashSet, LinkedHashSet, TreeSet에 넣어서 순서가 어떻게 다른지도 본다.

### 수업 때 결과가 이상했던 이유

수업 때는 슬라이드 코드를 거의 그대로 쳤다.

```java
set1.addAll(set2);     // 합집합
set2.retainAll(set3);  // 교집합
set3.removeAll(set1);  // 차집합
```

set3 = {9, 11, 7, 6, 2}에서 set1 = {1, 2, 3, 4, 5}를 빼면 {9, 11, 7, 6}이 나와야 할 것 같은데, 결과는 `9 11`이었다. 첫 줄에서 `set1.addAll(set2)`를 하는 순간 set1이 {1, 2, 3, 4, 5, 6, 7}로 바뀌어버렸기 때문이다. 슬라이드에도 "주의! 위에서 set3의 값이 변경되었다"라는 주석이 있었는데 그때는 무슨 말인지 몰랐다.

`addAll`, `retainAll`, `removeAll`은 새 집합을 만들어주는 게 아니라 부른 쪽 집합 자체를 바꾼다. 그래서 이번에는 매번 복사본을 만들어서 계산했다.

### 코드

```java
// 출력용 제네릭 메소드. E 자리에 Integer든 String이든 들어갈 수 있다
// main이 static이라 객체를 안 만들고 바로 부르려면 이 메소드도 static이어야 한다
public static <E> void show(String title, Set<E> s) {
    System.out.print(title + " : ");
    // Set은 인덱스가 없어서 for (int i...) + get(i)를 못 쓴다
    Iterator<E> it = s.iterator();
    while (it.hasNext()) {
        System.out.print(it.next() + " ");
    }
    System.out.println();
}

public static void main(String[] args) {
    Set<Integer> a = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));
    Set<Integer> b = new HashSet<>(Arrays.asList(3, 4, 5, 6, 7));

    // 원본 a를 지키려고 매번 new HashSet<>(a)로 복사해서 계산한다
    Set<Integer> union = new HashSet<>(a);
    union.addAll(b);                           // 합집합
    Set<Integer> inter = new HashSet<>(a);
    inter.retainAll(b);                        // 교집합
    Set<Integer> diff = new HashSet<>(a);
    diff.removeAll(b);                         // 차집합 A - B
    ...
    System.out.println("A가 A∩B를 포함? (containsAll) " + a.containsAll(inter));
    System.out.println("A가 B를 포함? (containsAll) " + a.containsAll(b));

    // 중복은 안 들어간다. add는 성공하면 true, 이미 있으면 false
    System.out.println("A에 3 다시 add : " + a.add(3) + ", 크기 " + a.size());
    System.out.println("A에 null add : " + a.add(null) + ", 한 번 더 : " + a.add(null));

    // 같은 순서로 넣고 세 가지 Set 비교
    String[] input = {"banana", "kiwi", "apple", "mango", "cherry", "apple"};
    Set<String> hash = new HashSet<>(Arrays.asList(input));
    Set<String> linked = new LinkedHashSet<>(Arrays.asList(input));
    Set<String> tree = new TreeSet<>(Arrays.asList(input));
    ...
}
```

`show` 메소드에 static을 붙인 건 금요일에 교수님께서 설명해주신 부분이다. main이 static이니까 main에서 `new Main()` 없이 바로 부르려면 이 메소드도 static이어야 한다.

#### 실행 전에 예상한 것

- 합집합 1~7, 교집합 3 4 5, 차집합 1 2가 나올 것 같다.
- `a.containsAll(inter)`는 교집합이 A 안에 있으니 true, `a.containsAll(b)`는 6, 7이 A에 없으니 false.
- 3을 다시 넣으면 false이고 크기는 그대로 5. null은 처음 한 번만 true, 두 번째는 false.
- 문자열 비교는 apple이 두 번 들어가니까 셋 다 5개. HashSet은 순서가 뒤죽박죽, LinkedHashSet은 넣은 순서(banana kiwi apple mango cherry), TreeSet은 알파벳 순(apple banana cherry kiwi mango).

#### 실제 결과

![Lab#5 HashSet 실행 결과]({{ '/assets/images/java-programming-2/g3-lab5-hashset.png' | relative_url }})

전부 예상대로 나왔다. 복사본으로 계산하니까 수업 때처럼 이상한 값이 안 나온다.

하나 신기했던 건 숫자 HashSet이다. 순서를 안 지킨다고 했는데 1 2 3 4 5, 1 2 3 4 5 6 7처럼 작은 순서대로 나왔다. 찾아보니까 Integer의 hashCode는 그 숫자 자체라서, 작은 숫자들은 해시 테이블에서 번호 순서대로 칸에 들어가서 우연히 정렬된 것처럼 보이는 거였다. 교수님께서 "재수가 좋으면 순서대로 나올 수도 있다"고 하신 게 이거였다. 문자열로 넣으니까 HashSet은 banana apple cherry kiwi mango처럼 넣은 순서도, 알파벳 순서도 아닌 순서로 나왔다.

### 실습 후기

- `addAll` 같은 벌크 연산은 원본을 바꾼다. 파이썬의 `a | b`처럼 새 집합을 돌려줄 거라고 생각하면 틀린다.
- HashSet 결과가 정렬돼 보인다고 순서가 있는 게 아니다. 순서가 필요하면 LinkedHashSet이나 TreeSet을 쓰는 게 맞다.
- 중복을 넣으면 에러가 아니라 `false`가 돌아온다. 넣었는지 확인하고 싶으면 add의 리턴값을 보면 된다.

## 3. Lab#6: HashMap

### 실습 목표

HashMap에 put, get, remove를 해보고, 같은 키로 다시 넣으면 어떻게 되는지 본다. keySet, values, entrySet으로 키와 값을 꺼내는 제네릭 메소드를 만들고, HashMap, LinkedHashMap, TreeMap의 순서를 비교한다.

### 코드

```java
// K, V 두 개를 쓰는 멀티 타입 파라미터 제네릭 메소드
public static <K, V> void show(Map<K, V> map) {
    // keySet의 리턴 타입은 Set. 키는 중복이 안 되니까
    Set<K> keys = map.keySet();
    System.out.println("Key    : " + keys);
    // values의 리턴 타입은 Collection. 값은 중복될 수 있으니까 Set이 아니다
    Collection<V> values = map.values();
    System.out.println("Values : " + values);
    // entrySet: 키와 값을 한 쌍(Map.Entry)으로 묶어서 Set으로 준다
    for (Map.Entry<K, V> entry : map.entrySet()) {
        System.out.println(entry.getKey() + " : " + entry.getValue());
    }
}

public static void main(String[] args) {
    Map<Integer, String> map = new HashMap<>();
    for (int i = 1; i <= 5; i++) {
        map.put(i, "data#" + i);
    }
    System.out.println(map);
    System.out.println("get(111) = " + map.get(111));   // 없는 키

    map.put(111, "data#111");
    map.remove(1);
    map.put(3, "new_data#3");               // 같은 키에 다시 넣으면?
    System.out.println("put(3) 후 크기 = " + map.size());
    show(map);

    // 메뉴판으로 HashMap / LinkedHashMap / TreeMap 비교
    // menu#3, menu#5, menu#1, menu#4, menu#2 순서로 넣는다
    ...
    linkedMenu.put("menu#5", "아이스티");    // 이미 있는 키에 새 값

    hashMenu.forEach((k, v) -> System.out.println(k + " => " + v));   // 람다식 인자 2개
    ...
}
```

메뉴는 슬라이드에 있던 메가커피 메뉴를 그대로 썼다. 대신 넣는 순서를 일부러 섞었다.

#### 실행 전에 예상한 것

- 처음 map은 {1=data#1, ..., 5=data#5}. 없는 키 111을 get하면 null.
- 111을 넣고 1을 지우고 3을 다시 넣으면, 3은 새 칸이 생기는 게 아니라 값만 new_data#3으로 바뀐다. 그래서 크기는 5(2, 3, 4, 5, 111).
- HashMap 메뉴는 순서가 뒤죽박죽, LinkedHashMap은 넣은 순서(3, 5, 1, 4, 2), TreeMap은 키 순서(1~5).
- LinkedHashMap에서 menu#5를 아이스티로 바꿔도 자리는 두 번째 그대로일 것 같다. 키는 그대로고 값만 바뀌니까.

#### 실제 결과

![Lab#6 HashMap 실행 결과]({{ '/assets/images/java-programming-2/g3-lab6-hashmap.png' | relative_url }})

예상대로였다. 같은 키로 put하면 크기는 그대로고 값만 바뀐다. 슬라이드에 "중복된 키와 값을 저장하면 마지막에 저장된 값으로 대체"라고 한 게 이거다.

HashMap 메뉴는 menu#5, #4, #3, #2, #1로 거꾸로 나왔다. 슬라이드 결과(3, 5, 4, 1, 2)랑도 달랐다. 해시 값에 따라 칸이 정해지는 거라 순서를 믿으면 안 된다는 걸 다시 느꼈다.

LinkedHashMap에서 menu#5를 아이스티로 바꿨더니 맨 뒤로 가지 않고 원래 자리(두 번째)에 그대로 있었다. 순서는 "처음 넣은 순서"를 기억하는 거였다.

### 실습 후기

- Map은 Collection이 아니라서 바로 for-each를 못 돌린다. entrySet으로 Set을 받아서 돌아야 한다. 그림에서 Map만 Collection 밑에 없던 이유를 코드로 느꼈다.
- keySet은 Set, values는 Collection으로 리턴 타입이 다르다. 키는 중복이 안 되고 값은 될 수 있어서다.
- 람다식 `forEach((k, v) -> ...)`가 제일 편했다. entrySet으로 도는 것보다 훨씬 짧다.

## 4. Lab#8: PrettyPrinter 만들기

### 실습 목표

어떤 컬렉션이든 안에 뭐가 들었든 보기 좋고 안전하게 출력하는 `PrettyPrinter`를 만든다. 요구사항은 이랬다.

- 출력 메소드 이름은 `show()`
- 요소가 Integer, String, Double, 사용자 정의 클래스여도 문제없어야 한다
- Null-Safe
- List, Set, Map 범위 안에서
- 제네릭, 다형성, 컬렉션, 메소드 오버로딩을 활용

슬라이드 Lab#7과 노션 "PrettyPrinter 인터페이스" 페이지에 예시 코드가 있어서 그걸 먼저 읽어보고, 부족한 부분을 고쳐서 만들었다. 노션 버전의 for-each 구현 클래스에는 for-each 말고도 같은 내용을 `set.forEach(item -> ...)` 람다식, `set.forEach(System.out::println)` 메소드 참조, `map.forEach((key, value) -> ...)`, `map.entrySet().forEach(System.out::println)`으로 한 번씩 더 출력하는 부분이 들어 있었다. 컬렉션 메소드로 출력하는 방법을 한 번에 비교해볼 수 있어서 좋았다. 나는 결과 비교가 쉽도록 구현 클래스마다 한 가지 방법만 썼다.

### Lab#7을 보면서 고치고 싶었던 것

1. **`show(ArrayList<E> list)`로 받는다.** 슬라이드랑 노션 인터페이스 둘 다 이렇게 돼 있다. 그러면 LinkedList나 `Arrays.asList`로 만든 리스트는 못 넘긴다. 컬렉션 글에서 "인터페이스로 참조하라"고 배웠으니 `List<E>`로 받는 게 맞다. 실제로 넘겨보니까 컴파일 에러가 났다. 이건 퀴즈 4번으로 만들었다.
2. **null에 약하다.** 요소가 null인 건 `item + " "`에서 알아서 "null"로 찍히지만, 컬렉션 자체가 null이면 `for (E item : set)`에서 NullPointerException이 난다.
3. **빈 컬렉션이면 제목만 덩그러니 나온다.**
4. **문자열 `""`이나 `" "`은 출력하면 안 보인다.**

### 구조

```text
<<interface>> PrettyPrinter
  + show(Set<E>)
  + show(Map<K,V>)
  + show(List<E>)
  + text(Object) : String   (default)
      △                 △
ForEachPrinter    IteratorPrinter
```

Lab#7처럼 구현 클래스는 for-each 버전과 Iterator 버전 두 개로 만들었다. 테스트에서는 `PrettyPrinter` 타입 하나로 두 구현체를 갈아 끼운다.

### 코드

```java
// 어떤 컬렉션이든 안에 뭐가 들었든 보기 좋고 안전하게 출력하는 인터페이스
// show()를 Set, Map, List 버전으로 오버로딩했다
public interface PrettyPrinter {

    <E> void show(Set<E> set);

    <K, V> void show(Map<K, V> map);

    // Lab#7은 ArrayList<E>로 받았는데, 그러면 LinkedList나 Arrays.asList 결과는 못 넘긴다
    // 인터페이스 타입인 List<E>로 받아서 List 구현체면 전부 받을 수 있게 바꿨다
    <E> void show(List<E> list);

    // 요소 하나를 글자로 바꾸는 공통 규칙. 구현 클래스 두 개가 똑같이 쓰니까 default로 올려뒀다
    // null이면 NullPointerException 대신 "null"이라고 보여준다
    default String text(Object o) {
        if (o == null) return "null";
        if (o instanceof String) return "\"" + o + "\"";   // 문자열은 따옴표로 감싸서 빈 문자열도 보이게
        return o.toString();
    }
}
```

```java
// for-each로 출력하는 구현 클래스
public class ForEachPrinter implements PrettyPrinter {

    @Override
    public <E> void show(Set<E> set) {
        System.out.print("[Set 출력 : for-each] ");
        if (set == null) { System.out.println("(null 컬렉션)"); return; }   // 컬렉션 자체가 null
        if (set.isEmpty()) { System.out.println("(비어 있음)"); return; }
        for (E item : set) {
            System.out.print(text(item) + " ");
        }
        System.out.println("(" + set.size() + "개)");
    }

    @Override
    public <K, V> void show(Map<K, V> map) {
        System.out.println("[Map 출력 : for-each]");
        if (map == null) { System.out.println("  (null 컬렉션)"); return; }
        if (map.isEmpty()) { System.out.println("  (비어 있음)"); return; }
        // Map은 Collection이 아니라서 바로 for-each를 못 돌린다. entrySet()으로 Set을 받아서 돈다
        for (Map.Entry<K, V> entry : map.entrySet()) {
            System.out.println("  " + text(entry.getKey()) + " => " + text(entry.getValue()));
        }
    }

    // show(List<E>)도 Set과 같은 모양
}
```

```java
// Iterator로 출력하는 구현 클래스. 사용법(show)은 ForEachPrinter와 똑같고 도는 방법만 다르다
public class IteratorPrinter implements PrettyPrinter {
    ...
    @Override
    public <K, V> void show(Map<K, V> map) {
        System.out.println("[Map 출력 : Iterator]");
        if (map == null) { System.out.println("  (null 컬렉션)"); return; }
        if (map.isEmpty()) { System.out.println("  (비어 있음)"); return; }
        Iterator<Map.Entry<K, V>> it = map.entrySet().iterator();
        while (it.hasNext()) {
            Map.Entry<K, V> entry = it.next();   // next()는 한 바퀴에 한 번만 불러야 한다
            System.out.println("  " + text(entry.getKey()) + " => " + text(entry.getValue()));
        }
    }
    ...
}
```

사용자 정의 클래스로는 `Student`를 만들고 `toString()`을 오버라이딩했다. 안 하면 `prettyprinter.Student@1b6d3586`처럼 나온다.

```java
public class PrettyPrinterTest {
    public static void main(String[] args) {
        // 1. 요소 타입이 제각각인 컬렉션들
        Set<String> languages = new HashSet<>(Arrays.asList("Java", "Java", "Java", "C", "C++", "Python"));
        Set<Integer> lotto = new TreeSet<>(Arrays.asList(45, 3, 27, 3, 11));
        Map<String, Integer> menu = new LinkedHashMap<>();
        menu.put("라면", 2000);
        menu.put("아아", 1000);
        menu.put("아이스크림", null);        // 값이 null
        Map<String, Student> seat = new HashMap<>();
        seat.put("1번", new Student("짱구", 60));
        seat.put(null, new Student("유리", 95)); // HashMap은 키도 null 하나는 된다
        List<Double> scores = new ArrayList<>(Arrays.asList(85.5, 90.0, 78.5, null));
        List<Student> team = new LinkedList<>();  // ArrayList가 아닌 List
        team.add(new Student("철수", 100));
        team.add(null);
        List<String> blank = Arrays.asList("", " ", "끝");

        // 2. 빈 컬렉션, null 컬렉션
        Set<String> emptySet = new HashSet<>();
        List<Integer> nullList = null;

        // 3. 참조는 인터페이스 PrettyPrinter 하나. 구현체만 갈아 끼운다 (다형성)
        PrettyPrinter[] printers = { new ForEachPrinter(), new IteratorPrinter() };
        for (PrettyPrinter printer : printers) {
            System.out.println("########## " + printer.getClass().getSimpleName());
            printer.show(languages);
            printer.show(lotto);
            printer.show(menu);
            printer.show(seat);
            printer.show(scores);
            printer.show(team);
            printer.show(blank);
            printer.show(emptySet);
            printer.show(nullList);
            // printer.show(null);   // 에러! Set, Map, List 버전 중 뭘 불러야 할지 모른다
        }
    }
}
```

처음에는 Lab#7처럼 `Map.of("라면", 2000, "아아", 1000, "아이스크림", null)`로 null 값을 넣어보려고 했는데, Map.of는 null을 넣으면 그 자리에서 NullPointerException이 났다. 그래서 LinkedHashMap에 put으로 넣었다.

#### 실행 전에 예상한 것

- languages는 Java가 세 번 들어가도 Set이라 4개(Java, C, C++, Python). HashSet이라 순서는 섞인다. 문자열이라 따옴표가 붙는다.
- lotto는 TreeSet이라 3이 한 번만 들어가고 3 11 27 45로 정렬돼서 4개.
- menu는 LinkedHashMap이라 넣은 순서대로 나오고, 아이스크림 값은 에러 없이 null.
- seat은 null 키가 있어도 에러 없이 `null => 유리(95점)`이 찍힐 것 같다.
- scores는 null까지 4개, team은 LinkedList인데도 `show(List)`로 들어가서 `철수(100점) null`이 찍힌다.
- blank는 `"" " " "끝"`처럼 따옴표 덕분에 빈 문자열도 보인다.
- 빈 Set은 (비어 있음), null 리스트는 (null 컬렉션).
- ForEachPrinter와 IteratorPrinter는 도는 방법만 다르니까 결과가 완전히 같아야 한다.

#### 실제 결과

![Lab#8 PrettyPrinter 실행 결과]({{ '/assets/images/java-programming-2/g3-lab8.png' | relative_url }})

예상대로 위쪽 ForEachPrinter와 아래쪽 IteratorPrinter의 결과가 제목만 빼고 한 줄도 안 다르다. 사용하는 쪽(PrettyPrinterTest)은 `printer.show(...)`만 부르고, 안에서 for-each로 도는지 Iterator로 도는지는 모른다. 짱구 때 배운 "호출은 부모 타입으로, 실행은 실제 객체의 메소드로"가 인터페이스에서도 그대로였다.

seat은 null 키가 `"1번"`보다 먼저 나왔다. HashMap에서 null 키는 해시 값을 0으로 쳐서 맨 앞 칸에 들어간다고 한다.

마지막으로 주석 친 `printer.show(null)`을 풀어봤다.

![Lab#8 show(null) 컴파일 에러]({{ '/assets/images/java-programming-2/g3-lab8-ambiguous.png' | relative_url }})

`The method show(Set<Object>) is ambiguous for the type PrettyPrinter`, show가 모호하다(ambiguous)는 에러다. null은 Set에도 Map에도 List에도 들어갈 수 있으니까, 오버로딩된 show 중에 뭘 불러야 할지 컴파일러가 못 고른다. 바로 위의 `printer.show(nullList)`처럼 타입이 정해진 변수로 넘기면 List 버전이 불려서 문제가 없다. Null-Safe를 만들었는데 정작 null을 그냥 넘기면 컴파일이 안 되는 게 재밌었다.

에러가 있는데도 이클립스에서 실행은 됐다. 대신 콘솔에는 앞의 출력이 하나도 없이 `Unresolved compilation problem`이라는 Error만 찍혔다. 에러가 있는 50번 줄보다 앞에 있는 show들도 실행되지 않았다. 이클립스는 에러가 있어도 일단 실행시켜주지만, 에러가 난 메소드는 실행하자마자 이 Error를 던지는 것 같다.

### 실습 후기

- **제네릭:** `show` 하나에 `<E>`를 붙이니까 요소가 Integer든 Student든 전부 받는다. 타입마다 메소드를 따로 안 만들어도 된다.
- **오버로딩:** Set, Map, List는 모양이 달라서 show를 세 개로 오버로딩했다. 대신 null처럼 셋 다 될 수 있는 값을 넘기면 모호해진다.
- **다형성:** 인터페이스 타입 하나로 구현체를 갈아 끼웠다. 나중에 표 모양으로 출력하는 `TablePrinter`를 만들어도 테스트 코드는 안 고쳐도 된다.
- **파라미터 타입은 인터페이스로:** `ArrayList<E>` 대신 `List<E>`로 받으니까 LinkedList도, Arrays.asList도 다 들어왔다. 받는 쪽은 최대한 넓게 받는 게 좋다.
- **default 메소드:** 두 구현체가 똑같이 쓰는 `text()`를 인터페이스에 default로 올려두니까 중복이 없어졌다.
- 궁금한 점: `show(List<E>)`와 `show(Set<E>)`를 합쳐서 `show(Collection<?> c)` 하나로 만들어도 될 것 같은데, 그러면 Set이랑 List 제목을 다르게 찍기가 어려워진다. 어느 쪽이 더 좋은 설계일까?

## 마치며

제네릭은 "약속한 타입만 넣게 하는 박스"라는 게 이번 실습으로 확실해졌다. 약속을 어기면 실행하기 전에 에디터가 먼저 빨간 줄로 알려준다.

반대로 컴파일러가 못 잡는 것도 있었다. `(ArrayList<String>) Arrays.asList(...)`처럼 내가 강제로 캐스팅하면 컴파일러는 믿고 넘어가고, 실행할 때 터진다. 제네릭을 쓰는 이유가 이런 형변환을 없애려는 거였는데, 내가 직접 형변환을 넣어서 그 장점을 스스로 버린 셈이었다.

Set과 Map은 순서가 있는 것처럼 보여도 믿으면 안 된다. 숫자 HashSet은 정렬돼 보였고, HashMap 메뉴는 슬라이드랑 다른 순서로 나왔다.
