

> #### Effective Java 3/E
> 2020-05-25
>
> 아이템 #01 생성자 대신 정적 팩터리 메서드를 고려하라

<br><br><br><br><br>





# 01. 생산자 대신 정적 팩터리 메서드를 고려하라

생성자를 사용하게 되면 매개 변수가 증가하고, 같은 역할을 하는 클래스의 생성자를 반복 생성하게 되어 코드가 복잡해진다
(디자인 패턴에서 말하는 팩터리 패턴과 다른 개념)

Boolean 클래스의 예 (일반 타입 boolean을 객체 타입 Boolean으로 박싱)

``` java
public static Boolean valueOf(boolean b) {
  return b ? Boolean.TRUE : Boolean.FALSE;
}
```




<br><br>


## 이름을 가질 수 있다

* 생성자와 해당 생성자에 넘기는 매개 변수만으로는 객체의 특성을 제대로 설명할 수 없다
* 정적 팩터리 메서드는 이름을 가지고 있으며, 객체와 메서드의 역할에 대한 설명을 담을 수 있다 (클라이언트 코드의 가독성 향상)
* 같은 시그니처를 갖는 생성자를 여러 개 정의할 필요가 있을 때는 그 생성자들을 정적 팩터리 메서드로 바꾸고, 메서드 이름을 보면 차이가 명확히 드러나도록 작명


## 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다

* 변경 불가능 클래스라면 이미 만들어둔 객체를 활용할 수도 있고, 만든 객체를 캐시해놓고 사용할 수도 있다
* 동일한 객체가 요청되는 일이 잦고, 특히 객체를 만드는 비용이 클 때 유용하다
* 어떤 시점에 어떤 객체가 얼마나 존재할지 정밀하게 제어할 수 있다

#### 예시
`valueOf`과 같은 메서드는 객체를 리턴하지만, 인스턴스를 새로 생성하지 않는다
매개변수에 따라 Boolean.TRUE, Boolean.FALSE 라는 이미 만들어진 값을 리턴한다
따라서 자주 호출되는 객체에 대해 성능을 끌어올릴 수 있다. (플라이웨이트 패턴과 비슷한 기법)


## 반환값 자료형의 하위 자료형 객체를 반환할 수 있다

* public 정적 팩터리 메서드가 반환하는 객체의 클래스가 public일 필요가 없다
* 메서드에 주어지는 인자를 이용하면 어떤 클래스의 객체를 만들지도 동적으로 결정할 수 있다
* API를 호출하는 코드 쪽에서는 내부적으로 어떤 클래스가 이용되는지 알 수가 없다 (알 필요도 없다)

#### 예시

``` java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
}
```

클라이언트는 팩터리 메서드가 반환하는 객체의 실제 클래스를 알 수 없다.
단지 EnumSet의 하위 클래스라는 사실만 중요할 뿐.

## 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다

**서비스 제공자 프레임워크(Service Provider Framework)**를 만드는 근간이 된다. (e.g. JDBC)

* 서비스 인터페이스 (Service Interface): 구현체의 동작을 정의 (e.g. JDBC Connection)
* 제공자 등록 API (Provider Registration API): 제공자가 구현체를 등록 (e.g. DriverManager.registerDriver)
* 서비스 접근 API (Service Access API): 클라이언트가 서비스의 인터페이스를 얻을 때 사용 (e.g. DriverManager.getConnection)
* 서비스 제공자 인터페이스 (Service Provider Interface): 부가적. (e.g. JDBC Driver)


<br><br>

## 생성자를 만들지 않고 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다

* 하위 클래스를 만들기 위해서는 public 혹은 protected 수준의 생성자 필요
* 계승(inheritance) 대신 구성(composition) 기법을 쓰도록 장려
* 불변 타입 클래스를 만들 경우 오히려 장점으로 작용하기도 한다

## 정적 팩터리 메서드가 다른 정적 메서드와 확연히 구분되지 않는다

* API 설명에 명확하게 드러나지 않는다
* 클래스나 인터페이스 주석을 통해 정적 팩터리 메서드임을 표시
* 흔히 쓰이는 정적 팩터리 메서드의 이름을 사용



#### 흔히 쓰이는 정적 팩터리 메서드의 이름


valueOf: 인자로 주어진 값과 같은 값을 갖는 객체를 반환
of: valueOf를 더 간단하게 쓴 것

``` java
// 예시
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
Set<Rank> faceCards - EnumSet.of(JACK, QUEEN, KING);
``` 

getInstance: 인자에 기술된 객체 반환, 싱글톤 패턴을 따를 경우, 인자 없이 항상 같은 객체 반환 

``` java
// 예시
StackWalker luke = StackWalker.getInstance(options);
``` 

newInstance: getInstance와 같지만 호출할 때마다 다른 객체 반환

``` java
// 예시
Object newArray = Array.newInstance(classObject, arrayLen);
```

from: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드

``` java
// 예시
Date d = Date.from(instant);

```



> 결론: 무조건 생성자만 쓰는 습관을 고치자
> 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 장단점을 이해하고 적절히 사용하자
> 보통은 정적 팩터리 메서드가 더 좋은 점이 많다
