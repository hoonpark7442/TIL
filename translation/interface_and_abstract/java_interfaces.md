https://www.baeldung.com/java-interfaces 번역

# 1. Overview

이 아티클에서는 자바의 인터페이스에 대해 설명한다. 또한 자바가 다형성과 다중 상속을 구현하기 위해 그것들을 어떻게 사용하는지도 볼 것이다.

# 2. What Are Interfaces in Java?

자바에서 인터페이스는 메서드와 상수 변수의 집합을 포함하는 abstract 타입이다. 자바에서 핵심 개념 중 하나이며 추상화 및 다형성, 다중 상속을 달성하는 데 사용된다.

간단한 예를 보자.

```java
public interface Electronic {

    // Constant variable
    String LED = "LED";

    // Abstract method
    int getElectricityUse();

    // Static method
    static boolean isEnergyEfficient(String electtronicType) {
        if (electtronicType.equals(LED)) {
            return true;
        }
        return false;
    }

    //Default method
    default void printDescription() {
        System.out.println("Electronic Description");
    }
}
```

implements 키워드를 사용하여 자바 클래스에 인터페이스를 구현할 수 있다. 
방금 만든 인터페이스를 구현하는 컴퓨터 클래스도 만들어보자.

```java
public class Computer implements Electronic {

    @Override
    public int getElectricityUse() {
        return 1000;
    }
}
```

### 2.1. Rules for Creating Interfaces

인터페이스에서는 다음을 사용할 수 있다.

- constants variables
- abstract methods
- static methods
- default methods

또한 다음과 같은 것들을 기억해야 한다.

- 인터페이스를 직접 인스턴스화 할 수 없다.
- 인터페이스는 메서드나 변수가 없는 빈 인터페이스일 수 있다.
- 컴파일러 오류가 발생하기 때문에 인터페이스 정의에서 final 키워드를 사용할 수 없다.
- 모든 인터페이스 선언에는 public 또는 디폴트 접근지정자가 있어야 한다. abstract 지정자는 컴파일러에 의해 자동으로 추가된다.
- 인터페이스 메서드는 protected나 final일 수 없다.
- 자바9까지는 인터페이스 메서드가 private일 수 없었지만 자바9는 인터페이스에 private 메서드를 정의할 수 있는 가능성을 도입했다.
- 인터페이스 변수는 정의상 public, static, 그리고 final 이므로 가시성을 변경할 수 없다.

# 3. What Can We Achieve by Using Them?

### 3.1. Behavioral Functionality

우리는 인터페이스를 사용하여 관련 없는 클래스에서 사용할 수 있는 특정 동작 기능을 추가한다. 예를 들어, Comparable, Comparator, 및 Cloneable은 관련없는 클래스로 구현할 수 있는 자바 인터페이스이다. 

(생략)

### 3.2. Multiple Inheritances

자바 클래스는 단일 상속을 지원한다. 그러나 인터페이스를 사용하여 다중 상속을 구현할 수 있다.

예를 들어 아래 예제에서는 Car 클래스가 Fly 및 Transform 인터페이스를 구현한다. 그렇게 함으로써 fly 와 transform이라는 메서드를 상속한다.

```java
public interface Transform {
    void transform();
}

public interface Fly {
    void fly();
}

public class Car implements Fly, Transform {

    @Override
    public void fly() {
        System.out.println("I can Fly!!");
    }

    @Override
    public void transform() {
        System.out.println("I can Transform!!");
    }
}
```

### 3.3. Polymorphism

먼저 다형성이란 무엇인가 부터 알아보자. 런타임 동안 객체가 다른 형태를 취할 수 있는 기능?능력? 이다. 좀 더 구체적으로 말하면, 런타임에 특정 객체 타입과 관련된 오버라이드된 메서드의 실행이다. 

자바에서 우리는 인터페이스를 사용하여 다형성을 달성할 수 있다. 예를 들어 Shape 인터페이스는 원형 또는 사각형과 같은 다양한 형태를 취할 수 있다.

Shape 인터페이스를 정의해보자.

```java
public interface Shape {
    String name();
}
```

Circle 클래스도 만들어보자

```java
public class Circle implements Shape {

    @Override
    public String name() {
        return "Circle";
    }
}
```

Square 클래스를 만들자.

```java
public class Square implements Shape {

    @Override
    public String name() {
        return "Square";
    }
}
```

마지막으로 Shape 인터페이스와 그 구현을 사용하여 다형성이 작동하는 것을 봐보자. Shape 객체를 인스턴스화 하고 리스트에 추가한 다음 마지막으로 루프에서 이름을 인쇄하자.

```java
List<Shape> shapes = new ArrayList<>();
Shape circleShape = new Circle();
Shape squareShape = new Square();

shapes.add(circleShape);
shapes.add(squareShape);

for (Shape shape : shapes) {
    System.out.println(shape.name());
}
```

# 4. Default Methods in Interfaces

(생략)

# 5. Interface Inheritance Rules

인터페이스를 통해 다중 상속을 달성하려면 몇 가지 규칙을 기억해야 한다.

### 5.1. Interface Extending Another Interface

인터페이스가 다른 인터페이스를 extends할 때 해당 인터페이스의 추상 메서드를 모두 상속한다. 두 가지 인터페이스인 HasColor와 Shape을 만들어보자.

```java
public interface HasColor {
    String getColor();
}

public interface Box extends HasColor {
    int getHeight()
}
```

위의 예에서 Box는 extends 키워드를 사용하여 HasColor를 상속받는다. 이렇게 하면 Box 인터페이스가 getColor() 메서드를 상속한다. 따라서 Box 인터페이스에는 getColor()와 getHeight()라는 두 가지 메서드가 있게 된다.

### 5.2. Abstract Class Implementing an Interface

추상 클래스는 인터페이스를 구현할 때 모든 추상 메서드와 기본 메서드를 상속한다. 아래 예를 보자.

```java
public interface Transform {
    
    void transform();
    default void printSpecs(){
        System.out.println("Transform Specification");
    }
}

public abstract class Vehicle implements Transform {}
```

이 예에서 Vehicle 클래스는 두 가지 메서드를 상속한다. abstract transform 메서드와 디폴트 printSpecs 메서드이다.

# 6. Functional Interfaces

(생략)

# 7. Conclusion

이 아티클에서는 자바 인터페이스에 대한 개요를 알아봤고 다형성과 다중 상속을 달성하기 위해 자바 인터페이스를 사용하는 방법에 대해 얘기했다.