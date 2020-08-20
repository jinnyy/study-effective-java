# item 60. 정확한 답이 필요하다면 float와 double은 피하라
float, double 대신 int, long, BigDecimal


<br><br>

## 1. float과 double을 피하자

* float, double 타입은 특히 금융 관련 계산과는 맞지 않는다.
* 0.1 혹은 음의 거듭 제곱수(10^-1, 10^-2 등)를 표현할 수 없기 때문이다.

### 예
``` java
System.out.println(1.03 - 0.42);
System.out.println(1.00 - 9 * 0.10);
```
결과
```
0.6100000000000001
0.09999999999999998
```


``` java
// 코드 60-1 오류 발생! 금융 계산에 부동소수 타입을 사용했다. (356쪽)
public static void main(String[] args) {
    double funds = 1.00;
    int itemsBought = 0;
    for (double price = 0.10; funds >= price; price += 0.10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");  // 3개 구입
    System.out.println("잔돈(달러): " + funds);   // 잔돈: 0.3999999999999999
}
```


<br><br>


## 2. BigDecimal, int, long을 사용하자

### BigDecimal
장점
- 정확한 값을 얻을 수 있다.

단점
- 기본 타입보다 쓰기가 훨씬 불편하다.
- 기본 타입보다 훨씬 느리다.

``` java
// 코드 60-2 BigDecimal을 사용한 해법. 속도가 느리고 쓰기 불편하다. (356쪽)
public static void main(String[] args) {
    final BigDecimal TEN_CENTS = new BigDecimal(".10");

    int itemsBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for (BigDecimal price = TEN_CENTS;
         funds.compareTo(price) >= 0;
         price = price.add(TEN_CENTS)) {
        funds = funds.subtract(price);
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");   // 4개 구입
    System.out.println("잔돈(달러): " + funds);     // 잔돈(달러): 0.00
}
```

### int, long
대안으로 int나 long을 사용 가능하다.
- 다룰 수 있는 값의 크기가 제한됨
- 소수점을 직접 관리해야 함
(이 예시에서는 달러 대신 센트로 연산을 수행하면 해결됨)


``` java
// 코드 60-3 정수 타입을 사용한 해법 (357쪽)
public static void main(String[] args) {
    int itemsBought = 0;
    int funds = 100;
    for (int price = 10; funds >= price; price += 10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입"); // 4개 구입
    System.out.println("잔돈(센트): " + funds);   // 잔돈(센트): 0
}
```
