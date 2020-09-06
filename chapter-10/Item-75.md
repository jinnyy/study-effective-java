## Item 75. 예외의 상세 메시지에 실패 관련 정보를 담으라

### 에러 메세지

예외가 발생했을 때 해당 상황에 대해서 모든것을 파악할 수 있는 메세지를 남겨야 한다

즉, **예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담아야 한다.**



개발자가 보는 것이므로 친절한것보다는, **사후 처리를 수월하게 할 수 있는 포맷과 가독성이 우선**시 되어야 한다.





### 예외 생성자

필요한 정보들을 예외 생성자에 모두 받아서 상세 메시지까지 미리 생성해놓는 방법도 고민 해 볼 수 있다. 

가령 리스트의 경우, 리스트에 존재하는 필드값들을 사용해서 다음과 같이 작성 가능하다.

```java
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
	super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));
  
  this.lowerBound = lowerBound;
  this.upperBound = upperBound;
  this.index = index;
}
```



