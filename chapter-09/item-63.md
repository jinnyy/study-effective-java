#### Effective Java 3/E
> 2020-08-20
>
> 아이템 #63 문자열 연결은 느리니 주의하라

<br><br><br><br><br>





# 63. 문자열 연결은 느리니 주의하라

- 문자열 연결 연산자(+): 여러 문자열을 하나로 합쳐주는 방법. 많이 사용 시 성능 저하. (n^2, 문자열은 불변이므로 양쪽 모두 복사)
- StringBuilder 또는 StringBuffer를 사용하자


<br><br>

## StringBuilder 와 StringBuffer 의 차이
- StringBuffer는 멀티스레드 환경을 염두에 두고 설계 (method들마다 synchronized 걸려 있음)
- 멀티스레드 환경에서 thread-safe 하게 쓰고 싶다면 StringBuffer, 싱글스레드라면 StringBuilder

<br>

## 컴파일러의 문자열 연결 최적화
- jdk 1.5부터는 컴파일 단계에서 문자열 연결 연산자(+)를 자동으로 최적화

``` java
// String plusOpStr1 = "abc";
String plusOpStr1 = "a" + "b" + "c"; 

// String plusOpStr2 = (new StringBuilder("x")).append(plusOpStr1).append("y").toString();
String plusOpStr2 = "x" + plusOpStr1 + "y";


String plusOpStr3 = "";
for (int i = 0; i < 100; i++) {
    // plusOpStr3 = (new StringBuilder(String.valueOf(plusOpStr3))).append(f).toString();
    plusOpStr3 += "f"; 
}
```
