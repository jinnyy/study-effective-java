#### Effective Java 3/E
> 2020-06-08
>
> 아이템 #25 톱레벨 클래스는 한 파일에 하나만 담으라

<br><br><br><br><br>





# 25. 톱레벨 클래스는 한 파일에 하나만 담으라

소스 파일 하나에 톱레벨 클래스를 여러개 선언하여도 자바 컴퍼일러는 문제삼지 않지만, 문제 되는 경우가 존재한다.

#### 한 파일에 클래스 2개가 정의되어있는데, 다른 파일에 똑같은 클래스명으로 클래스 2개가 정의되어 있는 경우

``` java
// Utensil.java, Dessert.java 에 동시 정의
class Utensil {
   static final String NAME = "pot";
}

class Dessert {
   static final String NAME = "pie";
}
```

컴파일에 실패하거나 컴파일 순서에 따라 어떻게 동작할지 예측할 수 없게 된다
* `javac Main.java Dessert.java` Utensil 참조를 먼저 만나 중복 발생으로 컴파일 에러
* `javac Dessert.java Main.java` 정상 컴파일

> 결론: 한 파일에는 하나의 톱레벨 클래스, 인터페이스만 작성
> * 톱레벨 클래스들을 서로 다른 소스 파일로 분리
> * 정적 팩터리 메서드: 다른 클래스에 딸린 부차적인 클래스라면 사용

``` java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```
