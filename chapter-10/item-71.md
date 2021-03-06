# item 71. 필요 없는 검사 예외 사용은 피하라


### 불필요한 검사 예외는 클라이언트에게 고통을 줄 수 있다

* API를 제대로 사용해도 발생할 수 있는 예외이거나, 프로그래머가 의미 있는 조치를 취할 수 있는 경우라면 검사 예외를 던져도 괜찮지만 그렇지 않다면...
* 프로그래머가 예외를 어떻게 다룰지 생각해보자

```java
} catch (TheCheckedException e) {
  throw new AssertionError(); // 일어날 수 없다!
}
```
```java
} catch (TheCheckedException e) {
  e.printStackTrace(); // ...
  System.exit(1);
}
```

* 메서드가 단 하나의 검사 예외만 던질 때 프로그래머에게 지우는 부담은 더욱 커진다 - 하나의 예외 때문에 try 블록을 추가해야 하고 스트림에서 직접 사용하지 못하게 된다

<br>

### 검사 예외를 피하는 방법

1. 적절한 결과 타입을 담은 **빈 옵셔널 반환**

   * 단, 예외를 발생한 이유를 알려주는 부가 정보를 담을 수 없다
2. 검사 예외를 던지는 메서드를 2개로 쪼개기 -> **상태 검사 메서드 + 비검사 예외를 던지는 메서드**
   * 유연한 API를 제공할 수 있다 - 클라이언트가 메서드가 성공하리라는 걸 안다거나 실패시 스레드를 중단하길 원한다면 간단하게 작성할 수 있다 (ex. `obj.action(args);`)
   * 상태 검사 메서드를 사용하기 때문에, 1) 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인에 의해 상태가 변할 수 있을 경우, 2) 상태 검사 메서드가 기존 메서드의 작업 일부를 중복 수행하는 경우에는 적절하지 않다

<br>

```java
// 검사 예외를 던지는 메서드

try {
  obj.action(args)
} catch (TheCheckedException e) {
  ... // 예외 상황에 대처한다
}
```

```java
// 상태 검사 메서드와 비검사 예외를 던지는 메서드

if (obj.actionPermitted(args)) {
  obj.action(args);
} else {
  ... // 예외 상황에 대처한다
}
```

<br>

> 세 줄 요약:
> - 꼭 필요한 곳에만 사용한다면 검사 예외는 프로그램의 안전성을 높여주지만, 남용하면 클라이언트에게 고통을 줄 수 있다
> - API 호출자가 예외 상황에서 복구할 방법이 없다면 비검사 예외를 던지자
> - 복구가 가능하고 호출자가 그 처리를 해주길 바란다면, 우선 옵셔널을 고려해보자. 옵셔널만으로는 상황을 처리하기에 충분한 정보를 제공할 수 없을 때만 검사 예외를 던지자.
