#### Effective Java 3/E
> 2020-06-08
>
> 아이템 #22 인터페이스는 타입을 정의하는 용도로만 사용하라

<br><br><br><br><br>


* 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.
  - 즉, 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것이다.
* 인터페이스는 오직 이 용도로만 사용되어야 한다.

### 잘못된 예 - 상수 인터페이스
``` java
// 코드 22-1 상수 인터페이스 안티패턴 - 사용금지! (139쪽)
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```
* 상수 인터페이스: static final 필드로만 가득 찬 인터페이스
* 상수 인터페이스를 구형하는 것은 내부 구현을 클래스의 API로 노출하는 행위임
* 사용자 입장에서 의미가 없는 정보를 노출함으로써 혼란을 줄 수 있음
* 클라이언트 코드가 내부 구현에 해당하는 (인터페이스 내부의) 상수들에 종속되게 함
  - 다음 릴리즈에서 해당 상수를 더이상 사용하지 않더라도 바이너리 호환성을 위해 상수 인터페이스를 유지해야 함
* final이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위 클래스의 이름공간(name spacce)이 그 인터페이스가 정의한 상수들로 오염됨


### 해결방법
상수를 공개할 목적이라면
* 특정 클래스나 인터페이스와 강하게 연관된 상수라면 -> 그 클래스나 인터페이스 자체에 추가
  - 예: Integer.MIN_VALUE, Integer.MAX_VALUE
* 열거(enum) 타입으로 나타내기 적합하면 enum으로 만들어 공개
* 인스턴스화할 수 없는 유틸리티 클래스에 담아 공개
  - 예
  ``` java
  public class PhysicalConstants {
      // 아보가드로 수 (1/몰)
      public static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

      // 볼츠만 상수 (J/K)
      public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

      // 전자 질량 (kg)
      public static final double ELECTRON_MASS      = 9.109_383_56e-31;
  }
  ```
  
상수를 공개할 목적이 아니라면
* 클래스 내부에서 private으로 구현하면 되겠죠?


