## 웹 애플리케이션과 싱글톤

**싱글톤 방식 주의점**
싱글톤 객체는 반드시 무상태(stateless)로 설계해야 한다. 즉,
1. 특정 클라이언트에 의존적인 필드가 있으면 안되며
2. 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
3. 가급적 읽기만 해야하며
4. 필드 대신에 자바에서 공유되지 않는 지역변수나 파라미터, 쓰레드로컬 등을 사용해야 한다. 


### ApplicationContext에서 어떻게 싱글톤 레지스트리 기능을 제공할 수 있을까
- 사실 ApplicationContext에 AppConfig를 넘기게 되면 AppConfig를 그대로 쓰지 않는다. 
- 일단 ApplicationContext에 넘긴 AppConfig 역시 스프링 빈으로 등록되어야 한다. 이때 AppConfig를 그대로 빈으로 등록하는 것이 아니라 CGLIB이라는 바이트 코드 조작 라이브러리를 사용하여 AppConfig를 상속받은 임의의 클래스를 빈으로 등록하게 된다. 

- 아래와 같은 코드가 동적으로 만들어질 것으로 예상할 수 있다.

**예상코드**
```java
@Bean
public MemberRepository memberRepository() {
	if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) { 
		return 스프링 컨테이너에서 찾아서 반환;
	} else { //스프링 컨테이너에 없으면
		기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록 return 반환
	} 
}
```

- 즉 @Bean으로 수동 등록할 때 내부의 의존관계를 직접 메서드 호출로 주입하는 경우 @Configuration 적용이 안되어있다면 싱글톤이 보장 안된다. @Configuration 어노테이션을 적용했을 때 AppConfig는 CGLIB에 의해 코드 조작이 되어 빈으로 등록되게 된다.
