

> #### Effective Java 3/E
> 2020-05-25
>
> 아이템 #02 생성자에 매개변수가 많다면 빌더를 고려하라

<br><br><br><br><br>





# 02. 생성자에 매개변수가 많다면 빌더를 고려하라

생성자와 정적 팩터리 메서드의 공통적인 단점은 **선택적 매개변수**가 많을 때 적절히 대응하기 어렵다는 것이다

## 점층적 생성자 패턴

* N번째 인자에 넘기는 것이 무엇인지 해당 생성자의 시그니처를 반드시 봐야만 한다 (물론 IntelliJ가 똑똑하다)
* 매개변수가 많아질 수록 많은 생성자를 작성해야하고, 복잡해진다

#### 예시
2개의 필수 인자와 4개의 선택인자가 있는 NutritionFacts(영양정보)라는 class가 있다고 생각해보자

``` java
public class NutritionFacts {
   private final int servingSize;
   private final int servings;
   private final int calories;
   private final int fat;
   private final int sodium;
   private final int carbohydrate;

   public NutritionFacts(int servingSize, int servings,
                         int calories, int fat, int sodium, int carbohydrate) {
      this.servingSize = servingSize;
      this.servings = servings;
      this.calories = calories;
      this.fat = fat;
      this.sodium = sodium;
      this.carbohydrate = carbohydrate;
   }

   public NutritionFacts(int servingSize, int servings,
                         int calories, int fat, int sodium) {
      this(servingSize, servings, calories, fat, sodium, 0);
   }

   public NutritionFacts(int servingSize, int servings,
                         int calories, int fat) {
      this(servingSize, servings, calories, fat, 0);
   }
}

NutritionFacts salmon = new NutritionFacts(123, 2, 34, 4, 5555, "salmon");
NutritionFacts rice = new NutritionFacts(null, 2, null, 4, 1234, "rice");
```

## 자바 빈즈 패턴

* 매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식

``` java
public class NutritionFacts {
   private final int servingSize;
   private final int servings;
   private int calories = 0;
   private int fat = 0;
   private int sodium = 0;
   private int carbohydrate = 0;

   public NutritionFacts(int servingSize, int servings) {
      this.servingSize = servingSize;
      this.servings = servings;
   }

   public void setCalories(int calories) { this.calories = calories; }
   public void setFat(int fat) { this.fat = fat; }
   public void setSodium(int sodium) { this.sodium = sodium; }
   public void setCarbohydrate(int carbohydrate) { this.carbohydrate = carbohydrate; }
}
```

* 객체의 불변성을 보장하지 못한다
* 객체의 생성이 한 번에 끝나지 않고, 원하는 값을 넣어주기 위해서는 set 메서드를 연속해서 호출해야 한다
* 이후 set 메서드를 호출하여 값을 변조한다면 객체의 일관성을 훼손한다 (cf. freeze 등의 방법을 사용할 수 있긴 함)
* 위의 경우 쓰레드 안정성까지 보장해줘야 한다

#### 불변 vs. 불변식
* 불변: 어떤 객체에 대해서 어떠한 변경도 허용하지 않겠다 (e.g. String 객체)
* 불변식: 반드시 만족해야 하는 조건 (불변은 아니다) (e.g. 리스트 객체의 size 값이 0보다 커야 한다)

## 빌더 패턴

필요한 객체를 직접 만드는 대신 Builder 객체를 얻은 후 setter 들을 호출한 뒤 build()를 통해 필요한 객체를 얻는 것

* 점층적 생성자와 자바빈즈 패턴의 장점만 모아둔 가장 좋은 방법
* 필수 인자라면 new NutritionFacts.Builder("필수", "인자") 처럼 쓸 수도 있다
* build()가 호출되는 시점에서 검증, 불변화 등 수행
* 단점은 Builder부터 짜야한다는 것

``` java
public class NutritionFacts {
   private final int servingSize;
   private final int servings;
   private final int calories;
   private final int fat;
   private final int sodium;
   private final int carbohydrate;

   public static class NutriFactsBuilder {
      // 필수 매개변수
      private final int servingSize;
      private final int servings;

      // 선택 매개변수 - 기본값으로 초기화
      private int calories = 0;
      private int fat = 0;
      private int sodium = 0;
      private int carbohydrate = 0;

      public NutriFactsBuilder(int servingSize, int servings) {
         this.servingSize = servingSize;
         this.servings = servings;
      }

      public NutriFactsBuilder calories(int val) { calories = val; return this; }
      public NutriFactsBuilder fat(int val) { fat = val; return this; }
      public NutriFactsBuilder sodium(int val) { sodium = val; return this; }
      public NutriFactsBuilder carbohydrate(int val) { carbohydrate = val; return this; }
      public NutritionFacts build() { return new NutritionFacts(this);  }
   }

   private NutritionFacts(NutriFactsBuilder builder) {
      servingSize = builder.servingSize;
      servings = builder.servings;
      calories = builder.calories;
      fat = builder.fat;
      sodium = builder.sodium;
      carbohydrate = builder.carbohydrate;
   }
}

NutritionFacts salmon = new NutritionFacts.Builder()
                                .calories(123)
                                .sodium(2)
                                .carbohydrate(5555)
                                .build();
```

#### lombok의 @Builder 어노테이션

* builder를 짜는데 필요한 코드량이 많다
* class 위에 어노테이션만 달면 바로 클래스명.Builder().build(); 를 사용 가능하다
* 단점: 필수 인자를 쓸 수 없으며, build() 호출 시점에 검증을 통해 예외를 던지는 커스터마이징이 불가능하다

#### 계층 클래스

* 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다
* 각 계층 클래스에 관련 빌더를 멤버로 정의하고, 추상 클래스에는 추상 빌더를, 구현 클래스에는 구현 빌더를 갖게 한다

``` java
public abstract class Pizza{
   public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
   final Set<Topping> toppings;

   abstract static class Builder<T extends Builder<T>> {
      EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
      public T addTopping(Topping topping) {
         toppings.add(Objects.requireNonNull(topping));
         return self();
      }

      abstract Pizza build();

      protected abstract T self();
   }

   Pizza(Builder<?> builder) {
      toppings = builder.toppings.clone();
   }
}
```

* NyPizza는 size라는 값을 필수로 받아야 하며, Calzone는 sauseInside를 통해 소스를 넣을지 말지를 결정한다
* self라는 추상메서드를 더해 하위 클래스에서 형변환하지 않고도 메서드 체이닝이 가능하도록 한다 (`simulated self-type`)


> 결론: 확장 가능성이 높은 객체에 대해서는 처음부터 빌더 패턴을 고려하자
