

> #### Effective Java 3/E
> 2020-05-25
>
> 아이템 #08, #09


<br><br>
<p align="center">
  * 이 글은 잘 정리된 블로그 내용을 가져왔음 *
</p>
<br>


<br><br><br><br><br>




# 08.  finalizer와 cleaner 사용을 피하라

## Finalizer
``` java
@Override
public void finalize() {
    // ...
}
```
finalize 메서드를 Override 하면 해당 객체가 JVM 에게 Garbage Collection 을 해야 할 대상이 될 때 호출된다. 객체가 없어지기 전 다른 연관 자원을 정리하려는 의도로 작성된다.

하지만 이 메서드는 사용해서는 안되며 실제로 java9+ 부터 Deprecated 되어버렸다.

오류/시점/성능/수행성 뭐 하나 보장하지 못하며 때로는 영원히 수행되지 않거나 불행하게도 Lock 이 걸려 프로그램 전체가 블럭될 수도 있다.

java9 에서는 대안으로 Cleaner 를 지원하게 되었다

<br><br>


## Cleaner
Java9 에서 도입된 소멸자로 생성된 Cleaner 가 더 이상 사용되지 않을 때 등록된 스레드에서 정의된 클린 작업을 수행한다.

혹은 명시적으로 clean 을 수행할수도 있다. 보통 AutoCloseable을 구현해서 try-with-resource 와 같이 사용한다. (이 편이 추천된다)

``` java
public class CleaningRequiredObject implements AutoCloseable {

    private static final Cleaner cleaner = Cleaner.create​();

    private static class CleanData implements Runnable {

        @Override
        public void run() {
            // 여기서 클린 작업 수행
        }
    }

    private final CleanData;
    private final Cleaner.Cleanable cleanable

    public CleaningRequiredObject() {
        this.cleanData = new CleanData();

        // 등록
        this.cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() {
        cleanable.clean();
    }
}
```

자칫 Clean 작업을 실제로 수행할 클래스의 디자인에 실패해서 다른 외부 참조나 의존성이 걸릴 경우, 최악의 경우 순환의존성 덕분에 GC의 기회가 없어질 수도 있다.

이를 피하기 위해 보통은 AutoCloseable - try-with-resource 로 안전장치를 거는 편이 좋다.

<br><br>


# 09. try-finally보다는 try-with-resources를 사용하라
## try-with-resource
I/O 등의 작업에서는 어떤 모듈이 사용이 종료될 경우 해당 자원을 해지하고 없애야 할때가 많다.

보통 그럴 경우 try-finally 구문으로 처리하는데 이 경우 코드가 상당히 지저분하다.

또한 작업 메서드가 오류가 나더라도 close 를 해야 할 경우와 close 자체에서도 오류를 던지는 경우가 있어 그 두 부분을 전부 try-catch 하다보면 코드 가독성은 현저하게 저하된다.

try-with-resource 로 이런 고충을 한방에 날려버릴 수 있다.

다음 Worker 클래스는 테스트를 위해 명시적으로 오류를 내고 있다. 이 경우에 try-with-resource 를 사용하면 아주 깔끔한 코드가 나오며, 오류또한 잘 캡처된다.

``` java
public class Boss {

    public static class Worker implements AutoCloseable {
        
        public String work() {
            throw new RuntimeException("work Exception!");
        }

        @Override
        public void close() {
            throw new RuntimeException("close Exception!");
        }
    }

    public static void main(String...args) {

        // 짧다!
        try(Worker worker = new Worker()) {
            worker.work();
        }
    }
}
```
출력 오류는 다음과 같다

```
Exception in thread "main" java.lang.RuntimeException: work Exception!
	at Boss$Worker.work(Boss.java:6)
	at Boss.main(Boss.java:16)
	Suppressed: java.lang.RuntimeException: close Exception!
		at Boss$Worker.close(Boss.java:10)
		at Boss.main(Boss.java:18)
```

* Cleaner / Finalizer 둘다 애매하다.
* 하지만 둘다 일반적으로는 사용이 불필요하다.

* 둘다 성능에 문제가 많고, Serialize 를 통한 보안 이슈가 존재하며, 수행 시점이 보장되지 않는다.
* JVM 구현에 따라 동작도 매우 달라질 여지가 많다.

* 사용할수밖에 없을때는 다음과 같은 케이스가 있다
	* JNI
	* Off-Heap 메모리 사용시 (DirectBuffer 류 사용시)


<br><br><br>



### 출처

* [객체 소멸자의 슬픈 디자인 (Effective java 3th - Item8, 9)](https://blog.javarouka.me/2018/11/26/Finalizer%EC%99%80-Cleaner/)

