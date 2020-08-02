# item 55. 옵셔널 반환은 신중히 하라

## Optional이란

자바8 이전에 메서드가 값을 반환 할 수 없을 때 취할 수 있는 선택지는 크게 두가지 였다.


<br>
1. null 반환

   **문제점**

   > null방어코드를 작성해야함 만약 그렇지 않을 경우 추적이 제대로 되지 않는 NullPointException 발생 가능성

2. 예외를 던짐

   **문제점**

   > 예외처리는 진짜 예외적인 상황을 위해 존재하는 기능으로, 값이 비었다. 를 표현하는데에는 적합하지 않음
   >
   > Stack전체를 캡처하는것의 비용



자바8부터는 Optional이 대안으로 등장했다. `Optional<T>` 는 null 이 아닌 T타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다. 즉, 원소를 최대 1개 가질 수 있는 '불변' 컬렉션이다.



보통 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야 할 때 T 대신 `Optional<T>` 를 반환하도록 선언하면 된다.



<br><br>

## 주의점

### null을 반환하는 Optional 메서드
```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  if(c.isEmpty())
    return Optional.empty();
  
  E result = null;
  for (E e : c)
    if(result == null || e.compareTo(result) >0)
      result = Objects.requireNonNull(e);
  
  return Optional.of(result);
}
```

옵셔널은 위와과 같이`Optional.empty()` , `Optional.of(value)` 와 같은 정적팩토리 적절히 사용하여 구현할 수 있다.

그런데 간혹 Optional의 반환형이라고 해서 null을 반환해도 된다고 착각하는 경우가 있는데, Optional 객체로 감싼 value가 null이어도 된다는 뜻이지, 해당 메서드의 반환값이 null일 경우 이전에 null을 반환하는 일반적 상황의 메서드와 동일한 문제가 발생할 것이다.



### 컨테이너 타입의 옵셔널

null이 의심되는 상황에서는 Collection.emptyList(혹은 Map 등..)을 사용하지, Optional을 사용하는것은 바람직하지 않다.

<br>

### 박싱된 기본 타입 옵셔널

OptionalInt, OptionalLong, OptionalDouble을 사용하자.

<br>

### Map의 값으로 사용

옵셔널을 Map의 key, value, 배열의 원소로 사용하는게 적절한 상황은 거의없다.

만약 사용할 경우, 맵안에 키가 없다는 사실을 나타내는 방법이 두가지나 생기게 되는것이므로 쓸데없는 복잡성을 야기한다.


<br>
<br>

## 어떻게 사용하나요

**반환값이 옵셔널** 인 경우는 이를 사용하는 클라이언트는 이렇게 생각하면 된다.

=> **반환값이 있을수도 있고, 없을수도 있다**

즉, 결과가 없을 수 있으며, 클라이언트가 상황을 특별하게 처리해야 하는 경우 사용한다.

<br>

옵셔널은 검사 예외와 취지가 비슷하다. 따라서 메서드 또한, Null일 경우와 아닐경우의 대신할 값을 설정하는 방식이다.

1. 기본값으로 대치

   ```java
   String lastWordInLexicon = max(words).orElse("단어 없음...");
   ```

2. 원하는 예외를 던짐

   ```Java
   Toy myToy = max(toys),orElseThrow(TemperTantrumException::new);
   ```

3. 항상 값이 채워져 있는경우

   ```java
   Element lastNobleGas = max(Elements.NOBLE_GASES).get();
   ```

   > 애초에 값이 항상 채워있을 경우엔 Optional 을 사용하지 않는것을 추천하지만, 인터페이스 수준에서 반환값이 Optional인 경우도 있으므로



기본값을 설정하는 비용이 커서 부담이 될 수 있다. 이럴때는 `Supplier<T>` 를 인수로 받는 `orElseGet` 을 사용 할 수 있다.



Optional은 엄연히 새로 할당해서 생성해주어야 하는 객체이고, 안에서 값을 꺼내려면 메서드를 호출해야하므로 **성능이 중요한 상황에서는 옵셔널이 맞지 않을 수 있다**.



<br>

## 고급 메서드 사용 예시

값을 좀더 다양한 형태로 처리하기 위해 다음과 같은 메서드들이 지원된다

`filter`, `map`, `flatMap`, `ifPresent` 



```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID: " + (parentProcess.isPresent() ? String.valueOf(parentProcess.get().pid()) : "N/A"));
```

```java
System.out.println("부모 PID: " + ph.parent().map(h -> String.value(h.pid))).orElse("N/A"));
```

<br>

스트림을 사용한다면 옵셔널들을 `Stream<Optional<T>>`로 받아서 그중 채워진 옵셔널들에서 값을 뽑아 `Stream<T>`에 건네 처리하는 경우도 있음

```java
streamOfOptionals
  .filter(Optional::isPresent)
  .map(Optional::get)
```

<br>



자바 9에서는 Optional에 stream() 메서드가 추가되었다. 이 메서드는 Optional을 Stream으로 변환해주는 어댑터로, 옵셔널에 값이 있으면 그 값을 원소로 담은 스트림으로, 값이 없으면 빈 스트림으로 변환해준다.

이를 사용하면 위의 코드가 아래처럼 단순화된다

```java
streamOfOptionals
  .flatMap(Optional::stream)
```

