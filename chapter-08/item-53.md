# item 53. 가변인수는 신중히 사용하라

## 가변인수(varargs)란?

메소드의 마지막 인수로 받을 수 있는 여러개의 값을 받을 수 있는 특별한 형태의 argument

인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네주는 형태로 동작.





## 문제

1. 인수를 넣지않아도 호출되므로 **런타임 에러** 에 취약하다

   > 배열 순회하며 연산하는 경우 등..

   ```java
   static int sum(int... args) {
     int sum = 0;
     for (int arg : args)
       sum += arg;
     return sum;
   }
   ```

   **Solution**

   ```java
   static int min(int firstArg, int... remainArgs) {
   	// contents...
   }
   ```

2. 성능에 악영향을 끼친다
   > 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화 함
   
   **Solution**
   
   ```
   public void foo() { }
   public void foo(int a1) { }
   public void foo(int a1, int a2) { }
   public void foo(int a1, int a2, int a3) { }
   public void foo(int a1, int a2, int a3, int... rest) { }
   ```
   
   > EnumSet의 정적 팩토리도 이 기법을 사용해 열거타입 집합 생성 비용을 최소화한다
   
   ![image](https://user-images.githubusercontent.com/16698414/89122749-d9ddd200-d504-11ea-8861-e2fe3c55136d.png)



