---
title: "[자프실2] 배틀 리팩토링: 삼국지, 해리포터, 몬스터 배틀"
date: 2026-09-15 20:30:00 +0900
series: "JAVA프로그래밍및실습II"
categories:
  - 강의
tags:
  - Java
  - OOP
  - 리팩토링
  - 상속
  - 과제
excerpt: "교수님이 리팩토링하라고 주신 지난 학기 배틀 코드 3개에서 숨은 문제를 테스트 코드로 재현하고, 원본은 그대로 둔 채 새 프로젝트에서 고쳤다."
toc: true
toc_sticky: true
---

2주차 과제 중 리팩토링 부분이다. 과제 전체(Vehicle-Driver 리팩토링, 짱구네 집, 하늘 싱글톤)는 [2주차 과제 글]({{ site.baseurl }}{% post_url java-programming-2/2026-09-15-week2-assignment %})에 정리했다.

교수님이 리팩토링하라고 주신 코드는 지난 학기 자프실1 배틀 과제 코드 세 개였다. 폴더 이름에 힌트가 붙어 있었다.

| 폴더 | 게임 | 힌트 |
| --- | --- | --- |
| 1_상속 | 삼국지 배틀 (RTKBattle) | 상속을 잘못 썼다 |
| 2_상속 | 해리포터 배틀 (OOP_Battle) | 상속을 1번과 다른 방식으로 잘못 썼다 |
| 3_모듈화_비교문 | 몬스터 배틀 (battle) | 모듈화가 안 돼 있고 비교문이 많다 |

세 개 다 실행은 잘 된다. 교수님도 "실행은 너무 잘 되는데 열어보면 치명적인 문제가 있다"고 하셨다.

압축을 풀어서 이클립스로 가져오니 1번이랑 3번은 한글이 다 깨졌다. 파일이 MS949로 저장돼 있어서 UTF-8로 바꾼 사본을 `Battle1_RTKBattle_Original`처럼 따로 만들었다. 원본 코드는 건드리지 않고, Vehicle-Driver 때처럼 문제를 재현하는 `BugDemo`를 넣어 확인한 다음 `Battle1_RTKBattle_Refactor`처럼 새 프로젝트에서 고쳤다.

## 1. 삼국지 배틀: 자식이 부모 필드를 또 선언했다

### 지금 코드 상태

![원본 삼국지 배틀 Player 쪽 구조]({{ '/assets/images/java-programming-2/battle1-original-player-uml.png' | relative_url }})

![원본 삼국지 배틀 무기 쪽 구조]({{ '/assets/images/java-programming-2/battle1-original-weapon-uml.png' | relative_url }})

![원본 삼국지 배틀 게임 창]({{ '/assets/images/java-programming-2/battle1-original-game.png' | relative_url }})

실행하면 캐릭터 두 명이 정해지고, 공격하기와 무기 사용 버튼으로 싸운다. 겉으로 보기에는 멀쩡하다.

`Player`에 이름, hp, power, mp, 이미지, 전용무기가 있는데 관우, 유비, 장비, 조조가 이걸 전부 또 선언하고 있었다. 관우 코드는 이렇다.

```java
public class 관우 extends Player {

	private static final String String = null;
	public String 이름;
	private int hp;
	private int power;
	private int mp;
	private String imgFile1;
	private String imgFile2;
	public 청룡언월도 청룡 = new 청룡언월도(50,90);

	public 관우(String 이름, int hp, int power, int mp, String imgFile1, String imgFile2, 무기 전용무기) {
		super(이름, hp, power, mp, imgFile1, imgFile2, 전용무기);
	}

	public void 무기로공격(Player target) {
		int 운장mp = this.getMP();
		int targethp = target.getHP();

		운장mp -= 청룡.get필요mp();
		targethp -= 청룡.get데미지();

		this.setMP(운장mp);
		target.setHP(targethp);
	}
}
```

### 코드 읽으면서 이상했던 부분

- 관우, 유비, 장비, 조조가 `Player`에 있는 필드 6개를 똑같이 다시 선언했다. 수업에서 교수님이 PlantUML로 보여주신 "Player에 있는데 유비에서 또 선언한" 그 문제다.
- 생성자로 `전용무기`를 받아서 부모에 넘기는데, 정작 `무기로공격`은 클래스 안에서 따로 만든 무기(`청룡`, `쌍고` 등)를 쓴다.
- `무기로공격`은 네 클래스에 변수 이름(운장mp, 현덕mp...)만 바꿔서 그대로 복사돼 있다.
- 청룡언월도, 장팔사모, 쌍고검, 청강검도 부모 `무기`에 있는 `필요mp`, `데미지`를 또 선언했다.
- 관우에는 `private static final String String = null;`이라는 쓸 데 없는 필드가 있다.
- `Win`에서 플레이어와 상대를 고르는 if문은 조건마다 `Math.random()`을 새로 부르고, 마지막 조건이 `else if (플레이어생성 != 3)`이라 상대가 안 정해질 수 있어 보였다.
- 공격 버튼을 누르면 내 공격, 상대 반격, 내 hp 확인, 상대 hp 확인 순서로 진행된다.
- `Main`은 `Win`과 다른 hp로 캐릭터를 또 만들고, 값 하나만 출력하고 끝난다.

### 실행 전에 예상한 것

- **문제1:** 관우 객체를 `Player` 타입으로 보면 부모 쪽 이름("관우")이 나오고, `관우` 타입으로 보면 자식이 새로 만든 이름이 나오는데 아무도 값을 안 넣었으니 null일 것 같다.
- **문제2:** 유비에게 필요mp 10, 데미지 999짜리 무기를 넣어줘도, 유비 클래스 안에서 만든 쌍고검(30, 30)으로 공격해서 조조 hp는 30만 깎이고 mp도 30을 쓸 것 같다.
- **문제3:** `Win`의 선택 조건을 그대로 옮겨서 10만 번 돌리면 상대가 안 정해지는 경우가 몇 퍼센트는 나올 것 같다.
- **문제4:** 둘 다 hp 10일 때 내가 먼저 상대를 쓰러뜨려도 상대가 반격하고, 내 hp부터 확인하니까 결과는 패배로 나올 것 같다.

### 실제 결과

![원본 삼국지 배틀 BugDemo 실행 결과]({{ '/assets/images/java-programming-2/battle1-bugdemo.png' | relative_url }})

예상대로 나왔다. 같은 객체인데 보는 타입에 따라 이름이 "관우"도 되고 null도 되는 게 짱구 퀴즈에서 본 거랑 연결돼서 신기했다. 문제3은 10만 번 중 2097번(2.1%), 그러니까 대충 50번 실행하면 한 번은 게임이 켜지자마자 에러가 난다. hp가 -40, -20처럼 음수로 내려가는 것도 같이 보였다.

### 어떻게 고쳤나

**캐릭터마다 다른 건 능력치와 무기뿐이라, 공통 동작은 전부 `Player`에 두고 자식은 값만 넘기게 했다.** 필드는 전부 private으로 막아서 자식이 다시 선언할 이유가 없게 했다.

```java
public abstract class Player {

	public static final int MAX_MP = 100;

	private final String 이름;
	private final int maxHp;
	private int hp;
	private final int power;
	private int mp;
	private final String imgFile1;
	private final String imgFile2;
	private final 무기 전용무기;

	// 네 클래스에 복사돼 있던 코드를 한 곳으로. 넣어준 전용무기를 쓴다.
	public boolean 무기로공격(Player target) {
		if (!can무기사용()) return false;
		mp -= 전용무기.get필요mp();
		target.피해입기(전용무기.get데미지());
		return true;
	}

	protected void 피해입기(int damage) {
		hp = Math.max(0, hp - damage);
	}
	// 생성자와 getter는 생략
}
```

```java
public class 관우 extends Player {

	public 관우() {
		this(new 청룡언월도(50, 90));
	}

	public 관우(무기 전용무기) {
		super("관우", 500, 50, 10, "관우.png", "관우2.png", 전용무기);
	}
}
```

- 무기 자식 클래스의 중복 필드도 지우고, 무기 성능은 만든 뒤 바뀔 일이 없어서 `final`로 바꿨다.
- hp는 0 아래로, mp는 100 위로 안 가게 막았다.
- `Win` 안에 섞여 있던 대진 짜기와 턴 규칙을 `Battle` 클래스로 뺐다. 화면 없이도 테스트할 수 있다.

```java
// 조건마다 Math.random()을 부르는 대신, 섞어서 앞의 두 명을 뽑는다
public static Battle 랜덤대진() {
	List<Player> 후보 = new ArrayList<Player>(Arrays.asList(new 관우(), new 유비(), new 조조(), new 장비()));
	Collections.shuffle(후보);
	return new Battle(후보.get(0), 후보.get(1));
}

// 공격 버튼과 무기 버튼에 복사돼 있던 흐름을 하나로. 상대가 쓰러지면 반격하기 전에 끝낸다.
public 결과 한턴(boolean 무기사용) {
	if (!행동(플레이어, 상대, 무기사용)) return 결과.MP부족;
	if (상대.isDead()) {
		기록("당신은 삼국을 통일했습니다! 축하합니다.");
		return 결과.승리;
	}
	행동(상대, 플레이어, 상대.can무기사용());
	플레이어.회복MP(10);
	상대.회복MP(10);
	if (플레이어.isDead()) {
		기록("당신이 패배함으로써 삼국 통일의 꿈이 물건너갔습니다...");
		return 결과.패배;
	}
	return 결과.계속;
}
```

- `Win`은 그리기와 버튼 연결만 한다. 똑같이 생긴 진행바, 초상화, 버튼 만드는 코드는 작은 메소드로 묶었다.
- `Main`은 게임을 시작하는 한 줄로 정리했다.

### 고친 후

![리팩토링한 삼국지 배틀 Player 쪽 구조]({{ '/assets/images/java-programming-2/battle1-refactor-player-uml.png' | relative_url }})

![리팩토링한 삼국지 배틀 무기 쪽 구조]({{ '/assets/images/java-programming-2/battle1-refactor-weapon-uml.png' | relative_url }})

![리팩토링한 삼국지 배틀 게임 창]({{ '/assets/images/java-programming-2/battle1-refactor-game.png' | relative_url }})

화면 구성은 원본과 똑같다. 오른쪽 기록 창에 턴마다 누가 누구를 공격했는지 찍힌다.

실행 전에 예상한 것:

- 이름은 `Player`에만 있으니 어느 타입으로 봐도 "관우"
- 필요mp 10, 데미지 77 무기를 넣으면 조조 hp 77이 깎이고 mp 10을 쓴다
- 섞어서 뽑으니 10만 번 돌려도 상대가 비거나 같은 캐릭터끼리 붙는 경우는 0번
- 상대가 먼저 쓰러지면 반격 없이 승리, 내 hp는 10 그대로

![리팩토링한 삼국지 배틀 RefactorCheck 실행 결과]({{ '/assets/images/java-programming-2/battle1-refactorcheck.png' | relative_url }})

예상한 대로 전부 고쳐졌다.

## 2. 해리포터 배틀: 넣은 값이 무시되고 static이 공유된다

### 지금 코드 상태

![원본 해리포터 배틀 구조]({{ '/assets/images/java-programming-2/battle2-original-uml.png' | relative_url }})

![원본 해리포터 배틀 실행 화면]({{ '/assets/images/java-programming-2/battle2-original-game.png' | relative_url }})

원본 `Main`을 그대로 실행한 화면이다. 창이 뜨다가 콘솔에 Exception이 나고 Harry 이미지 자리는 비어 있다(문제3). 가운데 능력치도 `Main`에서 넣은 500, 30이 아니라 100, 50이다(문제1).

```java
public abstract class Player implements Attackable {
	public String name;
	private int hp;
	private int power;
	public static String imgFile;

	public ArrayList<Weapon> weapons;
	...
}

public class Harry extends Player {

	public Harry (String name, int hp, int power) {
		super("Harry", 100, 50);
	}

	@Override
	public void attack(Player attacker, Player target) {
		// TODO Auto-generated method stub
	}
}
```

### 코드 읽으면서 이상했던 부분

- `Harry(String name, int hp, int power)`가 받은 값을 버리고 `super("Harry", 100, 50)`으로 넘긴다. Hermione, Ron, Fluffy도 똑같다. `Main`에서는 `new Harry("Harry", 500, 30)`으로 만들고 있어서 의도한 값이랑 실제 값이 다를 것 같았다.
- 이미지 파일 이름 `imgFile`이 `public static`이다. static은 모든 객체가 같이 쓰는 값이라 Harry와 Fluffy가 같은 이미지를 가질 수밖에 없다. 그런데 이 값을 넣어주는 곳도 없다.
- `Attackable`의 `attack(attacker, target)`은 모든 캐릭터가 TODO로 비워뒀고, 실제로는 `Player`의 `attack(target)`을 쓴다. 게다가 `View`의 `Main`까지 `Attackable`을 구현하고 있다. Main은 공격하는 캐릭터가 아니니까 is-a가 안 맞는다.
- `Weapon`도 `Attackable`을 구현하는데, `wand`, `gun`, `flute`는 `Weapon`을 상속하지 않은 빈 클래스다. `weapons` 리스트는 선언만 있고 한 번도 안 쓴다.
- `GameWindow`는 공격하기 전에 승패를 확인해서, 상대 hp가 0이 된 뒤 한 번 더 눌러야 끝난다. 두 공격 버튼을 아무 때나 누를 수 있다.

### 실행 전에 예상한 것

- **문제1:** Harry에 hp 500, power 30을 넣어도 100, 50이 들어가고, Hermione에 이름이나 값을 뭘 넣어도 "Hermione", 100, 50일 것 같다.
- **문제2:** Harry에 img1.png, Fluffy에 img2.png를 넣으면 static이라 둘 다 마지막에 넣은 img2.png가 된다.
- **문제3:** Main처럼 이미지를 안 넣으면 경로가 `/images/null`이 돼서 파일을 못 찾고, `new ImageIcon(null)`에서 NullPointerException이 날 것 같다.
- **문제4:** `attack(harry, fluffy)`는 비어 있어서 hp가 안 줄고, `attack(fluffy)`는 power 50만큼 준다.

### 실제 결과

![원본 해리포터 배틀 BugDemo 실행 결과]({{ '/assets/images/java-programming-2/battle2-bugdemo.png' | relative_url }})

예상대로였다. 문제3은 원래 `Main`을 그대로 실행해도 창이 뜨는 도중에 멈추는 원인이었다. static 하나 때문에 이미지가 섞이고, 값을 안 넣으니 에러까지 이어졌다.

### 어떻게 고쳤나

- `imgFile`을 static에서 인스턴스 필드로 바꾸고, 이름과 함께 private으로 막았다.
- 이름과 이미지는 캐릭터마다 정해져 있으니 생성자에서 받지 않고, 능력치만 받아서 그대로 넘긴다.

```java
public class Harry extends Player {

	public Harry() {
		this(500, 30);
	}

	public Harry(int hp, int power) {
		super("Harry", hp, power, "img1.png");
	}
}
```

- `Attackable`은 실제로 쓰는 `attack(target)` 하나만 약속하게 바꾸고, 비어 있던 `attack(attacker, target)`을 모든 캐릭터에서 지웠다. `Main`은 `Attackable`을 구현하지 않는다.
- 무기는 공격하는 주체가 아니라서 `Weapon`에서 `Attackable`을 뺐다. `Wand`, `Gun`, `Flute`는 `Weapon`을 상속하게 하고, 쓰지 않던 `weapons` 리스트 대신 무기 하나를 장착하면 그만큼 공격력이 오르게 했다.

```java
public abstract class Weapon {
	private final String name;
	private final int power;
	...
}

public class Wand extends Weapon {
	public Wand() {
		super("지팡이", 20);
	}
}
```

- `GameWindow`는 공격한 다음에 승패를 확인하고, 두 버튼을 번갈아 누르게 했다. 기본 생성자에서 이미지를 하드코딩하던 코드와 콘솔 디버그 출력, 쓰지 않는 import도 정리했다.

### 고친 후

![리팩토링한 해리포터 배틀 구조]({{ '/assets/images/java-programming-2/battle2-refactor-uml.png' | relative_url }})

![리팩토링한 해리포터 배틀 게임 창]({{ '/assets/images/java-programming-2/battle2-refactor-game.png' | relative_url }})

두 캐릭터 이미지가 다 보이고, Harry가 넣은 값 그대로 hp 500, power 30으로 시작한다. 공격한 쪽 버튼은 비활성화돼서 번갈아 누르게 된다.

실행 전에 예상한 것:

- 넣은 값(500/30, 700/40)이 그대로 들어간다
- Harry는 img1.png, Fluffy는 img2.png로 따로 가진다
- 두 이미지 모두 경로를 찾는다
- 맨손 공격은 30, 지팡이(20)를 들면 50만큼 준다
- Main은 Attackable이 아니고, Wand, Gun, Flute는 Weapon이다

![리팩토링한 해리포터 배틀 RefactorCheck 실행 결과]({{ '/assets/images/java-programming-2/battle2-refactorcheck.png' | relative_url }})

## 3. 몬스터 배틀: 복사된 비교문과 화면에 묶인 캐릭터

### 지금 코드 상태

![원본 몬스터 배틀 구조]({{ '/assets/images/java-programming-2/battle3-original-uml.png' | relative_url }})

![원본 몬스터 배틀 게임 창]({{ '/assets/images/java-programming-2/battle3-original-game.png' | relative_url }})

몇 턴 진행한 화면이다. 턴마다 기록 창에 공격, 연주, 치료 결과가 쌓인다.

`Player.attack`은 랜덤 값 r에 따라 if문 6개로 나뉘는데, 데미지에 더하는 숫자만 0~5로 다르고 나머지는 전부 똑같다.

```java
int r=(int) (Math.random() * 1000) % 6;
if(r==0) {
	target.hp-=(this.power-target.protection);
	System.out.printf("%s의 공격이 %s에게 %d 만큼의 데미지를 입혔습니다!",this.name,target.name,this.power-target.protection);
	System.out.println();
	String str= this.name +"의 공격이" + target.name+ "에게"+(this.power-target.protection)+"만큼의 데미지를 입혔습니다!\n";
	Mywin.ta.append(str);
}
else if(r==1) {
	target.hp-=(this.power-target.protection+1);
	...
}
// r==5까지 똑같이 반복
```

### 코드 읽으면서 이상했던 부분

- **복사된 비교문:** `Player.attack`에 6개, `Jester`에 10개, `BoneCourtier`에 6개, `Vestal`에 6개. 전부 숫자 하나만 다르다.
- **화면에 묶인 캐릭터:** 캐릭터 클래스가 화면 클래스의 `static` 텍스트창 `Mywin.ta`에 직접 글을 쓴다. `playerStress`는 `JLabel`을 받아서 이미지까지 바꾼다. 창 없이는 캐릭터 코드를 돌려볼 수가 없어 보였다.
- **제한 없는 값:** 성녀 치료는 최대 hp를 확인하지 않고, 광대 연주는 스트레스를 0 밑으로도 뺀다.
- **상속 문제도 있다:** `Heroes`가 부모 `Player`에 있는 `stress`를 `private int stress`로 또 선언했고, `Heroes.playerStress(Player)`는 어디서도 안 불린다. `Monster` 생성자는 `super(...)`를 안 쓰고 필드를 하나씩 다시 넣는다.
- **붕괴 횟수가 이상한 곳에 쌓인다:** `playerStress(Player pl, ...)`는 영웅 `pl`의 스트레스를 보는데, 붕괴 횟수 `stressCount`는 `this`, 즉 공격한 몬스터 쪽 값을 쓴다.
- **화면 코드 복사:** `Mywin`은 캐릭터 8명마다 라벨, 진행바, 버튼 코드가 복사돼 있고, 다음 차례를 정하는 같은 if문이 `runStage`, `heroAttack`(3번), `monsterAttack`까지 5군데에 있다. "p1이나 p4면 공격, p3면 치료"처럼 자리 번호로 역할을 나눈다.
- **몬스터 공격 대상:** 죽은 영웅은 `heroes` 목록에서만 지우고 `heroButtons` 목록은 그대로 둔 채 `heroButtons.get(랜덤)`을 누른다. `Main` 맨 아래에도 "인덱스 문제로 오류남 잘 해보시길..."이라는 주석이 있었다.
- 이벤트 처리 중에 `Thread.sleep(1000)`을 불러서 그동안 화면이 멈춘다.

### 실행 전에 예상한 것

- **문제1:** 창을 안 띄우고 `attack()`을 부르면 `Mywin.ta`가 null이라 NullPointerException이 날 것 같다.
- **문제2:** 최대 hp 33인 Crusader를 30번 치료하면 한 번에 0~5씩 계속 올라가서 33을 훨씬 넘을 것 같다.
- **문제3:** 스트레스 0인 영웅에게 10번 연주하면 음수가 될 것 같다.
- **문제4:** `Player`와 `Heroes` 둘 다에 stress 필드가 선언돼 있다.
- **문제5:** 영웅 한 명이 죽으면 랜덤 범위는 0~2인데 버튼 목록은 4개 그대로라, 죽은 영웅 버튼이 3번에 1번꼴로 눌리고 마지막 Crusader 버튼은 아예 안 눌릴 것 같다.
- **문제6:** 같은 몬스터가 스트레스 30인 영웅 두 명에게 차례로 판정을 하면, 첫 번째 영웅이 붕괴한 순간(4번 중 3번) 몬스터 쪽 횟수가 1이 돼서 두 번째 영웅은 판정을 건너뛸 것 같다.

### 실제 결과

![원본 몬스터 배틀 BugDemo 실행 결과]({{ '/assets/images/java-programming-2/battle3-bugdemo.png' | relative_url }})

예상대로 나왔다. 문제2는 hp가 최대 33인데 91까지 올라갔고, 문제3은 스트레스가 -33이 됐다. 문제5는 죽은 영웅 버튼이 10만 번 중 33414번(33.4%) 눌렸다. 죽은 영웅 버튼을 누르면 "이미 사망하였습니다" 메시지만 나오고 몬스터 차례가 넘어가지 않아서 게임이 멈춘다. 코드를 짠 사람이 주석에 적어둔 "인덱스 문제"가 바로 이거였다. 문제6은 1000번 중 754번으로 4번 중 3번(75%)에 가깝게 나왔다.

### 어떻게 고쳤나

**복사된 비교문은 계산식 한 줄로 바꿨다.**

```java
@Override
public void attack(Player target) {
	if (!hits(target)) {
		BattleLog.write("공격이 빗나갔습니다!");
		return;
	}
	BattleLog.write("공격이 성공했습니다!");
	int damage = Math.max(0, power - target.protection + roll(6));
	target.takeDamage(damage);
	BattleLog.write(name + "의 공격이 " + target.name + "에게 " + damage + "만큼의 데미지를 입혔습니다!");
	...
}
```

광대는 `roll(10) + 1`, 해골 궁정인은 `5 + roll(6)`, 성녀는 `roll(6)` 한 줄씩이다.

**캐릭터와 화면을 떼어냈다.** 캐릭터는 `BattleLog`에만 기록하고, 화면은 리스너를 등록해서 받아 간다.

```java
public final class BattleLog {

	private static Consumer<String> listener = message -> {
	};

	public static void setListener(Consumer<String> newListener) {
		listener = newListener;
	}

	public static void write(String message) {
		System.out.println(message);
		listener.accept(message);
	}
}
```

```java
// Mywin에서
BattleLog.setListener(message -> ta.append(message + "\n"));
```

**값에 한계를 두고, 스트레스는 영웅만 갖게 했다.** 치료는 `Math.min(maxHp, hp + amount)`, 스트레스는 0~100 사이로만 움직인다. `stress`와 붕괴 여부는 `Heroes`에만 두고, 붕괴 판정은 영웅이 자기 값으로 한다.

```java
public abstract class Heroes extends Player {

	private int stress;
	private boolean broken;   // 원래는 공격한 몬스터 쪽 stressCount를 썼다

	protected void addStress(int amount) {
		stress = Math.max(0, Math.min(MAX_STRESS, stress + amount));
	}

	public StressState checkStress() { ... }
}
```

화면 이미지를 바꾸는 일은 `checkStress()`가 돌려준 결과(평소, 영웅의기상, 붕괴, 심장마비)를 보고 `Mywin`이 한다.

**몬스터가 공격할 영웅은 살아 있는 영웅 중에서 직접 고른다.** 버튼 목록 인덱스를 쓰지 않으니 어긋날 일이 없다.

```java
public Player chooseTarget(List<Player> players) {
	List<Player> aliveHeroes = new ArrayList<Player>();
	for (Player player : players) {
		if (player instanceof Heroes && !player.isDead()) aliveHeroes.add(player);
	}
	return aliveHeroes.isEmpty() ? null : aliveHeroes.get(roll(aliveHeroes.size()));
}
```

**`Mywin`의 복사된 코드는 반복문과 메소드 하나로 줄였다.**

- 캐릭터 한 명의 라벨, 진행바, 버튼을 `Slot`으로 묶어서 반복문으로 8개를 만든다.
- 다음 차례를 정하는 코드는 `nextTurn()` 하나로 모았다. 몬스터 차례는 자동으로 처리하고, 라운드 중간에 죽은 캐릭터는 건너뛴다.
- 자리 번호 비교 대신 캐릭터가 `targetsAllies()`(아군 대상인지)와 `getTurnMessage()`(차례 안내 문구)로 알려준다. 광대와 성녀만 이 두 메소드를 오버라이딩한다.
- `Thread.sleep`은 지웠다.

### 고친 후

![리팩토링한 몬스터 배틀 구조]({{ '/assets/images/java-programming-2/battle3-refactor-uml.png' | relative_url }})

![리팩토링한 몬스터 배틀 게임 창]({{ '/assets/images/java-programming-2/battle3-refactor-game.png' | relative_url }})

원본과 같은 배치로, 같은 순서대로 턴이 돌아간다. 캐릭터 코드를 화면에서 떼어냈지만 기록 창 문구는 그대로 나온다.

실행 전에 예상한 것:

- 창 없이도 `attack()`이 되고, BoneSoldier hp가 조금 깎인다
- 30번 치료해도 hp는 33
- 10번 연주해도 스트레스는 0
- stress 필드는 `Heroes`에만 있다
- 죽은 영웅은 0번, Crusader는 살아 있는 3명 중 1명이라 약 33%
- 두 번째 영웅도 매번 판정을 받아서 건너뛴 횟수는 0번

![리팩토링한 몬스터 배틀 RefactorCheck 실행 결과]({{ '/assets/images/java-programming-2/battle3-refactorcheck.png' | relative_url }})

예상대로였다. Crusader는 10만 번 중 33559번(33.6%) 골라져서 3명 중 1명꼴로 나왔다.

## 4. 알게 된 점

- 1번과 2번은 둘 다 "상속을 잘못 썼다"는 힌트였는데 양상이 달랐다. 1번은 **자식이 부모 것을 또 선언하고 복사**한 경우고, 2번은 **부모에 넘길 값을 버리거나, 공유하면 안 되는 값을 static으로 둔** 경우였다. 둘 다 결국 "부모가 이미 가진 걸 자식이 믿지 않고 따로 챙긴" 문제였다.
- 3번에도 상속 문제(`Heroes`의 stress 중복)가 숨어 있었다. 모듈화 문제만 있을 줄 알았는데 한 코드에 여러 문제가 섞여 있었다.
- 숫자만 다른 if문이 보이면 계산식으로 바꿀 수 있는지부터 보면 된다. 3번은 이것만으로 코드가 확 줄었다.
- 캐릭터가 화면을 직접 건드리면 테스트 코드 하나 돌리기도 어렵다. `BattleLog`로 떼어내고 나서야 창 없이 RefactorCheck를 돌릴 수 있었다.
- 원래 코드를 짠 사람이 주석에 "인덱스 문제로 오류남"이라고 남겨둔 문제가 두 목록의 인덱스가 어긋나서 생긴다는 걸 직접 재현해서 확인할 수 있었다.

## 5. 궁금한 점

- 1번 `Battle`처럼 규칙을 화면에서 떼어내는 게 교수님이 말씀하신 "상속을 회피하는 방법(연관 관계)"이랑 같은 이야기일까?
- 3번 `BattleLog`를 static으로 만들었는데, static을 줄이려면 캐릭터가 로그 객체를 생성자로 받는 게 더 나을까?
- 캐릭터 역할을 `targetsAllies()`로 구분했는데, 역할이 더 늘어나면 인터페이스(예: `Healer`, `Attacker`)로 나누는 게 나을까?

## 마치며

세 코드 모두 실행은 잘 되는 코드였는데, 테스트 코드로 하나씩 찔러보니까 생각보다 문제가 많았다. 특히 자식 클래스가 부모 필드를 다시 선언하는 것, 똑같은 코드를 복사해서 숫자만 바꾸는 것, 캐릭터가 화면을 직접 건드리는 것 세 가지는 이번에 확실히 눈에 익었다. 다음에 내 코드를 짤 때도 이 세 가지부터 확인해봐야겠다.
