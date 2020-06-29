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

<br><br>


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


## 요약

* 애너테이션이 명명패턴을 이용할 때 보다 확실히 낫다.
* 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.
* 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다.



