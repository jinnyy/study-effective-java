#### Effective Java 3/E
> 2020-06-08
>
> 아이템 #21 인터페이스는 구현하는 쪽을 생각해 설계하라

<br><br><br><br><br>





## 인터페이스 메서드 추가
자바 8 전에는 기존 구현체를 깨뜨리지 않고는 인터페이스에 메서드를 추가할 방법이 없었다.
* 기존 인터페이스에 메서드를 추가했을 때, 정의되지 않은 추가 된 메서드로 인해 컴파일 오류가 발생한다.
  - 기존 구현체에 (그 메소드가) 이미 존재할 가능성이 아주 낮기 때문
* 자바 8에 와서 기존 인터페이스에 메서드를 추가할 수 있도록 **디폴트 메서드** 가 나왔다. 
  - 그러나 모든 기존 구현체들과 매끄럽게 연동되리라는 보장은 없다. 
  - 게다가 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어려운 법이다.



#### 예 - Collection 인터페이스에 추가된 removeIf 메소드

``` java
default boolean removeIf(Predicate<? super E> filter) {
	Objects.requireNonNull(filter);
	boolean removed = false;
	final Iterator<E> each = iterator();
	while (each.hasNext()) {
		if (filter.test(each.next())) {
			each.remove();
			removed = true;
		}
	}
	return removed;
}
```

위 코드를 보면 Predicate가 true를 반환하는 모든 원소를 제거한다. 코드를 이보다 더 범용적으로 구현하기도 어렵겠지만, 
그렇다고 해서 현존하는 모든 Collection 구현체와 잘 어우어지는 것은 아니다.

이의 대표적인 예가 org.apache.commons.collections.collection.SynchronizedCollecion이다. 
이는  클라이언트가 제공한 객체로 락을 거는 능력을 추가로 제공한다. 
즉, 모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스(아이템18)이다.

그러나 removeIf에서는 동기화에 관해 아무것도 모르므로 락 객체를 사용할 수 없다. 
그 결과 멀티 스레드의 환경에서 removeIf를 호출할 경우 ConcurrentModificationException이 발생하거나 다른 예기치 못한 결과로 이어질 수 있다.

이처럼 디폴트 메서드는 컴파일에 성공하더라도 기존 구현체에 **런타임 오류** 를 일으킬 수 있다. 
그러므로 인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야 한다.

기존 인터페이스에 디폴트 메소드로 새 메소드를 추가하는 일은 꼭 필요한 경우가 아니라면 피해야 한다.
추가하려는 메소드가 기존 메소드와 충졸하지 않을지 심사숙고해야함도 당연하다.

<br><br><br>




## 핵심

자바 8 부터 인터페이스에는 디폴트 메서드가 도입되었고, 이를 통해 기존 인터페이스에 새로운 메서드를 추가할 수 있게 되었다.
* 그러나 디폴트 메서드를 통해 새로운 메서드를 추가하더라도 모든 기존 구현체들과 매끄럽게 연동되지 않을 수 있다.
* 따라서 인터페이스를 설계할 때는 세심한 주의를 기울여야 한다. 그리고 인터페이스를 수정하는 것이 가능한 경우도 있겠지만, 절대 그 가능성에 기대해서는 안 된다.

테스트도 해야 한다.
* 서로 다른 방식으로 최소한 세 가지 구현
* 각 인터페이스의 인스턴스를 다양한 작업에 활용하는 클라이언트 여러 개 구현




<br><br><br><br><br>

## References
* [이펙티브 자바 3E 책 & 코드](https://github.com/WegraLee/effective-java-3e-source-code)
* [블로그 백수개발자-item21](https://happy-playboy.tistory.com/entry/Item-21-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EB%8A%94-%EA%B5%AC%ED%98%84%ED%95%98%EB%8A%94-%EC%AA%BD%EC%9D%84-%EC%83%9D%EA%B0%81%ED%95%B4-%EC%84%A4%EA%B3%84%ED%95%98%EB%9D%BC)
