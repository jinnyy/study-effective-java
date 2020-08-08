# item 58. 전통적인 for문 보다는 for-each문을 사용하라

for-each문의 정식 이름은 향상돤 for문(enhanced **for** statement)이다.

<br><br><br>


## for-each문을 씁시다

#### for-each를 사용하지 않았을 때 문제점
``` java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
  Element e = i.next();
  ... // e로 무엇인가를 한다.
}

for (int i = 0; i< a.length; i++) {
  ... // a[i]로 무엇인가를 한다.
}
```
- 반복자와 인덱스 변수는 모두 코드를 지저분하게 할 뿐. 진짜 필요한 것은 원소들 뿐이다.
- 쓰이는 요소 종류가 늘어나면 오류가 생길 가능성이 높아진다. 혹시라도 잘못 사용했을 때 컴파일러가 잡아주리라는 보장도 없다.
- 컬렉션이냐 배열이냐에 따라 코드 형태가 상당히 달라진다.

<br>

:high_brightness: 이런 문제들은 for-each문을 사용하면 모두 해결된다. :high_brightness:

<br>

#### 코드 58-4 버그를 찾아보자.
``` java
// 버그를 찾아보자.
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
    NINE, TEN, JACK, QUEEN, KING }

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

public static void main(String[] args) {
    List<Card> deck = new ArrayList<>();

    for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
        for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
            deck.add(new Card(i.next(), j.next()));

}
```
* 문제: 바깥 컬렉션(suits)의 반복자에서 next 메소드가 너무 많이 불린다는 것이다.
* 마지막 줄의 i.next()는 숫자(Suit) 하나당 한 번씩만 불려야 하는데, 안쪽 반복문에서 호출되는 바람에 카드(Rank) 하나당 한 번씩 불리고 있다.
* 그래서 숫자가 바닥나면 NoSuchElementException을 던진다.



<br><br>

## for-each문을 사용할 수 없는 상황
아래의 상황 중 하나에 속할 때는 일반적인 for문을 사용하되 [이번 아이템에서 언급한 문제들](#for-each를-사용하지-않았을-때-문제점)을 경계하기 바람

1. 파괴적인 필터링 (destructive filtering)
  - 컬렉션을 순회하면서 제거해야 한다면 iterator의 remove 메서드를 호출해야 한다.
  - 자바 8부터는 Collection의 removeIf 메서드를 통해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.
2. 변형 (transformin)
  - 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 **교체**해야 한다면 리스트의 iterator나 배열의 인덱스를 사용해야 한다.
3. 병렬 반복 (parallel iteration)
  - 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.


`for-each` 문은 컬렉션과 배열은 물론 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다.
Iterable 인터페이스는 다음과 같이 메서드가 단 하나뿐이다.
``` java
public interface Iterable<E> {
  // 이 객체의 원소들을 순회하는 반복자를 반환한다.
  Iterator<E> iterator();
}
```

<br><br>

## 결론
* 처음부터 Iterable을 직접 구현하기는 까다롭지만, 원소들의 묶음을 표현하는 타입을 작성해야 한다면 Iterable을 구현하는 쪽으로 고민해보기 바란다.
* 그러면 그 타입을 사용하는 프로그래머가 for-each를 사용할 때마다 여러분에게 감사할 것이다.
