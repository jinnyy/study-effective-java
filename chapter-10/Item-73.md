## Item73. 추상화 수준에 맞는 예외를 던져라



### 예외 번역

저수준 예외를 그대로 위로 올리면 내부 구현 방식을 그대로 드러내서 윗 레벨 API를 오염시킨다.

상위 계층에서는 저수준 예외를 잡아서 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다.

이러한 방식을 **예외 번역** 이라한다.



```java
/**
* @throws IndexOutOfBoundsException '예외 설명'
*/
public E get(int index) {
	ListIterator<E> i = listIterator(index);
  try {
    return i.next();
  } catch (NoSuchElementException e) {
    throw new IndexOutOfBoundsException("인덱스: " + index);
  }
}
```

> AbstractSequentialList.class (extends List) 의 추상화된 예외 생성 예시





### 예외 연쇄

저수준 예외가 디버깅에 도움이 된다면 에러를 고수준 예외로 실어서 던지자.

```java
class HigherLevelException extends Exception {
  HigherLevelException(Throwable cause) {
    super(cause);
  }
}
```



### 정리

예외를 처리하는 다양한 방식들이 있지만, 제일 좋은 방식은 가능하다면 저수준 API에서  메서드가 반드시 성공하도록 상위 메서드에서 매개변수 검증 등의 방식으로 미리 예외가 발생하지 않도록 하는것이다.

