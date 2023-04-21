# 정리
- 변하는 것과 변하지 않는 것을 분리 
- 핵심 기능과 부가 기능 분리 

## 템플릿 메서드 패턴
- 템플릿패턴은 추상클래스로 만드는데, 변하지 않는 부분은 execute()에 두고 변하는 부분은 추상메서드로, 네이밍은 call()로 해서 둔다
- 템플릿 메서드 패턴은 부모 클래스에 변하지 않는 템플릿 코드를 둔다. 그리고 변하는 부분은 자식 클래스에 두고 `상속과 오버라이딩`을 사용해서 처리한다.
- 템플릿 메서드 패턴은 이렇게 다형성을 사용해서 변하는 부분과 변하지 않는 부분을 분리하는 방법이다.
- V4 는 단순히 템플릿 메서드 패턴을 적용해서 소스코드 몇줄을 줄인 것이 전부가 아니다. 로그를 남기는 부분에 단일 책임 원칙(SRP)을 지킨 것이다. 변경 지점을 하나로 모아서 변경에 쉽게 대처할 수 있는 구조를 만든 것이다.

- 단점
	- 자식 클래스 입장에서는 부모 클래스의 기능을 전혀 사용하지 않는데, 부모 클래스를 알아야한다. 이것은 좋은 설계가 아니다. 그리고 이런 잘못된 의존관계 때문에 부모 클래스를 수정하면, 자식 클래스에도 영향을 줄 수 있다.
	- 추가로 템플릿 메서드 패턴은 상속 구조를 사용하기 때문에, 별도의 클래스나 익명 내부 클래스를 만들어야 하는 부분도 복잡하다.

## 전략 패턴
- 전략 패턴은 변하지 않는 부분을 Context 라는 곳에 두고, 변하는 부분을 Strategy 라는 인터페이스를 만들고 해당 인터페이스를 구현하도록 해서 문제를 해결한다. 상속이 아니라 위임으로 문제를 해결하는 것이다.
- 컨텍스트(문맥)는 크게 변하지 않지만, 그 문맥 속에서 strategy 를 통해 일부 전략이 변경된다 생각하면 된다.
- 바로 스프링에서 의존관계 주입에서 사용하는 방식이 바로 전략 패턴이다.
- 이는 선조립, 후실행 방식이다.
- 필드에 두는게 아니라 파라미터로 전략을 받을 수도 있다. 좀 더 유연한 방법. 아래의 템플릿 콜백 패턴이다.

## 템플릿 콜백 패턴
- ContextV2 는 변하지 않는 템플릿 역할을 한다. 그리고 변하는 부분은 파라미터로 넘어온 Strategy 의 코드를 실행해서 처리한다. 이렇게 다른 코드의 인수로서 넘겨주는 실행 가능한 코드를 콜백(callback)이라 한다.
- 스프링에서는 ContextV2 와 같은 방식의 전략 패턴을 템플릿 콜백 패턴이라 한다. 전략 패턴에서 Context 가 템플릿 역할을 하고, Strategy 부분이 콜백으로 넘어온다 생각하면 된다.
- 스프링에서는 JdbcTemplate , RestTemplate , TransactionTemplate , RedisTemplate 처럼 다양한 템플릿 콜백 패턴이 사용된다. 스프링에서 이름에 XxxTemplate 가 있다면 템플릿 콜백 패턴으로 만들어져 있다 생각하면 된다.

정리끝
시작!!


# 템플릿 메서드 패턴 - 시작

## 핵심 기능 vs 부가 기능
- 핵심 기능은 해당 객체가 제공하는 고유의 기능
- 부가 기능은 핵심 기능을 보조하기 위해 제공되는 기능이다. 예를 들어서 로그 추적 로직, 트랜잭션 기능이 있다. 이러한 부가 기능은 단독으로 사용되지는 않고, 핵심 기능과 함께 사용된다. 예를 들어서 로그 추적 기능은 어떤 핵심 기능이 호출되었는지 로그를 남기기 위해 사용한다. 그러니까 핵심 기능을 보조하기 위해 존재한다.

## 변하는 것과 변하지 않는 것을 분리
- 좋은 설계는 변하는 것과 변하지 않는 것을 분리하는 것이다.
- 여기서 핵심 기능 부분은 변하고, 로그 추적기를 사용하는 부분은 변하지 않는 부분이다. 이 둘을 분리해서 모듈화해야 한다.
- 템플릿 메서드 패턴(Template Method Pattern)은 이런 문제를 해결하는 디자인 패턴이다.

## 템플릿 메서드 패턴 - 예제1
```java
@Slf4j
public class TemplateMethodTest {
	@Test
	void templateMethodV0() {
		logic1();
		logic2(); 
	}

	private void logic1() {
		long startTime = System.currentTimeMillis(); 

		//비즈니스 로직 실행
		log.info("비즈니스 로직1 실행");
		//비즈니스 로직 종료

		long endTime = System.currentTimeMillis(); 
		long resultTime = endTime - startTime; 

		log.info("resultTime={}", resultTime);
	}
	
	private void logic2() {
		long startTime = System.currentTimeMillis(); 
		//비즈니스 로직 실행
		log.info("비즈니스 로직2 실행");
		//비즈니스 로직 종료

		long endTime = System.currentTimeMillis();
		long resultTime = endTime - startTime;
		log.info("resultTime={}", resultTime);
	} 
}      
```
- 변하는 부분: 비즈니스 로직
- 변하지 않는 부분: 시간 측정

## 템플릿 메서드 패턴 - 예제2
### 템플릿 메서드 패턴 구조 그림

![](/assets/template-pattern.png)

### AbstractTemplate
```java
@Slf4j
public abstract class AbstractTemplate {
	public void execute() {
		long startTime = System.currentTimeMillis(); 

		//비즈니스 로직 실행
		call(); //상속
		//비즈니스 로직 종료

		long endTime = System.currentTimeMillis(); 
		long resultTime = endTime - startTime; 
		log.info("resultTime={}", resultTime);
	}

	protected abstract void call();
}
```
- **이 부분이 중요한 부분이다. 템플릿패턴은 이렇게 추상클래스로 만드는데, 변하지 않는 부분은 execute()에 두고 변하는 부분은 추상메서드로, 네이밍은 call()로 해서 둔다.**

- 템플릿 메서드 패턴은 이름 그대로 템플릿을 사용하는 방식이다. 템플릿은 기준이 되는 거대한 틀이다. 템플릿이라는 틀에 변하지 않는 부분을 몰아둔다. 그리고 일부 변하는 부분을 별도로 호출해서 해결한다.
- **템플릿 메서드 패턴은 부모 클래스에 변하지 않는 템플릿 코드를 둔다. 그리고 변하는 부분은 자식 클래스에 두고 `상속과 오버라이딩`을 사용해서 처리한다.**



**SubClassLogic1**
```java
@Slf4j
public class SubClassLogic1 extends AbstractTemplate {
	@Override
	protected void call() {
		log.info("비즈니스 로직1 실행"); 
	}
}
```
- 변하는 부분인 비즈니스 로직1을 처리하는 자식 클래스이다. 템플릿이 호출하는 대상인 call() 메서드를 오버라이딩 한다.

**SubClassLogic2**
```java
@Slf4j
public class SubClassLogic2 extends AbstractTemplate {
	@Override
	protected void call() {
		log.info("비즈니스 로직2 실행"); 
	}
}
```
- 변하는 부분인 비즈니스 로직2를 처리하는 자식 클래스이다. 템플릿이 호출하는 대상인 call() 메서드를 오버라이딩 한다.

**TemplateMethodTest - templateMethodV1() 추가**
```java
/**
* 템플릿 메서드 패턴 적용
*/
@Test
void templateMethodV1() {
    AbstractTemplate template1 = new SubClassLogic1();
    template1.execute();

    AbstractTemplate template2 = new SubClassLogic2();
    template2.execute();
}
```

- template1.execute() 를 호출하면 템플릿 로직인 AbstractTemplate.execute() 를 실행한다. 여기서 중간에 call() 메서드를 호출하는데, 이 부분이 오버라이딩 되어있다. 따라서 현재 인스턴스인 SubClassLogic1 인스턴스의 SubClassLogic1.call() 메서드가 호출된다.


- 중요!!!! **템플릿 메서드 패턴은 이렇게 다형성을 사용해서 변하는 부분과 변하지 않는 부분을 분리하는 방법이다.**

## 템플릿 메서드 패턴 - 적용1
```java
@RestController
@RequiredArgsConstructor
public class OrderControllerV4 {
	private final OrderServiceV4 orderService;
	private final LogTrace trace;

	@GetMapping("/v4/request")
	public String request(String itemId) {
		AbstractTemplate<String> template = new AbstractTemplate<>(trace) {
			@Override
			protected String call() {
				orderService.orderItem(itemId);
				return "ok";
			} 
		};

		return template.execute("OrderController.request()");
	}
}
```
- `AbstractTemplate<String>`
	- 제네릭을 String 으로 설정했다. 따라서 AbstractTemplate 의 반환 타입은 String 이 된다.
- 익명 내부 클래스
	- 익명 내부 클래스를 사용한다. 객체를 생성하면서 AbstractTemplate 를 상속받은 자식 클래스를 정의했다. 따라서 별도의 자식 클래스를 직접 만들지 않아도 된다.

## 템플릿 메서드 패턴 - 적용2
- 템플릿 메서드 패턴 덕분에 변하는 코드와 변하지 않는 코드를 명확하게 분리했다.
- 로그를 출력하는 템플릿 역할을 하는 변하지 않는 코드는 모두 AbstractTemplate 에 담아두고, 변하는 코드는 자식 클래스를 만들어서 분리했다.

- 바로 위 코드는 핵심 기능과 템플릿을 호출하는 코드가 섞여 있다.
- 그 전의 코드들은 핵심기능과 부가기능이 섞여있는 상태였다.
- V4는 템플릿 메서드 패턴을 사용한 덕분에 핵심 기능에 좀 더 집중할 수 있게 되었다.

### 좋은 설계란?
- 진정한 좋은 설계는 바로 변경이 일어날 때 자연스럽게 드러난다.

### 단일 책임 원칙(SRP)
- **V4 는 단순히 템플릿 메서드 패턴을 적용해서 소스코드 몇줄을 줄인 것이 전부가 아니다. 로그를 남기는 부분에 단일 책임 원칙(SRP)을 지킨 것이다. 변경 지점을 하나로 모아서 변경에 쉽게 대처할 수 있는 구조를 만든 것이다.**


## 템플릿 메서드 패턴 - 정의
- GOF 디자인 패턴에서는 템플릿 메서드 패턴을 다음과 같이 정의했다.

> 템플릿 메서드 디자인 패턴의 목적은 다음과 같습니다. "작업에서 알고리즘의 골격을 정의하고 일부 단계를 하위 클래스로 연기합니다. 템플릿 메서드를 사용하면 하위 클래스가 알고리즘의 구조를 변경하지 않고도 알고리즘의 특정 단계를 재정의할 수 있습니다." [GOF]

- 풀어서 설명하면 다음과 같다. 부모 클래스에 알고리즘의 골격인 템플릿을 정의하고, 일부 변경되는 로직은 자식 클래스에 정의하는 것이다. 이렇게 하면 자식 클래스가 알고리즘의 전체 구조를 변경하지 않고, 특정 부분만 재정의할 수 있다. **결국 상속과 오버라이딩을 통한 다형성으로 문제를 해결하는 것이다.**

### 하지만.. 중요!!!
- 템플릿 메서드 패턴은 상속을 사용한다. 따라서 상속에서 오는 단점들을 그대로 안고간다. 특히 자식 클래스가 부모 클래스와 컴파일 시점에 강하게 결합되는 문제가 있다. 이것은 의존관계에 대한 문제이다.
- 자식 클래스 입장에서는 부모 클래스의 기능을 전혀 사용하지 않는다. 그럼에도 불구하고 템플릿 메서드 패턴을 위해 자식 클래스는 부모 클래스를 상속 받고 있다.
- 상속을 받는 다는 것은 특정 부모 클래스를 의존하고 있다는 것이다. 자식 클래스의 extends 다음에 바로 부모 클래스가 코드상에 지정되어 있다. 따라서 부모 클래스의 기능을 사용하든 사용하지 않든 간에 부모 클래스를 강하게 의존하게 된다. 여기서 강하게 의존한다는 뜻은 자식 클래스의 코드에 부모 클래스의 코드가 명확하게 적혀 있다는 뜻이다
- UML에서 상속을 받으면 삼각형 화살표가 자식 -> 부모 를 향하고 있는 것은 이런 의존관계를 반영하는 것이다.

- **자식 클래스 입장에서는 부모 클래스의 기능을 전혀 사용하지 않는데, 부모 클래스를 알아야한다. 이것은 좋은 설계가 아니다. 그리고 이런 잘못된 의존관계 때문에 부모 클래스를 수정하면, 자식 클래스에도 영향을 줄 수 있다.**
- **추가로 템플릿 메서드 패턴은 상속 구조를 사용하기 때문에, 별도의 클래스나 익명 내부 클래스를 만들어야 하는 부분도 복잡하다.**

- 템플릿 메서드 패턴과 비슷한 역할을 하면서 상속의 단점을 제거할 수 있는 디자인 패턴이 바로 전략 패턴(Strategy Pattern)이다.


# 전략 패턴 - 시작
## 전략 패턴 - 예제1
- 탬플릿 메서드 패턴은 부모 클래스에 변하지 않는 템플릿을 두고, 변하는 부분을 자식 클래스에 두어서 상속을 사용해서 문제를 해결했다.
- **전략 패턴은 변하지 않는 부분을 Context 라는 곳에 두고, 변하는 부분을 Strategy 라는 인터페이스를 만들고 해당 인터페이스를 구현하도록 해서 문제를 해결한다. 상속이 아니라 위임으로 문제를 해결하는 것이다.**
- 전략 패턴에서 Context 는 변하지 않는 템플릿 역할을 하고, Strategy 는 변하는 알고리즘 역할을 한다.
- GOF 디자인 패턴에서 정의한 전략 패턴의 의도는 다음과 같다.
	- 알고리즘 제품군을 정의하고 각각을 캡슐화하여 상호 교환 가능하게 만들자. 전략을 사용하면 알고리즘을 사용하는 클라이언트와 독립적으로 알고리즘을 변경할 수 있다.

![](/assets/strategy.png)

**Strategy 인터페이스**
```java
package hello.advanced.trace.strategy.code.strategy;
  public interface Strategy {
      void call();
}
```
- 이 인터페이스는 변하는 알고리즘 역할을 한다.

**StrategyLogic1**
```java
@Slf4j
public class StrategyLogic1 implements Strategy {
	@Override
	public void call() {
		log.info("비즈니스 로직1 실행"); 
	}
}
```
- 변하는 알고리즘은 Strategy 인터페이스를 구현하면 된다. 여기서는 비즈니스 로직1을 구현했다.

**StrategyLogic2**
```java
@Slf4j
public class StrategyLogic2 implements Strategy {
	@Override
	public void call() {
		log.info("비즈니스 로직2 실행"); 
	}
}
```

**ContextV1**
```java
/**
* 필드에 전략을 보관하는 방식
*/
@Slf4j
public class ContextV1 {
	
	private Strategy strategy;

	public ContextV1(Strategy strategy) {
		this.strategy = strategy;
	}

	public void execute() {
		long startTime = System.currentTimeMillis(); 

		//비즈니스 로직 실행
		strategy.call(); //위임
		//비즈니스 로직 종료
		
		long endTime = System.currentTimeMillis(); 
		long resultTime = endTime - startTime; 
		log.info("resultTime={}", resultTime);
	} 
}
```
- **ContextV1 은 변하지 않는 로직을 가지고 있는 템플릿 역할을 하는 코드이다. 전략 패턴에서는 이것을 컨텍스트(문맥)이라 한다.**
- **쉽게 이야기해서 컨텍스트(문맥)는 크게 변하지 않지만, 그 문맥 속에서 strategy 를 통해 일부 전략이 변경된다 생각하면 된다.**
- Context 는 내부에 Strategy strategy 필드를 가지고 있다. 이 필드에 변하는 부분인 Strategy 의 구현체를 주입하면 된다.
- 전략 패턴의 핵심은 Context 는 Strategy 인터페이스에만 의존한다는 점이다. 덕분에 Strategy 의 구현체를 변경하거나 새로 만들어도 Context 코드에는 영향을 주지 않는다.
- **어디서 많이 본 코드 같지 않은가? 그렇다. 바로 스프링에서 의존관계 주입에서 사용하는 방식이 바로 전략 패턴이다.**

**ContextV1Test - 추가**
```java
/**
*전략 패턴 적용
*/
@Test
void strategyV1() {
	Strategy strategyLogic1 = new StrategyLogic1();
	ContextV1 context1 = new ContextV1(strategyLogic1);
	context1.execute();

	Strategy strategyLogic2 = new StrategyLogic2();
	ContextV1 context2 = new ContextV1(strategyLogic2);
	context2.execute();
}
```
- 전략 패턴을 사용해보자. 코드를 보면 의존관계 주입을 통해 ContextV1 에 Strategy 의 구현체인 strategyLogic1 를 주입하는 것을 확인할 수 있다. 이렇게해서 Context 안에 원하는 전략을 주입한다. 이렇게 원하는 모양으로 조립을 완료하고 난 다음에 context1.execute() 를 호출해서 context 를 실행한다.

**선 조립, 후 실행**
- 여기서 이야기하고 싶은 부분은 Context 의 내부 필드에 Strategy 를 두고 사용하는 부분이다.
- 이 방식은 Context 와 Strategy 를 실행 전에 원하는 모양으로 조립해두고, 그 다음에 Context 를 실행하는 선 조립, 후 실행 방식에서 매우 유용하다.
- Context 와 Strategy 를 한번 조립하고 나면 이후로는 Context 를 실행하기만 하면 된다.

- 그니까 Context에 Strategy를 딱 조립해 두고 사용하는 거잖아.

- **스프링을 애플리케이션 개발할 때 애플리케이션 로딩 시점에 의존관계 주입을 통해 필요한 의존관계를 모두 맺어두고 난 다음에 실제 요청을 처리하는 것과 같은 원리**

- 이 방식의 단점은 Context 와 Strategy 를 조립한 이후에는 전략을 변경하기가 번거롭다는 점
- Context 에 setter 를 제공해서 Strategy 를 넘겨 받아 변경하면 되지만, Context 를 싱글톤으로 사용할 때는 동시성 이슈 등 고려할 점이 많다. 그래서 전략을 실시간으로 변경해야 하면 차라리 이전에 개발한 테스트 코드 처럼 Context 를 하나더 생성하고 그곳에 다른 Strategy 를 주입하는 것이 더 나은 선택일 수 있다.

- 이렇게 먼저 조립하고 사용하는 방식보다 더 유연하게 전략 패턴을 사용하는 방법을 없을까?

## 전략 패턴 - 예제3
- 이번에는 전략을 실행할 때 직접 파라미터로 전달해서 사용해보자.
```java
@Slf4j
public class ContextV2 {
	public void execute(Strategy strategy) {
		long startTime = System.currentTimeMillis(); 

		//비즈니스 로직 실행
		strategy.call(); //위임
		//비즈니스 로직 종료

		long endTime = System.currentTimeMillis(); 
		long resultTime = endTime - startTime; 
		log.info("resultTime={}", resultTime);
	}
}
```

```java
@Test
void strategyV1() {
    ContextV2 context = new ContextV2();
    context.execute(new StrategyLogic1());
    context.execute(new StrategyLogic2());
}
```
- Context 와 Strategy 를 '선 조립 후 실행'하는 방식이 아니라 Context 를 실행할 때 마다 전략을 인수로 전달한다
- 이전 방식보다 원하는 전략을 더욱 유연하게 변경할 수 있다.

### 정리
- 이전 방식은 필드에 Strategy를 저장하는 방식으로 전략 패턴 구사함
	- 선 조립, 후 실행에 적합
	- Context를 실행하는 시점에는 이미 조립이 끝났기에 전략을 신경쓰지 않고 단순히 실행만 하면 된다.

- Context2는 파라미터에 Strategy를 전달받는 방식으로 전략 패턴 구사함
	- 실행할 때 마다 전략을 유연하게 변경 가능
	- 단점 역시 실행할 때 마다 전략을 계속 지정해줘야 한다는 점.


# 템플릿 콜백 패턴 - 시작
- ContextV2 는 변하지 않는 템플릿 역할을 한다. 그리고 변하는 부분은 파라미터로 넘어온 Strategy 의 코드를 실행해서 처리한다. 이렇게 다른 코드의 인수로서 넘겨주는 실행 가능한 코드를 콜백(callback)이라 한다.
- 쉽게 이야기해서 callback 은 코드가 호출( call )은 되는데 코드를 넘겨준 곳의 뒤( back )에서 실행된다는 뜻이다.
- ContextV2 예제에서 콜백은 Strategy 이다.
- 여기에서는 클라이언트에서 직접 Strategy 를 실행하는 것이 아니라, 클라이언트가 ContextV2.execute(..) 를 실행할 때 Strategy 를 넘겨주고, ContextV2 뒤에서 Strategy 가 실행된다

```
<참고>
자바 언어에서 콜백
자바 언어에서 실행 가능한 코드를 인수로 넘기려면 객체가 필요하다. 자바8부터는 람다를 사용할 수 있다. 자바 8 이전에는 보통 하나의 메소드를 가진 인터페이스를 구현하고, 주로 익명 내부 클래스를 사용했다. 최근에는 주로 람다를 사용한다.
```

**템플릿 콜백 패턴**
- **스프링에서는 ContextV2 와 같은 방식의 전략 패턴을 템플릿 콜백 패턴이라 한다.** 전략 패턴에서 Context 가 템플릿 역할을 하고, Strategy 부분이 콜백으로 넘어온다 생각하면 된다.
- 참고로 템플릿 콜백 패턴은 GOF 패턴은 아니고, 스프링 내부에서 이런 방식을 자주 사용하기 때문에, 스프링 안에서만 이렇게 부른다. 전략 패턴에서 템플릿과 콜백 부분이 강조된 패턴이라 생각하면 된다.
- **스프링에서는 JdbcTemplate , RestTemplate , TransactionTemplate , RedisTemplate 처럼 다양한 템플릿 콜백 패턴이 사용된다. 스프링에서 이름에 XxxTemplate 가 있다면 템플릿 콜백 패턴으로 만들어져 있다 생각하면 된다.**


## 정리
- 지금까지 우리는 변하는 코드와 변하지 않는 코드를 분리하고, 더 적은 코드로 로그 추적기를 적용하기 위해 고군분투 했다. 템플릿 메서드 패턴, 전략 패턴, 그리고 템플릿 콜백 패턴까지 진행하면서 변하는 코드와 변하지 않는 코드를 분리했다. 그리고 최종적으로 템플릿 콜백 패턴을 적용하고 콜백으로 람다를 사용해서 코드 사용도 최소화 할 수 있었다.

## 한계
- 그런데 지금까지 설명한 방식의 한계는 아무리 최적화를 해도 결국 로그 추적기를 적용하기 위해서 원본 코드를 수정해야 한다는 점이다. 클래스가 수백개이면 수백개를 더 힘들게 수정하는가 조금 덜 힘들게 수정하는가의 차이가 있을 뿐, 본질적으로 코드를 다 수정해야 하는 것은 마찬가지이다.










