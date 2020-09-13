# item 79. 과도한 동기화는 피하라
과도한 동기화는 **성능**을 떨어뜨리고, **교착상태**에 빠뜨린다.
예측할 수 없는 동작을 낳기도 한다.

<br><br><br>

응답 불가와 안전 실패를 피하려면 동기화 메소드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.

* 동기화된 영역 안에서는 재정의할 수 있는 메소드는 호출하면 안됨
* 동기화된 영역 안에서는 클라이언트가 넘겨준 함수 객체(아이템24)를 호출해서도 안 된다.

동기화된 영역을 포함한 클래스 관점에서는 이런 메소드는 모두 `외계인`이다.

<br>

그 메소드가 
- 무슨 일을 할 지 알지 못하고
- 통제할 수 없다

``` java
// 코드 79-1 잘못된 코드. 동기화 블록 안에서 외계인 메서드를 호출한다. (420쪽)
   private final List<SetObserver<E>> observers
           = new ArrayList<>();

   public void addObserver(SetObserver<E> observer) {
       synchronized(observers) {
           observers.add(observer);
       }
   }

   public boolean removeObserver(SetObserver<E> observer) {
       synchronized(observers) {
           return observers.remove(observer);
       }
   }

   private void notifyElementAdded(E element) {
       synchronized(observers) {
           for (SetObserver<E> observer : observers)
               observer.added(this, element);	// <- 외계인 메소드
       }
   }
```

<br>


## 발생 가능한 문제
* 예외 발생
* 교착상태
* 안전 실패 (데이터 훼손)

<br>

## 해결
* 외계인 메소드 호출을 동기화 블록 바깥으로 옮긴다.

``` java
// 코드 79-3 외계인 메서드를 동기화 블록 바깥으로 옮겼다. - 열린 호출 (424쪽)
   private void notifyElementAdded(E element) {
       List<SetObserver<E>> snapshot = null;
       synchronized(observers) {
           snapshot = new ArrayList<>(observers);
       }
       for (SetObserver<E> observer : snapshot)
           observer.added(this, element);
   }
```

<br><br>


## 기본 규칙

동기화 영역에서는 가능한 한 일을 적게 하는 것

<br>


## 성능
과도한 동기화가 초래하는 비용
* 락을 얻는 데 드는 CPU 시간 (minor)
* 경쟁하느라 낭비하는 시간 (major)
	- 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간
	- 가상머신의 코드 최적화를 제한

<br><br>



## 가변 클래스를 작성할 때 가이드
다음 중 하나를 따르자.

1. 동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 하자.
	- 예: java.util (구식..이 된 Vetor, Hashtable 제외)
2. 동기화를 내부에서 수행해 스레드 안전한 (thread safe) 클래스로 만들자.
	- 단, 클라이언트다 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 두 번째 방법을 선택
	- 예: java.util.concurrent (item 81)

선택하기 어렵다면, 동기화하지 말고, 문서에 "스레드 안전하지 않다"라고 명시하자.

<br><br><br>




## 결론

* 교착 상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메소드를 절대 호출하지 말자.
	- 동기화 영역 안에서의 작업은 최소로 줄이자.
* 가변 클래스를 설계할 대는 스스로 동기화해야 할 지 고민하자.
	- 합당한 이유가 있을 때만 내부에서 동기화하자
	- 동기화했는지 여부를 문서에 정확히 밝히자

