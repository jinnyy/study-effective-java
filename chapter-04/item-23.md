#### Effective Java 3/E
> 2020-06-08
>
> 아이템 #23 태그 달린 클래스보다는 클래스 계층구조를 활용하라

<br><br><br><br><br>





# 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

태그 달린 클래스는 2가지 이상의 값을 표현하는 클래스이다.


``` java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```
위의 코드들은 불필요한 코드들이 너무 많고, 장황하며, 오류를 내기 쉽고 비효율적이다.

``` java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override
    double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    
    @Override
    double area() { return length * width; }
}
```

위와 같이 클래스 계층 구조를 사용할 경우, **컴파일 타임의 검사**를 최대한 활용할 수 있다. (추상메소드 구현 여부 등)
또한 **다른 도형을 추가할 때도 확장성** 있는 형태이다.
