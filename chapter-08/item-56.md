# item 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라

<br>

## JavaDoc

자바독은 소스코드 파일에서 문서화 주석이라는 특수한 형태로 기술된 설명을 추려 API문서로 변환해준다


![](https://user-images.githubusercontent.com/16698414/89122742-caf71f80-d504-11ea-909c-068398c278f3.png)




<br>


### 주의점

1. API를 올바로 문서화 하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 함

2. How 가 아닌 What을 기술할것.

3. 클라이언트가 메서드를 호출하기 위한 전제조건, 성공적으로 수행된 후 만족해야하는 사후조건을 모두 나열

4. 일반적으로 전제조건은 @throws 태그로 비검사 예외를 선언하여 암시적으로 기술해야 함

5. 부작용또한 문서화 해야한다. (부작용: 시스템의 상태에 어떤 변화를 가져오는 것)

   > 예를들어 백그라운드 스레드를 시작시키는 메서드일 경우 그 사실을 문서에 밝혀야 함

6. 제너릭 타입이나 제너릭 메서드 문서화시, 타입 매개변수에도 주석을 달아 설명해야함

   ```java
   /**
   * An object that maps keys to values. A map cannot contain
   * duplicate keys; each key can map to at most one value.
   *
   * (Remainder omitted)
   *
   * @param <K> the type of keys maintained by this map
   * @param <V> the type of mapped values
   */
   public interface Map<K, V> {...}
   ```

7. 열거타입에는 상수들에도 주석을 달아주어야 함

8. 에너테이션 문서화시 멤버들에도 모두 주석을 달아주어야 함

9. 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 함. 직렬화 할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 함.

   ![](https://user-images.githubusercontent.com/16698414/89122737-bfa3f400-d504-11ea-833d-91af22222f07.png)


<br>


### 사용법

1. HTML문서로 되므로, 태그를 통해 간단하게는 줄바꿈 부터, 표까지 표현 가능하다.

2. code 표현:  `{@code ... 코드 ...}` 

3. `@implSpec` 주석: 해당 메서드와 하위 클래스 사이의 계약을 설명. 하위 클래스들이 메서드를 상속하거나 super 키워드를 이용해 호출 할 때, 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용하도록 해주어야 함.

   ```java
   /**
   * Returns true if this collection is empty.
   *
   * @implSpec
   * This implementation returns {@code this.size == 0}.
   *
   * @return true if this collection is empty
   */
   public boolean isEmpty() { ... }
   ```

4. 각 문서의 첫번째 문장은 요약설명(summary description)으로 간주된다.

   > 정규식상 {<마침표> <공백> <다음 문장 시작> } 이 만족될 경우 요약설명 문구가 생성되므로, 일반 문장 설명 상황에서 마침표가 등장할경우 {@literal 문장} 와 같이 표현해주어야 함
   >
   > java10부터는 {@summary 문장} 으로 요약설명을 대체할 수 있게 되었음

5. 메서드 요약설명은 **동사**
6. 클래스, 인터페이스, 필드 요약설명은 **명사**

