# 필드 동기화 - 개발

```java
public class FieldLogTrace implements LogTrace {
	private static final String START_PREFIX = "-->";
	private static final String COMPLETE_PREFIX = "<--";
	private static final String EX_PREFIX = "<X-";

	private TraceId traceIdHolder; //traceId 동기화, 동시성 이슈 발생

	@Override
	public TraceStatus begin(String message) {
		...
	}

	...(생략)
```

- 기존에 파라미터로 넘기던 TraceId를 필드로 두어 관리하는 코드이다.
- FieldLogTrace를 빈으로 등록 후 사용한다.

- 기존 코드에 FieldLogTrace 적용(빈으로 등록해놔서 굳이 컨트롤러 -> 서비스 안넘겨도 객체가 싱글톤으로 동일한 객체로 동작..?)
```java
@RestController
@RequiredArgsConstructor
public class OrderControllerV3 {
	private final OrderServiceV3 orderService;
	private final LogTrace trace;

	@GetMapping("/v3/request")
	public String request(String itemId) {
		TraceStatus status = null;
		try {
			status = trace.begin("OrderController.request()");
			orderService.orderItem(itemId);
			trace.end(status);
			return "ok";
		} catch (Exception e) { trace.exception(status, e);
			throw e; //예외를 꼭 다시 던져주어야 한다.
		} 
	}
}
```

```java
@Service
@RequiredArgsConstructor
public class OrderServiceV3 {
	private final OrderRepositoryV3 orderRepository;
	private final LogTrace trace;
	
	public void orderItem(String itemId) {
		TraceStatus status = null;
		try {
			status = trace.begin("OrderService.orderItem()");
			orderRepository.save(itemId);
			trace.end(status);
		} catch (Exception e) {
			trace.exception(status, e);
			throw e;
		} 
	}
}
```

# 필드 동기화 - 동시성 문제
- 1초안에 api 여러번 날려보자
- 기대한것과전혀다른문제가발생한다. 트랜잭션ID도동일하고,level도뭔가많이꼬인것같다.분명히 테스트 코드로 작성할 때는 문제가 없었는데, 무엇이 문제일까?

## 동시성 문제
- `FieldLogTrace`는 싱글톤으로 등록된 스프링 빈이다. 이 객체의 인스턴스가 애플리케이션에 딱 1개 존재한다는 뜻
- 이렇게 하나만 있는 인스턴스의 `FieldLogTrace.traceIdHolder` 필드를 여러 쓰레드가 동시에 접근해서 문제 발생

- 동시성 문제는 지역 변수에서는 발생하지 않는다. 지역 변수는 쓰레드마다 각각 다른 메모리 영역이 할당된다.
- 동시성 문제가 발생하는 곳은 같은 인스턴스의 필드(주로 싱글톤에서 자주 발생), 또는 static 같은 공용 필드에 접근할 때 발생한다.
- 동시성 문제는 값을 읽기만 하면 발생하지 않는다. 어디선가 값을 변경하기 때문에 발생한다.

# ThreadLocal - 소개
- 쓰레드 로컬은 해당 쓰레드만 접근할 수 있는 특별한 저장소를 말한다. 쉽게 이야기해서 물건 보관 창구를 떠올리면 된다.
- 여러 사람이 같은 물건 보관 창구를 사용하더라도 창구 직원은 사용자를 인식해서 사용자별로 확실하게 물건을 구분해준다.
- 쓰레드 로컬을 사용하면 각 쓰레드마다 별도의 내부 저장소를 제공한다. 따라서 같은 인스턴스의 쓰레드 로컬 필드에 접근해도 문제 없다.

![](/assets/threadlocal.png)

- 자바는 언어차원에서 쓰레드 로컬을 지원하기 위한 java.lang.ThreadLocal 클래스를 제공한다.
- ThreadLocal 사용법
	- 값저장: ThreadLocal.set(xxx)
	- 값조회: ThreadLocal.get()
	- 값제거: ThreadLocal.remove()

> 주의: 해당 쓰레드가 쓰레드 로컬을 모두 사용하고 나면 ThreadLocal.remove() 를 호출해서 쓰레드 로컬에 저장된 값을 제거해주어야 한다.


# 쓰레드 로컬 - 주의사항
- 쓰레드 로컬의 값을 사용 후 제거하지 않고 그냥 두면 WAS(톰캣)처럼 쓰레드 풀을 사용하는 경우에 심각한 문제가 발생할 수 있다.

1. 사용자A가 저장 HTTP를 요청했다.
2. WAS는 쓰레드 풀에서 쓰레드를 하나 조회한다.
3. 쓰레드 thread-A 가 할당되었다.
4. thread-A 는 사용자A 의 데이터를 쓰레드 로컬에 저장한다.
5. 쓰레드 로컬의 thread-A 전용 보관소에 사용자A 데이터를 보관한다.
6. 사용자A의 HTTP 응답이 끝난다.
7. WAS는 사용이 끝난 thread-A 를 쓰레드 풀에 반환한다. 쓰레드를 생성하는 비용은 비싸기 때문에 쓰레드를 제거하지 않고, 보통 쓰레드 풀을 통해서 쓰레드를 재사용한다.
8. thread-A 는 쓰레드풀에 아직 살아있다. 따라서 쓰레드 로컬의 thread-A 전용 보관소에 사용자A 데이터도 함께 살아있게 된다.
9. 사용자B가 조회를 위한 새로운 HTTP 요청을 한다.
10. WAS는 쓰레드 풀에서 쓰레드를 하나 조회한다. 쓰레드 thread-A 가 할당되었다. (물론 다른 쓰레드가 할당될 수 도 있다.)
11. 이번에는 조회하는 요청이다. thread-A 는 쓰레드 로컬에서 데이터를 조회한다.
12. 쓰레드 로컬은 thread-A 전용 보관소에 있는 사용자A 값을 반환한다.
13. 결과적으로 사용자A 값이 반환된다. 사용자B는 사용자A의 정보를 조회하게 된다.
































