# item 52. 다중정의는 신중히 사용하라

재정의(override)한 메서드는 런타임에 동적으로 선택되며, **다중정의(overload)한 메서드는 컴파일타임에 정적으로 선택된다**

<br>

```java
public class CollectionClassifier {
  public static String classify(Set<?> s) { return "집합"; }
  public static String classify(List<?> l) { return "리스트"; }
  public static String classify(Collection<?> c) { return "그 외"; }
  
  public static void main(String[] args) {
    Collection<?>[] collections = {
      new HashSet<String>(),
      new ArrayList<BigInteger>(),
      new HashMap<String, String>().values()
    };
    
    for (Collection<?> c: collections)
      System.out.println(classify(c));  // 뭐가 출력될까용
  }
}
```

1. 매개변수 수가 같은 다중정의는 만들지 말자
   - 가변인수(varargs)를 사용하는 메서드라면 다중정의를 아예 하지 말자 (아이템 53에 예외를 기술하였다고 한다)

2. 다중정의 하는 대신 메서드 이름을 다르게 지어주자
   - ex. `ObjectOutputStream` 의 `writeBoolean(boolean)`, `writeInt(int)`, `writeLong(long)`
   
3. 생성자는 이름을 다르게 지을 수 없으니 정적 팩터리를 활용하라

4. 매개변수 중 하나 이상이 근본적으로 다르다면 (radically different: 두 타입의 null이 아닌 값을 서로 형변환할 수 없다) 헷갈릴 일은 없을 것이다
   - Object 외의 클래스 타입과 배열 타입은 근본적으로 다르며, Serializable 과 Cloneable 외의 인터페이스 타입과 배열 타입도 근본적으로 다르다
   - 관련 없는 (상하위 관계가 아님) 클래스는 근본적으로 다르다
   - 주의 ❗️ **1) 오토박싱의 등장으로 기본 타입과 참조 타입(Object)은 근본적으로 다르지 않게 되었으며, 2) 서로 다른 함수형 인터페이스는 근본적으로 다르지 않다**

5. 다중 정의 메서드가 똑같이 동작하도록 만든다 - 더 특수한 다중정의 메서드에서 더 일반적인 다중정의 메서드로 일을 넘겨버린다

```java
public final class String {

  public boolean contentEquals(CharSequence cs) { ... }

  public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence) sb);
  }
}
```

<br>

## +) appendix (radically different)

1. 오토박싱의 등장으로 **기본 타입**과 **참조 타입**이 더 이상 근본적으로 다르지 않게 되었다

   - `List` 의 `remove` 메서드는 `Object` 과 `int` 를 받는 메서드를 다중정의했다

| Inteface | Method                                  |
|----------|-----------------------------------------|
| Set      | remove(Object o)                        |
| List     | remove(Object o) <br> remove(int index) |


```java
for (int i  = -3; i < 3; i++) {
  set.add(i);
  list.add(i);
}

for (int i = 0; i < 3; i++) {
  set.remove(i);              // [-3, -2, -1]
  list.remove(i);             // [-2, 0, 2] 🤔
}
```

```java
for (int i = 0; i < 3; i++) {
  list.remove((Integer) i);   // [-3, -2, -1] 🙂
}
```

<br>

2. 람다와 메서드 참조의 등장 - **함수형 인터페이스**라도 서로 근본적으로 다르지 않다

   - `ExecutorService` 의 `submit` 메서드는 `Runnable` 과 `Callable` 을 받는 메서드를 다중정의했다

> 함수형 인터페이스: 1개의 추상메소드를 가지고 있는 인터페이스 
> (ex. Function<T, R>, Predicate<T>, [Runnable](https://docs.oracle.com/javase/7/docs/api/java/lang/Runnable.html), [Callable](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Callable.html))

<img width="1380" alt="스크린샷 2020-07-19 오후 5 10 19" src="https://user-images.githubusercontent.com/39719108/87870454-2690bb80-c9e3-11ea-904b-05619ca90540.png">

<details>
    <summary>올바른 println 메서드를 선택하게 했을 경우 - 정상적으로 동작</summary>

<img width="800" alt="스크린샷 2020-07-19 오후 5 10 53" src="https://user-images.githubusercontent.com/39719108/87870450-1bd62680-c9e3-11ea-9b65-90202f3bdcc4.png">

</details>

