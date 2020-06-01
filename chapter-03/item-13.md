# 13. clone 재정의는 주의해서 진행하라

## 1. clone의 명세는 허술하다

* `Cloneable`은 믹스인 인터페이스로, 이를 구현한 객체는 **복제 가능함**을 명시하기 위해 사용된다.

  * 그러나 Cloneable은 인터페이스만 있을 뿐 **구현해야 할 메소드가 없다!** clone 메서드는 쌩뚱맞게도 Object 클래스에 protected로 정의되어 있기 때문에, Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메소드를 호출할 수 없다.
  * Cloneable은 상위 클래스인 Object에 정의된 clone의 동작 방식을 결정한다. Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 필드를 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 `CloneNotSupportException`을 던진다.


```java
public interface Cloneable {
}
```
```java
public class Object {
  protected native Object clone() throws CloneNotSupportedException;
  ...
}
```

> 실무에서는 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다. 강제할 수 없고 허술하게 기술된 프로토콜을 지킨 결과로 깨지기 쉽고 위험하고, 모순적인 매커니즘이 탄생한다. **생성자를 호출하지 않고도 객체를 생성할 수 있게 된다.**

<br>

## 2. clone의 재정의시 주의해야 할 점들

* 일어날 수 없는 예외를 위한 try-catch 처리

```java
public class PhoneNumber implements Cloneable {
  @Override public PhoneNumber clone() {
    try {
      return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {  // unchecked exception 이었어야 한다
      throw new AssertionError();
    }
  }
  ...
}
```

* 가변 객체를 참조하는 클래스.. 🤯

  * Object의 clone 기본 규약은 shallow copy를 이용하기 때문에, 배열과 같은 가변 필드는 원본 필드와 객체를 공유하게 된다.
  * 가변 객체의 clone을 재귀적으로 호출한다 -> **가변 객체를 참조하는 필드는 final로 선언하라는 일반 용법**과 충돌한다
  * 가변 객체가 deep copy(깊은 복사)를 지원하도록 보장한다
  * 가변 객체를 재생성하는 고수준 메서드를 호출한다 -> 느리고, 필드 단위 객체 복사를 우회하기 때문에 Cloneable 아키텍처와 어울리지 않는 방식이다

```java
public class Stack implements Cloneable {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack() {
    this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  
  @Override public Stack clone() {
    try {
      Stack result = super.clone();
      result.elements = elements.clone(); // 가변 객체의 clone을 재귀적으로 호출한다
      return result
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
}
```

```java
public class HashTable implements Cloneable {
  private Entry[] buckets = ...;
   
  private static class Entry {
    final Object key;
    Object value;
    Entry next;
    
    Entry(Object key, Object value, Entry next) {
      this.key = key;
      this.value = value;
      this.next = next;
    }

    Entry deepCopy_recursive() {
      return new Entry(key, value,
        next == null ? null : next.deepCopy());
    }
    
    // deep copy의 재귀적 호출은 스택 오버플로우를 일으킬 위험이 있기 때문에, 되도록이면 반복적 호출을 이용하자
    Entry deepCopy_iterative() {
      Entry result = new Entry(key, value, next);
      for (Entry p = result; p.next != null; p = p.next)
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
      return result;
    }
  }
  
  @Override public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = new Entry[buckets.length];
      for (Entry bucket : buckets) {
        if (bucket != null)
          result.buckets[i] = buckets[i].deepCopy_iterative();
      return result;
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
  ...
}
```

* 상속용 클래스는 Cloneable을 구현해서는 안 된다

  * 부모 클래스의 clone 메서드에서 재정의 가능한 메서드를 호출했다면, 자식 클래스에서 `super.clone();`을 호출했을 때 자식 클래스에서 재정의한 메서드를 호출하게 되기 때문이다
  * Object와 마찬가지로 clone 메서드를 구현하되 Cloneable 인터페이스는 구현하지 않아서 하위 클래스에게 구현 여부를 선택하게 하거나, clone을 동작하지 않게 구현하고 하위 클래스에서 재정의하지 못하게 한다

```java
@Override
protected final Object clone() throws CloneNotSupportedException {
  throw new CloneNotSupportedException();
}
```

* Cloneable을 구현한 스레드 세이프 클래스를 작성할 때는 clone 메서드 역시 동기화해야 한다.

<br>

## 3. clone보다 복사 생성자와 복사 팩토리를 사용하라
* 생성자를 쓰지 않는 (위험천만한) 객체 생성 메커니즘을 사용하지 말고 복사 생성자와 복사 팩토리를 사용하자 😇
* 변환 생성자, 변환 팩터리: 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다

  * 에를 들어, `new TreeSet<>(s)` 라는 구문으로 HashSet 객체 s를 TreeSet 타입으로 복제할 수 있다

```java
public Yum(Yum yum) {...};
```
```java
public static Yum newInstance(Yum yum) {...};
```

<br>

> 네 줄 요약: Cloneable이 몰고 온 모든 문제를 되짚어봤을 때, 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다. final 클래스라면 Cloneable을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 드물게 허용해야 한다(아이템 67). 기본 원칙은 **'복제 기능은 생성자와 팩터리를 이용하는 게 최고'** 라는 것이다. 단, 배열만은 clone 메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다.
