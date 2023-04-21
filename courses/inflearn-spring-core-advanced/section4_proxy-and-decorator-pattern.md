# 예제 프로젝트 만들기 

예제는 크게 3가지 상황으로 만든다.
v1 - 인터페이스와 구현 클래스 - 스프링 빈으로 수동 등록 
v2 - 인터페이스 없는 구체 클래스 - 스프링 빈으로 수동 등록 
v3 - 컴포넌트 스캔으로 스프링 빈 자동 등록

# 프록시, 프록시 패턴, 데코레이터 패턴 - 소개

- 클라이언트와 서버의 기본 개념 정의
	- 클라이언트는 서버에 필요한 것을 요청하고, 서버는 클라이언트의 요청을 처리

- 클라이언트와 서버 개념에서 일반적으로 클라가 서버를 직접 호출하고 처리 결과를 직접 받는다. 이것을 직접 호출이라 한다.
- 클라가 요청한 결과를 서버에 직접 요청하는 것이 아니라 어떤 대리자를 통해 대신 간접적으로 서버에 요청할 수 있다. 이러한 대리자를 프록시라 한다.
- 객체에서 프록시가 되려면 클라는 서버에게 요청을 한 것인지, 프록시에게 요청을 한 것인지조차 몰라야 한다.
- 즉 서버와 프록시는 같은 인터페이스를 사용해야 한다. 
- 그리고 클라가 사용하는 서버 객체를 프록시 객체로 변경해도 클라 코드를 변경하지 않고 동작할 수 있어야 한다.

![](/assets/proxy1.png)

- 클래스 의존관계를 보면 클라는 서버 인터페이스에만 의존한다. 그리고 서버와 프록시가 같은 인터페이스를 사용한다. 따라서 DI를 사용해서 대체 가능하다.

![](/assets/proxy2.png)
![](/assets/proxy3.png)

- 런타임 객체 의존 관계를 살펴보자
- 런타임에 클라 객체에 DI를 사용해서 클라 -> 서버 에서 클라 -> 프록시로 객체 의존관계를 변경해도 클라 코드를 전혀 변경하지 않아도 된다. 클라 입장에서는 변경 사실 조차 모른다.
- DI를 사용하면 클라 코드의 변경없이 유연하게 프록시를 주입할 수 있다.

### 프록시의 주요 기능
- 접근제어
	- 권한에 따른 접근 차단
	- 캐싱
	- 지연 로딩
- 부가기능 추가
	- 원래 서버가 제공하는 기능에 더해서 부가 기능 수행
	- 예) 요청 값이나 응답 값을 중간에 변경
	- 예) 실행 시간을 측정해서 추가 로그를 남김

### GOF 디자인 패턴
- 둘 다 프록시를 사용하는 방법이지만 GOF 디자인 패턴에서는 이 둘을 의도에 따라서 프록시 패턴과 데코레이터 패턴으로 구분한다.
- 프록시 패턴: 접근 제어가 목적
- 데코레이터 패턴: 새로운 기능 추가가 목적

- 둘 다 프록시를 사용하지만 의도가 다르다는 점이 핵심이다. 
- 용어가 프록시 패턴이라고 해서 이 패턴만 프록시를 사용하는 것은 아니다. 데코레이터 패턴도 프록시를 사용한다.

> 참고: 프록시라는 개념은 클라이언트-서버 라는 큰 개념안에서 자연스럽게 발생할 수 있다. 프록시는 객체 안에서의 개념도 있고 웹 서버에서의 프록시도 있다. 객체 안에서 객체로 구현되어 있는가, 웹서버로 구현되어 있는가 처럼 규모의 차이가 있을 뿐 **근본적인 역할은 같다.**


## 프록시 패턴 - 예제코드 작성
![](/assets/proxy4.png)

- 프록시 패턴 적용 전 코드를 간단하게 작성해보자

```java
public interface Subject {
      String operation();
}
```

```java
public class RealSubject implements Subject {
	@Override
	public String operation() {
		log.info("실제 객체 호출"); sleep(1000);
		return "data";
	}

	private void sleep(int millis) {
		try {
			Thread.sleep(millis);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	} 
}
```

```java
public class ProxyPatternClient {

	private Subject subject;

	public ProxyPatternClient(Subject subject) {
		this.subject = subject;
	}

	public void execute() {
		subject.operation();
	}
}
```

테스트 코드

```java
public class ProxyPatternTest {
	@Test
	void noProxyTest() {
	    RealSubject realSubject = new RealSubject();
	    ProxyPatternClient client = new ProxyPatternClient(realSubject);
	    client.execute();
	    client.execute();
	    client.execute();
	} 
}
```

- 테스트 코드에서는 client.execute() 를 3번 호출한다. 데이터를 조회하는데 1초가 소모되므로 총 3 초의 시간이 걸린다.

- 그런데 이 데이터가 한 번 조회하면 변하지 않는 데이터라면 어딘가 보관해두고 이미 조회한 데이터를 사용하는 것이 성능상 좋다. 이를 캐싱이라한다.
- 프록시 패턴의 주요 기능은 접근 제어이다. 캐시도 접근 자체를 제어하는 기능 중 하나이다.
- 이미 개발된 로직을 전혀 수정하지 않고 프록시 객체 통해 캐시를 적용해보자

## 프록시패턴 - 예제코드2

![](/assets/proxy5.png)

```java
@Slf4j
public class CacheProxy implements Subject {

	private Subject target;
	private String cacheValue;

	public CacheProxy(Subject target) {
		this.target = target;
	}

	@Override
	public String operation() {
		log.info("프록시 호출");
		if (cacheValue == null) {
			cacheValue = target.operation();
		}
		return cacheValue;
	}
}
```

- 앞서 설명한 것 처럼 프록시도 실제 객체와 그 모양이 같아야 하기 때문에 Subject 인터페이스를 구현해야 함
- `private Subject target` : 클라이언트가 프록시를 호출하면 프록시가 최종적으로 실제 객체를 호출해야 한다. 따라서 내부에 실제 객체의 참조를 가지고 있어야 한다. 이렇게 프록시가 호출하는 대상을 target 이라 한다.

```java
public class ProxyPatternTest {
	@Test
	void noProxyTest() {
		RealSubject realSubject = new RealSubject();
		ProxyPatternClient client = new ProxyPatternClient(realSubject);
		client.execute();
		client.execute();
		client.execute();
	}

	@Test
	void cacheProxyTest() {
		Subject realSubject = new RealSubject();
		Subject cacheProxy = new CacheProxy(realSubject);
		ProxyPatternClient client = new ProxyPatternClient(cacheProxy);
		client.execute();
		client.execute();
		client.execute();
	} 
}
```

**cacheProxyTest()**
- realSubject와 cacheProxy를 생성하고 둘을 연결한다
- 결과적으로 cacheProxy가 realSubject를 참조하는 런타임 객체 의존관계가 완성된다.
- 그리고 마지막으로 client에 realSubject가 아닌 cacheProxy를 주입한다. 
- 이 과정 통해 `client -> cacheProxy -> realSubject` 런타임 객체 의존 관계가 완성

**실행결과**

```
CacheProxy - 프록시 호출 
RealSubject - 실제 객체 호출 
CacheProxy - 프록시 호출 (즉시반환 0초)
CacheProxy - 프록시 호출 (즉시반환 0초)
```

- 결과적으로 캐시 프록시를 도입하기 전에는 3초가 걸렸지만 캐시 프록시 도입 이후에는 최초 한 번만 1초가 걸리고 이후에는 즉시 반환한다.

### 정리
- 프록시 패턴 핵심은 realSubject 코드와 클라이언트 코드를 전혀 변경하지 않고 프록시를 도입해서 접근 제어를 했다는 점
- 그리고 클라 코드의 변경 없이 자유롭게 프록시를 넣고 뺄 수 있다. 
- 실제 클라 입장에서는 프록시 객체가 주입되었는지 실제 객체가 주입되었는지 알지 못한다.


## 데코레이터 패턴 - 예제 코드1

- 데코레이터 패턴 도입 전

![](/assets/decorator1.png)

```java
public interface Component {
      String operation();
}


@Slf4j
public class RealComponent implements Component {
	@Override
	public String operation() {
		log.info("RealComponent 실행");
		return "data";
	}
}

@Slf4j
public class DecoratorPatternClient {
	private Component component;
	public DecoratorPatternClient(Component component) {
		this.component = component;
	}

	public void execute() {
		String result = component.operation();
		log.info("result={}", result);
	} 
}

@Slf4j
public class DecoratorPatternTest {
	@Test
	void noDecorator() {
		Component realComponent = new RealComponent();
		DecoratorPatternClient client = new DecoratorPatternClient(realComponent);
		client.execute();
	}
}
```

- 테스트 코드는 client -> realComponent 의 의존관계를 설정하고, client.execute() 를 호출

## 데코레이터 패턴 - 예제코드2

### 부가 기능 추가
- 프록시 통해서 할 수 있는 기능은 접근 제어 + 부가기능 추가
- 프록시로 부가 기능 추가하는 것을 데코레이터 패턴이라 한다.
- 데코레이터 패턴: 원래 서버가 제공하는 기능에 더해 부가 기능 수행

### 응답 값을 꾸며주는 데코레이터

![](/assets/decorator2.png)

```java
@Slf4j
public class MessageDecorator implements Component {
	
	private Component component;

	public MessageDecorator(Component component) {
		this.component = component;
	}
	
	@Override
	public String operation() {
		log.info("MessageDecorator 실행");
		String result = component.operation();
		String decoResult = "*****" + result + "*****"; log.info("MessageDecorator 꾸미기 적용 전={}, 적용 후={}", result, decoResult);
		
		return decoResult;
	}
}

```

(test 및 실행결과 생략)

## 데코레이터 패턴 - 예제 코드 3

- 실행 시간 측정하는 데코레이터 추가해보자
![](/assets/decorator3.png)

```java
@Slf4j
public class TimeDecorator implements Component {
	private Component component;

	public TimeDecorator(Component component) {
		this.component = component;
	}

	@Override
	public String operation() {
		log.info("TimeDecorator 실행");
		long startTime = System.currentTimeMillis();
		String result = component.operation();
		long endTime = System.currentTimeMillis();
		long resultTime = endTime - startTime; log.info("TimeDecorator 종료 resultTime={}ms", resultTime); return result;
	} 
}
```

테스트코드

```java
@Test
void decorator2() {
    Component realComponent = new RealComponent();
    Component messageDecorator = new MessageDecorator(realComponent);
    Component timeDecorator = new TimeDecorator(messageDecorator);
    DecoratorPatternClient client = new DecoratorPatternClient(timeDecorator);
    client.execute();
}
```
- client -> timeDecorator -> messageDecorator -> realComponent 의 객체 의존관계를 설정하고, 실행한다.

**실행결과**

```
TimeDecorator 실행
MessageDecorator 실행
RealComponent 실행
MessageDecorator 꾸미기 적용 전=data, 적용 후=*****data***** TimeDecorator 종료 resultTime=7ms
result=*****data*****
```

- 실행결과 보면 TimeDecorator가 MessageDecorator를 실행하고 실행 시간 측정해서 출력한 것을 확인할 수 있다.

## 프록시 패턴과 데코레이터 패턴 정리

### GOF 데코레이터 패턴

![](/assets/decorator4.png)

- 생각해보면 Decorator 기능에 일부 중복이 있다. 꾸며주는 역할을 하는 Decorator들은 스스로 존재할 수 없다. 
- 항상 꾸며줄 대상이 있어야 한다. 따라서 내부에 호출 대상인 컴포넌트를 가지고 있어야 한다. 그리고 컴포넌트를 항상 호출해야 한다. 이 부분이 중복.
- 이런 중복 제거하기 위해 컴포넌트를 속성으로 가지고 있는 Decorator 라는 추상 클래스를 만드는 방법도 고민할 수 있다. 
- 이렇게 하면 추가로 클래스 다이어그램에서 어떤 것이 실제 컴포넌트인지, 데코레이터인지 명확하게 구분할 수 있다. 
- 여기까지 고민한 것이 바로 GOF에서 설명하는 데코레이터 패턴 기본 예제이다.

### 프록시 패턴 vs 데코레이터 패턴
- Decorator 라는 추상클래스를 만들어야 데코레이터 패턴일까?
- 프록시 패턴과 데코레이터 패턴은 그 모양이 거의 비슷한 거 같은데?

**의도**
- 사실 프록시 패턴과 데코레이터 패턴은 그 모양이 거의 같고 상황에 따라 정말 똑같을 때도 있다. 어떻게 구분하는 것일까
- 디자인 패턴에서 중요한 것은 해당 패턴의 겉모양이 아니라 그 패턴을 만든 의도가 더 중요. 따라서 의도에 따라 패턴 구분

- 프록시 패턴 의도: 다른 개체에 대한 접근 제어
- 데코레이터 패턴 의도: **객체에 추가 책임을 동적으로 추가하고** 기능 확장을 위한 유연한 대안 제공


# 인터페이스 기반 프록시 - 적용 
- 인터페이스와 구현체가 있는 V1 App에 지금까지 학습한 프록시를 도입해서 LogTrace 를 사용해보자. 프록시를 사용하면 기존 코드를 전혀 수정하지 않고 로그 추적 기능을 도입할 수 있다.

![](/assets/proxy6.png)

위 형태의 구조에 로그 추적용 프록시를 추가하는 과정이다. 이 부분은 생략.

```java
@Configuration
  public class InterfaceProxyConfig {
      @Bean
      public OrderControllerV1 orderController(LogTrace logTrace) {
          OrderControllerV1Impl controllerImpl = new
  OrderControllerV1Impl(orderService(logTrace));
          return new OrderControllerInterfaceProxy(controllerImpl, logTrace);
      }
      @Bean
      public OrderServiceV1 orderService(LogTrace logTrace) {
          OrderServiceV1Impl serviceImpl = new
  OrderServiceV1Impl(orderRepository(logTrace));
          return new OrderServiceInterfaceProxy(serviceImpl, logTrace);
      }
      @Bean
      public OrderRepositoryV1 orderRepository(LogTrace logTrace) {
          OrderRepositoryV1Impl repositoryImpl = new OrderRepositoryV1Impl();
          return new OrderRepositoryInterfaceProxy(repositoryImpl, logTrace);
      }
}
```

### V1 프록시 런타임 객체 의존 관계 설정

- 프록시 세팅이 끝났다. 이제 프록시 런타임 객체 의존 관계를 설정하면 된다. 
- 기존에는 스프링 빈이 orderControlerV1Impl, OrderServiceV1Impl같은 실제 객체를 반환했다. 
- 이제는 프록시를 사용해야 하므로 프록시를 생성하여 프록시를 실제 스프링 빈 대신 등록한다. 실제 객체는 스프링 빈으로 등록하지 않는다.
- 프록시는 내부에 실제 객체를 참조하고 있다. 예를 들어 OrderServiceInterfaceProxy는 내부에 실제 대상 객체인 OrderServiceV1Impl을 가지고 있다. 
- `proxy -> target`
- `orderServiceInterfaceProxy -> orderServiceV1Impl`
- 스프링 빈으로 실제 객체 대신 프록시 객체를 등록했기 때문에 앞으로 스프링 빈을 주입 받으면 실제 객체 대신 프록시 객체가 주입된다.

![](/assets/proxy7.png)

![](/assets/proxy8.png)

- 스프링 컨테이너에 프록시 객체가 등록된다. 스프링 컨테이너는 이제 실제 객체가 아니라 프록시 객체를 스프링 빈으로 관리한다.
- 이제 실제 객체는 스프링 컨테이너와는 상관이 없다. 실제 객체는 프록시 객체를 통해 참조될 뿐이다.
- 프록시 객체는 스프링 컨테이너가 관리하고 자바 힙 메모리에도 올라간다. 반면 실제 객체는 자바 힙 메모리에는 올라가지만 스프링 컨테이너가 관리하지는 않는다.

# 구체 클래스 기반 프록시 - 예제1

- 프록시 도입 전의 기본 코드

```java
@Slf4j
  public class ConcreteLogic {
  	 public String operation() { log.info("ConcreteLogic 실행");
          return "data";
      }
}
```

![](/assets/proxy9.png)

```java
public class ConcreteClient {
      private ConcreteLogic concreteLogic;
      
      public ConcreteClient(ConcreteLogic concreteLogic) {
          this.concreteLogic = concreteLogic;
			}

      public void execute() {
          concreteLogic.operation();
			} 
}
```

# 구체 클래스 기반 프록시 - 예제2
### 클래스 기반 프록시 도입
- 지금까지 인터페이스 기반으로 프록시를 도입했다.
- 그런데 자바 다형성은 인터페이스를 구현하든 아니면 클래스를 상속하든 상위 타입만 맞으면 다형성이 적용된다.
- 즉 인터페이스 없어도 프록시를 만들 수 있다.
- 이번에는 인터페이스가 아니라 클래스를 기반으로 상속을 받아서 프록시를 만들어보겠다.

![](/assets/proxy10.png)

```java
@Slf4j
public class TimeProxy extends ConcreteLogic {
	
	private ConcreteLogic realLogic;

	public TimeProxy(ConcreteLogic realLogic) {
		this.realLogic = realLogic;
	}

	@Override
	public String operation() {
		log.info("TimeDecorator 실행");
		long startTime = System.currentTimeMillis();
		String result = realLogic.operation();
		long endTime = System.currentTimeMillis();
		long resultTime = endTime - startTime; log.info("TimeDecorator 종료 resultTime={}", resultTime); return result;
	} 
}
```

- TimeProxy 프록시는 시간을 측정하는 부가 기능을 제공한다. 그리고 인터페이스가 아니라 클래스인 ConcreteLogic 를 상속 받아서 만든다.

```java
@Test
  void addProxy() {
      ConcreteLogic concreteLogic = new ConcreteLogic();
      TimeProxy timeProxy = new TimeProxy(concreteLogic);
      ConcreteClient client = new ConcreteClient(timeProxy);
      client.execute();
}
```
- 여기서 핵심은 ConcreteClient 의 생성자에 concreteLogic 이 아니라 timeProxy 를 주입하는 부분이다. 다형성 적용됨

# 구체 클래스 기반 프록시 - 적용
- 실제 코드에 적용할 시간. 이 부분은 생략하겠다.

### 클래스 기반 프록시의 단점
- super(null): 자바 기본 문법에 의해 자식 클래스를 생성할 때에는 항상 super()로 부모 클래스의 생성자를 호출해야 한다. 이 부분을 생략하면 기본 생성자가 호출된다.
- 그런데 부모 클래스인 OrderServiceV2는 기본 생성자가 없고 생성자에서 파라미터 1개를 필수로 받는다. 따라서 파라미터를 넣어서 super(..)를 호출해야 한다.
- 프록시는 부모 객체의 기능을 사용하지 않기 때문에 super(null)을 입력해야 된다. 
- 인터페이스 기반 프록시는 이런 고민을 하지 않아도 된다.


# 인터페이스 기반 프록시와 클래스 기반 프록시
### 프록시
- 프록시를 사용한 덕분에 원본 코드를 전혀 변경하지 안고 v, v2 애플리케이션에 LogTrace 기능을 적용할 수 있었다.

### 인터페이스 기반 프록시 vs 클래스 기반 프록시
- 인터페이스가 없어도 클래스 기반으로 프록시를 생성할 수 있다.
- 클래스 기반 프록시는 해당 클래스에만 적용할 수 있다. 
- 인터페이스 기반 프록시는 인터페이스만 같으면 모든 곳에 적용할 수 있다.
- 클래스 기반 프록시는 상속을 사용하기 때문에 몇가지 제약이 있다
	- 부모 클래스의 생성자를 호출해야 한다
	- 클래스에 final 키워드가 붙으면 상속이 불가능
	- 메서드에 final 키워드가 붙으면 해당 메서드를 오버라이딩할 수 없다

- 이렇게 보면 인터페이스 기반 프록시가 더 좋아보인다. 맞다. 인터페이스 기반 프록시는 상속이라는 제약에서 자유롭다.
- 프로그래밍 관점에서도 인터페이스를 사용하는 것이 역할과 구현을 명확히 나누기 때문에 더 좋다

- 인터페이스 기반 프록시 단점은 인터페이스가 필요하다는 그 자체이다. 인터페이스가 없으면 인터페이스 기반 프록시를 만들 수 없다.

- 이론적으로는 모든 객체에 인터페이스를 도입해서 역할과 구현을 나누는 것이 좋다. 이렇게 하면 역할과 구현을 나누어서 구현체를 매우 편리하게 변경할 수 있다.
- 하지만 실제로는 구현을 거의 변경할 일이 없는 클래스도 많다.

**결론**
- 실무에서는 프록시를 적용할 때 인터페이스 기반 클래스도 있고 구체 클래스 기반도 있다. 따라서 2가지 상황 모두 대응할 수 있어야 한다.

**너무 많은 프록시 클래스**
- 위의 예제 코드들 보면 프록시 클래스가 너무 많다.
- 프록시 클래스가 하는 일은 LogTrace를 사용하는 것인데 그 로직이 모두 똑같다. 대상 클래스만 다를 뿐이다.
- 만약 적용해야 하는 대상 클래스가 100개라면 프록시 클래스도 100개를 만들어야 한다.
- 프록시 클래스를 하나만 만들어서 모든 곳에 적용하는 방법은 없을까?
- 동적 프록시 기술이 바로 그것이다.









































