

> #### Effective Java 3/E
> 2020-05-25
>
> 아이템 #04 인스턴스화를 막으려거든 private 생성자를 사용하라

<br><br><br><br><br>





# 04. 인스턴스화를 막으려거든 private 생성자를 사용하라

유틸리티 클래스와 같은 도구용 클래스, 즉 정적 메서드와 멤버만을 가지고 있는 클래스는 인스턴스화가 필요치 않다 (static method 들로 구성하여 사용)

* 그러나 생성자를 따로 정의하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다
* 클라이언트 입장에서는 이 생성자가 자동 생성된 것인지 구분할 수 없고, 의도치 않게 인스턴스화 되는 상황이 발생할 수 있다

## 접근제어자를 private으로 하자

* 상속을 막는 효과도 있다
* 생성자 내부에서 `throw new AssertionError();`를 통해 클래스 안에서라도 실수로 생성자를 호출하지 않도록 할 수 있다

``` java
public class JacksonUtils {
    private JacksonUtils() {}
    public static <T> T convertObject(..) {...}
    public static String writeValueAsString(..) {...}
}
```

