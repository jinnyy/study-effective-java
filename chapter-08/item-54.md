# item 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

## null의 문제점

컬렉션이나 배열같은 컨테이너가 비었을 때, null을 반환하는 **메서드를 사용**하면 항시 방어코드가 존재해야한다.



**Solution**

빈 컨테이너를 사용한다

1. List

   - ```java
     public List<Cheese> getCheeses() {
     	return new ArrayList<>(cheesesInStock);
     }
     ```

   - ```java
     public List<Cheese> getCheeses() {
       return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
     }
     ```

2. Array

   - ```java
     public Cheese[] getCheeses() {
       return chessesInStock.toArray(new Cheese[0]);
     }
     ```

     객체 할당에 성능이 떨어지는 것이 우려된다면?
     
     ```java
     private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
     
     public Cheese[] getChesses() {
       return chessesInStock.toArray(EMPTY_CHEESE_ARRAY);
     }
     ```
     
     > 길이 0인 배열은 모두 불변인 특성을 사용



## 성능에 좋지 않다?

빈 컨테이너를 할당하는 것이 비용이 드니 null이 성능에 좋다?



=> **아니다**

1. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

   > `Collections.emptyList()`, `Collections.emptyMap()`
   >
   > `private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0]`

2. 애초에 성능 차이는 신경 쓸 수준의 이슈되는 정도가 아님.





