# item 51. 메서드 시그니처를 신중히 설계하라

a.k.a. API 설계 요령 모음집

<br>

### 1. 메서드 이름을 신중히 짓자

- 표준 명명 규칙 (아이템 68) 을 따라야 한다
- 같은 패키지에 속한 다른 이름들과 일관되게 짓는다
- 개발자 커뮤니티에서 널리 받아들여지는 이름을 사용한다
- 애매하면 자바 라이브러리의 API 가이드를 참조하라

<br>

### 2. 편의 메서드를 너무 많이 만들지 말자

- 메서드가 너무 많은 클래스는 개발자와 사용자를 모두 고통스럽게 만든다

<br>

### 3. 매개변수 목록은 짧게 유지하자 (4개 이하가 좋다)

- **같은 타입의 매개변수 여러 개가 연달아 나오는 경우가 특히 해롭다.** 사용자가 순서를 바꿔 입력하더라도 정상적으로 컴파일 되고 실행된다. 단지 의도와 다르게 동작할 뿐...

- 해결책 1) 여러 메서드로 쪼갠다 - **코드의 직교성**을 높여준다
   - as-is: `indexOfSubList(int fromIndex, int toIndex, object O)`
   - to-be: `subList(int fromIndex, int toIndex)` + `indexOf(Object o)`
- 해결책 2) 매개변수 여러 개를 묶어주는 도우미 클래스를 만든다
   - as-is: `pickupCard(RankEnum rank, SuitEnum suit)`
   - to-be: `pickupCard(Card card)`
- 해결책 3) [빌더 패턴](../chapter-02/item-02.md)을 메서드 호출에 응용한다

<br>

### 4. 매개변수 타입으로는 클래스보다는 인터페이스가 더 낫다 (아이템 64)

- 매개변수로 클래스를 사용하면 **클라이언트에게 특정 구현체만 사용하도록 제한**하는 꼴이며, 혹시라도 입력 데이터가 다른 형태로 존재한다면 명시한 특정 구현체의 객체로 옮겨 담느라 비싼 복사 비용을 치러야 한다.

<br>

### 5. boolean 보다는 원소 2개짜리 열거 타입이 낫다

- 단, 메서드 이름상 boolean을 받아야 의미가 더 명확할 때는 예외다
- 열거 타입을 사용하면 코드의 가독성이 높아지며 나중에 선택지를 추가하기도 쉽다
   - ex. `public enum TemperatureScale { FAHRENHEIT, CELSIUS }`