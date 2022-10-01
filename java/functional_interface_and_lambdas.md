이전 아티클에서 익명클래스에 대해 소개하였다(참고: 링크). 이러한 익명클래스와 관련된 한가지 문제는 익명클래스 구현이 매우 간단한 경우, 예를들어 하나의 메서드만 포함하는 인터페이스를 익명클래스로 구현한다면, 불필요하게 어렵게 보일수 있고 또한 가독성도 떨어진다는 점이다. 예를 들어보자.

```java
interface Calculate {
	int operation(int a, int b);
}
```

```java
private void calculateClassic() {
	Calculate calculateAdd = new Calculate() {
		@Override
		public int operation(int a, int b) {
			return a+b;
		}
	};

	System.out.println(calculateAdd.operation(1.2));
}
```
이렇게 다른 메서드에 특정 단일 기능(functionality)을 인수로서 전달하기 위해 익명클래스를 사용하게 된다. 

람다식을 사용하면 이러한 functionality를 메서드 인수로 넘길 수 있다. 

```java
// 람다식을 사용하여 리팩토링
private void calculateClassic() {
	Calculate calculateAdd = (a, b) -> a+b;
	
	System.out.println(calculateAdd.operation(1.2));
}
```
이렇게 람다식을 사용하면 단일 메서드 클래스의 인스턴스를 보다 압축적으로 표현이 가능하다.

위의 Calculate 인터페이스와 같이 오직 하나의 추상 메서드만 정의되어 있는 인터페이스를 함수형 인터페이스라 부른다.
람다로 만든 객체에 접근하기 위해서는 이러한 함수형 인터페이스가 필요하며 자바에서는 몇가지 이러한 함수형 인터페이스를 제공하고 있다.
- Runnable
- Supplier
- Consumer
- Function<T, R>
- Predicate

### 람다에서 사용하는 외부 지역 변수는 왜 final 혹은 effectively final이어야 하는가.

람다식에서는 외부 블록에 있는 변수에 접근이 가능하다. 이러한 변수에는 인스턴스 변수, 클래스 변수, 지역 변수가 포함된다. 
하지만 지역 변수의 경우 반드시 final 혹은 effectively final이어야 한다. 그렇지 않으면 컴파일 에러가 발생한다.

effectively final의 의미는 아래와 같다.
> A non-final local variable or method parameter whose value is never changed after initialization is known as effectively final.

즉 변수가 초기화된 이후 값이 한 번도 변경되지 않았다면 effectively final이라 할 수 있다. 자바 8 이전에는 final 키워드를 반드시 붙여야만 했으나 자바8부터는 일종의 syntactic sugar로서 이러한 변수를 final로 간주할 수 있게 되었다.

이렇게 외부 변수에 접근하는 람다식을 Capturing lambda라고 한다. 

#### Capturing lambda에서의 지역 변수 

```java
Supplier<Integer> incrementer(int start) {
  return () -> start++;
}
```

위의 코드는 컴파일 에러가 난다. 람다식 내에서 외부 지역 변수에 접근할 때 이 지역 변수가 final일 것임을 가정해두는데, 위의 코드에서는 start를 변경하려 하고 있기 때문이다.
final 혹은 effectively final한 지역 변수를 사용하는 이유는 람다식에서 지역 변수를 그대로 사용하지 못하고 복사하여 사용하기 때문이다. 만약 지역 변수가 변경 가능하다면 람다식에서 해당 변수를 사용할 때 정합성에 문제가 발생할 수도 있다.

그렇다면 왜 복사본을 사용할까?
위의 코드에서 보면 incrementer 메서드는 람다식을 리턴하고 있다. 
메서드가 호출될 때 스택에 스택프레임으로 올라가게 되고 리턴되면 메서드에서 사용된 지역 변수와 함께 스택에서 제거된다. 
람다식이 추후에 수행될 때 이 지역 변수는 이미 스택에서 제거어 참조할 수 없으므로 람다식에서는 지역 변수를 복사하여 갖고 있게 되는 것이다.
이렇게 start 복사본을 사용함으로써 위의 람다식은 메서드 밖에서도 살아있을 수 있게 된 것이다. 
같은 맥락에서, 람다식이 다른 스레드에서 돌아갈 경우에도 지역 변수가 final하지 못하다면 동시성 이슈가 발생될 여지가 있다. 
스레드는 각자의 스택을 갖고 지역변수를 그 스택에서 관리하게 된다. 기존 스레드에서 지역 변수를 변경하게 되었을 때 다른 스레드에서 돌고있는 람다식일 경우 그 값의 변경을 알 수가 없다.

```java
public void localVariableMultithreading() {
    boolean run = true;
    executor.execute(() -> {
        while (run) {
            // do operation
        }
    });
    
    run = false;
}
```

위의 코드와 같이, run 변수가 변경된 스레드와 while문을 포함한 람다식이 있는 스레드가 다르기 때문에 만약 run과 같이 final이 아닌 변경 가능한 변수라면 이슈가 발생될 여지가 크다.
