# 컴포넌트 스캔과 의존관계 자동 주입

### 작동 방식 및 스캔 범위
- @ComponentScan은 @Component가 붙은 모든 클래스를 스프링 빈으로 등록한다. 그리고 생성자에 @Autowired를 지정하면 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아 주입시켜준다. 
- 보통 스캔의 범위는 지정된 basePackage를 시작 위치로 하위 패키지 모두를 탐색하게된다. 디폴트로는 @ComponentScan은이 붙은 클래스의 패키지가 시작 위치가 된다. 

### 스프링부트에서의 컴포넌트 스캔
- @SpringBootApplication 어노테이션 안을 살펴보면 @ComponentScan을 확인할 수 있다. 
- 스캔 기본 대상 역시 @Controller, @Service, @Repository, @Configuration 등의 어노테이션이 붙어 있다면 스캔의 대상이 되는데, 안을 살펴보면 @Component 어노테이션이 붙어 있는 것을 알 수 있다. 

## 의존관계 주입 방법

- 의존관계 주입에는 크게 3가지가 있다(일반 메서드 주입까지 4가지이나 자주 사용되지 않기에 제외). 생성자 주입, 수정자 주입, 그리고 필드 주입이다.

### 필드 주입을 권장하지 않는 이유

1. DI 컨테이너 의존성으로 인해 테스트가 어려움
@Autowired를 통해 필드 주입을 하면 스프링을 통해서만 의존성 주입이 가능하게 된다. 즉 스프링(혹은 DI 컨테이너)에 의존적이라는 소리다. 이렇게 하면 스프링 빈들이 스프링의 DI 컨테이너에 강한 결합을 하게 된다. DI 컨테이너 없이도 단위테스트에서 (직접 의존성 주입을 하는 식으로) 인스턴스화 시킬 수 있어야 하는데 필드 주입의 경우 단위 테스트를 위해 스프링 프레임워크를 직접 띄워야 하는 경우가 생기는 것이다. 

2. 의존성 감춤
생성자나 수정자를 통한 의존성 주입의 경우 public으로 필요한 의존성에 대한 정보를 제공하게 된다. 하지만 필드 주입의 경우 의존성이 눈에 보이지 않는다.

3. 순환 참조
순환 참조란 클래스 A가 클래스 B를 참조하는데 클래스 B가 클래스 A를 다시 참조하는 경우 혹은 2개 이상의 클래스들이 다른 클래스들을 참조하며 순환이 발생하는 것을 말한다.

```java
class A {
	@Autowired
	private B b;

	public void aMethod() {
		b.bMethod();
	}
}

class B {
	@Autowired
	private A a;

	public void bMethod() {
		a.aMethod();
	}
}
```
이 경우 객체 생성 시점에는 문제가 보이지 않는다. 빈으로는 문제 없이 등록되는 것이다. 
하지만 실제 실행시점에 StackOverflowError가 발생한다. 서로 참조하는 문제 때문이다. 
(setter 주입의 경우도 마찬가지이다. 객체 생성시점에 순환참조가 발생하고 있는 걸 알 수 없기 때문에 setter 호출 시에 알게 되는 것이다)

생성자 주입의 경우 순환 참조가 발생했을 시에는 BeanCurrentlyCreationExeption을 발생시킴으로써 객체 생성 시점에 미리 알 수 있다.

4. 불변하게 선언이 불가
필드 주입을 하려면 final로 선언할 수 없다. 생성자 주입도 마찬가지이다. 그렇기에 객체 자체가 state safe하지 않다. 

```java
class A {
	@Autowired
	private SomeService service;

	public void aMethod() {
		service.doSomething();
		service = new SomeService();
	}
}
```

위와 같은 식의 코드가 작성될 가능성이 있다. 생성자 주입을 사용하여 final로 선언한다면 위와 같은 식으로 새 객체를 할당할 수 없게 된다.

### 생성자 주입을 사용해야 하는 이유

위의 설명에서 어느정도 생성자 주입을 사용해야 하는 이유가 나왔지만 다시 한 번 정리해 보자면 아래와 같다.

1. 불변
대부분의 의존관계 주입은 한 번 일어나면 애플리케이션 종료 전까지 의존관계를 변경할 일이 거의 없다. 즉 객체를 불변으로 설계하는 것이 유리하다는 뜻이다. 하지만 수정자 주입의 경우 setter 메서드를 퍼블릭으로 열어두고 의존성을 주입해야 한다. 즉 바로 위 코드예제 처럼 누군가 실수로 변경을 해버릴 수 있다는 것이며 **변경하면 안되는 메서드를 퍼블릭으로 열어두는 것은 좋은 설계가 아니기도 하다.**

2. 누락을 방지
수정자 주입의 코드를 테스트한다고 해보자.

```java
public class OrderService {
    private MemberRepository memberRepository;

    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    public Order createOrder(Long memberId, String item, int itemPrice) {
        // 코드 생략
    }
}
```

```java
class OrderServiceTest {
    @Test
		void createOrder() {
			OrderServiceImpl orderService = new OrderServiceImpl();
			orderService.createOrder(1L, "itemA", 10000);
		}
}
```
위의 테스트 코드는 작성 시에는 문제가 없다. 하지만 실행하게 되면 NPE가 뜨게 된다. 수정자 주입을 통한 의존관계 주입이 누락되어서이다.
반면 생성자 주입을 사용하게 되면 주입할 데이터가 누락시 바로 컴파일 오류가 발생하게 될 것이다.
`OrderServiceImpl orderService = new OrderServiceImpl(); -> 컴파일 에러 발생`

즉 사전에 미리 누락된 의존성을 캐치할 수 있다. 
수정자 주입 사용시 테스트 길이가 길어지고, 양도 많아지게 되면 분명 휴먼에러가 발생할 수 밖에 없을 것이다. 

3. final 키워드 사용 가능
위의 두가지 장점과 같은 맥락에서, 생성자 주입은 final 키워드를 사용할 수 있기 때문에 불변성을 갖고, 누락을 방지할 수 있게 된다. 
final 키워드가 붙어 있게 되면 값이 누락되는 오류를 미리 컴파일 시점에 막을 수 있고 수정자 주입 등과 같이 나중에 값을 따로 할당하여 불변성을 깨는 코드를 작성할 수 없게 된다.