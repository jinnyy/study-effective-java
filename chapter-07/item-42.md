

> #### Effective Java 3/E
> 2020-07-06
>
> 아이템 #42 익명 클래스보다는 람다를 사용하라

<br><br><br><br><br>





# 42. 익명 클래스보다는 람다를 사용하라
## 함수 객체
* 함수의 타입을 나타낼 때 사용. 추상 메서드를 하나만 담은 인터페이스 (또는 추상 클래스)
* 특정 함수나 동작을 나타내는 데 사용

<br><br>

## 익명 클래스
* 함수 객체를 만들 때 사용
* 코드가 길기 때문에 함수형 프로그래밍에 적합하지 않음

``` java
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("one", "two", "three", "four");

        Collections.sort(words, new Comparator<String>() {
            public int compare(String s1, String s2) {
                return Integer.compare(s1.length(), s2.length());
            }
        });
    }
}
```




<br><br>


## 람다 (Lambda)

* `JDK 1.8` 부터는 함수형 인터페이스를 만드는데 해당 인터페이스의 인스턴스를 람다식을 사용해 만들 수 있게 되었다
* 컴파일러가 알아서 반환 타입, 매개변수 타입을 추론하여 넣어준다 (컴파일러가 추론못하겠다고 오류를 뱉을 때만 직접 명시)
* 람다는 Lambda Factory 라고 불리는 call site 에 각종 정보(외부인자값 등)와 함께 저장
  * invokedynamic 명령어를 통해 메소드가 동적으로 바인딩되고 MethodHandle로 실행된다
* 익명내부클래스는 class load, new 로 객체를 생성할 때 메모리 할당, 초기화 등의 비용이 들어가지만, 람다는 컴파일 타임의 활동(jvm bytecode 레벨에서 invokedynamic 추가)이므로 런타임에 추가비용이 크지 않다

``` java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("one", "two", "three", "four");

        Collections.sort(words,
                (s1, s2) -> Integer.compare(s1.length(), s2.length()));
    }
}
```
<br><br>

* `JDK 1.8` 버전 이상을 사용하게 되면 `List 인터페이스` 에 추가된 `sort 메서드` 를 사용할 수 있다

``` java
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("kim", "taeng", "mad", "play");
        words.sort(Comparator.comparingInt(String::length));
    }
}
```
<br><br>

Operation enum의 인스턴스 필드에 람다 사용

``` java
enum Operation {ㄴ
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
        public double apply(double x, double y) { return x * y; }
    };
    
    private final String symbol;
   
    Operation(String symbol) { this.symbol = symbol; }
    
    @Override public String toString() { return symbol; } 
    public abstract double apply(double x, double y);
}
```

``` java
import java.util.function.DoubleBinaryOperator;

enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() { return symbol; }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}

public class Main {
    public static void main(String[] args) {
        // 사용은 아래와 같이
        Operation.PLUS.apply(2, 3);
    }
}
```
<br><br><br>

### 람다의 한계

* 이름이 없고, 문서화 할 수 없다
* 코드가 길어지거나, 동작이 명확하지 않아 설명이 필요할 때는 쓰지 않는 방향으로 리펙토링
* 추상 클래스의 인스턴스를 만들 때 람다를 사용할 수 없다
* 자기 자신 참조가 안된다
* 직렬화 형태가 구현별로 다를 수 있다 (private 정적 중첩 클래스의 인스턴스를 사용)

``` java
abstract class Hello {
    public void sayHello() {
        System.out.println("Hello!");
    }
}

public class Main {
    public static void main(String[] args) {
        Hello instance1 = new Hello() {
            private String msg = "Hi";
            @Override public void sayHello() {
                System.out.println(msg);
            }
        };

        Hello instance2 = new Hello() {
            private String msg = "Hola";
            @Override public void sayHello() {
                System.out.println(msg);
            }
        };

        // Hi!
        instance1.sayHello();

        // Hola!
        instance2.sayHello();
    }
}
```
<br><br>

``` java 
class Anonymous {
    public void say() {}
}

public class Main {
    public void someMethod() {
        List<Anonymous> list = Arrays.asList(new Anonymous());

        Anonymous anonymous = new Anonymous() {
            @Override
            public void say() {
                System.out.println("this instanceof Anonymous : " + (this instanceof Anonymous));
            }
        };
        
        // this instanceof Anonymous : true
        anonymous.say();

        // this instanceof Main : true
        list.forEach(o -> System.out.println("this instanceof Main : " + (this instanceof Main)));
    }

    public static void main(String[] args) {
        new Main().someMethod();
    }
}
```

<br><br>

> 익명 클래스는 타입의 인스턴스를 만들 때만 사용
> 람다는 작은 함수 객체를 쉽게 표현하고자 할 때 사용
