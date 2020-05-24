

> #### Effective Java 3/E
> 2020-05-25
>
> 아이템 #03 private 생성자나 열거 타입으로 싱글턴임을 보증하라

<br><br><br><br><br>





# 03. private 생성자나 열거 타입으로 싱글턴임을 보증하라

싱글턴(singleton)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다

* 단순히 함수를 사용하기 위한 유틸리티 객체나, 로깅 객체, 설계상 유일해야 하는 시스템 컴포넌트
* 생성자를 private으로 감춘다


## public static final 멤버변수

* static 영역이 로딩될 때 딱 1번만 호출
* 이 클래스가 싱글턴임이 명백히 드러나고, 코드가 간결하다

``` java
public class Elvis {
   public static final Elvis INSTANCE = new Elvis();
   private Elvis() { }
   
   public void leaveTheBuilding() {}
}
```

* 리플렉션 API인 `AccessibleObject.setAccessible`을 사용해 private 생성자를 호출할 수 있는데, 이는 생성자를 수정하여 두 번 호출되지 못하도록 예외를 던지게 하면 된다

#### 리플렉션 API
구체적인 클래스 타입을 알지 못해도, 그 클래스의 메소드, 타입, 변수들을 접근할 수 있도록 해주는 자바 API (Item 65)


## static factory method

* getInstance 메서드는 항상 같은 객체의 참조를 반환하므로, 다른 인스턴스는 만들어지지 않는다 (리플렉션 API에 대한 예외 처리는 별도로 필요)
* API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다 (getInstance 내부만 변경)
* 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다 (형변환을 유연하게 만든다)
* 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다

``` java
public class Elvis {
   private static final Elvis INSTANCE = new Elvis();
   private Elvis() { }
   public static Elvis getInstance() { return INSTANCE; }

   public void leaveTheBuilding() {}
}
```

#### 역직렬화 시 싱글턴이 깨지는 이슈

앞선 두 가지 모두 역직렬화 할 때 같은 타입의 인스턴스가 여러개 생길 수 있다

* 이미 직렬화된 Elvis를 다시 역직렬화할 때, private static final Elvis INSTANCE 이라면 여러 인스턴스가 생길 수 있는 것
* transient 키워드를 추가하고, readResolve()를 구현해서 어떤 값이 들어오든 버리고 transient Elvis INSTANCE 를 반환한다
    * 직렬화(Serialize) : JVM 메모리영역에 존재하는 인스턴스를 byte형태로 만드는 것
    * 역직렬화(Deserialize) : byte형태를 다시 JVM에 올리는 것 (json, xml로 많이들 직렬화/역직렬화)
    * transient: 그 필드를 직렬화 대상에서 제외하는 것
    * readResolve(): 역직렬화시 Serializable interface의 ObjectInputStream 메서드에서 사용, 클래스의 멤버변수(레퍼런스 타입)가 serializable 하지않을 경우, 이 멤버변수를 직렬화/역직렬화 해주기 위해 호출
    * writeReplace() : readResolve의 반대로 직렬화 시 호출

``` java
public class Elvis implements Serializable {
	private static final transient Elvis INSTANCE = new Elvis();

	public static Elvis getInstance() {
		return INSTANCE;
	}
	
	public Object readResolve() {
		return INSTANCE;
	}
}
```

## 열거 타입 싱글턴

* 간결하고, 추가 노력 없이 직렬화 할 수 있다
* 대부분의 상황에서는 이런 **열거타입으로 싱글턴을 만드는 것이 가장 좋은 방법**
* 싱글턴이 Enum 외의 클래스를 상속해야 한다면, 이 방법은 사용할 수 없다 (인터페이스는 구현 가능)
* javap로 디컴파일 해보면 public static final 멤버변수와 유사하다는 것을 알 수 있다

``` java
public enum Elvis {
   INSTANCE;
   
   public void leaveTheBuilding() { }
}
```
