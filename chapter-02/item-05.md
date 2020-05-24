

> #### Effective Java 3/E
> 2020-05-25
>
> 아이템 #05 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

<br><br><br><br><br>





# 05. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

많은 클래스가 하나 이상의 자원(bean)에 의존한다. 
정적 유틸성 클래스나 싱글턴을 이런 자원을 내부적으로 명시하면 문제가 발생한다.

``` java
// 한국어사전에 기반한 맞춤법검사기 유틸. 다른 언어사전으로 어떻게 갈아끼울 것인가?
public class SpellChecker {
    private static final Lexicon dictionary = new KoreanDic();
    private SpellChecker() {}
    public static boolean isValid(String word) { 
        // dictionary 를 이용한 검증로직..
    }
    public static List<String> suggestions(String typo) {
        // dictionary 를 이용한 제안로직..
    }
}

// 한국어사전에 기반한 맞춤법검사기 싱글톤. 다른 언어사전으로 어떻게 갈아끼울 것인가?
public class SpellChecker {
    private final Lexicon dictionary = new KoreanDic();
    private SpellChecker() {}
    public boolean isValid(String word) { 
        // dictionary 를 이용한 검증로직..
    }
    public List<String> suggestions(String typo) {
        // dictionary 를 이용한 제안로직..
    }
}
```
* 실제로는 사전에도 여러 언어의 사전이 있고, 특수 어휘용 사전이 별도로 있기도 한다
* 테스트 또한 용이하지 않다
* final 키워드를 삭제하고 setter 메서드 등을 통해 사전을 그때그때 교체하는 방법도 생각해 볼 수 있지만, 어색하고, 오류를 내기 쉬우며, 멀티 스레드에서는 사용할 수 없다


## 의존객체 주입 패턴

* 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글톤 방식이 적합하지 않다
* 이럴 때는 인스턴스를 생성할 때 생성자, static factory method, builder로 필요한 자원을 넘겨주는 방식이 좋다
* 불변을 보장하며, 같은 자원을 사용하려는 다른 클라이언트들과 안심하고 객체를 공유할 수 있다

``` java
private final Lexicon dictionary;
public SpellChecker(Lexicon dictionary) {
    this.dictionary = dictionary;
}
```

* 의존객체를 통째로 넘겨주는 것이 아니라, 한 단계 더 들어가서 의존객체를 생성하는 Factory를 넘겨주게 할 수도 있다
* 자바8에서는 특히 이것을 명확하게 나타낼 수 있는데, Supplier 를 아래처럼 이용하는 것이다

public SpellChecker(Supplier<? extends Lexicon> dicFactory) {
    this.dictionary = dicFactory.get();
}

#### 팩토리

팩토리란 팩토리 메서드 패턴(Factory Method Pattern)[Gamma95]에서 언급하는 형태의 클래스이며, 호출할때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다

* 의존 객체 주입을 직접적으로 사용하는 경우는 거의 드물다. 의존성이 수천개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 하기 때문이다
* 큰 규모의 프로젝트에서는 주로 **스프링(Spring)**과 같은 의존 객체 주입 프레임워크 사용
