#### Effective Java 3/E
> 2020-09-21
>
> 아이템 #83 지연 초기화는 신중히 사용하라

<br><br><br><br><br>





# 83. 지연 초기화는 신중히 사용하라
대부분의 상황에서 지연 초기화는 사용하지 않는 것이 좋음

## 지연 초기화를 꼭 해야 한다면
- 인스턴스 필드: 이중검사 관용구 사용하기

``` java
// https://github.com/jbloch/
// Double-check idiom for lazy initialization of instance fields - Page 334
private volatile FieldType field4;
 
private FieldType getField4() {
    FieldType result = field4;
    if (result != null)    // First check (no locking)
        return result;

    synchronized(this) {
        if (field4 == null) // Second check (with locking)
            field4 = computeFieldValue();
        return field4;
    }
}
```
- 멀티 스레드 환경에서 싱글턴 객체를 만들어야 할 때: 
https://medium.com/@joongwon/multi-thread-환경에서의-올바른-singleton-578d9511fd42

<br>

- 정적 필드: 지연 초기화 홀더 클래스 관용구 사용하기

``` java
// https://github.com/jbloch
// Lazy initialization holder class idiom for static fields
 
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}
 
private static FieldType getField() { return FieldHolder.field; }
```
