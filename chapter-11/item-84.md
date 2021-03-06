#### Effective Java 3/E
> 2020-09-21
>
> 아이템 #84 프로그램의 동작을 스레드 스케줄러에 기대지 말라

<br><br><br><br><br>





# 84. 프로그램의 동작을 스레드 스케줄러에 기대지 말라

- 다른 플랫폼으로의 이식성이 떨어짐
- Thread priorities 는 가장 나중에 생각할 문제 (중요하지 않다)
- 만약 robustness를 올리고자 한다면 (실행 가능한)스레드 개수가 processor의 수보다 크게 많지 않도록 잘 조절하는 것으로 충분
  - 전체 스레드수 = 대기중인 스레드수(실행 가능하지 않은 스레드 수) + 실행중인 스레드 수

## Thread Scheduler
- 어떤 스레드를 오래 실행할까?

## 실행 가능한 스레드 수를 적게 유지하는 법
- 당장 처리해야 할 작업이 없다면 실행되어서는 안된다
- 스레드 풀 크기 적절히 설정, 작업은 짧게 유지
- 절대 busy waiting 없도록 (공유 객체가 바뀔 때까지 쉬지 않고 검사하지 않도록)

``` java
public class SlowCountDownLatch { // 바쁜 대기 버전 CountDownLatch 구현
  private int count;

  public SlowCountDownLatch(int count) {
    if (count < 0)
      throw new IllegalArgumentException(count + " < 0");
    this.count = count;
  }

  public void await() {
    while (true) {
      synchronized(this) {
        if (count == 0)
          return;
      }
    }
  }
  public synchronized void countDown() {
    if (count != 0)
      count--;
  }
}
```

## Thread.yield
- 테스트할 수단도 없고, 다음 jvm에서 꼭 성능이 좋아지리라는 보장도 없음
