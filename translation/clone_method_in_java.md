https://www.geeksforgeeks.org/clone-method-in-java-2/ 번역

객체 복제는 객체의 정확한 복사본을 만드는 것을 말한다. 현재 객체의 클래스의 새 인스턴스를 만들고 이 객체의 해당 필드 내용을 사용하여 새로이 생성된 객체의 모든 필드를 초기화 한다.

### Using Assignment Operator to create a copy of the reference variable(대입 연산자(=)를 사용하여 참조 변수의 복사본 생성)

자바에는 객체의 복사본을 만드는 연산자가 없다. C++과 달리 자바에서는 대입 연산자(=)를 사용하면 객체가 아닌 참조 변수의 복사본이 생성된다. 아래의 프로그램을 예로 들어보자.

```java
// Java program to demonstrate that assignment operator
// only creates a new reference to same object
import java.io.*;
 
// A test class whose objects are cloned
class Test {
    int x, y;
    Test()
    {
        x = 10;
        y = 20;
    }
}
 
// Driver Class
class Main {
    public static void main(String[] args)
    {
        Test ob1 = new Test();
 
        System.out.println(ob1.x + " " + ob1.y);
 
        // Creating a new reference variable ob2
        // pointing to same address as ob1
        Test ob2 = ob1;
 
        // Any change made in ob2 will
        // be reflected in ob1
        ob2.x = 100;
 
        System.out.println(ob1.x + " " + ob1.y);
        System.out.println(ob2.x + " " + ob2.y);
    }
}
```

결과

```java
10 20
100 20
100 20
```

### Creating a copy using the clone() method

객체의 복사본을 만들 클래스는 해당 클래스 또는 상위 클래스 중 하나에 public clone 메서드가 있어야 한다. 

clone()을 구현하는 모든 클래스는 super.clone()을 호출하여 복제된 객체 참조를 얻어야 한다. 

클래스도 java.lang.Cloneable 인터페이스를 구현해야 한다. 그렇지 않으면 해당 클래스의 객체에서 clone() 메서드를 호출할 때 CloneNotSupportedException 에러를 발생시킨다. 


### Usage of clone() method -Shallow Copy

참고 - 아래 코드 예제에서 clone() 메서드는 다른 해시코드 값을 가진 완전히 새로운 객체를 만든다. 즉 별도의 메모리 위치에 만들어진다. 그러나 Test 객체 c가 Test2 안에 있기 때문에 기본형 타입은 딥카피를 달성했지만 이 Test 객체 c는 여전히 t1과 t2 간에 공유된다. 이를 극복하기 위해 객체 변수 c에 대한 딥카피를 명시적으로 수행해야 하며, 이는 나중에 다룰 예정이다.

```java
// A java program to demonstrate shallow copy using clone()
import java.util.ArrayList;

// An object reference of this class is contained by Test2
class test {
	int x, y;
}

// Contains a reference of Test and implements clone with shallow copy.
class Test2 implements Cloneable {
	int a;
	int b;
	Test c = new Test();
	public Object clone() throws CloneNotSupportedException {
		return super.clone();
	}
}

// Driver class
public class Main {
	public static void main(String args[]) throws CloneNotSupportedException {
		Test2 t1 = new Test2();
		t1.a = 10;
		t1.b = 20;
		t1.c.x = 30;
		t1.c.y = 40;

		// Creating a copy of objcet t1 and passing it to t2
		Test2 t2 = (Test2)t1.clone();

		// Change in primitive type of t2 will not be reflected in t1 field
		t2.a = 100;

		// Change in object type field will be reflected in both t2 and t1(shallow copy)
		t2.c.x = 300;

		System.out.println(t1.a + " " + t1.b + " " + t1.c.x + " " + t1.c.y);
		System.out.println(t2.a + " " + t2.b + " " + t2.c.x + " " + t2.c.y);
	}
}
```

[Output]
```java
10 20 300 40
100 20 300 40
```

위의 예에서 t1.clone() 은 객체 t1의 얕은 복사본은 리턴한다. 객체의 깊은 복사본을 얻으려면 복사본을 얻은 후 추가적인 수정이 들어가야 한다.

### Deep Copy vs Shallow Copy

- 얕은 복사는 객체를 복사하는 방법이며 복제를 할 때 디폴트로 얕은 복사를 행하게 된다. 이 메서드에서는 이전 객체 X의 필드가 새 객체 Y로 복사된다. 객체 타입 필드를 복사하는 동안 참조값이 Y로 복사되게 된다. 즉 객체 Y는 X가 가리키는 가리키는 위치와 동일한 위치를 가리키게 된다. 만약 필드값이 기본타입이면 기본 타입의 값이 복사된다. 
- 따라서 객체 X 또는 Y 둘 중 하나에서 참조된 객체의 값이 변경되게 되면 다른 객체에도 반영된다. 

얕은 복사는 만들기 쉽고 비용이 저렴하다. 위의 예에서는 얕은 복사본을 만들었다.

### Usage of clone() method – Deep Copy 

객체 X의 깊은 복사본을 만들어 새 객체 Y에 배치하려면 참조된 객체 필드의 생성되고 이러한 참조가 객체 Y에 배치되어야 한다. 즉, 객체 X또는 Y의 참조된 객체 필드에서 변경한 내용은 해당 객체에만 반영되고 다른 객체에는 반영되지 않게 된다. 아래 예제에서는 객체의 깊은 복사본을 만들어 볼 예정이다.
깊은 복사는 모든 필드를 복사하고 필드가 가리키는 동적으로 할당된 메모리의 복사본을 만든다. 깊은 복사는 객체가 참조하는 객체가 참조하는 객체와 함께 복사될 때 발생한다.

```java
// A Java program to demonstrate
// deep copy using clone()
 
// An object reference of this
// class is contained by Test2
class Test {
    int x, y;
}
 
// Contains a reference of Test and
// implements clone with deep copy.
class Test2 implements Cloneable {
    int a, b;
    Test c = new Test();
 
    public Object clone() throws CloneNotSupportedException
    {
        // Assign the shallow copy to
        // new reference variable t
        Test2 t = (Test2)super.clone();
 
        // Creating a deep copy for c
        t.c = new Test();
        t.c.x = c.x;
        t.c.y = c.y;
 
        // Create a new object for the field c
        // and assign it to shallow copy obtained,
        // to make it a deep copy
        return t;
    }
}
 
public class Main {
    public static void main(String args[])
        throws CloneNotSupportedException
    {
        Test2 t1 = new Test2();
        t1.a = 10;
        t1.b = 20;
        t1.c.x = 30;
        t1.c.y = 40;
 
        Test2 t3 = (Test2)t1.clone();
        t3.a = 100;
 
        // Change in primitive type of t2 will
        // not be reflected in t1 field
        t3.c.x = 300;
 
        // Change in object type field of t2 will
        // not be reflected in t1(deep copy)
        System.out.println(t1.a + " " + t1.b + " " + t1.c.x
                           + " " + t1.c.y);
        System.out.println(t3.a + " " + t3.b + " " + t3.c.x
                           + " " + t3.c.y);
    }
}
```

[Output]

```java
10 20 30 40
100 20 300 40
```

위의 예제에서 Test 클래스에 대한 새 객체가 클론 메서드로 반환될 객체를 복사하도록 할당되었음을 알 수 있다. 이로인해 t3는 객체 t1의 깊은 복사본을 얻을 것이다. 따라서 t3에 의한 c 객체 필드의 변경 사항은 t1에 반영되지 않는다.


### Advantages of clone method

- 대입 연산자를 사용하여 객체 참조를 다른 참조 변수에 할당하면 이전 객체의 동일한 주소 위치를 가리키고 객체의 새 복사본이 생성되지 않는다. 이로 인해 참조 변수의 변경 사항이 원래 객체에도 반영된다.
- 복사생성자(copy constructor)를 사용하면 모든 데이터를 명시적으로 복사해야 한다. 즉 생성자에 있는 클래스의 모든 필드를 명시적으로 재할당해야한다. 그러나 클론 메서드에서는 새 복사본을 만드는 작업이 메서드 자체에 의해 수행된다. 따라서 추가 처리를 피하기 위해 object cloning을 사용한다. 

