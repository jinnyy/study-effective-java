# 39. 명명 패턴보다 애너테이션을 사용하라

<br><br><br>


## 명명 패턴
: 이름으로 기능을 구분해서 실행하는 패턴

* 예: JUnit에서 버전 3까지 테스트 메소드 이름을 test로 시작하게 했다.

효과적인 방법이지만 **단점** 도 크다.
1. 오타가 나면 안 된다.
2. 올바른 프로그램 요소에서만 사용되리라고 보장할 방법 없다.
3. 프로그램 요소를 매개변수로 전달할 방법이 없다.

이런 문제를 해결해 주는 개념으로 JUnit4 부터는 애너테이션을 도입하였다.

<br>


## 마커 애너테이션 타입 선언

@Test와 같은 애너테이션을 아무 매개변수 없이 단순히 대상에 마킹(Marking)한다는 뜻에서 마커 애너테이션 (Marker Annotation)이라고 한다.
이 애너테이션을 사용하면 @Test 애너테이션에 오타를 내면 컴파일 오류를 내준다.

#### 마커 애너테이션 타입 선언

``` java
/**
* 테스트 메서드임을 선언하는 애너테이션이다.
* 매개변수 없는 정적메서드 전용이다.
*/
@Retention(RetentionPolicy.RUNTIME) // @Test가 Runtime에도 유지되어야 함
@Target(ElementType.METHOD)         // @Test가 메소드 선언에만 시용되어야 함
public @interface Test{
}
```

* 주석에 있는 내용을 강제하려면 애너테이션과 별개로 애너테이션 처리기를 직접 구현해야 한다.


#### 마커 애너테이션을 사용한 프로그램 예

``` java
public class Sample {
    @Test
    public static void m1() {
        //성공
    }
  
    public static void m2() {
        //실행되지 않는다.
    }
    @Test
    public static void m3() {
        //실패
        throw new RuntimeException("실패");
    }
    public static void m4() {
        //실행되지 않는다.
    }
    @Test
    public void m5() {
        //잘못 사용한 예
        //static method가 아니다.
    }
    public static void m6() {
        //실행되지 않는다.
    }
    @Test
    public static void m7() {
        //실패
        throw new RuntimeException("실패");
    } 
    public static void m8() {
        //실행되지 않는다.
    }
}
```
* 1개 성공, 2개 실패, 1개 잘못 사용
* 잘못 사용한 경우 (m5)
* @Test를 붙이지 않은 4개의 메소드는 테스트 도구가 무시

애너테이션은 Sample클래스의 의미에 직접적으로 영향을 주지는 않는다.
그저 애너테이션에 관심있는 프로그램에게 추가 정보를 제공할 뿐이다.
다시 말하면, 프로그램 코드에의 의미는 그대로 둔 채 애너테이션에 관심있는 도구에서 특별히 처리하도록 하는 것이다.

<br>


## 메타 애너테이션 (Meta Annotation)

애너테이션 타입에 다는 애너테이션을 메타 애너테이션 (Meta Annotation)이라 한다.

### 메타 애너테이션의 종류

* @Documented: 문서에도 애너테이션 정보가 표현되게 함
* @Inherited: 자식클래스가 애너테이션을 상속받을 수 있게 함
* @Repeatable: 애너테이션을 반복적으로 사용할 수 있게 함
* @Retention(RetentionPolicy): 애너테이션의 범위를 지정 (어느 시점까지 유효한지?)
  - RetentionPolicy.RUNTIME: 컴파일 이후에도 JVM에 의해 참조가 가능 - 보통 이거로 설정
  - RetentionPolicy.CLASS: 컴파일러가 클래스를 참조할 때 까지 유효
  - RetentionPolicy.SOURCE: 애너테이션 정보가 컴파일 이후 사라짐
* @Target(ElementType[]): 애너테이션이 적용될 위치를 선언
  - ElementType.PACKAGE: 패키지 선언시
  - ElementType.TYPE: 타입 선언시
  - ElementType.CONSTRUCTOR: 생성자 선언시
  - ElementType.FIELD: 멤버 변수 선언시
  - ElementType.METHOD: 메소드 선언시
  - ElementType.ANNOTATION_TYPE: 어노테이션 타입 선언시
  - ElementType.LOCAL_VARIABLE: 지역 변수 선언시
  - ElementType.PARAMETER: 매개 변수 선언시
  - ElementType.TYPE_PARAMETER: 매개 변수 타입 선언시
  - ElementType.TYPE_USE: 타입 사용시

<br><br><br>


## 매개변수가 있는 애너테이션

매개변수를 추가하여 특정 예외를 던져야만 테스트가 성공하도록 할 수 있다.

``` java
// 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MadExceptionTest {
    Class<? extends Throwable> value();
}
```

Class<? extends Throwable> 의 와일드 카드 타입
* Throwable을 확장한 클래스의 Class 객체
* 모든 예외(와 오류) 타입을 다 수용한다.

``` java
// 코드 39-5 매개변수 하나짜리 애너테이션을 사용한 프로그램 (241쪽)
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // 성공해야 한다.
        int i = 0;
        i = i / i;
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {  // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }  // 실패해야 한다. (예외가 발생하지 않음)
}
```

<br><br><br>



## 매개변수가 Array인 애너테이션


배열 매개변수를 받게 변경할 수 있다. 
앞서 살펴본 방식보다 문법적으로 조금 더 유연함을 기대할 수 있다. 
게다가 앞선 @MadExceptionTest의 매개변수를 배열로 변경함에도 기존의 애너테이션 샘플 코드는 수정할 필요가 없다.

``` java
// 코드 39-6 배열 매개변수를 받는 애너테이션 타입 (242쪽)
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MadExceptionTest {
    // 배열로 변경한다.
    Class<? extends Throwable>[] value();
}
```

``` java
// 배열 매개변수를 받는 애너테이션을 사용하는 프로그램 (242-243쪽)
public class Sample3 {
    // 이 변형은 원소 하나짜리 매개변수를 받는 애너테이션도 처리할 수 있다. (241쪽 Sample2와 같음)
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // 성공해야 한다.
        int i = 0;
        i = i / i;
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {  // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }  // 실패해야 한다. (예외가 발생하지 않음)

    // 코드 39-7 배열 매개변수를 받는 애너테이션을 사용하는 코드 (242-243쪽)
    @ExceptionTest({ IndexOutOfBoundsException.class,
                     NullPointerException.class })
    public static void doublyBad() {   // 성공해야 한다.
        List<String> list = new ArrayList<>();

        // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
        // NullPointerException을 던질 수 있다.
        list.addAll(5, null);
    }
}
```

``` java
// 마커 애너테이션과 배열 매개변수를 받는 애너테이션을 처리하는 프로그램 (243쪽)
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }

            // 배열 매개변수를 받는 애너테이션을 처리하는 코드 (243쪽)
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    Class<? extends Throwable>[] excTypes =
                            m.getAnnotation(ExceptionTest.class).value();
                    for (Class<? extends Throwable> excType : excTypes) {
                        if (excType.isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed)
                        System.out.printf("테스트 %s 실패: %s %n", m, exc);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n",
                passed, tests - passed);
    }
}
```

<br><br><br>


## 반복 가능 애너테이션
자바 8부터는 앞서 살펴본 배열 매개변수 대신 애너테이션에 `@Repeatable` 메타애너테이션을 사용하여 여러 개의 값을 받을 수 있다. 
단, 아래와 같이 `@Repeatable`을 달고 있는 애너테이션을 반환하는 컨테이너 애너테이션을 하나 더 정의하고 `@Repeatable`에 이 컨테이너 애녀테이션의 class 객체를 매개변수로 전달해야 한다.

컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다. 
그리고 컨테이너 애너테이션 타입에는 적절한 `보존 정책(@Retention)`과 `적용 대상(@Target)`을 명시해야 한다. 그렇지 않으면 컴파일되지 않는다.

``` java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(MadExceptionContainer.class)
public @interface MadExceptionTest {
    Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MadExceptionContainer {
    MadExceptionTest[] value();
}
```

테스트하는 코드와 애너테이션 처리 코드를 작성해보자. 반복 가능 애너테이션을 여러 개 다는 경우와 하나만 달았을 때를 구분하기 위하여 ‘컨테이너’ 애너테이션 타입이 적용된다. 
getAnnotationsByType 메서드는 둘의 차이를 구분하지 않지만 isAnnotationPresent 메서드는 구분한다.

따라서 달려있는 애너테이션 수와 상관없이 모두 검사하기 위해 둘을 따로따로 검사해야 한다.

<br><br><br>


## 요약

* 애너테이션이 명명패턴을 이용할 때 보다 확실히 낫다.
* 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.
* 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다.



<br><br>

## 참고
* https://jaehun2841.github.io/2019/02/04/effective-java-item39/#%EB%B0%98%EB%B3%B5-%EA%B0%80%EB%8A%A5-%EC%95%A0%EB%84%88%ED%85%8C%EC%9D%B4%EC%85%98-repeatable
* https://madplay.github.io/post/prefer-annotations-to-naming-patterns
