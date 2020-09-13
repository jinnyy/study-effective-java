# item 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라

<br><br>

java.util.concurrent 패키지의 실행자 프레임워크(Executor Framework)
: 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다.

``` java
// (초판의 코드보다 모든 면에서 뛰어난) 작업 큐 생성
ExecutorService exec = Executors.newSingleThreadExecutor();

// 실행할 태스크 넘기는 방법
exec.execute(runnable);

// 종료
exec.shutdown();
```

<br>

### 주요 기능

* 특정 태스크가 완료되기를 기다린다. (get())
* 태스크 모음 중 아무것 하나(invokeAny()) 혹은 모든 태스크(invokeAll())가 완료되기를 기다린다.
* 실행자 서비스가 종료하기를 기다린다. (awaitTermination())
* 완료된 태스크들의 결과를 차례로 받는다. (ExecutorCompletionService 이용)
* 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다. (ScheduledThreadPoolExecutor 이용)

<br><br>

### 작은 프로그램 / 가벼운 서버 - Executors.newCachedThreadPool
* 특별히 설정할 것이 없음. 일반적인 용도에 적합하게 동작
* 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행됨
* 가용한 스레드가 없다면 새로 하나를 생성함
> * 그래서 서버가 무겁다면 CPU 이용률이 100%로 치닫고, 새로운 태스크가 도착하는 족족 또 다른 스레드를 생성하며 상황을 악화시킨다.


### 무거운 프로덕션 서버의 경우에는 
* 스레드 개수를 고정한 Executors.newFixedThreadPool을 선택하거나
* 완전히 통제할 수 있는 ThreadPoolExecutor를 사용하는 편이 훨씬 낫다.

<br><br>


### 스레드를 직접 다루지 말자.
직접 큐를 손수 만드는 일은 삼가야 하고, 스레드를 직접 다루는 것도 일반적으로 삼가야 한다.
* 스레드를 직접 다루면 Thread가 작업 단위와 수행 매커니즘 역할을 모두 수행하게 된다.
* 반면 실행자 프레임워크에서는 작업 단위와 실행 매커니즘이 분리된다.

<br>

### 태스크
* Runnble
* Callable (Runnable과 비슷하지만 값을 반환하고 임의의 예외를 던질 수 있다.)

<br>

### 실행자 서비스
태스크를 실행하는 일반적인 매커니즘.
* 태스크 수행을 실행자 서비스에 맡기면 원하는 태스크 수행 정책을 선택할 수 있고, 언제든 변경 가능하다.
* 핵심: 실행자 프레임워크가 작업 수행을 담당해준다는 것

<br>

자바 7부터 실행자 프레임워크는 포크-조인(fork-join) 태스크를 지원하도록 확장됐다. 
**ForkJoinTask**의 인스턴스는 작은 하위 태스크로 나뉠 수 있고 ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리하며, 
일을 먼저 끝낸 스레드가 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있다.

