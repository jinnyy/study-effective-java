#### Effective Java 3/E
> 2020-09-21
>
> 아이템 #82 스레드 안전성 수준을 문서화하라

<br><br><br><br><br>





# 82. 스레드 안전성 수준을 문서화하라
` synchronized` 키워드가 있다고 해서 스레드 안전한 것일까? <br>
스레드 안전성에도 어느 정도의 수준인지 나뉜다

## 스레드 안전성 수준
- 불변 (immutable)
  - 외부 동기화도 필요 없음 ex. String, Long, BigInteger
- 무조건적 스레드 안전 (unconditionally thread-safe)
  - 값이 변할 수 있지만 내부에서 충실히 동기화. 외부 동기화 필요 없음.
  - ex. AtomicLong, ConcurrentHashMap
- 조건부 스레드 안전 (conditionally thread-safe)
  - 일부 메서드는 동시에 사용하려면 외부 동기화 필요
  - ex. Collections.synchronized 레퍼 클래스가 반환한 메서드
- 스레드 안전하지 않음 (not thread-safe)
  - 수정될 수 있으며 동시에 사용하려면 각각의 메서드 호출을 클라이언트가 선택한 외부 동기화 로직으로 감싸야 한다
  - ArrayList, HashMap
- 스레드 적대적 (thread-hostile)
  - 외부 동기화를 사용하더라도 멀티스레드 환경에서 안전하지 않음

## 문서화 방법
- 어떠한 순서로 호출할 때 외부 동기화 로직이 필요한지
- 그 순서대로 호출하려면 어떤 락을 얻어야만 하는지

- Collections.synchronizedMap API 문서:
``` java
/**
 * It is imperative that the user manually synchronize on the returned
 * map when iterating over any of its collection views
 * 반환된 맵의 콜렉션 뷰를 순회할 때 반드시 그 맵으로 수동 동기화하라
 * 
 *  Map m = Collections.synchronizedMap(new HashMap());
 *      ...
 *  Set s = m.keySet();  // Needn't be in synchronized block
 *      ...
 *  synchronized (m) {  // Synchronizing on m, not s!
 *      Iterator i = s.iterator(); // Must be in synchronized block
 *      while (i.hasNext())
 *          foo(i.next());
 *  }
 */
```

## 외부에 공개된 Lock
- 클라이언트가 공개된 락을 가지고 놓지 않는 서비스 거부 공격(denial-of-service attack)을 수행할 수 있음
- synchronized 메서드도 공개된 락에 속함
- 비공개 락 객체를 사용하는 것이 좋다

``` java
// 비공개 락 객체, final 선언!
private final Object lock = new Object();

public void someMethod() {
    synchronized(lock) {
        // do something
    }
}
```
- 우연히라도 락 객체가 교체되는 상황을 방지
- 클라이언트 또는 이를 상속하는 하위 클래스에서 동기화 로직을 깨뜨리는 것을 예방
