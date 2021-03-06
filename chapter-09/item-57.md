# item 57. 지역변수의 범위를 최소화하라

<br>

## 순서
1. 지역변수는 가장 처음 쓰일 때 선언한다.
2. 거의 모든 지역변수는 선언과 동시에 초기화해야 한다.
3. 메소드를 작게 유지하고 한 가지 기능에 집중한다.


<br><br><br>


## 1. 지역변수는 가장 처음 쓰일 때 선언한다.

지역변수의 범위를 줄이는 가장 강력한 기법은 역시 '**가장 처음 쓰일 때 선언하기**'이다.
* [아이템 15 - 클래스와 멤버의 접근권한을 최소화하라](https://github.com/jinnyy/study-effective-java/blob/master/chapter-04/EffectiveJava%20Item15_18.pdf)와 취지가 비슷하다.
* 유효 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고 오류 가능성은 낮아진다.
* 사용 하려면 멀었는데 미리 선언부터 해두면 코드가 어수선해져 가독성이 떨어진다. -> 실제로 사용하는 시점에 타입과 초기값이 기억나지 않을 수도 있다.

<br><br>


## 2. 거의 모든 지역변수는 선언과 동시에 초기화해야 한다.
* 초기화할 때 필요한 정보가 충분하지 않다면 충분해질 때까지 선언을 미룬다.
* 예: for문

#### 코드 57-1
``` java
for (Element e : collection) {
  ...
}
```
* Element e는 선언된 for문 안에서만 유효하다.

#### for문이 좋다
* 반복자(iterator)를 사용해야 하는 상황이면 for-each문 대신 전통적인 for문을 사용하는 것이 낫다.
* while문보다 for문이 낫다.
  - 런타임 에러 vs 컴파일 에러
  - 더 짧아서 가독성이 좋다.

<br><br>


## 3. 메소드를 작게 유지하고 한 가지 기능에 집중한다.

* 한 메소드에서 여러 기능을 처리한다면 -> 그중 한 기능과만 관련된 지역변수라도 다른 (기능을 수행하는) 코드에서 접근할 수 있을 것이다.
* 메소드를 기능별로 쪼개면 해결된다.




