
> #### Effective Java 3/E
> 2020-07-06
>
> 아이템 #44 표준 함수형 인터페이스를 사용하라

<br><br><br><br><br>



# 44. 표준 함수형 인터페이스를 사용하라

## 표준 함수형 인터페이스

* java.util.function 패키지에 총 43개의 인터페이스
* 기본 6개의 인터페이스를 기억하고 나머지는 유추해서 사용


인터페이스 | 함수 시그니처 | 예시
----------- | ---------- | ---------
UnaryOperator | T apply(T t) |  String::toLowerCase 
BinaryOperator | T apply(T t1, T t2) | BigInteger::add 
Predicate | boolean test(T t) | Collection::isEmpty 
Function<T,R> | R apply(T t) | Arrays::asList 
Supplier | T get() | Instant::now 
Consumer | void accept(T t) | System.out::println

<br><br>

Example: 

``` java 
@FunctionalInterface interface EldestEntryRemovalFunction<K,V>{
  boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}

// 표준 함수형 인터페이스 사용
BiPredicate<Map<K,V>, Map.Entry<K,V>>
```

## 전용 함수형 인터페이스 구현이 필요한 경우

* 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다
* 반드시 따라야 하는 규약이 있다
* 유용한 디폴트 메서드를 제공할 수 있다

#### @FunctionalInterface

* 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다
* 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다
* 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다


<br><br>

> 용도에 맞는 것이 있다면 직접 구현하지 말고 표준 함수형 인터페이스를 사용하자
> 표준 함수형 인터페이스는 대부분 기본 타입 지원 (박싱된 타입 사용 X -> 성능이 안좋음)
> 직접 만든 함수형 인터페이스에는 @FunctionalInterface 사용