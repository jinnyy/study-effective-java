
> #### Effective Java 3/E
> 2020-07-06
>
> 아이템 #45 스트림은 주의해서 사용하라

<br><br><br><br><br>





# 45. 스트림은 주의해서 사용하라

## 스트림

* 다량의 데이터 처리 작업을 돕고자 자바 8에서 추가
* 스트림 (Stream): 데이터의 유한/무한 시퀀스
* 스트림 파이프라인 (Pipeline): 데이터 원소들로 수행하는 연산단계
* 스트림의 데이터 원소들은 어디에서든지 가져올 수 있다 (객체 참조나 기본 타입 지원)
* 기본 타입:  IntStream, LongStream, DoubleStream과 같은 Stream을 사용하는 것이 성능이 좋다

#### 파이프라인: 소스 스트림 -> 중간 연산 -> 종단 연산
종단 연산
* forEach(Consumer<? super T> consumer) : Stream의 요소를 순회
* count() : 스트림 내의 요소 수 반환
* max(Comparator<? super T> comparator) : 스트림 내의 최대 값 반환
* min(Comparator<? super T> comparator) : 스트림 내의 최소 값 반환
* allMatch(Predicate<? super T> predicate) : 스트림 내에 모든 요소가 predicate 함수에 만족할 경우 true
* anyMatch(Predicate<? super T> predicate) : 스트림 내에 하나의 요소라도 predicate 함수에 만족할 경우 true
* noneMatch(Predicate<? super T> predicate) : 스트림 내에 모든 요소가 predicate 함수에 만족하지않는 경우 true
* sum() : 스트림 내의 요소의 합 (IntStream, LongStream, DoubleStream)
* average() : 스트림 내의 요소의 평균 (IntStream, LongStream, DoubleStream)

<br><br>

중간 연산
* filter(Predicate<? super T> predicate) : predicate 함수에 맞는 요소만 사용하도록 필터
* map(Function<? Super T, ? extends R> function) : 요소 각각의 function 적용
* flatMap(Function<? Super T, ? extends R> function) : 스트림의 스트림을 하나의 스트림으로 변환
* distinct() : 중복 요소 제거
* sort() : 기본 정렬
* sort(Comparator<? super T> comparator) : comparator 함수를 이용하여 정렬
* skip(long n) : n개 만큼의 스트림 요소 건너뜀
* limit(long maxSize) : maxSize 갯수만큼만 출력

<br><br>

* 스트림 파이프라인은 지연 평가(lazy evaluation): 종단 연산이 호출될 때 평가 (종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않음)
* 스트림을 잘못 사용하면 읽기 힘들고, 유지보수가 어렵다


``` java 
// 일반 loop
public static void main(String[] args) {
    File dectionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    Map<String, Set<String>> groups = new HashMap<>();
    try (Scanner s = new Scanner(dectionary)) {
        while(s.hasNext()) {
            String word = s.next();
            groups.computeIfAbsent(alphabetize(word), 
                                   (unused) -> new TreeSet<>()).add(word);
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
    
    for(Set<String> group : groups.values()) {
        if(group.size() >= minGroupSize) {
            System.out.println(group.size() + ": " + group);
        }
    }
}

private static String alphabetize(String s) {
    char[] a = s.toCharArray();
    Arrays.sort(a);
    return new String(a);
}
```
<br>

``` java
// 과도한 Stream
public static void main(String[] args) throws IOException {
    File dectionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    try(Stream<String> words = Files.lines(dectionary.toPath())) {
        words.collect(
            groupingBy(word -> word.chars().sorted()
                       .collect(StringBuilder::new,
                                (sb, c) -> sb.append((char) c),
                                StringBuilder::append).toString()))
            .values().stream()
            .filter(group -> group.size() >= minGroupSize)
            .map(group -> group.size() + ": " + group)
            .forEach(System.out::println);
    }
}
```

<br>

``` java
// 적절한 Stream
public static void main(String[] args) throws IOException {
    File dectionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    try(Stream<String> words = Files.lines(dectionary.toPath())) {
        words.collect(groupingBy(word -> alphabetize(word)))
            .values().stream()
            .filter(group -> group.size() >= minGroupSize)
            .forEach(group -> System.out.println(group.size() + ": " + group));
    }
}
```
* 람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인 가독성이 유지된다
* 도우미 메서드를 적절히 활용 (e.g. alphabetize)
* 기존 코드는 스트림을 사용하도록 리팩터링 (더 나아질 때만)

<br><br>

## 코드 블럭 vs. 람다
* 코드블럭에서는 범위 안의 지역변수를 읽고 수정할 수 있다
* 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있다 (지역 변수를 수정하는 것은 불가능)
* 코드 블럭에서는 return을 이용해 메서드를 빠져나갈 수 있다
* 코드 블럭에서는 break, continue문을 통해 블럭 바깥의 반복문을 종료 할 수 있다
* 메서드 선언에 명시된 예외(Exception)을 던질 수 있다
* 하지만 람다로는 불가능한 것들

``` java
// stream + lambda 에서 순차증가하는 방법

// 배열 이용. 멀티스레드에선 안전X
int[] count = {0};
IntStream.range(0, 1000000).forEach((i) -> {
    count[0]++;
    System.out.println(count[0]);
});
 
// Atomic Reference 이용. 권장
AtomicInteger count = new AtomicInteger();
IntStream.range(0, 1000000).forEach((i) -> {
    System.out.println(count.incrementAndGet());
});
```

<br>

## 이럴 때 스트림을 사용하자

* 원소들의 시퀀스를 일관되게 반환한다
* 원소들의 시퀀스를 필터링한다
* 원소들의 시퀀스를 하나의 연산을 사용해 결합한다
* 원소들의 시퀀스를 컬렉션에 모은다
* 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다

<br>

## 이럴 때 스트림을 사용하지 말자

* 스트림 파이프라인이 여러 연산단계로 구성될 때, 각 단계의 값들을 동시에 접근하고자 할때
  * 스트림 파이프라인은 연산이 지나가면 원래값을 잃는 구조이기 때문이다.
  * cf. 매핑객체를 이용하면 방법이 있으나 지저분해짐

<br><br>

``` java
public static void main(String[] args) {
    primes().map(p -> TWO.pow(p.intValueExact().subtract(ONE)))
            .filter(mersenne -> mersenne.isProbablePrime(50))
            .limit(20)
            .forEach(System.out::println);
}
```

``` java
// loop
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for(Suit suit : Suit.values()) 
        for(Rank rank : Rank.values())
            result.add(new Card(suit, rank));
    return result;
}

// stream
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
    .flatMap(suit -> Stream.of(Rank.values())
                      .map(rank -> new Card(suit, rank)))
    .collect(toList());
}
```

> * 스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택
> * 스트림과 반복을 잘 조합해서 사용하는 것이 제일 좋다
> * 함수형 프로그래밍에 익숙하다면 스트림을 시도해보자
