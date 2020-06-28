# 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

<br><br><br>


`열거 타입(enum type)`은 거의 모든 상황에서 `안전 타입 열거 패턴(typesafe enum pattern)`보다 우수하다.
* 예외: `타입 안전 열거 패턴`은 확장 가능. `열거 타입`은 확장 불가능 
* 즉 `열거 타입`은 열거한 값들을 그대로 가져온 뒤 값을 추가하여 사용 불가

<br>

대부분의 경우 열거 타입을 확장하는 건 좋지 않다.

- 확장한 타입의 원소는 기반 타입의 원소로 취급하지만, 그 반대는 성립하지 않는다면 이상함
- 기반 타입과 확장된 타입들의 원소 모두를 순회하는 방법도 불분명
- 확장성을 높이려면 고려할 요소가 늘어남 -> 설계와 구현이 복잡해짐


<br><br>




## 확장할 수 있는 열거 타입이 어울리는 쓰임

* operation code (op code)


<br><br>




## 열거 타입을 확장할 수 있는 방법

기본 아이디어: enum 타입이 interface를 implement할 수 있다는 사실을 이용

#### 단계
1. 연산 코드용 interface를 정의한다.
2. enum 타입이 이 인터페이스를 구현하게 한다.
  enum 타입이 그 인터페이스의 표준 구현체 역할을 한다.


#### 예
열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다.

``` java
// 코드 38-1 인터페이스를 이용해 확장 가능 열거 타입을 흉내 냈다. (232쪽)
public interface Operation {
    double apply(double x, double y);
}
```

``` java
// 코드 38-1 인터페이스를 이용해 확장 가능 열거 타입을 흉내 냈다. - 기본 구현 (233쪽)
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

지수 연산(EXP)과 나머지 연산(REMAINDER)를 추가하려면
* Operation 인터페이스를 구현한 열거 타입을 작성하면 됨

 ``` java
 // 코드 38-2 확장 가능 열거 타입 (233-235쪽)
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}

```


<br><br>

새로 작성한 연산은 기존 연산을 쓰던 곳이면 어디든 쓸 수 있다.
apply가 interface (Operation)에 선언되어 있으니 enum 타입에 따로 추상 메소드로 선언하지 않아도 된다.

### 1. Enum타입을 넘겨 순회하기
개별 인스턴스 뿐 아니라 타입 수준에서도 기본 열거 타입 대신 확장된 열거 타입을 넘겨 
확장된 열거 타입의 원소 모두를 사용하게 할 수도 있다.

``` java
// 열거 타입의 Class 객체를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예 (234쪽)
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}
private static <T extends Enum<T> & Operation> void test(
        Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```

- Class 객체가 enum 타입이고 (동시에) Operation의 하위 타입이어야 한다.



### 2. Collection<? extends Operation> 넘겨 순회하기
(class 객체 대신) 한정적 와일드카드 타입을 넘기는 방법

``` java
// 컬렉션 인스턴스를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예 (235쪽)
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

// Operation의 하위 타입인 T 가 담긴 Collection 타입을 인자로 넘기면 됨
private static void test(Collection<? extends Operation> opSet,
                         double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}

```

- 장점: 그나마 덜 복잡, test 메소드가 살짝 더 유연해졌다. (여러 구현 타입의 연산을 조합해 호출 가능)
- 단점: 특정 연산에서 enumSet(아이템 36)과 enumMap(아이템 37)을 사용하지 못한다.


<br><br>


## 문제점 
enum 타입끼리 구현을 상속할 수 없다는 점

### 해결 방법
아무 상태에도 의존하지 않는 경우
* 디폴트 구현(아이템 20)을 이용해 인터페이스에 추가
* Operation 예시의 경우 연산 기호를 저장하고 찾는 로직이 BasicOperation과 ExtendedOperation 모두에 들어가야 함
* 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메소드로 분리해 코드 중복 제거


<br><br><br>




## 핵심 정리

* 열거 타입 자체는 확장할 수 없다.
* 인터페이스와 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다.
* 클라이언트는 이 인터페이스를 구현해 자신만의 열거(혹은 다른) 타입을 만들 수 있다.

