# DI와 IoC
```java
public class MemberService {
	//  private MemberRepository memberRepository = new MemoryMemberRepository();
	private MemberRepository memberRepository = new JdbcMemberRepository();
}
```

- 문제가 없어 보일수도 있으나 위 코드는 OCP, DIP 모두를 지키지 못한다. 
- OCP: 위 주석처리에서도 볼 수 있듯이 repository 구현체가 변경되면 클라이언트인 MemberService 코드에 변경이 일어난다.
- DIP: MemberService 클래스는 memberRepository 인터페이스 뿐만 아니라 JdbcMemberRepository 구현체에도 의존한다.
- 즉 다형성 만으로는 OCP, DIP를 지킬 수 없는 것이다. 가장 근본적인 문제는 MemberService에서 어떤 repository를 사용할지를 결정하여 생성하고 연결하는 관심사를 떠앉고 있다는 점이다. 

## 관심사 분리를 통한 해결

```java
public class MemberService {
	private MemberRepository memberRepository;

	public MemberService(MemberRepository memberRepository) {
		this.memberRepository = memberRepository;
	}	
}
```

- memberRepository는 외부에서 주입받게끔 하고 MemberService 본연의 책임에만 집중할 수 있도록 한다.
- 이제 memberRepository 역할에 맞는 구현체를 생성하고 지정하는 책임을 지는 config용 클래스를 만들자.

```java
public class AppConfig {
	public MemberService memberService() {
  	return new MemberServiceImpl(memberRepository());
	}

	public MemberRepository memberRepository() {
		return new MemoryMemberRepository();
	}	
}
```

**AppConfig 활용**
```java
public class MemberApp {
	public static void main(String[] args) {
		AppConfig appConfig = new AppConfig();
		MemberService memberService = appConfig.memberService();
		Member member = new Member(1L, "memberA", Grade.VIP);
		memberService.join(member);
	} 
}
```

위의 과정을 통해 SRP(관심사 분리), DIP, OCP 등을 지킬 수 있게 되었다.

## IoC, DI, 컨테이너

- AppConfig 처럼 객체를 외부에서 생성하고 관리하며, 의존관계를 연결해주는 것을 IoC 컨테이너 혹은 DI 컨테이너라 한다. 
- 또한 기존에 MemberService 구현객체가 직접 필요한 구현체, 예를 들면 MemoryMemberRepository 과 같은 객체를 생성하고 연결했다면 이러한 제어 흐름을 컨테이너 혹은 프레임워크에 넘겨주는 것을 제어의 역전, 즉 IoC라 부른다.
- 그리고 위에서 본 의존성 주입, 즉 DI가 이러한 IoC를 구현하기 위한 하나의 패턴인 것이다.

- 이렇게 의존성 주입을 통해 
1. 서로 강하게 결합되었던 클래스들의 결합도를 낮출 수 있고
2. 객체의 유연성을 높일 수 있으며
3. 무엇보다 테스트 작성이 용이해 질 수 있다.

### 스프링 컨테이너
AppConfig를 Spring에서 관리하게끔 변경해보자.

```java
@Configuration
public class AppConfig {
	@Bean
	public MemberService memberService() {
		return new MemberServiceImpl(memberRepository());
	}

	@Bean
	public MemberRepository memberRepository() {
		return new MemoryMemberRepository();
	}
}
```

```java
public class OrderApp {
	public static void main(String[] args) {
		// AppConfig appConfig = new AppConfig();
		// MemberService memberService = appConfig.memberService();
		// OrderService orderService = appConfig.orderService();

		ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
		
		MemberService memberService = applicationContext.getBean("memberService", MemberService.class);

		long memberId = 1L;
		Member member = new Member(memberId, "memberA", Grade.VIP);
		memberService.join(member);
	}
}
```

- 위의 ApplicationContext를 스프링 컨테이너라 부르며 이 컨테이너가 스프링에서의 IoC 컨테이너이다.
- 이 스프링 컨테이너는 빈이라 불리우는 객체를 초기화하고 관리하며 의존관계를 조립하는 역할을 한다. 더불어 빈의 라이프 사이클도 관리한다.

> Bean이란? 스프링 빈은 스프링 IoC 컨테이너에 의해 관리되는 객체를 말한다. 스프링 IoC 컨테이너에 의해 초기화 되고, 조립되고, 관리되는 객체이다.

- ApplicationContext에는 직접 객체를 생성하고 의존성 연결을 해주는 코드가 들어있진 않다. 그렇기에 @Configuration이 붙어있는 config용 클래스를 설정 정보로 사용한다. 해당 설정 파일 안에 @Bean이 달린 메서드를 모두 호출 후 반환된 객체를 스프링 컨테이너에 등록하게 된다.
- 그렇다면 직접 AppConfig 설장 파일을 만들어서 사용하는 것과 ApplicationContext로 관리하는 것의 차이는 무엇일까? 물론 ApplicationContext로 관리한다는 것은 스프링 프레임워크에서 관리한다는 뜻이니 여러면에서 이점이 있겠으나 그 중 하나를 꼽자면 바로 스프링 빈이 싱글톤으로 관리된다는 점이다.
