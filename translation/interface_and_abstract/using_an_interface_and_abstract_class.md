https://www.baeldung.com/java-interface-vs-abstract-class 번역

# 1. Introduction

추상화는 객체 지향 프로그래밍의 핵심 기능 중 하나이다. 단순한 인터페이스를 통해 기능을 제공하는 것 만으로도 구현 복잡성을 숨길 수 있다. 자바에서 우리는 인터페이스나 추상 클래스를 사용하여 추상화를 달성한다.

이 아티클에서는 애플리케이션을 설계하는 동안 인터페이스를 사용해야 하는 때와 추상클래스를 사용해야 하는 때에 대해 설명한다. 또한 그들 사이의 주요차이점과, 우리가 무엇을 달성하고자 하는지를 기반으로 둘 중 어떤 것을 선택해야 하는지도 살펴본다.

# 2. Class vs. Interface

먼저 일반 실체 클래스와 인터페이스의 차이점을 알아보자. 클래스는 객체 생성의 블루프린트 역할을 하는 사용자 정의 유형이다. 그것은 각각 객체의 상태와 행동을 나타내는 속성과 메서드를 가질 수 있다.

인터페이스 또한 구문적으로 클래스와 유사한 사용자 정의 유형이다. 인터페이스를 구현하는 클래스에 의해 오버라이드 될 필드 상수 및 메서드 시그니쳐의 컬렉션을 가질 수 있다.

이에 더해 자바 8의 새로운 기능들은 역호환성을 지원하기 위해 인터페이스에서 정적 및 디폴트 메서드를 지원한다. 인터페이스의 메서드는 그들이 정적이나 디폴트가 아니라면 암묵적으로 추상 메서드이며 모두 public이다.

그러나 자바 9부터는 인터페이스에 private 메서드를 추가할 수도 있다.

# 3. Interface vs. Abstract Class

추상 클래스는 abstract 키워드를 사용하여 선언된 클래스에 지나지 않는다. 또한 abstract 키워드를 사용하여 메서드 시그니쳐를 선언할 수 있으며 서브클래스가 선언된 모든 메서드를 구현하도록 강제한다. 클래스에 추상 메서드가 있다면 그 클래스 자체도 추상 클래스여야 한다.

추상 클래스는 필드 및 메서드 제어자에 대한 제한이 없는 반면 인터페이스에서는 기본적으로 public이다. 우리는 추상 클래스에 인스턴스 및 정적 초기화 블록을 가질 수 있으나 인터페이스에는 그것들을 가질 수 없다. 추상 클래스는 자식 객체의 인스턴스화 동안 실행될 생성자를 가질 수도 있다.

자바 8은 하나 이상의 선언된 추상 메서드를 제한하지 않는 인터페이스인 functional 인터페이스를 도입했다. 정적 메서드와 디폴트 메서드가 아닌 단일 추상 메서드를 가진 인터페이스는 functional 인터페이스로 간주된다. 이 기능을 사용하여 선언할 추상 메서드의 수를 제한할 수 있다. 추상 클래스에서 우리는 추상 메서드 선언의 수에 대한 이러한 제한을 가질 수 없다.

추상 클래스는 어떤 면에서는 인터페이스와 유사하다.
- 둘 다 인스턴스화 할 수 없다. 즉 new TypeName() 문을 사용하여 객체를 인스턴스화할 수 없다. 앞서 말한 문구를 사용했다면 익명 클래스를 사용하여 모든 메서드를 오버라이드해야한다. 
- 둘 다 인터페이스의 정적 및 디폴트 메서드, 추상클래스의 인스턴스 메서드, 추상클래스의 추상 메서드와 같은 일련의 메서드를 포함할 수 있다.

# 4. When to Use an Interface

인터페이스를 사용해야 하는 몇 가지 시나리오를 살펴보자.

- 다중 상속을 사용하여 문제를 해결해야 하며 서로 다른 클래스 계층으로 구성된 경우
- 관련 없는 클래스가 우리의 인터페이스를 구현할 때. 예를 들어 Comparable은 두 객체를 비교하기 위해 오버라이드 될 수 있는 compareTo() 메서드를 제공한다. 
- 애플리케이션 기능을 계약으로 정의해야 하지만 누가 이 동작을 구현하는지 상관하지 않는 경우, 즉 써드파티 벤더가 이를 완전히 구현해야 한다.

“A is capable of [doing this]” 라고 하려 할 때 인터페이스를 사용하는 것을 고려해봐라. 예를 들어 “Clonable is capable of cloning an object”, “Drawable is capable of drawing a shape” 등 이다.

인터페이스를 사용하는 예를 살펴보자.

```java
public interface Sender {
    void send(File fileToBeSent);
}
```

```java
public class ImageSender implements Sender {
    @Override
    public void send(File fileToBeSent) {
        // image sending implementation code.
    }
}
```

여기서 Sender는 메서드 send()가 있는 인터페이스이다. 따라서 "Sender is capable to send file"을 인터페이스로 구현했다. ImageSender는 이미지를 타겟으로 전송하기 위한 인터페이스를 구현한다. 또한 위의 인터페이스를 사용하여 VideoSender, DocumentSender를 구현하여 다양한 작업을 수행할 수 있다.

위의 인터페이스와 구현된 클래스를 사용하는 유닛테스트 케이스를 고려해보자.

```java
@Test
void givenImageUploaded_whenButtonClicked_thenSendImage() { 
 
    File imageFile = new File(IMAGE_FILE_PATH);
 
    Sender sender = new ImageSender();
    sender.send(imageFile);
}
```

# 5. When to Use an Abstract Class

이제 추상 클래스를 사용해야 하는 몇 가지 시나리오를 살펴보자.

- 서브클래스가 오버라이드하는 공통적인 기본 클래스 메서드를 제공하여 코드(많은 관련 클래스들 간에 코드 공유)에서 상속개념을 사용하려고 할 때
- 요구사항을 지정하고 일부 세부 구현 정보만 지정한 경우
- 추상클래스를 확장하는 클래스에 여러 공통 필드 또는 메서드가 있을 때(non-public 지정자 필요)
- 객체 상태를 수정하기 위해 non-final or non-static 메서드를 원할 경우

"A is a B"일 때 추상클래스와 상속을 사용하는 것을 고려해라. 

예를 살펴보자.

```java
public abstract class Vehicle {
    
    protected abstract void start();
    protected abstract void stop();
    protected abstract void drive();
    protected abstract void changeGear();
    protected abstract void reverse();
    
    // standard getters and setters
}
```

```java
public class Car extends Vehicle {

    @Override
    protected void start() {
        // code implementation details on starting a car.
    }

    @Override
    protected void stop() {
        // code implementation details on stopping a car.
    }

    @Override
    protected void drive() {
        // code implementation details on start driving a car.
    }

    @Override
    protected void changeGear() {
        // code implementation details on changing the car gear.
    }

    @Override
    protected void reverse() {
        // code implementation details on reverse driving a car.
    }
}
```

위의 코드에서 Vehicle 클래스는 다른 추상 메서드와 함께 추상 클래스로 정의되었다. 실제 차량의 일반 작동을 제공하며 몇 가지 공통 기능도 갖추고 있다. Vehicle 클래스를 확장하는 Car 클래스는 차량의 구현 세부 정보를 제공하며 모든 메서드를 오버라이드한다(“Car is a Vehicle”)

따라서 우리는 Vehicle 클래스를 자동차와 버스와 같은 실제 개별 차량에 의해 기능을 구현할 수 있는 추상 클래스로 정의했다. 

이제 위의 코드를 사용하는 간단한 유닛테스트를 살펴보자.

```java
@Test
void givenVehicle_whenNeedToDrive_thenStart() {

    Vehicle car = new Car("BMW");

    car.start();
    car.drive();
    car.changeGear();
    car.stop();
}
```

# 6. Conclusion

이 아티클을 통해 인터페이스와 추상 클래스에 대한 개요와 그들 사이의 주요 차이점에 대해 논의했다.