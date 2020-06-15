# 28. 배열보다는 리스트를 사용하라

<br>

## 1. 배열 vs 리스트

1. 배열은 공변이고 리스트는 불공변이다

   * String이 Object의 하위타입이므로 배열 String[]는 배열 Object[]의 하위 타입이 된다
   * 서로 다른 타입 Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아니다
   * 배열과 리스트 모두 Long용 저장소에 String을 넣을 수는 없다. 다만 배열에서는 실수를 런타임에서 알게 되지만, **리스트를 사용하면 컴파일할 때 바로 알 수 있다**

```java
/* Runtime Error */

Object[] array = new Long[1];
array[0] = "타입이 달라 넣을 수 없다."  // ArrayStoreException

/* Compile Error */

List<Object> list = new ArrayList<Long>();
list.add("타입이 달라 넣을 수 없다.")
```

2. 배열은 실체화된다

   * 배열은 런타임에도 타입 정보를 인지하고 확인한다
   * 제네릭은 타입 정보를 컴파일 타임에만 검사하며 런타임에는 소거한다

* 실체화 불가 타입

   * `E`, `List<E>`, `List<String>`과 같이 소거 매커니즘으로 인해 실체화 되지 않아서 런타임에는 컴파일 타임보다 타입 정보를 적게 가지는 타입을 말한다
   * 매개변수화 타입 가운데 실체화 가능 타입은 `List<?>`와 같은 비한정적 와일드카드 타입뿐이다

<br>

## 2. 제네릭 배열 생성은 허용되지 않는다

* new List<E>[], new List<String>[], new E[] 식으로 작성하면 컴파일시 오류를 일으킨다

```java
List<String>[] stringLists = new List<String>[1]; // 제네릭 배열 생성이 허용된다고 가정해보자
List<Integer> intList = List.of(42);

Object[] objects = stringLists;
// List<String> 배열을 Object 배열에 할당 - 배열은 공변이기 때문에 성공한다

objects[0] = intList;
// List<Integer> 인스턴스를 Object 배열에 저장 - 제네릭이 소거되어 성공한다

string s = stringLists[0].get(0);                 // ClassCastException 💣
```


<br>

## 3. 배열로 형변환할 때 오류가 뜨는 경우 제네릭을 사용하라

* 배열로 형변환할 때 생성 오류, 비검사 형변환 오류가 뜨는 경우 대부분 E[] 대신 List<E>를 사용하면 된다

<br>

* before 🤯 - choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 하며, 형변환 오류의 가능성이 있다
```java
public class Chooser {
  private final Object[] choiceList;
  
  public Chooser(Collection choices) {
    choiceArray = choices.toArray();
  }
  
  public Object choose() {
    Random rnd = ThreadLocalRandomm.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```
* 1차 after 🤨 - 제네릭 사용
```java
choiceArray = (T[]) choices.toArray();      // incompatible types error
```

* 2차 after 😇 - 제네릭 및 리스트 사용
```java
public class Chooser<T> {
  private final List<T> choiceList;         // 배열 대신 리스트를 사용해 타입 안정성 확보
  
  public Chooser(Collection<T> choices) {
    choiceList = new ArrayList<>(choices);
  }
  
  public T choose() {
    Random rnd = ThreadLocalRandomm.current();
    return choiceList.get(rnd.nextInt(choiceList.size()));
  }
}
```

<br>

> 세 줄 요약:
> 1) 배열은 공변이고 실체화되는 반면, 제네릭은 불공변이고 타입 정보가 소거된다.
> 2) 배열은 런타임시 타입 안전하며, 제네릭은 컴파일시 타입 안전하다.
> 3) 둘을 섞어쓰다가 컴파일 오류나 경고를 만나면, 배열을 리스트로 대체하라.
