https://www.baeldung.com/java-static 번역

# 1. Overview

이 튜토리얼에서는 자바의 static 키워드를 살펴볼 예정이다. 

변수, 메서드, 블로 및 네스티드 클래스에 static 키워드를 적용할 수 있는 방법과 그 차이가 무엇인지 알아보자.

# 2. The Anatomy of the static Keyword (static 키워드의 구조)

자바에서 static 키워드는 특정 멤버가 해당 유형의 객체가 아닌 해당 유형 자체에 속한다는 것을 의미한다.

즉 클래스의 모든 객체에서 공유되는 static 멤버의 인스턴스를 오직 하나만 만든다.

![static_example](/assets/static_example_1.jpeg)
[이미지출처](https://www.baeldung.com/java-static)<br/>

# 3. The static Fields (or Class Variables)

자바에서 필드를 static으로 선언하면 해당 필드의 정확히 딱 하나의 복사본이 생성되어 해당 클래스의 모든 객체 간에 공유된다. 

우리가 클래스를 얼마나 많이 초기화 했는지는 중요치않다. 클래스에 속하는 정확히 단 하나의 정적 필드 카피본만이 존재한다. 이 정적 필드의 값은 동일하거나 다른 클래스(..?)의 모든 객체에서 공유된다.

메모리 관점에서 정적 변수는 힙 메모리에 저장된다.

### 3.1. Example of the static Field

여러 속성(인스턴스 변수)이 있는 Car 클래스가 있다고 해보자.

Car 클래스로부터 새 객체를 초기화 할 때마다 각 새 객체에는 이러한 인스턴스 변수의 고유한 복사본(copy)이 있다.

하지만 초기화된 Car 객체 수 카운트가 관리되며 모든 객체에 걸쳐 공유되고 객체 초기화가 일어날 때 증가한다고 가정해보자. 

이 때 필요한 것이 static 변수이다.

```java
public class Car {
    private String name;
    private String engine;
    
    public static int numberOfCars;
    
    public Car(String name, String engine) {
        this.name = name;
        this.engine = engine;
        numberOfCars++;
    }

    // getters and setters
}
```

우리가 초기화 하는 이 클래스의 모든 객체에 대해 numberOfCars 변수의 동일한 카피가 증가된다. 

따라서 이러한 경우에 이와 같은 것들은 true일 것이다.

```java
@Test
public void whenNumberOfCarObjectsInitialized_thenStaticCounterIncreases() {
    new Car("Jaguar", "V8");
    new Car("Bugatti", "W16");
 
    assertEquals(2, Car.numberOfCars);
}
```

### 3.2. Compelling Reasons to Use static Fields

- 변수 값이 객체와 독립적일 때
- 값이 모든 객체에서 공유되어야 하는 경우

### 3.3. Key Points to Remember

- 정적 변수는 클래스에 속하므로 클래스 이름을 사용하여 직접 액세스할 수 있다. 따라서 어떠한 객체 참조도 필요하지 않다.
- 정적 변수는 클래스 수준에서만 선언 가능하다.
- 객체를 초기화 하지 않아도 정적 필드에 액세스 할 수 있다.
- 마지막으로 객체 참조(ford.numberOfCars++)를 사용하여 정적 필드에 접근할 수 있다. 하지만 인스턴스 변수인지 클래스 변수인지 파악하기 어렵기 때문에 피해야 한다. 대신 항상 클래스 이름을 사용하여 정적 변수를 참조해야 한다(Car.numberOfCars++)

# 4. The static Methods (Or Class Methods)

정적 필드와 마찬가지로 정적 메서드도 객체 대신 클래스에 속한다. 따라서 사용자가 객체를 생성하지 않고 호출할 수 있다.

### 4.1. Example of static Method

우리는 일반적으로 객체 생성에 의존하지 않는 작업을 수행하기 위해 정적 메서드를 사용한다. 

해당 클래스의 모든 객체에서 코드를 공유하기 위해 정적 메서드로 코드를 작성한다.

```java
public static void setNumberOfCars(int numberOfCars) {
    Car.numberOfCars = numberOfCars;
}
```

또한 일반적으로 정적 메서드를 사용하여 유티리티 또는 헬퍼 클래스를 만들어 이러한 클래스의 새 객체를 만들지 않고 가져올 수 있다.

JDK의 Collections 또는 Math 유틸리티 클래스, Apache의 StringUtils 또는 Spring 프레임워크의 CollectionUtils를 살펴보고 모든 메서드가 정적임을 주목해라.

### 4.2. Compelling Reasons to Use static Methods

- 정적 변수 및 객체에 종속되지 않는 기타 정적 메서드에 액세스 하기 위해
- 정적 메서드는 유틸리티 및 헬퍼 클래스에 널리 사용된다.

### 4.3. Key Points to Remember

- 자바의 정적 메서드는 컴파일 시간에 처리된다. 메서드 오버라이딩이 런타임 다형성의 일부이기 때문에 정적 메서드는 오버라이드 될 수 없다.
- 추상 메서드는 정적일 수 없다.
- 정적 메서드는 this키워드나 super키워드를 사용할 수 없다.
- 아래의 객체 및 클래스 메서드, 변수의 조합은 유효하다
	- 인스턴스 메서드는 인스턴스 메서드와 인스턴스 변수 모두에 직접 액세스 가능
	- 인스턴스 메서드는 정적 변수와 정적 메서드에도 직접 액세스 가능
	- 정적 메서드는 모든 정적 변수 및 기타 정적 메서드에 액세스 가능
	- 정적 메서드는 인스턴스 변수 및 인스턴스 메서드에 직접 액세스 할 수 없다. 액세스 하려면 객체 참조가 필요하다.

# 5. A static Block
(생략)

# 6. A static Class

자바 프로그래밍 언어를 사용하면 클래스 내에 클래스를 만들 수 있다. 한 곳에서만 사용할 수 있는 강력한 그룹화 방법을 제공한다. 이는 코드를 좀 더 체계적이고 읽기 쉽게 유지하는데 도움이 된다.

중첩 클래스 아키텍쳐는 다음 두 가지로 나뉜다.

- static으로 선언한 중첩클래스. 정적 중첩 클래스라 불린다
- non-static한 중첩 클래스. 이너클래스라 불린다.

이 두가지 중첩 클래스의 주요 차이점은 다음과 같다. 이너클래스는 enclosing 클래스(including private)의 모든 멤버에 액세스할 수 있다. 반면 정적 중첩 클래스는 외부 클래스의 정적 멤버에만 액세스 할 수 있다.

실제로 정적 중첩 클래스는 다른 최상위 클래스처럼 동작하지만 더 나은 패키징 편의를 제공하기 위해 이 클래스에 액세스 할 유일한 클래스로 enclosed 되어있다(무슨 소리인지 모르겠음)

### 6.1. Example of static Class

싱글콘 객체를 만들기 위해 가장 널리 사용되는 접근법은 정적 중첩 클래스를 사용하는 것이다.

```java
public class Singleton  {
    private Singleton() {}

    private static class SingletonHolder {
        public static final Singleton instance = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHolder.instance;
    }
}

```

이 방법은 동기화가 필요하지 않고 구현이 쉽기 때문에 사용한다.

### 6.2. Compelling Reasons to Use a static Inner Class

- 한 곳에서만 사용될 클래스를 그룹화하면 캡슐화가 고도화된다.
- 우리는 해당 코드를 사용할 유일한 장소에 더 가까이 둘 수 있다. 이렇게 하면 가독성이 향상되고 유지관리가 쉽다.
- 중첩된 클래스에서 해당 클래스 인스턴스 멤버에 대한 액세스 권한이 필요하지 않으면 정적 클래스로 선언하는 것이 좋다. 이러한 방식으로 (둘러싸인)외부 클래스와 결합되지 않으므로 힙이나 스택 메모리가 필요하지 않기 때문에 더 최적화된다.

### 6.3. Key Points to Remember

- 정적 중첩 클래스는 (둘러싸인) 외부 클래스의 인스턴스 멤버에 액세스할 수 없다. 객체의 참조를 통해서만 액세스할 수 있다.
- 정적 중첩 클래스는 private을 포함하여 둘러싸인 클래스의 모든 정적 멤버에 접근 가능하다. 
- 자바 프로그래밍 사양에서는 최상위 클래스를 정적 클래스로 선언할 수 없다. 오직 중첩클래스만 정적 클래스로 만들 수 있다.










