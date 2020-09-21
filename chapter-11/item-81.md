#### Effective Java 3/E
> 2020-09-21
>
> 아이템 #81 wait와 notify보다는 동시성 유틸리티를 애용하라

<br><br><br><br><br>





# 81. wait와 notify보다는 동시성 유틸리티를 애용하라
`java.util.concurrent` 패키지의 고수준 동시성 유틸리티: 실행자 프레임워크, 동시성 컬렉션 그리고 동기화 장치 <br>
실행자 프레임워크는 아이템 80에서 설명


## 동시성 컬렉션
- List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 추가한 것
- 동기화를 내부에서 수행 (동시성 무력화 불가)
- 외부에서 Lock을 걸 경우 오히려 속도가 느려진다

### 상태 의존적 메소드
여러 동작을 하나의 원자적 동작으로 묶는 메서드
- 동시성 컬렉션의 동시성을 무력화하지 못하기 때문에 여러 메서드를 원자적으로 묶어 호출할 수 없다
- 몇몇 상태 의존적 메소드는 일반 컬렉션 인터페이스의 디폴트 메서드 형태로 추가됨 ex. Map의 putIfAbsent 

``` java
private static final ConcurrentMap<String, String> map =
        new ConcurrentHashMap<>();

public static String intern(String s) {
    String result = map.get(s);
    if (result == null) {
        result = map.putIfAbsent(s, s);
        if (result == null) {
            result = s;
        }
    }
    return result;
}
```

- 동기화한 컬렉션보다 동시성 컬렉션을 사용 (synchronizedMap보다는 ConcurrentHashMap)

### ConcurrentMap
- HashMap, ConCurrentHashMap, HashTable: Map 인터페이스 구현체
- key, value 값의 null 허용 여부와 속도, 동기화 보장 측면에서 차이가 있음

#### HashMap
- synchronized 키워드가 없기 때문에 동기화가 보장되지 못함
- 값을 찾는 속도가 상당히 빠르다
- key, value에 null 값을 허용
`Map m = Collections.synchronizedMap(new HashMap(...));`의 형태 선호

#### ConCurrentHaspMap
- HashMap의 멀티스레드 환경에서의 동기화 처리로 인한 문제점을 보완한 것
- key, value에 null 값을 허용하지 않음

#### HashTable
- 메서드에 전부 synchronized 키워드
- 메서드 호출 전 쓰레드간 동기화 락을 통해 멀티 쓰레드 환경에서 data의 무결성을 보장
- key, value에 null 값을 허용하지 않음
- 락때문에 속도는 느리지만, data의 안정성이 높고 신뢰가 높은 컬렉션

<br>

## 동기화 장치
- 스레드가 다른 스레드를 기다릴 수 있게 하여 서로의 작업을 조율할 수 있도록 도와준다
- Ex. CountDownLatch, Semaphore, CyclicBarrier, Exchanger, Phaser

``` java
CountDownLatch ready = new CountdownLatch(1);
ready.await();
ready.countDown();
```

- `CountDownLatch`: 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다린다
  - 생성자 인자로 받는 정수값은 래치의 countdown 메서드를 몇 번 호출해야 대기하고 있는 스레드들을 깨우는지 결정

``` java
public class CountDownLatchTest {
    public static void main(String[] args) {

        ExecutorService executorService = Executors.newFixedThreadPool(5);
        try {
            long result = time(executorService, 3,
                    () -> System.out.println("hello"));
            System.out.println(result);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            executorService.shutdown();
        }
    }

    public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                // 타이머에게 준비가 됐음을 알린다.
                ready.countDown();
                try {
                    // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    start.await();
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    // 타이머에게 작업을 마쳤음을 알린다.
                    done.countDown();
                }
            });
        }

        ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다.
        done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
        return System.nanoTime() - startNanos;
    }
}
```

#### 스레드 기아 교착상태(Thread starvation deadlock) 

time 메서드에 넘겨진 실행자(executor)는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있다. 그렇지 못하면 이메서드는 결코 끝나지 않을 것이다. 

<br>

## wait와 notify 메서드
- 새로운 구현을 할 때는 동시성 유틸리티 사용
- wait나 notify를 사용해야만 할 때는 동기화 영역 안에서 사용해야 하며, 항상 반복문 안에서 사용
- wait는 항상 while문 안에서, notify 대신 notifyAll 사용하기

``` java
synchronized (obj) {
    while (조건이 충족되지 않았다) {
        obj.wait(); // 락을 놓고, 깨어난 후 다시 락을 잡기
    }

    ... // 조건이 충족되고 난 후 동작
}
```

- 반복문은 wait 호출 전후로 조건이 만족하는지를 검사하는 역할
- 응답 불가 예방: 조건이 이미 충족되었는데 스레드가 notify 또는 notifyAll 호출 후 대기 상태에 빠지면 다시 스레드를 깨울 수 없음
- 안전 실패 예방: 대기 후에 조건을 검사하여 조건을 충족하지 않았을 때 다시 대기하게 하는 것
- 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황:
  - notify를 호출하여 대기 중인 스레드가 깨어나는 사이에 다른 스레드가 락
  - 조건이 만족되지 않았지만 실수 혹은 악의적으로 notify를 호출
  - 대기 중인 스레드 중 일부만 조건을 충족해도 notifyAll로 모든 스레드를 깨우는 경우
  - 대기 중인 스레드가 드물게 notify 없이 깨어나는 경우. 허위 각성(spurious wakeup)


#### notify vs notifyAll
notify 스레드 하나만 깨우며, notifyAll은 모든 쓰레드를 깨운다.

- notifyAll을 사용하면 관련 없는 스레드가 실수로 혹은 악의적으로 wait 호출하는 공격으로부터 보호
