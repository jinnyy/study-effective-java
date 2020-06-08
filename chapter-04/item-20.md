#### Effective Java 3/E
> 2020-06-08
>
> 아이템 #20 추상 클래스보다는 인터페이스를 우선하라

<br><br><br><br><br>


## 자바가 제공하는 다중 구현 메커니즘

<table>
<thead>
  <tr>
    <th class="tg-baqh"></th>
    <th class="tg-baqh">인터페이스</th>
    <th class="tg-baqh">추상 클래스</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-baqh">공통점</td>
    <td class="tg-0lax" colspan="2">둘 모두 인스턴스 메소드를 구현 형태로 제공 가능</td>
  </tr>
  <tr>
    <td class="tg-baqh" rowspan="4">차이점</td>
    <td class="tg-0lax">추상 클래스가 정의한 타입을 구현하는 클래스는 <br>반드시 추상 클래스의 하위 클래스가 되어야 함</td>
    <td class="tg-0lax">인터페이스가 선언한 메소드를 모두 정의하고 <br>그 일반 규약을 잘 지킨 클래스라면<br>다른 어떤 클래스를 상속했든 같은 타입으로 취급됨</td>
  </tr>
  <tr>
    <td class="tg-0lax">기존 클래스에도 손쉽게 새로운 인터페이스를<br>구현해넣을 수 있다.<br>- 인터페이스가 요구하는 메소드를 추가하고,<br>클래스 선언에 implements만 추가하면 끝</td>
    <td class="tg-0lax">기존 클래스 위에 새로운 추상 클래스를 끼워넣기 <br>어려움<br>- 두 클래스가 같은 추상 클래스를 확장하려면 <br>그 클래스는 공통 조상이어야 함</td>
  </tr>
  <tr>
    <td class="tg-0lax">믹스인 정의에 적합</td>
    <td class="tg-0lax">클래스는 두 개의 부모를 둘 수 없음</td>
  </tr>
  <tr>
    <td class="tg-0lax">
      계층 구조가 없는 타입 프레임워크 제작 가능<br>
      예) 가수, 작곡가, 싱어송라이터<br>
      interface 싱어송라이터 extends 가수, 작곡가
    </td>
    <td class="tg-0lax">
      클래스는 계층이 있음..<br>같은 기능을 구현하려면 고도비만 계층구조가 됨
    </td>
  </tr>
</tbody>
</table>
<br>

> `참고`
> #### mix-in 정의
> * 믹스인은 특정 기능(행위)만을 담당하는 클래스로, 단독 사용(Stand-alone use)이 아닌 다른 클래스에 탑재되어 사용될 목적으로 작성된 (조각) 클래스를 의미
>
> 출처: [위키피디아](https://en.wikipedia.org/wiki/Mixin)

<br><br>



## 골격 구현 (Skeletal Implementation)
인터페이스와 추상 골격 구현 클래스를 함께 제공함으로써 인터페이스와 추상 클래스의 장점을 모두 취할 수 있다.

* 템플릿 메소드 패턴
  * 인터페이스로는 타입을 정의하고, 필요하다면 디폴트 메소드 몇개도 함께 제공
  * 골격 구현 클래스는 나머지 메소드들까지 구현
* 관례: 인터페이스 이름이 Inferface 이면 골격 구현 클래스 이름은 AbstractInterface로 짓는다.


#### 예
``` java
// 코드 20-1 골격 구현을 사용해 완성한 구체 클래스 (133쪽)
public class IntArrays {
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);

        // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
        // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
        return new AbstractList<>() {
            @Override public Integer get(int i) {
                return a[i];  // 오토박싱(아이템 6)
            }

            @Override public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;     // 오토언박싱
                return oldVal;  // 오토박싱
            }

            @Override public int size() {
                return a.length;
            }
        };
    }

    public static void main(String[] args) {
        int[] a = new int[10];
        for (int i = 0; i < a.length; i++)
            a[i] = i;

        List<Integer> list = intArrayAsList(a);
        Collections.shuffle(list);
        System.out.println(list);
    }
}
```



``` java
// 코드 20-2 골격 구현 클래스 (134-135쪽)
public abstract class AbstractMapEntry<K,V>
        implements Map.Entry<K,V> {
    // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
    @Override public V setValue(V value) {
        throw new UnsupportedOperationException();
    }
    
    // Map.Entry.equals의 일반 규약을 구현한다.
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(),   getKey())
                && Objects.equals(e.getValue(), getValue());
    }

    // Map.Entry.hashCode의 일반 규약을 구현한다.
    @Override public int hashCode() {
        return Objects.hashCode(getKey())
                ^ Objects.hashCode(getValue());
    }

    @Override public String toString() {
        return getKey() + "=" + getValue();
    }
}
```
