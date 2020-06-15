# 29. 이왕이면 제네릭 타입으로 만들라

<br>

## 1. 일반 클래스를 제네릭 클래스로 만드는 방법

1. 클래스 선언에 타입 매개변수를 추가한다
2. 코드에 쓰인 Object를 적절한 타입 매개변수로 변경한다
3. 비검사 경고를 제거한다

```java
public class Stack {
  private Object[] elements;
  
  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  
  public void push(Object e) {
    elements[size++] = e;
  }
  
  public Object pop() {
    Object result = elements[--size];
    elements[size] = null;
    return result;
  }
}
```

* 위 Stack 클래스는 스택에서 꺼낸 객체를 형변환할 때 런타임 오류가 날 위험이 있기 때문에 제네릭 적용이 필요하다!

<br>

## 2. 배열을 사용한 코드를 제네릭으로 만드는 방법 - 실체화 불가 타입으로는 배열을 만들 수 없다

* **배열 형변환**: Object 배열을 생성해 제네릭 배열로 형변환 한다

  * 힙 오염을 발생시킬 수 있다 (아이템 32)
  * 코드가 더 짧고, 형변환을 배열 생성시 단 한번만 해주면 되기 때문에 현업에서 선호된다

```java
private E[] elements;

// 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다
@SuppressWarnings("unchecked")
public Stack() {
  elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

* **원소 형변환**: 배열 필드 타입을 E[]에서 Object[]로 변경하고, 필요시 원소를 형변환한다

```java
private Object[] elements;

public E pop() {
  ...
  // push에서 E 타입만 허용하므로 이 형변환은 안전하다
  @SupressWarnings("unchecked") E result = (E) elements[--size];
  
  elements[size] = null;
  return result;
}
```

* 두 방법 모두 ClassCastException을 발생시킬 수 있으므로 @SuppressWarnings 애너테이션으로 경고를 숨긴다

* 지금까지 설명한 Stack 예는 [아이템 28. 배열보다는 리스트를 우선하라]와 상충되어 보인다. **사실 제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하지도, 꼭 더 좋은 것도 아니다**

<br>

## 3. 한정적 타입 매개변수

```java
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```
* DelayQueue의 원소에서 형변환 없이 Delayed 클래스의 메서드를 호출할 수 있으며, ClassCastException을 걱정할 필요가 없다

<br>

> 한 줄 요약:
> 1) 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.
