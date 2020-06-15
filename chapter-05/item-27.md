# 27. 비검사 경고를 제거하라

> **What is an "unchecked" warning?**
> A warning by which the compiler indicates that it cannot ensure type safety. The term "unchecked" refers to the fact that the compiler and the runtime system do not have enough type information to perform all type checks that would be necessary to ensure type safety. In this sense, certain operations are "unchecked". 

<br>

## 1. 비검사 경고를 제거하라

* 비검사 경고를 제거함으로써 타입 안정성이 보장된다! 즉 런타임에 ClassCastException이 발생할 일이 없고 의도한 대로 잘 동작하리라고 확신할 수 있다.

```java
Set<Lark> exaltation = new HashSet();   // unchecked conversion
Set<Lark> exaltation = new HashSet<>(); // good 👍
```

<br>

## 2. SuppressWarnings("unchecked")

* 경고를 제거할 수는 없지만 타입 안전성을 확신할 수 있다면 SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨기자

  * 애너테이션은 어떤 선언에도 달 수 있지만, 가능한 한 좁은 범위에 적용하자 (ex. 변수 선언, 아주 짧은 메서드, 생성자)
  * 안전한 이유를 주석으로 남겨라

<br>

* 타입 안정성이 보장됨을 알지만 비검사 경고가 발생하는 예

```java
public <T> T[] toArray(T[] a) {
  if (a.length < size)
    return (T[]) Arrays.copyOf(elements, size, a.getClass()); // unchecked cast warning
  ...
}
```
* 경고를 숨겨줍시다 🥳
```java
public <T> T[] toArray(T[] a) {
  if (a.length < size)
    // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
    @SuppressionWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
    return result;
  ...
}
```

<br>

> 두 줄 요약:
> 1) 모든 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻하니 최선을 다해 제거하라.
> 2) 경고를 없앨 방법을 찾지 못하겠다면, 그 코드가 타입 안전함을 증명하고 가능한 한 범위를 좁혀 @SuppressWarnings("unchecked") 애너테이션으로 경고를 숨겨라. 그런 다음 경고를 숨기기로 한 근거를 주석으로 남겨라
