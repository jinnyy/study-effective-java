#### Effective Java 3/E
> 2020-06-08
>
> 아이템 #24 멤버 클래스는 되도록 static 으로 만들라

<br><br><br><br><br>





# 24. 멤버 클래스는 되도록 static 으로 만들라

중첩 클래스(nested class)는 다른 클래스 안에 정의된 클래스를 말한다 (맴버 클래스와 같은 말로 이해).  <br>
중첩 클래스는 자신을 감싼 바깥 클래스에만 쓰여야하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

## 중첩 클래스는 보통 왜 사용하는가?

* 클래스들의 논리적인 그룹을 나타낼 때 쓴다. (model 객체에서 상위모델과 하위모델이 있을 때)
* 향상된 캡슐화
* 좋은 가독성과 유지보수성

#### 내부 클래스 (Inner Class)

* Non-Static Nested Class
* 비정적 맴버 클래스, 익명 클래스, 지역 클래스
* 밖에 있는 클래스의 자원을 직접 사용

``` java
class Outer { 
    변수; 
    메소드;

    public class Inner { } 
}

// 객체 생성
Outer Instance1 = new Outer(); 
Outer.Inner Instance2 = Instance1.new Inner();
```

* 컴파일 시 클래스 파일 Outer.class, Outer$Inner.class 생성

* 지역 클래스 (Method Local Inner Class)
  * 메소드 내부에 클래스를 정의하는 경우. 메소드 내의 지역변수처럼 쓰인다.
  * 메소드 내부에서 new 한 뒤 사용해야 한다. 메소드 밖에서 사용할 수 없다(지역변수 룰)
  * 비정적 문맥에서만 바깥 인스턴스 참조, 정적 맴버는 가질 수 없다.
  * 짧게 작성할 것.
  * 컴파일 시 클래스 파일: Outer.class, Outer$숫자Inner.class

``` java
class Outer { 
    변수; 
    메소드1; 
    메소드2() { 
        지역변수; 
        class Inner { } 
    } 
}
```

* 익명 클래스 (Anonymous Inner Class)
  * 인스턴스 이름이 없다. 생성과 동시에 부모에게서 상속을 받아 내부에서 오버라이딩해서 사용.
  * 상속은 받아야하지만, 한번만 사용할 것이라서 extends 문법을 굳이 사용안함
  * 비정적 맥락에서만 외부 클래스 자원 사용 가능, 정적 맥락에서도 상수 변수 외 정적 맴버 가질 수 없음
  * 이 외에도 제약이 많다 (instanceOf 등 불가, 여러 인터페이스 구현 불가능, 인터페이스 구현하면서 다른 클래스 상속 불가)
  * 가독성을 위해 짧아야 한다.
  * 람다에게 자리를 물려주었다 (정정 팩터리 메서드를 구현할 때 사용 가능)
  * 
  * 컴파일 시 클래스 파일: Outer.class, Outer$숫자.class

``` java
class Inner { 
    변수; 
    메소드; 
} 

class Outer { 
    변수; 
    메소드1; 
    메소드2() { 
        지역변수; 
        new Inner() { 
            override된 내용들.. 
        } 
    } 
}
```

#### 정적 맴버 클래스

* Static Nested Class
* 중첩클래스는 바깐 클래스 자원에 static 키워드가 안붙었다면 사용할 수 없다.
* Outer 클래스의 객체가 없어도 Inner 클래스의 객체 생성이 가능하다.

``` java
class Outer { 
    변수; 
    메소드;
    
    public static class Inner { } 
}
```

중첩클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들다. <br>
비정적(non-static)일 경우 바깥 인스턴스에 대한 숨은 참조(바깥 클래스명.this)를 가져올 수 있게 되는데 비용이 들어간다. <br>
생성자를 호출할 때 자동으로 만들어지는 것이 보통이지만, 수동으로 만들 수도 있다. <br>
비정적 맴버 클래스의 인스턴스 안에 관계 정보가 만들어져 메모리 공간을 차지하고, 생성도 느리다. <br>
이 참조때문에 GC가 제때 수거하지 못해서 메모리 누수가 생길 수도 있다. <br>
비정적 멤버클래스는 바깥 인스턴스에 접근할 일이 있을 때 사용한다. (어댑터를 정의할 때: 컬렉션 패키지가 컬렉션 뷰 keySet, entrySet, values 등을 반환할 때)

> 어댑터: 어떤 클래스의 인스턴스를 감싸 다른 클래스의 인스턴스처럼 보이게 하는 뷰

``` java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    public Collection<V> values() {
        Collection<V> vs = values;
        if (vs == null) {
            vs = new Values();
            values = vs;
        }
        return vs;
    }

    final class Values extends AbstractCollection<V> {
        public final void clear() {
            HashMap.this.clear();
        }
        ...
    }
    ...
}
```
톱레벨 클래스 내부에서만 쓰인다면 **private 중첩 클래스**로 작성하여 접근범위를 최소화시키는 것이 좋다.

> 결론: 중첨 클래스에는 네 가지가 있고, 각각 쓰임이 다른다.
> * 메서드 밖에서 사용해야 하거나 메서드 안에 정의하기에 길이가 길다면 맴버 클래스로 만든다.
> * 맴버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적으로, 그렇지 않으면 정적으로.
> * 중첨 클래스가 한 메서드 안에서만 쓰이면서 생성 지점이 한 곳이며, 이미 적합한 클래스나 인터페이스가 있다면 익명 클래스.
> * 그렇지 않으면 지역 클래스
