# 다형성이란

- 다형성이란 하나의 메서드나 클래스가 있을 때 이것들이 다양한 방법으로 동작하는 것을 의미
	- 오버로딩. 같은 이름 다른 동작 방법.

- 예제 코드
```java
public class CalculatorDemo {
    public static void execute(Calculator cal){
        System.out.println("실행결과");
        cal.run();
    }
    public static void main(String[] args) { 
        Calculator c1 = new CalculatorDecoPlus();
        c1.setOprands(10, 20);
         
        Calculator c2 = new CalculatorDecoMinus();
        c2.setOprands(10, 20);
         
        execute(c1);
        execute(c2);
    }
}
```

- execute 메서드의 매개변수 타입이 Calculator가 아니라면 위의 로직을 처리할 수 없을 것이다. 아래와 같이 될 것이다
```java
public static void execute(CalculatorDecoPlus cal){
    System.out.println("실행결과");
    cal.run();
}

public static void execute(CalculatorDecoMinus cal){
    System.out.println("실행결과");
    cal.run();
}
```
- 코드의 중복이 일어난다.
- Calculator타입을 매개변수로 받게됨으로써 execute 메서드 입장에서는 매개변수로 전달된 값이 Calculator 이거나 그 자식이라면 run 메서드를 가지고 있다는 것을 보장받을 수 있게 된다.
- 즉 다형성이란 하나의 클래스(Calculator)가 다양한 동작 방법(ClaculatorDecoPlus, ClaculatorDecoMinus)을 가지고 있는데 이 것을 다형성이라 할 수 있다.

- 인터페이스와 다형성
	- 특정한 인터페이스를 구현하고 있는 클래스가 있을 때 이 클래스의 데이터 타입으로 인터페이스를 지정할 수 있다.
	- 예제코드
	```java
	package org.opentutorials.javatutorials.polymorphism;
	interface I2{
	    public String A();
	}
	interface I3{
	    public String B();
	}
	class D implements I2, I3{
	    public String A(){
	        return "A";
	    }
	    public String B(){
	        return "B";
	    }
	}
	public class PolymorphismDemo3 {
	    public static void main(String[] args) {
	        D obj = new D();
	        I2 objI2 = new D();
	        I3 objI3 = new D();
	         
	        obj.A();
	        obj.B();
	         
	        objI2.A();
	        //objI2.B();
	         
	        //objI3.A();
	        objI3.B();
	    }
	}
	```

	- 주석처리된 메서드 호출은 오류가 발생한다. objI2.b()에서 오류가 발생하는 이유는 objI2의 데이터 타입이 인터페이스 I2이기 때무니. 인터페이스 I2는 메서드 A만을 정의하고 있고 I2를 데이터 타입으로 하는 인스턴스는 마치 메서드 A만 가지고 있는 것처럼 동작하기 때문
	- 인스턴스 objI2의 데이터 타입을 I2로 한다는 것은 인스턴스를 외부에서 제어할 수 있는 조작 장치를 인스턴스 I2의 맴버로 제한한다는 의미가 된다. 인스턴스 I2와 I3로 인해서 하나의 클래스가 다양한 형태를 띄게 되는 것이다.
