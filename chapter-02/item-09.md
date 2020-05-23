
> #### Effective Java 3/E
> 2020-05-25
> 아이템 #09 try-finally보다는 try-with-resources를 사용하라

<br><br><br><br><br>


# 09. try-finally보다는 try-with-resources를 사용하라

* close() 메소드를 이용해 닫아줘야 하는 자원이 많다.
* 예: InputStream, OutputStream, java.sql.Connection 등

<br>


## 닫아줘야 하는 자원이 여러개일 때

#### 예 1

``` java
static String firstLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
  try {
  	return br.readLine();
  } finally {
  	br.close();
  }
}
```
- 꽤 나쁘지 않은 느낌...?
	
<br>


#### 예 2

``` java
static String copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
  try {
  	OututStream out = new FileOutputStream(dst);
    try {
    	byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0)
      	out.write(buf, 0, n);
		} finally {
    	out.close();
    }
  } finally {
  	in.close();
  }
}
```

- 코드가 너무 더러워진다
- close가 되지 않을 가능성이 있다
	- 예외는 try와 finally 블록 모두에서 발생 가능
	- 예: 기기에 물리적인 문제가 생긴다면?
  - *readLine* 메소드가 예외를 던지고, 같은 이유로 close 메소드도 실패


<br><br>



## 해결 방법

`try-with-resources` 사용

* 자바 7 부터 사용 가능
* (이 구조를 사용하려면) AutoCloseable 인터페이스를 구현해야 함
	* 단순히 void를 반환하는 close 메소드 하나만 정의한 인터페이스 
* 짧고 읽기 수월할 뿐 아니라 문제 진단하기도 더 쉽다

#### 예 3 - `예 1`을 재작성

``` java
static String firstLineOfFile(String path) throws IOException {
  try (BufferedReader br = new BufferedReader(new FileReader(path))) {
  	return br.readLine();
  }
}
```

#### 예 4 - `예 2`를 재작성

``` java
static String copy(String src, String dst) throws IOException {
  try (InputStream in = new FileInputStream(src);
  	OututStream out = new FileOutputStream(dst)) {
    	byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0)
      	out.write(buf, 0, n);
	}
}
```


#### 예 5 - catch 절 사용

``` java
static String firstLineOfFile(String path, String defaultVal) throws IOException {
  try (BufferedReader br = new BufferedReader(new FileReader(path))) {
  	return br.readLine();
  } catch (IOException e) {
    return defaultVal;
  }
}
```




