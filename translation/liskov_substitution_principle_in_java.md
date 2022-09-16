https://www.baeldung.com/java-liskov-substitution-principle 번역

# 1. Overview

SOLID 디자인 원리는 로버트 C. 마틴이 그의 2000년 논문, Design Principles and Design Patterns에서 소개되었다. SOLID 디자인 원칙은 우리가 더 유지보수가 가능하고 이해하기 쉬우며 유연한 소프트웨어를 만드는 데 도움을 준다.

이 아티클에서는 리스코프 대체 원리에 대해 말해볼 예정이다.

# 2. The Open/Closed Principle

리스코프 치환 원칙을 이해하기 위해서는 먼저 개방/폐쇄 원칙을 이해해야 한다. 개방/폐쇄 원칙의 목표는 소프트웨어를 설계하도록 장려하여 새로운 코드를 추가하는 것만으로 새로운 기능을 추가할 수 있게 한다. 이것이 가능할 때 우리는 느슨하게 결합하고 따라서 쉽게 유지관리할 수 있는 애플리케이션을 만들 수 있다.

# 3. An Example Use Case

개방/폐쇄 원칙을 이해하기 위해 뱅킹 어플리케이션 예제를 봐보자.

## 3.1. Without the Open/Closed Principle

우리의 뱅킹 어플리케이션은 두가지 계좌 타입을 지원한다. "current"와 "savings"이다. 이들은 각각 CurrentAccount와 SavingsAccount 클래스로 대변된다. BankingAppWithdrawalService는 사용자에게 인출 기능을 제공한다.


![open_closed_example_1](/assets/open_closed_example_1.webp)
[이미지출처](https://www.baeldung.com/java-liskov-substitution-principle)<br/>

안타깝게도 이 디자인을 확장하는 데 문제가 있다. BankingAppWithdrawalService는 계좌의 두가지 구체적인 구현을 알고 있다. 따라서 BankingAppWithdrawalService는 새로운 계정 유형이 도입될 때 마다 변경되어야 한다.


## 3.2. Using the Open/Closed Principle to Make the Code Extensible

개방/폐쇄 원칙을 준수하도록 재설계 해보자. 새로운 계정 유형이 필요할 때 Account base 클래스를 사용하는 것으로 수정해보자.

![open_closed_example_1](/assets/open_closed_example_2.webp)
[이미지출처](https://www.baeldung.com/java-liskov-substitution-principle)<br/>

이런식으로 CurrentAccount와 SavingsAccount가 extend 할 수 있는 새로운 추상 클래스인 Account를 추가하자.

BankingAppWithdrawalService는 더이상 구체적인 계정 클래스에 의존하지 않는다. 지금은 추상 클래스에만 의존하기 때문에 새 계정 유형을 도입할 때 변경할 필요가 없다. 

따라서 BankingAppWithdrawalService는 새로운 계정 타입의 확장에 대해 개방되어있지만 새로운 계정 타입의 경우 통합을 위해 변경할 필요가 없다는 점에서 수정에는 닫혀있다. 


## 3.3 자바 코드

Account 클래스를 정의해보자.

```java
public abstract class Account {
	protected abstract void deposit(BigDecimal amount);

	protected abstract void withdraw(BigDecimal amount);
}
```

BankingAppWithdrawalService 클래스를 정의해보자.

```java
public class BankingAppWithdrawalService {
	private Account account;

	public BankingAppWithdrawalService(Account account) {
		this.account = account;
	}	

	public void withdraw(BigDecimal amount) {
		account.withdraw(amount)
	}
}
```

어떻게 이 설계에서 새로운 계정 타입이 리스코프치환원칙을 위반할 수 있는지 살펴보자.


## 3.4. A New Account Type

은행은 이제 고객들에게 높은 이자를 받는 정기 예금 계좌를 제공하기를 원한다. 이를 지원하기 위해, 새로운 FixedTermDepositAccount 클래스를 만들자. 현실 세계의 정기예금 계좌는 일종의 계좌이다. 이는 우리의 객체 지향 설계에서 상속을 의미한다.
FixedTermDepositAccount를 Account의 서브클래스로 만들어보자.

```java
public class FixedTermDepositAccount extends Account {
	// overriden methods
}
```

지금까지는 좋다. 하지만 은행은 정기예금 계좌에 대한 인출을 허용하지 않으려 한다. 
이는 새로운 FixedTermDepositAccount 클래스는 Account에서 정의한 withdraw 메서드를 의미있게 제공할 수 없단 것이다. 이에 대한 일반적인 해결 방법 중 하나는 FixedTermDepositAccount가 메서드에서 UnsupportedOperationException 에러를 발생시키는 것이다.

```java
public class FixedTermDepositAccount extends Account {
	@Override
	protected void deposit(BigDecimal amount) {
		// Deposit into this account
	}

	@Override
	protected void withdraw(BigDecimal amount) {
		throw new UnsupportedOperationException("Withdrawals are not supported")
	}
}
```

## 3.5. Testing Using the New Account Type

이 새로운 클래스를 BankingAppWithdrawalService와 함께 사용해보자.

```java
Account myFixedTermDepositAccount = new FixedTermDepositAccount();
myFixedTermDepositAccount.deposit(new BigDecimal(1000.00));

BankingAppWithdrawalService withdrawalService = new BankingAppWithdrawalService(myFixedTermDepositAccount);
withdrawalService.withdraw(new BigDecimal(100.00));
```

당연히 뱅킹 애플리케이션은 에러를 발생시킨다. 

유효한 객체 조합이 오류를 발생시킨다면 이 설계에는 분명히 문제가 있다.


## 3.6. What Went Wrong?

BankingAppWithdrawalService는 Account 클래스의 클라이언트이다. 이는 Account과 서브타입 모두 Account 클래스가 인출 방법에 대해 지정한 동작을 보증할 것으로 예상한다. 
하지만 withdraw 메서드를 지원하지 않음으로써 FixedTermDepositAccount는 이 메서드 명세를 위반한다. 따라서 FixedTermDepositAccount를 Account로 확실하게 대체할 수 없다. 
다른말로 하자면 FixedTermDepositAccount가 리스코프치환원칙을 위반한 셈이다.


## 3.7. Can't We Handle the Error in BankingAppWithdrawalService?

우리는 클라이언트의 Account withdraw 메서드를 호출할 때 발생할 수 있는 오류를 인지하도록 디자인을 수정할 수 있다. 하지만 이는 클라이언트가 예기치 않은 동작에 대한 특별한 지식을 가지고 있어야 한다는 것을 의미한다. 이는 개방/폐쇄 원칙을 깨기 시작한다. 
즉 개방/폐쇄 원칙이 잘 작동하기 위해서는 클라이언트 코드를 수정할 필요 없이 모든 서브타입이 슈퍼타입을 대체할 수 있어야 한다. 리스코프 치환 원칙을 고수하는 것은 이러한 개체 가능성을 보장한다. 

이제 리스코프 치환 원칙에 대해 자세히 살펴보자


# 4. 리스코프 치환 원칙

## 4.1. 정의

로버트 마틴은 아래와 같이 요약했다.

> Subtypes must be substitutable for their base types.

Barbara Liskov는 1988년에 이를 정의했고 보다 수학적 정의를 제공했다.

> 만약 타입 S의 각 객체 o1에 대해 T의 관점에서 정의된 모든 프로그램 P에 대해 T 타입의 객체 o2가 있다면, o1이 o2를 대체 했을 때 P의 동작에 변화가 없다면 S는 T의 서브타입이다.

## 4.2. When Is a Subtype Substitutable for Its Supertype?

서브타입은 슈퍼타입을 자동으로 대체할 수 없다. 대체할 수 있으려면 서브타입은 슈퍼타입처럼 행동해야 한다. 

객체의 동작은 클라이언트가 믿을 수 있는 계약이다. 동작은 퍼블릭 메서드, 인풋에 대한 제약 조건, 객체가 겪는 상태 변화, 메서드 실행의 부작용에 의해 지정된다. 

자바에서 서브타입을 입력하려면 기본 클래스의 속성 및 메서드를 하위 클래스에서 사용할 수 있어야 한다. 

하지만 behavioral subtyping은 서브타입이 슈퍼타입의 모든 메서드를 제공할 뿐만 아니라 슈퍼타입의 행동 규격을 준수해야 한다는 것을 의미한다. 이렇게 하면 슈퍼타입 동작에 대한 클라이언트의 모든 가정이 서브타입에 의해서도 충족되는 것을 보장한다. 

이것이 리스코프 치환 원칙이 객체지향 설계에 가져오는 추가적인 제약이다. 

이제 앞에서 마주친 문제를 해결하기 위해 뱅킹 애플리케이션을 리팩터링 해보겠다.


# 5. 리팩토링

뱅킹 예제에서 우리가 찾아낸 문제를 해결하기 위해 근본적인 원인을 이해해보자.

## 5.1. The Root Cause

이 예에서 우리의 FixedTermDepositAccount는 Account의 서브타입 처럼 동작하지 않았다.

Account 설계는 모든 Account 타입이 인출을 허용한다고 잘못 가정 했다. 이에 따라 인출을 지원하지 않는 FixedTermDepositAccount를 포함한 모든 Account의 서브타입이 인출 메서드를 이어받았다.

Account의 계약(constract)을 연장하여 해결할 수 있지만 다른 해결책이 있다.


## 5.2. Revised Class Diagram


계좌 계층을 다시 설계해보자.

![liskov_example_1](/assets/liskov_example_1.webp)
[이미지출처](https://www.baeldung.com/java-liskov-substitution-principle)<br/>

모든 계정이 인출을 지원하지 않기 때문에 Account 클래스에서 인출 메서드를 새로운 추상 서브클래스인 WithdrawableAccount 으로 옮겼다. CurrentAccount와 SavingsAccount 모두 인출이 가능하다. 그래서 그들은 이제 새로운 WithdrawableAccount의 서브클래스가 되었다. 

이는 BankingAppWithdrawalService가 인출 기능을 제공할 수 있는 올바른 유형의 계정을 신뢰할 수 있다는 것을 의미한다. 

## 5.3. Refactored BankingAppWithdrawalService

BankingAppWithdrawalService는 이제 WithdrawableAccount를 사용해야할 필요가 생겼다.

```java
public class BankingAppWithdrawalService {
    private WithdrawableAccount withdrawableAccount;

    public BankingAppWithdrawalService(WithdrawableAccount withdrawableAccount) {
        this.withdrawableAccount = withdrawableAccount;
    }

    public void withdraw(BigDecimal amount) {
        withdrawableAccount.withdraw(amount);
    }
}
```

FixedTermDepositAccount의 경우, 우리는 Account를 부모 클래스로 유지한다. 따라서 안정적으로 이행할 수 있는 입금 행위만 상속하고 인출 메서드는 더이상 상속하지 않는다. 이 새로운 디자인은 앞에서 본 문제들을 피한다. 


# 6. 규칙

이제 제대로, 잘 작동하는 서브타입을 생성하기 위해 우리가 따라야하고 사용해야하는 메서드 시그니쳐, invariants, 전제조건 및 사후조건에 관한 몇 가지 규칙/기법에 대해 알아보자. 

바바라 리스코프와 존 구태그는 그들의 책에서 이러한 규치을 시그니쳐 룰, 프로퍼티 룰, 메서드 룰의 세가지 범주로 분류했다.

이러한 practices 중 일부는 이미 자바의 오버라이딩 룰에 의해 시행되고 있다.

여기서 몇가지 용어를 알아두어야 한다. 넓은 타입이 더 일반적이다. 예를 들어 객체는 모든 자바 객체를 의미할 수 있다.


## 6.1. Signature Rule – Method Argument Types

이 규칙은 오버라이드 된 서브타입 메서드 인수 타입이 슈퍼 타입 메서드 인수 타입과 동일하거나 더 넓을 수 있다고 명시한다. 

자바의 메서드 오버라이딩 규칙은 오버라이드된 메서드 인수 유형이 슈퍼타입 메서드와 정확히 일치하도록 하여 이 규칙을 지원한다.


## 6.2. Signature Rule – Return Types

오버라이드된 서브타입 메서드의 리턴 타입은 슈퍼 유형 메서드의 리턴 타입보다 좁을 수 있다. 이를 리턴 타입의 공분산이라 한다. 공분산은 서브 타입이 슈퍼 타입 대신 허용될 때를 나타낸다. 자바는 리턴 타입의 공분산을 지원한다. 예를 들어 보자.

```java
public abstract class Foo {
    public abstract Number generateNumber();    
    // Other Methods
}
```

Foo의 generateNumber 메서드는 리턴타입이 Number이다. 이제 더 좁은 유형의 Integer를 리턴하여 이 메서드를 오버라이드하자.

```java
public class Bar extends Foo {
    @Override
    public Integer generateNumber() {
        return new Integer(10);
    }
    // Other Methods
}
```

Integer IS-A Number 기 때문에 Number가 예상되는 클라이언트 코드는 Foo를 Bar로 문제없이 대체할 수 있다.

반면에 Bar에서 오버라이드 된 메서드가 Number보다 넓은 유형, 예를 들어 Object를 리턴하는 경우(Truck이라하자), Number의 리턴타입에 의존하는 모든 클라이언트 코드는 트럭을 처리할 수 없다.

다행히 자바 메서드 오버라이드 규칙은 더 넓은 타입을 리턴하는 오버라이드 메서드를 방지한다.

## 6.3. Signature Rule – Exceptions

서브타입 메서드는 슈퍼타입 메서드보다 더 적거나 더 좁은(그러나 추가적이거나 더 넓지 않은) 예외를 던질 수 있다. 

클라이언트 코드가 서브타입을 대체할 때 슈퍼타입 메서드보다 더 적은 예외를 던지는 메서드를 처리할 수 있기 때문에 이는 이해될 수 있다. 그러나 서브 타입의 메서드가 확인된 새 예외 또는 더 광범위한 예외를 발생시키면 클라이언트 코드가 망가진다. 

자바 메서드 오버라이딩 규칙은 이미 확인된 예외(checked exceptions)에 대해 이 규칙을 적용한다. 그러나 오버라이딩 메서드는 오버라이드된 메서드가 예외를 선언했느지 여부에 관계없이 RuntimeException 을 던질 수 있다. 

## 6.4. Properties Rule – Class Invariants

클래스 Invariants(불변식?)는 객체의 모든 유효한 상태에 대해 참이어야 하는 객체 속성에 대한 주장이다. 예를 들어 보자.

```java
public abstract class Car {
    protected int limit;

    // invariant: speed < limit;
    protected int speed;

    // postcondition: speed < limit
    protected abstract void accelerate();

    // Other methods...
}
```

Car 클래스는 속도가 항상 리미트 미만이어야 하는 클래스 불변식을 갖는다. 불변식 규칙은 모든 서브타입 메서드(상속되고 새로운)가 슈퍼타입의 클래스 불변식을 유지하거나 강화해야 한다고 명시한다. 

불변식을 보존하는 Car의 서브클래스를 정의하자.

```java
public class HybridCar extends Car {
    // invariant: charge >= 0;
    private int charge;

      @Override
    // postcondition: speed < limit
    protected void accelerate() {
        // Accelerate HybridCar ensuring speed < limit
    }

    // Other methods...
}
```

이 예에서 Car의 불변식은 HybridCar의 오버라이드된 accelerate() 메서드에 의해 보존된다. HybridCar는 자체 클래스 불변식인 charge >= 0을 추가로 정의하며 이는 전혀 문제가 없다. 

반대로 클래스 불변식이 서브타입에 의해 보존되지 않으면 슈퍼타입에 의존하는 클라이언트 코드는 망가진다.


## 6.5. Properties Rule – History Constraint

히스토리 제약 조건은 서브클래스 메서드(상속되거나 새로운)가 기본 클래스에서 허용하지 않는 상태 변경을 허용하지 않아야 한다고 명시한다.

예를 들어보자.

```java
public abstract class Car {

    // Allowed to be set once at the time of creation.
    // Value can only increment thereafter.
    // Value cannot be reset.
    protected int mileage;

    public Car(int mileage) {
        this.mileage = mileage;
    }

    // Other properties and methods...

}
```

Car 클래스는 마일리지 속성에 대한 제약을 지정한다. 마일리지 속성은 생성 시 한 번만 설정할 수 있으며 그 이후에는 재설정할 수 없다. 

이제 Car를 상속받는 ToyCar를 정의해보자.

```java
public class ToyCar extends Car {
    public void reset() {
        mileage = 0;
    }

    // Other properties and methods
}
```

ToyCar에는 마일리지 속성을 재설정하는 추가 메서드가 있다. 이렇게 함으로써 ToyCar는 부모가 마일리지 속성에 부과하는 제약을 무시했다. 이렇게 하면 제약 조건에 의존하는 모든 클라이언트 코드는 망가진다. 그래서 ToyCar는 자동차를 대체할 수 없다. 

마찬가지로 기본 클래스에 불변 속성이 있는 경우 서브클래스는 이 속성을 수정하도록 허용하지 않아야 한다. 이것이 immutable 클래스가 final이어야 하는 이유이다.


## 6.6. Methods Rule – Preconditions

메서드를 실행하기 전에 전제 조건이 충족되어야 한다. 매개 변수 값과 관련된 전제 조건의 예를 살펴보자.

```java
public class Foo {

    // precondition: 0 < num <= 5
    public void doStuff(int num) {
        if (num <= 0 || num > 5) {
            throw new IllegalArgumentException("Input out of range 1-5");
        }
        // some logic here...
    }
}
```

여기서 doStuff 메서드의 전제 조건은 num 파라미터 값이 1과 5사이여야한다고 명시한다. 우리는 메서드 내부의 레인지 체크를 통해 이 전제 조건을 적용했다. 서브타입은 오버라이드되는 메서드의 전제조건을 약화시킬 수는 있지만 강화시킬수는 없다. 서브타입이 전제조건을 약화시키면 슈퍼타입 메서드로 부과되는 제약을 완화한다. 

이제 약해진 전제조건으로 doStuff 메서드를 오버라이드해보자.

```java
public class Bar extends Foo {

    @Override
    // precondition: 0 < num <= 10
    public void doStuff(int num) {
        if (num <= 0 || num > 10) {
            throw new IllegalArgumentException("Input out of range 1-10");
        }
        // some logic here...
    }
}
```

여기서 오버라이드 된 doStuff 메서드에서 전제조건은 0 < num <= 10으로 약화되어 num에 대한 더 넓은 범위의 값을 허용한다. Foo.doStuff에 유효한 모든 num값은 Bar.doStuff에도 유효하다. 따라서 Foo.doStuff의 클라이언트는 Foo를 Bar로 대체할 때 차이를 인식하지 못한다. 

반대로 서브타입이 전제 조건을 강화할 때(예를 들어 0 < num <= 3) 그것은 슈퍼타입보다 더 엄격한 제한을 적용하게된다. 예를 들어 num의 값 4와 5는 Foo.doStuff에는 유효하지만 Bar.doStuff에서는 더이상 유효하지 않다.

이렇게 하면 이 새로운 엄격한 제약 조건을 예상하지 않는 클라이언트 코드는 망가지게 된다.


## 6.7. Methods Rule – Postconditions

사후조건은 메서드가 실행된 후에 충족되어야 하는 조건이다. 예를 들어보자

```java
public abstract class Car {

    protected int speed;

    // postcondition: speed must reduce
    protected abstract void brake();

    // Other methods...
}
```

여기서 Car의 브레이크 메서드는 메서드 실행이 종료될 때 Car의 속도가 감소해야 하는 사후 조건을 명시한다. 서브타입은 오버라이드되는 메서드에 대한 사후 조건을 강화(약하게 할 수는 없다)할 수 있다. 서브타입이 사후 조건을 강화하면 슈퍼타입 메서드 이상의 것을 제공한다.

예를 들어보자.

```java
public class HybridCar extends Car {

   // Some properties and other methods...

    @Override
    // postcondition: speed must reduce
    // postcondition: charge must increase
    protected void brake() {
        // Apply HybridCar brake
    }
}
```

HybridCar에서 오버라이드 된 브레이크 메서드는 charge도 증가하도록 추가로 보장함으로써 사후 조건을 강화한다. 결과적으로 Car 클래스에서 브레이크 메서드의 사후 조건에 의존하는 모든 클라이언트 코드는 HybridCar를 Car로 대체할 때 차이를 인식하지 못한다.

반대로 HybridCar가 오버라이드된 브레이크 메서드의 사후 조건을 약화시킨다면 더 이상 속도가 감소한다는 보장이 없다. 이것은 HybridCar가 Car의 대체품으로 주어진 클라이언트 코드를 망가뜨릴 수 있다.


# 7. Code Smells

슈퍼타입을 대체할 수 없는 서브타입을 현실 세계에서 어떻게 찾을 수 있을까.

리스코프 치환 원칙에 위배되는 일반적인 코드스멜을 살펴보자.


## 7.1. A Subtype Throws an Exception for a Behavior It Can't Fulfill

우리는 앞서 뱅킹 애플리케이션의 예에서 이러한 예를 볼 수 있었다. 

리팩토링 전에 Account 클래스는 서브 클래스인 FixedTermDepositAccount가 원하지 않는 인출 메서드가 있었다. FixedTermDepositAccount 클래스는 인출메서드에 UnsupportedOperationException 에러를 던져 이 문제를 해결했다. 하지만 이는 상속 계층 모델링의 약점을 은폐하기 위한 hack에 불과하다.


## 7.2. 서브타입은 수행할 수 없는 동작에 대한 구현을 제공하지 않는다.

이것은 상기 코드스멜의 변형이다. 서브타입은 동작을 수행할 수 없으므로 재정의된 메서드에서 아무 작업도 수행하지 않는다. 

FileSystem interface를 정의해보자.

```java
public interface FileSystem {
    File[] listFiles(String path);

    void deleteFile(String path) throws IOException;
}
```

FileSystem을 구현하는 ReadOnlyFileSystem을 정의해보자.


```java
public class ReadOnlyFileSystem implements FileSystem {
    public File[] listFiles(String path) {
        // code to list files
        return new File[0];
    }

    public void deleteFile(String path) throws IOException {
        // Do nothing.
        // deleteFile operation is not supported on a read-only file system
    }
}
```

여기서 ReadOnlyFileSystem은 deleteFile 작업을 지원하지 않으므로 구현을 제공하지 않는다.

## 7.3. The Client Knows About Subtypes

만약 클라이언트 코드가 instanceof 이나 downcasting을 사용해야 하는 경우, 개방/폐쇄 원칙과 리스코프 치환 원칙이 모두 위반되었을 가능성이 있다.

FilePurgingJob을 사용하여 설명해보자.

```java
public class FilePurgingJob {
    private FileSystem fileSystem;

    public FilePurgingJob(FileSystem fileSystem) {
        this.fileSystem = fileSystem;
    }

    public void purgeOldestFile(String path) {
        if (!(fileSystem instanceof ReadOnlyFileSystem)) {
            // code to detect oldest file
            fileSystem.deleteFile(path);
        }
    }
}
```

FileSystem 모델은 기본적으로 읽기전용 파일 시스템과 호환되지 않으므로 ReadOnlyFileSystem은 지원할 수 없는 deleteFile 메서드를 상속한다. 이 예제 코드는 서브 타입 구현을 기반으로 특수 작업을 수행하기 위해 instanceof 을 사용한다.

## 7.4. A Subtype Method Always Returns the Same Value

이것은 다른 것들보다 훨씬 더 미묘한 위반이며 발견하기 어렵다. 이 예에서 ToyCar는 항상 remainingFuel 속성에 대해 고정 값을 반환한다. 

 ```java
public class ToyCar extends Car {

    @Override
    protected int getRemainingFuel() {
        return 0;
    }
}
```

이것은 인터페이스와 값이 무엇을 의미하느냐에 따라 다르지만 일반적으로 객체의 변경 가능한 상태 값이 되어야 하는 것을 하드코딩 하는 것은 서브클래스가 슈퍼타입을 충족시키지 못하고 있으며 실제로 그것을 대체할 수 없다는 신호이다. 


# 8. Conclusion

이 아티클에서는 리스코프 치환 원칙을 살펴보았다. 

리스코프 치환 원칙은 좋은 상속 계층을 모델링하는데에 도움이 된다. 개방/폐쇄 원칙을 준수하지 않는 모델 계층 구조를 방지하는데 도움이 된다.

리스코프 치환 원칙을 준수하는 모든 상속 모델은 암묵적으로 개방/폐쇄 원칙을 따를 것이다.

먼저 개방/폐쇄 원칙을 따르려 하지만 리스코프 치환 원칙에 위배되는 활용 사례를 살펴보았다. 다음으로 리스코프 치환 원칙의 정의, behavioral subtyping의 개념, 서브타입이 반드시 따라야 하는 규칙을 살펴보았다. 

마지막으로 기존 코드의 위반을 탐지하는 데 도움이 될 수 있는 몇 가지 일반적인 코드 스멜에 대해 살펴보았다. 
