# item 89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

### 싱글턴의 직렬화
* `readResolve`를 사용하면 `readObject`가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다
  * 역직렬화한 객체의 클래스가 정의한 `readResolve`가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다
  * 새로 생성된 객체의 참조는 유지하지 않으므로 바로 가비지 컬렉션의 대상이 된다
* `readResolve`를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 `transient`로 선언해야 한다 - 그렇지 않으면 아래와 같이 공격 가능하다

```java
public class Elvis implements Serializable {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { }
  
  private Object readResolve() {
    return INSTANCE;
  }
}
```

```java
public class Elvis implements Serializable {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { }
  
  // non-transient 참조 필드 - Elvis의 readResolve 메서드가 실행되기 전에 역직렬화 된다
  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };
  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }
  
  private Object readResolve() {
    return INSTANCE;
  }
}
```

```java
// 도둑 클래스
public class ElvisStealer implements Serializable {
  static Elvis impersonator;
  private Elvis payload;
  
  private Object readResolve() {
    impersonator = payload;
    
    // ClassCastException이 발생하지 않도록 Elvis의 필드의 원래 타입에 맞는 값을 반환한다
    return new String[] { "A Fool Such as I" };
  }
  private static final long serialVersionUID = 0;
}
```

```java
public class ElvisImpersonator {
  // Elvis의 non-transient 참조 필드를 ElvisStealer로 바꿔친 바이트 스트림
  private static final byte[] serializedForm = {...}
  
  public static void main(String[] args) {
    Elvis elvis = (Elvis) deserialize(serializedForm);
    Elvis impersonator = ElvisStealer.impersonator;
    
    elvis.printFavorites();
    impersonator.printFavorites();
  } 
}

>> [Hound Dog, Heartbreak Hotel]
>> [A Fool Such as I]
```

* 따라서 `enum` 타입으로 싱글턴을 구현하는 게 더 쉽고 안전하다!

```java
public enum Elvis {
  INSTANCE;
  
  private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };
  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }
}
```
