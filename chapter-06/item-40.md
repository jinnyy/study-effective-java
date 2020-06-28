# @Override 애너테이션을 일관되게 사용하라


<br><br><br>



## 버그 찾기

``` java
// 코드 40-1 영어 알파벳 2개로 구성된 문자열(바이그램)을 표현하는 클래스 - 버그를 찾아보자. (246쪽)
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```

<br>


## 정답
* main 메소드를 보면, 소문자 2개로 이루어진 바이그램 26개를 10번 반복해 집합에 추가한 다음, 집합의 크기를 출력함
* Set은 중복을 허용하지 않으므로 26이 출력될 것 같지만, 실제로는 260이 출력된다.


<br>


## 원인
* equals()를 `override`하지 않고 `overload` 했습니다;
* Object의 equals()를 재정의하려면 매개변수 타입을 Object로 해야 하는데, Bigram으로 했다.










