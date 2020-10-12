# item 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스(= 직렬화 프로시)를 설계해 `private static`으로 선언한다

* 직렬화의 생성자를 이용하지 않고 인스턴스를 생성하는 기능을 제거함에 따라, 
* 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다

```java
public class Period implements Serializable {
  private final Date start;
  private final Date end;

  private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;

    SerializationProxy(Period p) {
      this.start = p.start;
      this.end = p.end;
    }
    
    // Period.SerializationProxy용 readResolve 메서드
    private Object readResolve() {
      return new Period(start, end); // public 생성자를 사용한다.
    }

    private final long serialVersionUID = 23409L;
  }

  // 직렬화 프록시 패턴용 writeReplace 메서드
  private Obejct writeReplace() {
    return new SerialziationProxy(this);
  }

  // 직렬화 프록시 패턴용 readObject 메서드
  private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다.");
  }
  
  ...
}
```

* 직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다

  * `EnumSet`은 (내부적으로) 열거 타입의 원소가 64개 이하면 `RegularEnumSet`을 사용하고 그보다 크면 `JumboEnumSet`을 사용한다
  * 직렬화 프록시 패턴에서는 원소 64개짜리 열거 타입을 가진 EnumSet을 직렬화한 다음 원소 5개를 추가하고 역직렬화해도 정상 작동한다

```java
private static class SerializationProxy <E extends Enum<E>> implements Serializable {
  private final Class<E> elementType;
  
  private final Enum<?>[] elements;
  
  SerializationProxy(EnumSet<E> set) {
    elementType = set.elementType;
    elements = set.toArray(new Enum<?>[0]);
  }
  
  private Object readResolve() {
    EnumSet<E> result = EnumSet.noneOf(elementType);
    for (Enum<?> e: elements)
      result.add((E)e);
    return result;
  }
  
  private final long serialVersionUID = 23409L;
}  
```

<br>

### 직렬화 프록시 패턴의 한계

* 클라이언트가 확장할 수 있는 클래스 혹은 객체 그래프에 순환이 있는 클래스에는 적용할 수 없다

  * 직렬화 프록시만 가졌을 뿐 실제 객체는 아직 만들어지지 않았기 때문에 `ClassCastException` 발생
* 퍼포먼스 저하 (저자피셜 14% 느림)
