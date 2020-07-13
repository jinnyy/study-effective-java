# item 49. 매개변수가 유효한지 검사하라

매개변수의 제약을 문서화하고 메서드 코드 시작 부분에서 명시적으로 검사한다

- **오류는 가능한 한 빨리 (발생한 곳에서) 잡아야 한다**
- 실패 원자성(호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 한다)을 해칠 수 있다
- 메서드는 최대한 범용적으로 설계하되, 구현하려는 개념 자체가 특정한 제약을 내재한 경우에만 유효성을 검사한다.

<br>

## public, protected 메서드

- `@throws` 자바독 태그를 사용해 매개변수의 제약과 이를 어겼을 때 발생하는 예외를 함께 기술한다.
    - 클래스의 모든 public 메서드에 적용해야 하는 제약은 클래스 수준 주석으로 기술하라
- 예외는 대개 `IllegalArgumentException`, `IndexOutOfBoundsException`, `NullPointerException` 중 하나가 될 것이다
- 가능하다면 Object의 유효성 검증 메소드를 사용하라
    - null 검사: `requireNonNull(T obj, String message)`
    - 범위 검사: `checkFromIndexSize(int fromIndex, int size, int length)`, `checkFromToIndex(int fromIndex, int toIndex, int length)`, `checkIndex(int index, int length)`

```java
/* ...
 * <p>All methods and constructors in this class throw
 * {@code NullPointerException} when passed
 * a null object reference for any input parameter.
 * ...
 */

public class BigInteger extends Number implements Comparable<BigInteger> {

  /**
    * Returns a BigInteger whose value is {@code (this mod m}).  This method
    * differs from {@code remainder} in that it always returns a
    * <i>non-negative</i> BigInteger.
    *
    * @param  m the modulus.
    * @return {@code this mod m}
    * @throws ArithmeticException {@code m} &le; 0
    * @see    #remainder
    */
  
  public BigInteger mod(BigInteger m) {
    if (m.signum <= 0)
      throw new ArithmeticException("BigInteger: modulus not positive");
    ...
  }
}
```
source code: [java.math.BigInteger](https://hg.openjdk.java.net/jdk8/jdk8/jdk/file/tip/src/share/classes/java/math/BigInteger.java)

<br>

## private 메서드

- 유효한 값만이 메서드에 넘겨지리라는 것을 (validation method가 아닌) **당신이 보증해야 한다**
- 단언문 (assert)을 사용해 매개변수 유효성을 검증하라
    - 실패하면 AssertionError를 던진다
    - (`-ea` 혹은 `--enableassertions` 와 같은 플래그로) assertion 사용을 명시적으로 지정하지 않으면 **런타임에 제거된다.**

<br>

## 정적 팩터리 메서드, 생성자 - 나중에 쓰기 위한 매개변수는 특히 더 신경써서 검사하라 🧐

- 문제가 된 매개변수를 받은 시점과 오류 발생 시점이 달라 디버깅이 상당히 괴로워질 수 있다

```java
static List<Integer> intArrayAsList(int[] a) {
  Objects.requireNonNull(a);
  
  return new AbstractList<Integer>() {
    @Override public Integer get(int i) { return a[i]; }
    ...
  };
}
```

<br>

## 예외

- 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때
- 계산 과정에서 암묵적으로 검사가 수행될 때
    - ex. `Collections.sort(List)` throws ClassCastException
