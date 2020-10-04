# item 88. readObject 메서드는 방어적으로 작성하라

* 생성자와 접근자에서 **방어적 복사**를 사용하는 불변 클래스

```java
public final class Period {
  private final Date start;
  private final Date end;
  
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0)
      throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
  }
  
  public Date start() { return new Date(start.getTime()); }
  public Date end() { return new Date(end.getTime()); }
}
```

를 직렬화 해보자 🤓

<br>

* 기본 `readObject` 메서드 사용시 - 가짜 바이트 스트림 공격으로 불변식을 깨뜨리는 Period 인스턴스를 생성할 수 있다 🤯

```java
public class BogusPeriod {
  // 진짜 Period 인스턴스에서는 만들어질 수 없는 바이트 스트림
  private static final byte[] serializedForm = { ... };
  
  public static void main(String[] args) {
    Period p = (Period) deserialize(serializedForm);
    System.out.println(p);
  }
  
  //주어진 직렬화 형태(바이트 스트림)로부터 객체를 만들어 반환한다.
  static Object deserialize(byte[] sf) {
    try {
      return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
    } catch (IOException | ClassNotFoundException e) {
      throw new IllegalArgumentException(e);
    }
  }
}

>> Sun Oct 04 21:59:35 KST 2020 - Mon Oct 04 21:59:35 KST 2010
```

<br>

* 유효성 검사를 수행하는 커스텀 `readObject` 메서드 사용시 - 내부 필드 탈취 공격 🤯

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();
  
  // 불변식을 만족하는지 검사한다.
  if (start.compareTo(end) > 0)
    throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}
```

```java
public class MutablePeriod {
  public final Period period;
  
  public final Date start;
  public final Date end;
  
  public MutablePeriod() {
    try {
      ByteArrayOutputStream bos = new ByteArrayOutputStream();
      ObjectOutputStream out = new ObjectOutputStream(bos);
      
      // 유효한 Period 인스턴스를 직렬화 한다
      out.writeObject(new Period(new Date(), new Date()));
      
      byte[] ref = { 0x71, 0, 0x7e, 0, 5 };
      bos.write(ref);
      ref[4] = 4;
      bos.write(ref);
      
      // Period 역직렬화 후 Date 참조를 '훔친다'
      ObjectInputStream in = new ObjectInputStream(
        new ByteArrayInputStream(bos.toByteArray()));
      period = (Period) in.readObject();
      start = (Date) in.readObject();
      end = (Date) in.readObject();
    } catch (IOException | ClassNotFoundException e) {
      throw new AssertionError(e);
    }
  }
}     
```

```java
public static void main(String[] args) {
  MutablePeriod mp = new MutablePeriod();
  Period p = mp.period;
  Date pEnd = mp.end;
  
  pEnd.setYear(78);
  System.out.println(p);
}

>> Sun Oct 04 22:41:44 KST 2020 - Wed Oct 04 22:41:44 KST 1978
```

<br>

* 유효성 검사와 방어적 복사를 모두 수행하는 커스텀 `readObject` 메서드 사용시 - 안전 😎👍

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
  s.defaultReadObject();
  
  // 가변 요소들을 방어적으로 복사한다
  start = new Date(start.getTime());
  end = new Date(end.getTime());
  
  // 불변식을 만족하는지 검사한다
  if (start.compareTo(end) > 0)
    throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}
```

<br>

### readObject는 매개변수로 바이트 스트림을 받는 생성자라 할 수 있다
* 바이트 스트림이 진짜 직렬화된 인스턴스라고 가정해서는 안 된다!

* 안전한 `readObject` 메서드를 작성하는 지침
  * `private`이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 **방어적 복사**하라. 불변 클래스 내의 가변 요소가 여기 속한다.
  * 모든 불변식을 검사하여 어긋나는 게 발견되면 `InvalidObjectException`을 던진다. 방어적 복사 다음에는 반드시 **불변식 검사**가 뒤따라야 한다.
  * 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 `ObjectInputValidation` 인터페이스를 사용하라
  * (final이 아닌 직렬화 가능 클래스라면) 재정의 가능 메서드를 호출하지 말자

* 기존 `readObject` 메서드를 써도 좋을지 판단하는 간단한 방법

  * Q. `transient` 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가?
  * A. 답이 **아니오** 라면 1) 커스텀 readObject 메서드를 만들어 모든 유효성 검사와 방어적 복사를 수행하거나 2) 직렬화 프록시 패턴을 사용하라
