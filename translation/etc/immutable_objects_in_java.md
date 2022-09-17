https://www.baeldung.com/java-immutable-object 번역

# 1. 소개

객체 불변성에 대해 알아보자.

# 2. What's an Immutable Object?

불변 객체란 완전히 생성된 후에도 내부 상태가 일정하게 유지되는 객체이다. 이것은 불변 객체의 public API가 전체 라이프 사이클 동안 동일한 방식으로 동작할 것을 보장한다는 것을 의미한다.

스트링 클래스를 살펴보면 스트링의 API가 replace 메서드로 mutable한 동작을 제공하는 것처럼 보일지라도 원본 스트링은 변경되지 않는다.

```java
String name = "hoon";
String newName = name.replace("on", "--");

assertEquals("hoon", name);
assertEquals("ho--", newName);
```

API는 우리에게 읽기 전용 메서드를 제공하므로 객체의 내부 상태를 변경하는 메서드를 포함해서는 안된다.

# 3. The final Keyword in Java

자바에서 불변성을 달성하려고 하기 전에 파이널 키워드에 대해 얘기해야한다. 자바에서 변수는 기본적으로 mutable하며 이는 변수가 보유하고 있는 값을 변경할 수 있음을 의미한다.

변수를 선언할 때 final 키워드를 사용함으로써 자바 컴파일러는 우리가 그 변수의 값을 변경할 수 없도록 한다. 대신 컴파일타임 에러를 레포트한다.

```java
final String name = "baeldung";
name = "bael...";
```

final은 변수가 보유하고 있는 참조를 변경하는 것을 금지할 뿐이며 퍼블릭 API를 사용하여 참조하는 객체의 내부 상태를 변경하는 것을 보호하지는 않는다.

```java
final List<String> strings = new ArrayList<>();
assertEquals(0, strings.size());
strings.add("hoon");
assertEquals(0, strings.size());
```

리스트에 아이템을 추가하면 크기가 변경되기 때문에 두번째 assertEquals은 실패한다. 따라서 이 리스트는 불변 객체가 아니다.

# 4. Immutability in Java

이제 변수 내용 변경을 피하는 방법을 알았으므로 이를 사용하여 불변 객체의 API를 구축할 수 있다. 

불변 객체의 API를 구축하려면 API를 어떻게 사용하든 내부 상태가 변경되지 않도록 보장해야 한다. 

이를 위해 속성을 선언할 때 final을 사용하는 것이다.

```java
class Money {
    private final double amount;
    private final Currency currency;

    // ...
}
```

자바는 우리에게 amount의 값이 변하지 않을 것을 보장한다는 것을 주목해라. 모든 기본형 타입 변수의 경우에 적용된다.

그러나 이 예에서 우리는 currency가 변하지 않을 것이라는 것만 보장받으므로 우리는 Currency API가 변경으로부터 스스로를 보호할 것이란 것에 의존해야한다.

대부분의 경우 커스텀 값을 홀드하기 위해선 객체의 attributes가 필요하며 불변 객체의 내부상태를 초기화 하는 위치는 객체의 생성자이다.

```java
class Money {
    // ...
    public Money(double amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }

    public Currency getCurrency() {
        return currency;
    }

    public double getAmount() {
        return amount;
    }
}
```

앞서 말했듯이 불변 API의 요구사항을 충족하기 위해 우리의 Money 클래스는 읽기 전용 메서드만 있다. 

reflection API를 사용하여 불변성을 깨고 불변객체를 변경할 수 있다. 그러나 reflection은 불변 객체의 public API를 위반하므로 우리는 보통 이것을 피해야만 한다.

# 5. Benefits

불변 객체의 내부 상태는 시간에 따라 일정하게 유지되므로 여러 스레드에서 안전하게 공유할 수 있다.

우리는 또한 그것을 자유롭게 사용할 수 있고 그것을 참조하는 어떤 객체도 어떠한 차이도 알아차리지 못할것이며 우리는 불변 객체가 사이드이펙트로부터 자유롭다 말할 수 있다.

# 6. Conclusion

불변 객체는 시간에 따라 내부 상태를 바꾸지 않고 스레드 세이프 하며 사이드이펙트로부터 자유롭다. 이러한 특성 때문에 불변 객체는 다중 스레드 환경을 처리할 때 특히 유용하다.




















