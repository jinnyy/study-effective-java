# item 50. 적시에 방어적 복사본을 만들라

## 문제 상황: 불변식을 깨뜨리려고 혈안이 되어있는 클라이언트의 악의적인 행동

🤓 불변식(**시작 시각이 종료 시각보다 늦을 수 없다**)을 지키지 못한 클래스 :
```java
public final class Period {
  private final Date Start;
  private final Date end;
  
  public  Period(Date start, Date end) {
    if (start.compareTo(end) > 0)
        throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
    this.start = start;
    this.end = end;
  }
  
  public Date Start() {
    return start;
  }
  
  public Date end() {
    return end;
  }
}
```

<br>

😈 클라이언트의 공격 :
```java
Date yesterday = new Date();
Date tomorrow = new Date();

Peroid p = new Period(yesterday, tomorrow);
tomorrow.setYear(2019); // p의 내부를 변경한다
```

```java
Peroid p = new Period(yesterday, tomorrow);
p.end().setYear(2019); // p의 내부를 변경한다
```

<br>

일단 **가변 클래스 Date 대신 불변인 Instant를 사용**하는 것이 가장 이상적인 해결 방법이다. 그렇지만 이전에 작성된 낡은 코드라 이 방법을 써먹을 수 없는 경우라면 **방어적 복사**를 이용하자

<br>

## 해결 방법: 방어적 복사

### step 1. 생성자 - 매개변수의 방어적 복사본을 만든다 

```java
public  Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    
    if (start.compareTo(end) > 0)
        throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
  }
```

- 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사하라
   - 멀티스레딩 환경이라면 원본 객체의 유효성을 검사한 후 복사본을 만드는 찰나의 순간에 다른 스레드가 원본 객체를 수정할 위험이 있다
- 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안 된다

### step 2. 접근자 - 가변 필드의 방어적 복사본을 반환한다

```java
public Date start() {
  return new Date(start.getTime());
}

public Date end() {
  return new Date(end.getTime());
}
```

<br>

> 세 줄 요약:
> - 되도록 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어든다
> - 클래스가 클라이언트로부터 받거나 혹은 클라이언트로 반환하는 구성요소가 가변이라면 그 요소는 반드시 방어적으로 복사한다.
> - 단, 복사 비용이 너무 크거나 클라이언트가 그 요소를 악의적으로 수정할 일이 없음을 신뢰한다면 방어적 복사를 수행하는 대신 해당 구성요소를 수정했을 때 책임이 클라이언트에 있음을 문서에 명시한다.
