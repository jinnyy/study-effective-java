# item 78. 공유 중인 가변 데이터는 동기화해 사용하라
부제: 가변 데이터 공유하지 마

<br><br>

|                        | 원자성 (배타적 실행) | 스레드 간 통신 |
|------------------------|----------------------|----------------|
| `synchronized`         |           O          |        O       |
| `volatile`             |           X          |        O       |
| special types          |           O          |        O       |
| long, double 이외 타입 |           O          |        X       |

<br><br><br>


## 동기화의 기능
1. 배타적 실행: 스레드가 한 객체에 접근할 떄 lock을 걸고, 다른 스레드에서 접근 못 함
2. 스레드 간 통신: 다른 스레드에서 일어난 변화를 확인할 수 있게 함

- 동기화는 배타적 실행 뿐 아니라, 스레드 사이의 안정적인 통신에 꼭 필요하다.

> - long, double 외의 변수를 읽고 쓰는 동작은 원자적(atomic)이다.
> - 한 스레드가 저장한 값이 다른 스레드에 보이는가는 보장하지 않음


<br><br>

## 다른 스레드를 멈추는 작업
* Thread.stop()은 사용하지 말지. (deprecated)
	- java 11 부터는 Thread.stop(Throwable obj)이 제거되었다.
	- 데이터가 훼손되어 의도와 다른 결과가 발생 가능하다.
* 스레드를 멈추는 올바른 방법
	- 조건

<br>

### 예 - 동기화되지 않은 경우
(boolean 필드를 읽고 쓰는 작업은 원자적이라서 동기화를 제거한 경우)
``` java
// 코드 78-1 잘못된 코드 - 이 프로그램은 얼마나 오래 실행될까? (415쪽)
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)	// 변경된 stopRequested 값이 보이지 않을 수 있다.
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```
- 바뀐 stopRequested 값이 보이지 않아 멈추지 않았다.
- 원인: 동기화

<br>

최적화 전
``` java
while (!stopRequested)
	i++;
```

최적화 후
``` java
if (!stopRequested)
	while (true)
		i++;
```

> OpenJDK 서버 VM에서 적용하는 끌어올리기(hoisting) 최적화 기법

<br>


### 해결 1 - `synchronized`
* 메소드에 synchronized 키워드 사용
* 쓰기 메소드와 읽기 메소드를 모두 동기화했음에 주목!
* 1초 후에 종료된다.

``` java
// 코드 78-2 적절히 동기화해 스레드가 정상 종료한다. (416쪽)
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}  
```

> #### 참고
> 예시의 경우에는 `2. 스레드 간 통신` 목적으로만 사용된 것

<br>


### 해결 2 - `volatile`
* 필드에 volatile 키워드 사용
* volatile 한정자는 `1. 배타적 수행`과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다. (`2. 스레드 간 통신`만 관련 있음)

``` java
// 코드 78-3 volatile 필드를 사용해 스레드가 정상 종료한다. (417쪽)
public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

<br><br>

단, volatile은 주의해서 사용해야 한다. 

### 문제 (안전 실패)
매번 고유한 값을 반환하는 의도로 만들어진 메소드
``` java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
	return nextSerialNumber++;
}
```

* `++`에서 변수에 실제로는 두 번 접근함 (값을 읽고, 1 더한 새로운 값을 저장)
* 두 접근 사이에 다른 스레드가 값을 읽어가면 첫 번째 스레드와 두 번째 스레드가 같은 값을 리턴받음
	- 첫 번째 스레드: 3을 `읽음`
	- 첫 번째 스레드: 3+1 연산
	- 두 번째 스레드: 3을 읽음
	- 첫 번째 스레드: 3+1 결과를 `저장`
	- 두 번째 스레드: 3+1 연산
	- 두 번째 스레드: 3+1 결과를 `저장`
	- 첫 번째와 두 번째 스레드가 같은 값을 리런받음


> #### 안전 실패
> * 프로그램이 잘못된 결과를 계산해내는 위와 같은 오류

<br>

### 해결
1. 동기화하면 된다.
2. 아이템 59에 따라, java.util.concurrent.atomic 패키지의 AtomicLong을 사용
	- 이 패키지는 원자성(배타적 실행), 통신 모두 지원

* 참고: 예시의 경우 더 견고하게 하려면 int 대신 long을 사용하거나 최댓값에 도달하면 예외를 던지게 수정

``` java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
	return nextSerialNum.getAndIncrement();
}
```

<br><br><br>


## 결론
* 불변 데이터만 공유하거나 아무것도 공유하지 말자. (가변 데이터는 단일 스레드에서만 쓰도록 하자)
* 사실상 불변(effectively immutable): 어떤 객체에서 공유하는 부분만 동기화
* 안전 발행(safe publication): 다른 스레드에 사실상 불변인 객체를 건네는 행위


<br><br>

> #### 참고
> * [사실상 불변](https://sysgears.com/articles/effectively-immutable-objects/)
