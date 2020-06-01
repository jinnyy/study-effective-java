# 14. Comparable을 구현할지 고려하라

* `Comparable`은 믹스인 인터페이스로, 이를 구현한 객체는 **자연적인 순서(natural order)** 가 있음을 명시하기 위해 사용된다

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

<br>

## 1. `compareTo` 메서드 일반 규약

* 이 객체가 주어진 객체보다 **작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다.** 비교할 수 없는 타입의 객체가 주어지면 `ClassCastException`을 던진다.

* 반사성, 대칭성, 추이성을 만족한다. - equals와 마찬가지로 Comparable을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶다면 확장 대신 컴포지션을 사용하라
* (권장) `compareTo` 메서드로 수행한 동치성 테스트의 결과가 `equals`의 결과와 같아야 한다. 이 권고를 지키지 않을 경우 주석으로 그 사실을 명시하라

<br>

마지막 항목은 왜 권장일까? 🤔

동치성 테스트의 결과가 다르더라도 동작은 잘 한다! 그러나 클래스를 정렬된 컬렉션에 넣었을 경우 해당 컬렉션이 구현한 인터페이스(`Collection, Set, Map`)에 정의된 동작과 엇박자가 날 수 있다. 정렬된 컬렉션들은 동치성을 비교할 때 `compareTo`를 사용하기 때문이다.

다음은 Set 인터페이스의 문서에 나온 내용이다. 

> sets contain no pair of elements e1 and e2 such that **e1.equals(e2)**

Set 인터페이스는 equals 함수로 동치성을 비교해 중복된 element를 제거하는 반면,

> a sorted set performs all element comparisons using its **compareTo** (or compare) method

Set 인터페이스를 구현하고 순서를 보장한 SortedSet 인터페이스는 compareTo로 동치성을 비교한다.

compareTo와 equlas가 일관되지 않은 `BigDecimal` 클래스를 예로 들어보자. 빈 HashSet 인스턴스에 `new BigDecimal("1.0")`과 `new BigDecimal("1.00")`을 차례로 추가한다. 두 인스턴스는 equals 메소드로 비교하면 서로 다르기 때문에 `HashSet`은 원소를 2개 갖게 된다. 그러나 HashSet 대신 `TreeSet`을 사용하면 원소를 1개만 갖게 된다. compareTo 메소드로 비교하면 두 인스턴스가 같기 때문이다.

<br><br>

## 2. `compareTo` 메서드 작성 요령

* 객체 참조 필드를 비교하려면 `compareTo` 메서드를 재귀적으로 호출하라
* `Comparable`을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 할 때는 비교자(`Comparator`)를 대신 사용하라

```java
public interface Comparator<T> {
  int compare(T o1, T o2);
  ...
}
```
```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
  private final String s;
  
  public int compareTo(CaseInsensitiveString cis) {
    return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);   // java에서 제공하는 Comparator
  }
}
```

<br>

* 정수 기본 타입 필드를 비교할 때 관계 연산자 <, >를 사용하기 보다는 Wrapper 클래스의 `compare` 메서드를 사용하라
* 가장 핵심이 되는 필드부터 비교하라 - 비교해야 할 필드가 많을 때는 가독성을 취하고 약간의 성능을 포기해 메서드 연쇄 방식으로 비교자를 생성하는 방법도 있다

```java
public int compareTo(PhoneNumber pn) {
  int result = Short.compare(areaCode, pn.areaCode);    // 가장 중요한 필드
  if (result == 0) {
    result = Short.compare(prefix, pn.prefix);          // 두 번째로 중요한 필드
    if (result == 0)
      result = Short.compare(lineNum, pn.lineNum);      // 세 번째로 중요한 필드
  }
  return result;
}
```
```java
private final Comparator<PhoneNumber> COMPARATOR =
  comparingInt((PhoneNumber pn) -> pn.areaCode)
    .thenComparingInt(pn -> pn.prefix)
    .thenComparingInt(pn -> pn.lineNum);
    
public int compareTo(PhoneNumber pn) {
  return COMPARATOR.compare(this, pn);
}
```

<br>

* 값의 차를 반환하지 마라 - 정수 오버플로우 혹은 부동 소수점 계산 방식에 따른 오류를 낼 수 있다



```java
/* 🤮 해시코드 값의 차를 기준으로 하는 비교자 */

static Comparator<Object> hashCodeOrder = new Comparator<> () {
  public int compare(Object o1, Object o2) {
    return o1.hashCode() - o2.hashCode();
  }
}
```

```java
/* 😇 정적 compare 메서드를 활용한 비교자 */

static Comparator<Object> hashCodeOrder = new Comparator<> () {
  public int compare(Object o1, Object o2) {
    return Integer.compare(o1.hashCode(), o2.hashCode());
  }
}
```

```java
/* 😇 비교자 생성 메서드를 활용한 비교자 */

static Comparator<Object> hashCodeOrder = 
  Comparator.comparingInt(o -> o.hashCode());
```


<br><br>

> **네 줄 요약:** 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다. compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연산자는 쓰지 말아야 한다. 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.


