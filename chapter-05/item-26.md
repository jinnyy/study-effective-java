# 26. 로 타입은 사용하지 말라

> **제네릭 타입 (제네릭 클래스, 제네릭 인터페이스)** :
> 선언에 타입 매개변수(type parameter)를 사용하는 클래스 혹은 인터페이스. 컴파일시 타입을 체크해 타입 안정성을 보장하기 위해 사용한다.

<br>

## 1. 로 타입을 사용하면 제네릭이 안겨주는 안전성과 표현력을 모두 잃게 된다

* 로 타입 (raw type)

  * 제네릭 타입에서 타입 매개변수를 사용하지 않는 것을 말한다. 제네릭 이전 코드와 호환되도록 하기 위해 제공된다.
  * `List<E>`의 로 타입은 `List`다

* 잘못된 로 타입 사용 예제

```java
private final Collection stamps = ...;

stamps.add(new Coin(...)); // "unchecked call" 경고가 발생하긴 하지만, 엉뚱한 객체가 들어가도 오류 없이 컴파일되고 실행된다

Stamp stamp = (stamp) stamps.iterator().next(); // ClassCastException을 던진다
```

* 임의 객체를 허용하고 싶다면 `List<Object>`를 사용하라

  * `List<Object>`는 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달한 것이다.
  * `List`는 `List<String>`의 상위 타입이지만, `List<Object>`는 `List<String>`의 상위 타입이 아니다. 따라서, `List`를 받는 메서드에 `List<String>`을 넘길 수 있지만, `List<Object>`를 받는 메서드에는 `List<String>`을 넘길 수 없다.
  
  * 로타입 사용시 런타임 오류

  ```java
  public static void main(String[] args) {
    List<String> strings = new ArrayList<>
    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0); // 컴파일러가 자동으로 형변환 코드를 넣어준다 - ClassCastException
  }

  private static void unsafeAdd(List list, Object o) {
    list.add(o);
  }
  ```

  * `List<Object>` 사용시 컴파일 오류
  
  ```java
  public static void main(String[] args) {
    List<String> strings = new ArrayList<>
    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0); // incompatible types error
  }

  private static void unsafeAdd(List<Object> list, Object o) {
    list.add(o);
  }
  ```

<br>

## 2. 비한정적 와일드카드 타입을 사용하라

* 제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않을 때 비한정적 와일드카드 타입을 사용하라

  * **null 외에는 어떤 원소도 넣을 수 없기 때문에** 타입 불변성을 훼손하지 않는다

```java
static int numElementInCommon(Set s1, Set s2) {...}         // bad
static int numElementInCommon(Set<?> s1, Set<?> s2) {...}   // good
```

<br>

## 3. 예외: class 리터럴과 instanceof 연산자

* class 리터럴에는 매개변수화 타입을 사용할 수 없다. 예를 들어 List.class는 허용하고 List<String>.class는 허용하지 않는다.
* 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개 변수화 타입에는 적용할 수 없다. 코드의 가독성을 위해 로 타입으로 표기하자.

```java
if (o instanceof Set) {   // 로 타입
  Set<?> s = (set<?>) o;  // 와일드카드 타입
  ...
}
```

<br>

> 세 줄 요약:
> 1) 로 타입을 사용하면 런타임에 예외가 일어날 수 있으니 사용하면 안 된다.
> 2) 로 타입은 제네릭이 도입되기 이전 코드와의 호환성을 위해 제공될 뿐이다.
> 3) 매개변수화 타입인 Set<Object>, 와일드카드 타입인 Set<?> 은 안전하지만, 로 타입인 Set은 제네릭 타입 시스템에 속하지 않으며 안전하지 않다!
