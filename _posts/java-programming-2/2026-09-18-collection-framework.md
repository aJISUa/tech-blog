---
title: "[자프실2] 컬렉션 프레임워크: List, Set, Map"
date: 2026-09-18 20:10:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - 컬렉션
  - List
  - Set
  - Map
  - Iterator
excerpt: "제네릭으로 만든 자바 표준 자료구조인 컬렉션 프레임워크 구조를 보고, ArrayList, LinkedList, HashSet, TreeSet, HashMap, LinkedHashMap을 언제 쓰는지 정리했다."
toc: true
toc_sticky: true
---

[제네릭]({{ site.baseurl }}{% post_url java-programming-2/2026-09-16-generic-basics %})과 [와일드카드]({{ site.baseurl }}{% post_url java-programming-2/2026-09-18-wildcard-pecs %})에 이어서 금요일에는 컬렉션을 실습했다.

교수님께서 이번 주 목표를 한 문장으로 정리하셨다.

> 제네릭을 이해하자가 아니라, 제네릭으로 구현한 컬렉션을 잘 쓰자.

"욕심을 버렸더니 마음이 편안하다"고 하셨다. 핵심은 List, Set, Map 잘 쓰기다.

## 컬렉션은 왜 쓰나

컬렉션은 자바에서 자료구조를 구현해둔 클래스들이다. 객체를 효율적으로 추가, 삭제, 검색할 수 있게 해주는 라이브러리고, 전부 `java.util` 안에 있다.

배열은 왜 썼을까? 쉬워서. 대신 쉬운 만큼 불편했다.

- 저장할 수 있는 개수가 배열을 만들 때 정해진다. 몇 개가 들어올지 모르면 곤란하다
- 중간에 넣거나 빼려면 뒤를 다 밀고 당겨야 한다
- 중간이 빠지면 낱알 빠진 옥수수처럼 된다. 새로 넣으려면 어디가 비었는지 찾아야 한다. 자프실1 헌터&애니멀에서 사냥당한 동물 자리에 맨 마지막 동물을 옮겨 넣었던 게 바로 이거다

컬렉션을 쓰면 이런 걸 내가 안 만들어도 된다. 이제부터는 메소드를 직접 만드는 것보다 이미 있는 걸 불러다 쓰는 일이 많을 거라고 하셨다.

주의할 점은 컬렉션에는 객체만 들어간다는 것. `int`는 `Integer`로, `double`은 `Double`로 박싱해서 저장한다.

## 컬렉션 프레임워크 구조

```text
Iterable
 └ Collection
     ├ List  ── ArrayList ✔, LinkedList ✔, Vector ─ Stack
     ├ Queue ── PriorityQueue, Deque ─ ArrayDeque (LinkedList도 구현)
     └ Set   ── HashSet ✔, LinkedHashSet, SortedSet ─ TreeSet

Map ── HashMap ✔, LinkedHashMap, Hashtable, SortedMap ─ TreeMap
```

슬라이드 그림에서 초록색은 인터페이스, 주황색은 그걸 구현한 클래스다. 체크 표시한 게 제일 많이 쓰는 것들이다.

쓰는 방법은 짱구 때 배운 업캐스팅이랑 같다. 초록색 인터페이스로 참조하고, 생성은 구체적인 구현 클래스로 한다.

```java
List<String> a = new ArrayList<>();
List<String> b = new LinkedList<>();
```

`List`로 참조해두면 안에서 ArrayList로 구현됐는지 LinkedList로 구현됐는지 상관없이 쓰는 법이 같다. 내부 사정은 메모리가 안다고 하셨다.

### 안 쓰는 것들: Vector, Stack, Hashtable, Properties

그림에는 있지만 지금은 안 쓰는 것들이 있다. Vector, Stack, Hashtable, Properties는 컬렉션 프레임워크(JDK 1.2)가 만들어지기 전부터 있던 것들이라 이름도 컬렉션 명명법을 안 따른다. Hashtable은 `~Map`이 아니고 Properties도 그렇다. 이름만 봐도 "아, 옛날 거구나" 하면 된다.

기존 코드 때문에 남겨둔 것뿐이고 사용을 권장하지 않는다(deprecated 취급). 자바가 1.0부터 지금까지 계속 발전하면서, 안 쓰는 건 엄격하게 쓰지 말라고 권한다고 하셨다.

### Map은 Collection의 자식이 아니다

그림을 보면 궁금한 게 하나 생겨야 한다고 하셨다. List, Set, Map을 컬렉션의 대표 자료구조라고 했는데, Map은 Collection 밑에 없다. 연결선이 아예 없다.

Map은 키와 값을 쌍으로 저장해서 저장 방식 자체가 다르기 때문이다. 대신 Collection 모양으로 가져올 수 있는 메소드 세 개를 제공한다.

| 메소드 | 리턴 타입 | 뭘 주나 |
| --- | --- | --- |
| `keySet()` | `Set<K>` | 키 전체. 키는 중복이 안 되니까 Set |
| `values()` | `Collection<V>` | 값 전체. 값은 중복될 수 있어서 Set이 아니다 |
| `entrySet()` | `Set<Map.Entry<K,V>>` | 키와 값 한 쌍(엔트리) 전체 |

엄밀하게는 자식이 아니지만 이 세 가지로 컬렉션처럼 쓸 수 있게 만들어둔 거다.

## Collection 인터페이스의 공통 메소드

| 분류 | 메소드 | 설명 |
| --- | --- | --- |
| 기본 연산 | `size()`, `isEmpty()`, `contains(o)` | 몇 개? 비었어? 이거 있어? |
| | `add(e)`, `remove(o)`, `iterator()` | 넣어줘, 지워줘, 하나씩 돌려줘 |
| 벌크 연산 | `addAll(c)`, `containsAll(c)`, `removeAll(c)`, `clear()` | 몽땅 넣어줘, 이거 다 있어? 몽땅 지워줘, 싹 비워줘 |
| 배열 연산 | `toArray()` | 배열로 바꿔줘 |

여기서 교수님께서 물어보셨다. `get(int i)`처럼 인덱스를 쓰는 게 있나? 없다. Set이나 Queue는 인덱스 개념이 없으니까, 모든 컬렉션의 공통 기능만 모아둔 Collection에 인덱스 메소드를 넣으면 큰일 난다.

`addAll(Collection<? extends E> c)`에 와일드카드가 보인다. 이제 이게 무슨 뜻인지 읽을 수 있다.

## List

List는 배열 같은 거다. 저장 순서가 유지되고, 인덱스로 접근하고, 중복을 허용한다.

| 기능 | 메소드 | 설명 |
| --- | --- | --- |
| 추가 | `add(e)` / `add(index, e)` | 맨 끝에 추가 / 그 자리에 끼워 넣기 |
| | `set(index, e)` | 그 자리를 새 객체로 바꾸기 |
| 검색 | `get(index)`, `contains(o)`, `isEmpty()`, `size()` | |
| 삭제 | `remove(index)`, `remove(o)`, `clear()` | |

### ArrayList

배열의 장점(인덱스)이랑 리스트의 장점(넣었다 뺐다)을 합친 가변 크기 배열이다. 저장 용량을 넘으면 알아서 늘어난다.

```java
List<String> list = new ArrayList<>();
list.add("MILK");
list.add("BREAD");
list.add("BUTTER");            // [MILK, BREAD, BUTTER]
list.add(1, "APPLE");          // 인덱스 1에 삽입 -> [MILK, APPLE, BREAD, BUTTER]
list.set(2, "GRAPE");          // 인덱스 2를 대체 -> [MILK, APPLE, GRAPE, BUTTER]
list.remove(3);                // 인덱스 3 삭제  -> [MILK, APPLE, GRAPE]
```

파이썬 리스트 쓰는 거랑 거의 같다. 대신 대괄호 `list[i]`가 아니라 `list.get(i)`로 꺼낸다.

수업 때는 과일 이름으로 해봤다. 사과, 포도, 오렌지를 넣고 `set(1, "키위")`로 바꾸면 사과, 키위, 오렌지가 되고, 다시 무화과로 바꾸면 사과, 무화과, 오렌지가 된다.

배열을 리스트로 한 번에 바꾸려면 `Arrays.asList`를 쓴다. `add`를 스무 번 치기 귀찮을 때 쓰는 벌크용 표현이다.

```java
String[] ar = {"aaa", "bbb", "ccc", "ddd"};
List<String> slist = Arrays.asList(ar);
List<String> list = new ArrayList<>(Arrays.asList(ar));   // 공지 예제는 이렇게 한 번 더 감쌌다
```

왜 한 번 더 감싸는지는 수업 때 직접 에러를 겪어서 알게 됐다. [3주차 과제 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-20-week3-assignment %})에 정리했다.

### LinkedList

```java
List<String> list = new LinkedList<>();   // 바뀌는 건 여기뿐
```

양방향 포인터로 앞뒤를 체인처럼 연결한 리스트다. 중간에 넣거나 지울 때 바로 앞뒤 링크만 바꾸면 되니까, 삽입과 삭제가 자주 일어나는 곳에서는 ArrayList보다 성능이 좋다. List랑 Queue를 둘 다 구현해서 스택, 큐로도 쓸 수 있다.

코드는 생성하는 부분만 다르고 나머지는 똑같다. List 인터페이스로 접근하니까.

### Vector는 왜 안 쓰나

Vector는 ArrayList랑 거의 같은데 메소드가 `synchronized`로 동기화돼 있다. 동기화(synchronization)는 영상, 음성, 자막이 따로 놀지 않게 맞추는 것처럼, 여러 스레드가 동시에 접근해도 안전하게 맞춰주는 거다.

| | Vector | ArrayList |
| --- | --- | --- |
| 도입 | JDK 1.0 | JDK 1.2 |
| 용량 증가 | 100%씩 | 50%씩 |
| 동기화 | O | X |
| 현업에서는 | Stack과 함께 사용 비권장 | 필요하면 동기화 처리해서 사용 |

동기화가 돼 있으면 좋을 것 같은데, 제대로 안 쓰면 오히려 시스템에 해가 되는 경우가 많이 보고돼서 쓰지 말라고 한다. 싱글 스레드에서는 쓸데없이 느리기만 하다. 대신 `Collections.synchronizedList(list)`처럼 동기화를 따로 걸어서 쓴다. 스레드는 나중에 다룬다고 하셨다.

C++의 vector는 지금도 쓴다. 이름만 같고 완전히 다른 거다.

## 반복하는 방법들

인덱스로 도는 `for (int i = 0; ...)`는 List에서만 된다. 컬렉션을 쓰다 보면 더 편한 방법을 쓰게 된다.

```java
List<Integer> num = new ArrayList<>(Arrays.asList(1, 2, 3, 4));

for (int i = 0; i < num.size(); i++) System.out.println(num.get(i));   // 1. 인덱스
for (Integer i : num) System.out.println(i);                           // 2. for-each
num.forEach(i -> System.out.println(i));                               // 3. 람다식
num.forEach(System.out::println);                                      // 4. 메소드 참조

Iterator<Integer> e = num.iterator();                                  // 5. 반복자
while (e.hasNext()) System.out.println(e.next());
```

3번, 4번은 한 줄이 곧 반복문이다. 교수님께서 이런 코드는 AI가 무수하게 만들어준다고, 이해 못 하는 걸 꾸역꾸역 쓰라고는 안 하겠지만 "아 이거 반복하라는 뜻이구나" 하고 알아볼 수 있어야 한다고 하셨다. 모르겠으면 for문으로 바꿔서 써도 된다.

Iterator는 반복자 패턴이라는 이름이 붙은 정식 패턴이다. `hasNext()`로 다음이 있는지 보고 `next()`로 꺼낸다.

## Set

수학의 집합이다.

- 저장 순서가 유지되지 않는다
- 중복을 허용하지 않는다. null도 하나만
- 순서가 없으니 `get(index)`가 없다

MILK, BREAD, APPLE, BUTTER가 있는 Set에 BUTTER를 또 넣으면 에러가 나는 게 아니라 그냥 안 들어간다. `add`가 false를 돌려줄 뿐이다.

인덱스가 없으니까 `for (int i...)`는 쓸 수가 없다. 그러다 보니 for-each를 쓰게 되고, 람다식을 쓰게 되고, Iterator를 쓰게 된다고 하셨다.

교수님께서는 1, 2, 3, 4, 5를 넣어도 1, 3, 5, 4, 2처럼 나올 수 있다고 하셨다. 재수가 좋으면 넣은 순서대로 나오는 것처럼 보일 수도 있는데, 애초에 순서를 유지할 이유가 없는 자료구조다. 실제로 작은 정수를 넣었더니 정렬된 것처럼 나와서 헷갈렸는데, 왜 그런지는 과제 글 Lab#5에 정리했다.

### Set 구현 클래스 고르기

| 클래스 | 특징 | 언제 |
| --- | --- | --- |
| `HashSet` | 해시 테이블에 저장. 성능이 제일 좋다. 순서 없음 | 기본 |
| `LinkedHashSet` | 해시 + 연결 리스트. 넣은 순서를 기억한다 | 중복은 빼면서 순서는 유지하고 싶을 때 |
| `TreeSet` | 이진 검색 트리. 넣으면 알아서 정렬된다 | 정렬된 상태로 관리하고 싶을 때 |
| `EnumSet` | 중복되지 않는 상수 그룹 | |

해시는 데이터를 빨리 찾으려고 메타 정보를 저장해두는 방식이다. 트리는 3이 있을 때 1을 넣으면 작은 쪽(왼쪽), 12를 넣으면 큰 쪽(오른쪽)으로 가는 식으로 정렬하면서 관리한다. 검색 효율은 높지만 전체 성능은 HashSet을 못 따라간다.

수업 때 TreeSet에 숫자를 뒤죽박죽 넣었더니 작은 수부터 정렬돼서 나왔다. 이름 앞에 뭐가 붙었는지 보면 성격이 보인다.

| 붙은 말 | 성격 |
| --- | --- |
| Hash | 검색 기능 향상 |
| Linked | 연결 리스트 구조로 순서 유지, 추가/삭제 향상 |
| Tree | 정렬시켜 관리해서 검색 향상 |
| LinkedHash | Hash + Linked |

그래서 특징을 하나하나 외우지 말고 대표 구현 클래스를 직접 써보라고 하셨다.

### 집합 연산

파이썬에서는 `union`, `intersection`을 썼는데 자바는 메소드 이름이 좀 생소하다.

| 연산 | 메소드 | 의미 |
| --- | --- | --- |
| 합집합 | `s1.addAll(s2)` | s1을 s1 ∪ s2로 |
| 교집합 | `s1.retainAll(s2)` | s1을 s1 ∩ s2로 |
| 차집합 | `s1.removeAll(s2)` | s1을 s1 - s2로 |
| 부분집합 | `s1.containsAll(s2)` | s2가 s1의 부분집합이면 true |

두세 번 보면 익숙해진다고 하셨다. 주의할 건 새 집합을 돌려주는 게 아니라 s1 자체를 바꾼다는 점이다. 이걸 모르고 연달아 쓰면 결과가 예상이랑 달라진다. 실습하면서 직접 겪었다.

HashSet은 `hashCode()`로 같은 객체인지 판단한다. 이것도 퀴즈에서 확인해봤다.

## Map

키와 값의 쌍으로 저장한다. 파이썬의 딕셔너리다. 전화번호부나 데이터베이스처럼 굉장히 많이 쓴다고 하셨다.

- 키는 중복 불가. 아이디 같은 거니까 겹치면 큰일 난다. 그래서 키는 Set으로 구현돼 있다
- 값은 중복 허용
- 같은 키로 다시 넣으면 마지막에 넣은 값으로 대체된다
- 저장 순서를 유지하지 않는다

| 기능 | 메소드 | 설명 |
| --- | --- | --- |
| 추가 | `put(key, value)` | add가 아니라 put. 키와 값 두 개를 넣는다 |
| 검색 | `get(key)`, `containsKey(key)`, `containsValue(v)` | |
| | `keySet()`, `values()`, `entrySet()` | 제일 많이 쓰는 세 가지 |
| | `size()`, `isEmpty()` | |
| 삭제 | `remove(key)`, `clear()` | |

키랑 값이 한 쌍으로 묶인 걸 엔트리(`Map.Entry`)라고 부른다. `Map.Entry`는 Map 안에 들어 있는 내부 인터페이스라서, 하나씩 꺼내면서 `getKey()`, `getValue()`로 쓴다.

```java
Map<Integer, String> map = new HashMap<>();   // 키와 값의 타입을 정해준다 (멀티 타입 파라미터)

for (Map.Entry<Integer, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
map.forEach((k, v) -> System.out.println(k + " : " + v));   // 람다식은 인자가 두 개
```

List나 Set은 값이 하나라 람다식 인자도 하나인데, Map은 키와 값 두 개를 받는다. 100 짱구, 200 유리, 300 맹구 같은 게 한 줄로 찍힌다. 인덱스가 없는 자료구조에서는 이게 제일 편하다고 하셨다.

### Map 구현 클래스 고르기

| 클래스 | 특징 |
| --- | --- |
| `HashMap` | 제일 많이 쓴다. 성능 제일 좋음. 순서 보장 안 함 |
| `LinkedHashMap` | 넣은 순서를 유지한다 |
| `TreeMap` | 키를 기준으로 정렬한다. 숫자, 알파벳 대문자, 소문자, 한글 순 |
| `Hashtable`, `Properties` | 옛날 것. Vector처럼 동기화돼 있지만 사용 비권장 |

수업 때 좋아하는 메뉴를 넣어보라고 해서 메가커피 할메가미숫커피, 녹차라떼 같은 걸 넣어봤다. HashMap은 순서가 뒤죽박죽이고 LinkedHashMap은 넣은 순서 그대로 나온다. 같은 키에 값을 또 넣으면 앞의 값이 지워지는 효과가 있다. Set이랑 똑같다.

HashMap의 키로는 String을 많이 쓴다. String은 문자열이 같으면 같은 객체로 보도록 `hashCode()`와 `equals()`가 재정의돼 있어서다. 내가 만든 클래스를 키로 쓰려면 이 두 개를 직접 오버라이딩해서 "같은 키"의 기준을 정해줘야 한다.

### null은 들어갈까

Arrays.asList로 `85.5, 90.0, 78.5, null`을 넣으면 null이 들어갈까? 들어간다. null은 어디에나 들어갈 수 있는 값이라고 하셨다. 다만 예외도 있다. Lab#7 테스트 코드에 나온 `Map.of("라면", 2000, ...)`는 키나 값에 null을 넣으면 `NullPointerException`이 난다. 직접 해보고 알았다. 그래서 과제에서 null 값을 테스트할 때는 LinkedHashMap에 put으로 넣었다.

## Supplement: 더 알아둘 것

슬라이드 뒤쪽 보충 자료는 수업 때 다루지 않았는데 제목만 적어둔다.

- **Comparable과 Comparator:** TreeSet, TreeMap은 정렬 기준이 있어야 한다. 기본 기준은 `Comparable`의 `compareTo()`, 다른 기준은 `Comparator`의 `compare()`로 정한다. 기준 없는 객체를 TreeSet에 넣으면 `ClassCastException`이 난다
- **Stack과 Queue:** Stack은 Vector를 상속받아서 비권장이고 `ArrayDeque`를 쓴다. Queue는 LinkedList, ArrayDeque를 쓴다
- **동기화된 컬렉션:** `Collections.synchronizedList()`로 감싸거나, `java.util.concurrent`의 `ConcurrentHashMap`, `CopyOnWriteArrayList`를 쓴다

## 코딩 테스트 연습

코딩 테스트는 보통 파이썬이나 C++로 보지만, 자바에서 배운 컬렉션으로 풀어보라고 노션에 "자바 컬렉션 프레임워크(JCF) 실습 추천 문제" 페이지를 만들어두셨다. 자프실1 때는 중간고사, 기말고사도 이런 방식으로 봤다고 한다.

원래는 백준 문제였는데 백준 접속이 안 돼서 같은 개념을 연습할 수 있는 프로그래머스 문제로 바꿔두셨다. 프로그래머스에 로그인해서 문제 이름으로 검색하면 된다.

| 번호 | 연습할 자료구조 | 원래 백준 문제 | 프로그래머스 대체 문제 | 난이도 |
| --- | --- | --- | --- | --- |
| 1 | Set / Hash | 1764 듣보잡 | 폰켓몬 | Lv.1 |
| 2 | Set, 정렬이나 Map | 1269 대칭 차집합 | 완주하지 못한 선수 | Lv.1 |
| 3 | Map (양방향 조회) | 1620 포켓몬 마스터 | 달리기 경주 | Lv.1 |
| 4 | Map (빈도수 세기) | 10816 숫자 카드 2 | 귤 고르기 | Lv.2 |
| 5 | List 커서 / Stack | 1406 에디터 | [카카오] 표 편집 (입문용은 올바른 괄호) | Lv.3 (Lv.2) |
| 6 | List 순환 / Queue | 1158 요세푸스 문제 | 프로세스 | Lv.2 |

2번 완주하지 못한 선수는 동명이인이 있어서 그냥 Set으로 바꾸면 안 된다는 게 포인트였다. Set은 중복을 없애버리니까, 정렬해서 비교하거나 Map으로 이름별 인원수를 세야 한다.

노션에 적힌 학습 팁은 이랬다.

- 프로그래머스는 언어 제한이 없어서 파이썬으로 풀고 싶은 유혹이 들겠지만, 이번만큼은 반드시 자바 컬렉션을 직접 써서 풀기
- 알고리즘 문제를 자주 풀면 Lv.2나 Lv.3, 처음이면 Lv.1부터
- HashSet, HashMap, ArrayList, LinkedList, ArrayDeque 중에 뭘 고를지 내부 동작 원리와 시간 복잡도(O(1), O(log N), O(N))를 생각하면서 고르기
- 왜 이 상황에서 이 컬렉션을 골랐는지 이유를 적어두면 나중에 기술 면접 대비에도 도움이 된다

자료구조를 안 쓰고 막 풀어도 통과는 되겠지만, 그래도 자료구조를 써보면서 마무리하라고 하셨다. 남이 푼 걸 검색해서 보지 말고.

## 정리

- 컬렉션은 제네릭으로 만든 자바 표준 자료구조다. 인터페이스로 참조하고 구현 클래스로 생성한다
- List는 순서 O, 중복 O, 인덱스 O. 보통 ArrayList, 삽입/삭제가 잦으면 LinkedList
- Set은 순서 X, 중복 X, 인덱스 X. HashSet, 순서가 필요하면 LinkedHashSet, 정렬이 필요하면 TreeSet
- Map은 키-값 쌍. 키 중복 X, 값 중복 O. Collection의 자식은 아니지만 keySet, values, entrySet으로 가져온다
- Vector, Stack, Hashtable, Properties는 옛날 것이라 안 쓴다
- 인덱스가 없는 컬렉션은 for-each, Iterator, 람다식으로 돈다
