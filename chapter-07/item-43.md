
> #### Effective Java 3/E
> 2020-07-06
>
> 아이템 #43 람다보다는 메서드 참조를 사용하라

<br><br><br><br><br>





# 43. 람다보다는 메서드 참조를 사용하라

## 메서드 참조

함수 객체를 람다보다도 간결하게 만드는 방법
* 매개변수가 늘어날수록 메서드 참조로 줄일 수 있는 코드량이 많다
* (동시에 매개변수의 이름이 좋은 가이드가 되기도 한다)
* intelliJ도 메서드 참조를 사용하라고 워닝을 띄운다

``` java 
//java 8 map안에 추가된 merge
//before
map.merge(key, 1, (count, incr) -> count + incr);

//after
map.merge(key, 1, Integer::sum);
```

* 람다도 할 수 없는 일은 메서드 참조로도 할 수 없다 (예외: 제네릭 함수 타입 구현)

<br><br>

## 람다가 메소드 참조보다 간결한 경우

#### 주로 메서드와 람다가 같은 클래스에 있을 때

``` java
class GoshThisClassnameIsHumongous{
    //before
    service.execute(GoshThisClassnameIsHumongous::action);

    //after
    service.execute(() -> action());
}
```
* Function.identity() 보단  x -> x 
* 다만, 위의 둘은 완전히 같지 않다: (https://stackoverflow.com/questions/28032827/java-8-lambdas-function-identity-or-t-t)

<br><br>

## 메소드 참조의 유형

* 정적 메서드를 가리키는 참조 (`Integer.sum`)
* 수신 객체를 한정 (`Instant.now()::isAfter`)
* 수신 객체를 한정하지 않음 (`String::toLowerCase`)
* 클래스 생성자 (`TreeMap<K, V>::new`)
* 배열 생성자 (`int[]::new`)

<br><br>

## Call Stack 관점

* userList.forEach(user -> System.out.println(user)); : ArrayList.forEach -> Consumer.accept -> System.out.println
* userList.forEach(System.out::println); : Consumer.accept 생략
* (https://softwareengineering.stackexchange.com/questions/277473/is-there-a-performance-benefit-to-using-the-method-reference-syntax-instead-of-l)

<br><br>


> 메소드 참조는 람다의 대안이 될 수 있다
> 메소드 참조 쪽이 더 간결할 때 사용
