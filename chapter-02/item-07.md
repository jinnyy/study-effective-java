

> #### Effective Java 3/E
> 2020-05-25
> 아이템 #07 다 쓴 객체 참조를 해제하라

<br><br><br><br><br>



# 07. 다 쓴 객체 참조를 해제하라

메모리를 직접 관리해야 하는 C/C++과 달리, 자바는 가비지 컬렉터를 갖춘 언어이다.
그래서 프로그래머의 삶이 편안해지지만, 메모리 관리에 더 이상 신경 쓰지 않아도 되는 것은 아니다.

<details>
	<summary>요약</summary>
  
## 메모리 누수의 주범
1. 자기 메모리를 직접 관리하는 클래스
2. 캐시
3. 리스터(listner) 혹은 콜백(callback)이라고 부르는 것

</details>
<br><br>




#### 메모리 누수 찾기
``` java
public class Stack {
	private Obkect[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;
	
	public Stack() {
		elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}

	public void push(Object e) {
		ensureCapacity();
		elements[size++] = e;
	}
	
	public Object pop() {
		if(size == 0)
			throw new EmptyStackException();
		return elements[--size];
	}

}
```


<details>
  <summary>정답</summary>
<hr>

#### 정답
* 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다.
* 프로그램에서 그 객체들을 더이상 사용하지 않더라도!
* 스택이 그 객체들의 **`다 쓴 참조(obsolete reference)`**를 여전히 가지고 있기 때문

#### 해결방법
* 해당 참조를 다 썼을 때 null 처리 (참조 해제)
* 위 stack 예시의 경우 각 원소의 참조가 더 이상 필요없어지는 시점은 스택에서 꺼내질 때
* 제대로 구현한 pop 메소드:

``` java
public Object pop() {
	if(size == 0)
		throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] = null;		// null 처리
	return result;
}
```


> 그러나 모든 객체를 다 쓰자마자 일일이 null 처리하는 데 혈안이 될 필요는 없다.
> * 그럴 필요도 없고, 바람직하지도 않다.
> * 프로그램을 필요 이상으로 지저분하게 만들 뿐이다.

<hr>
</details>


<br><br><br>



## 1. 자기 메모리를 직접 관리하는 클래스

### 객체 참조를 해제하는 방법
* 참조를 담은 변수를 유효 범위 (scope) 밖으로 밀어내는 것 (가장 좋은 방법)
	* 변수의 범위를 최소가 되게 정의했다면(아이템 57) 이 일은 자연스럽게 이루어진다.
* 객체 참조를 null 처리하는 일은 예외적이어야 한다.
	* 그럼 언제?
		* 자기 메모리를 직접 관리할 때
		* 가비지 콜렉터가 모를 때!

<br><br>


## 2. 캐시
캐시도 메모리 누수 주범이다.

#### 문제가 되는 경우: 
* 객체 참조를 캐시에 넣고 나서 잊은 채 객체를 다 쓴 뒤에도 그냥 놔둘 때

#### 해법 (여러 가지)
* WeakHashMap
	* (운 좋게) 외부에서 키를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 상황이라면 WeakHashMap을 사용해 캐시를 만들자
* 시간이 지날 수록 엔트리의 가치를 떨어뜨리는 방식을 흔히 사용
	* 보통 캐시 엔트리의 유효 기간을 정확히 정의하기 어렵기 때문에

#### 추가
* ehCache의 경우 이런 옵션을 줄 수 있음
  * timeToIdleSeconds="300" 
  * timeToLiveSeconds="600"

<br><br>


## 3. 리스너(listner) (콜백(callback))

#### 문제
클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면, 
뭔가 조치해주지 않는 한 콜백은 계속 쌓여갈 것이다.


#### 해결
* 콜백을 약한 참조 (weak reference)로 저장하면 가비지 컬렉터가 즉시 수거해간다.
* 예: WeakHashMap에 키로 저장




