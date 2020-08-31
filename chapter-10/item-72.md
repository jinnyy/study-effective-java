# item 72. 표준 예외를 사용하라

* 자바 라이브러리에서 제공하는 표준 예외를 사용하라
   * 여러분의 API가 다른 사람이 익히고 사용하기 쉬워진다
   * 여러분의 API를 사용한 프로그램도 읽기 쉬워진다
   * 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다

* `Exception`, `RuntimeException`, `Throwable`, `Error`는 직접 재사용하지 말자. 이 예외들은 여러 성격의 예외들을 포괄하는 클래스이므로 안정적으로 테스트할 수 없다.


* 널리 재사용되는 예외

|   예외                             |   주요 쓰임                                                                          |   예                                                                       |
|------------------------------------|--------------------------------------------------------------------------------------|----------------------------------------------------------------------------|
|   `IllegalArgumentException`         |   허용하지 않는 값이 인수로 건네졌을 때                                              |                                                                            |
|   `IllegalStateException`            |   객체가 메서드를 수행하기에 적절하지 않은 상태일 때                                 |                                                                            |
|   `NullPointerException`             |   Null을 허용하지 않는 메서드에 null을 건넸을 때                                     |                                                                            |
|   `IndexOutOfBoundsException`        |   인덱스가 범위를 넘어섰을 때                                                        |                                                                            |
|   `ConcurrentModificationException`  |   허용하지 않는 동시 수정이 발견됐을 때  |   단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정할 때  |
|   `UnsupportedOperationException`    |   호출한 메서드를 지원하지 않을 때                                                   |   원소를 넣을 수만 있는 List 구현체에 remove를 호출했을 때                 |

<br>

Q. 카드 덱을 표현하는 객체가 있고, 인수로 건넨 수만큼의 카드를 뽑아 나눠주는 메서드 `getCard(int number)` 를 제공한다고 해보자. 이 때 덱에 남아있는 카드 수보다 큰 값을 건네면 어떤 예외를 던져야 할까?

<details>
<summary>🤓</summary>
<br>
<div markdown="1">

* 인수 값이 무엇이었든 어차피 실패했을 거라면 `IllegalStateException`을, 그렇지 않으면 `IllegalArgumentException`을 던지자.

</div>
</details>
