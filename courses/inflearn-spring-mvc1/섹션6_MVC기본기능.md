# HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form
- 클라이언트에서 서버로 요청 데이터를 전달할 때는 주로 다음 3가지 방법을 사용한다.
	- GET - 쿼리 파라미터
	- POST - HTML Form
		- content-type: application/x-www-form-urlencoded
		- 메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello&age=20
	- HTTP message body에 데이터를 직접 담아서 요청

## HTTP 요청 파라미터 - @RequestParam
- 스프링이 제공하는 @RequestParam 을 사용하면 요청 파라미터를 매우 편리하게 사용할 수 있다.

## HTTP 요청 파라미터 - @ModelAttribute
- 스프링 MVC는 @ModelAttribute가 있으면 다음을 실행
	- HelloData 객체 생성
	- 요청 파라미터 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티 세터를 호출해 파라미터 값 바인딩
- @ModelAttribute는 생략 가능. 하지만 @RequestParam 역시 생략 가능해 혼란 발생 가능
- 스프링은 생략시 다음과 같은 규칙 적용
	- String, int, Integer 같은 단순 타입 -> @RequestParam
	- 나머지 -> @ModelAttribute(argument resolver로 지정해둔 타입 외)

## HTTP 요청 메시지 - 단순 텍스트
- 요청 파라미터와 다르게 HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우는 @RequestParam, @ModelAttribute를 사용할 수 없다.
	- 물론 HTML Form 형식으로 전달되는 경우는 요청 파라미터로 인정된다

- HTTP 메시지 바디 데이터를 InputStream을 사용해서 직접 읽을 수 있다.
```java
public class RequestBodyStringController {
	@PostMapping("/request-body-string-v1")
	public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
		
		ServletInputStream inputStream = request.getInputStream();
		String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

		log.info("messageBody={}", messageBody);
		
		response.getWriter().write("ok");
	}
}
```

- 스프링 MVC는 다음 파라미터를 지원한다.
	- InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
	- OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력

```java
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
	
	String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
	
	log.info("messageBody={}", messageBody);
	responseWriter.write("ok");
}
```

- 스프링 MVC는 다음 파라미터를 지원한다.
	- HttpEntity: HTTP header, body 정보를 편리하게 조회
		- 메시지 바디 정보를 직접 조회
		- 요청 파리미터를 조회하는 기능과 관계 없음(@RequestParam, @ModelAttribute 등과 관계없다는 뜻)
	- HttpEntity는 응답에도 사용 가능
		- 메시지 바디 정보 직접 반환, 헤더 정보 포함 가능
	- HttpEntity를 상속받은 다음 객체들도 같은 기능 제공
		- RequestEntity, ResponseEntity

```java
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
  String messageBody = httpEntity.getBody();
  log.info("messageBody={}", messageBody);
  return new HttpEntity<>("ok");
}
```

> 스프링MVC 내부에서 HTTP 메시지 바디를 읽어서 문자나 객체로 변환해서 전달해주는데 이때 HTTP 메시지 컨버터(HttpMessageConverter)라는 기능을 사용한다. 


### @RequestBody

- @RequestBody를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 
- 만약 헤더 정보가 필요하다면 HttpEntity를 사용하거나 @RequestHeader를 사용하면 된다.
- 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 @RequestParam, @modelAttribute와는 전혀 관계가 없다.

**요청 파라미터 vs HTTP 메시지 바디**
- 요청 파라미터를 조회하는 기능: @RequestParam, @modelAttribute
- HTTP 메시지 바디를 직접 조회하는 기능: @RequestBody

### @ResponseBody
- @ResponseBody를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다. 

## HTTP 요청 메시지 - JSON
```java
@PostMapping("/request-body-json-v1")
public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
	ServletInputStream inputStream = request.getInputStream();
	String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
	
	log.info("messageBody={}", messageBody);
	HelloData data = objectMapper.readValue(messageBody, HelloData.class);

	log.info("username={}, age={}", data.getUsername(), data.getAge());
	response.getWriter().write("ok");
}
```
- HttpServletRequest를 사용해서 직접 HTTP 메시지 바디에서 데이터를 읽어와서, 문자로 변환한다.
- 문자로 된 JSON 데이터를 Jackson 라이브러리인 objectMapper 를 사용해서 자바 객체로 변환한다.

```java
@PostMapping("/request-body-json-v1")
public void requestBodyJsonV1(@RequestBody String messageBody) throws IOException {
	HelloData data = objectMapper.readValue(messageBody, HelloData.class);
  log.info("username={}, age={}", data.getUsername(), data.getAge());
  return "ok";
}
```
- 문자로 된 JSON 데이터인 messageBody 를 objectMapper 를 통해서 자바 객체로 변환한다.

- 문자로 변환하고 다시 json으로 변환하는 과정이 불편하다. @ModelAttribute처럼 한 번에 객체로 변환하는 방법을 살펴보자

### @RequestBody 객체 변환

```java
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData data) {
    log.info("username={}, age={}", data.getUsername(), data.getAge());
    return "ok";
}
```

**@RequestBody 객체 파라미터**
- @RequestBody에 직접 만든 객체를 지정할 수 있다.
- ttpEntity , @RequestBody를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
- HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON도 객체로 변환

**@RequestBody는 생략 불가능**
- HelloData에 @RequestBody를 생략하면 @ModelAttribute가 적용되어 버린다. 따라서 생략하면 HTTP 메시지 바디가 아니라 요청 파라미터를 처리하게 된다.

> HTTP 요청시 content-type이 application/json인지 꼭 확인해야 한다. 그래야 json을 처리할 수 있는 http 메시지 컨버터가 실행된다.

- 물론 HttpEntity를 사용해도 된다.

**@ResponseBody**
- 응답의 경우에도 @ResponseBody를 사용하면 해당 객체를 HTTP 메시지 바디에 직접 넣어줄 수 있다.
- 물론 HttpEntity를 사용해도 된다.

- @RequestBody 요청
	- JSON 요청 HTTP 메시지 컨버터 객체
- @ResponseBody 응답
	- 객체 HTTP 메시지 컨버터 JSON 응답

## HTTP 응답 - 정적 리소스, 뷰 템플릿
생략

### @RestController
- @RestController 애노테이션을 사용하면 해당 컨트롤러에 모두 @ResponseBody가 적용되는 효과가 있다. 
- @ResponseBody는 클래스 레벨에 두면 전체 메서드에 적용되는데 @RestController 애노테이션 안에 @ResponseBody가 적용되어 있다.


## HTTP 메시지 컨버터

### @ResponseBody 원리
- HTTP body에 문자 내용을 직접 입력
- viewResolver 대신 HttpMessageConverter가 동작
- 기본 문자처리: StringHttpMessageConverter
- 기본 객체처리: MappingJackson2HttpMessageConverter

> 응답의 경우 클라이언트 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입 정보 2가지를 조합해서 HttpMessageConverter가 선택된다. 

**스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다.**
- HTTP 요청: @RequestBody , HttpEntity(RequestEntity)
- HTTP 응답: @ResponseBody , HttpEntity(ResponseEntity)

### HTTP 메시지 컨버터 인터페이스
- HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다.
- canRead(), canWrite(): 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크
- read(), write(): 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

### 스프링부트 기본 메시지 컨버터

```
0 = ByteArrayHttpMessageConverter
1 = StringHttpMessageConverter
2 = MappingJackson2HttpMessageConverter
```

- 스프링부트는 대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정. 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.

**ByteArrayHttpMessageConverter**
- byte[] 데이터를 처리
- 클래스타입: byte[], 미디어타입: */*
- 요청 예) @RequestBody byte[] data
- 응답 예) @ResponseBody return byte[] 쓰기 미디어타입: application/octet-stream

**StringHttpMessageConverter**
- String 문자로 데이터를 처리
- 클래스타입: String, 미디어타입: */*
- 요청 예) @RequestBody String data
- 응답 예) @ResponseBody return "ok" 쓰기 미디어타입: text/plain

**MappingJackson2HttpMessageConverter**
- application/json
- 클래스타입: 객체 또는 HashMap, 미디어타입: application/json 관련
- 요청 예) @RequestBody HelloData data
- 응답 예) @ResponseBody return helloData 쓰기 미디어타입: application/json 관련

### HTTP 요청 데이터 읽기
- HTTP 요청이 오고 컨트롤러에서 @RequestBody, HttpEntity 파라미터를 사용한다.
- 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead()를 호출
	- 대상 클래스타입을 지원하는가, HTTP 요청의 Content-Type 미디어 타입을 지원하는가 체크
- canRead() 조건을 만족하면 read()를 호출해서 객체 생성하고 반환

### HTTP 응답 데이터 생성
- HTTP 요청 읽기와 동일하게 작동하나 canWrite() 및 write()를 호출

## 요청 매핑 핸들러 어댑터 구조
- HTTP 메시지 컨버터는 스프링 MVC 어디쯤에서 사용되는걸까

- 비밀은 애노테이션 기반의 컨트롤러, 즉 @RequestMapping을 처리하는 핸들러 어댑터인 RequestMappingHandlerAdapter에 있다.

![](/assets/argument-resolver.png)

### ArgumentResolver
- 애노테이션 기반 컨트롤러는 매우 다양한 파라미터를 사용할 수 있었다.
- HttpServletRequest, Model은 물론이고 @RequestParam, @ModelAttribute 같은 애노테이션, 그리고 @RequestBody, HttpEntity 같은 HTTP 메시지를 처리하는 부분까지 매우 큰 유연함을 보여주었다.
- 이게 다 ArgumentResolver 덕분

- 애노테이션 기반 컨트롤러를 처리하는 RequestMappingHandlerAdapter는 바로 이 ArgumentResolver를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)를 생성
- 이렇게 파라미터 값이 모두 준비되면 컨트롤러 호출하면서 값 넘겨줌
- 스프링은 30개가 넘는 ArgumentResolver를 기본으로 제공

```java
public interface HandlerMethodArgumentResolver {
	boolean supportsParameter(MethodParameter parameter);

	@Nullable
	Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
      NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;
}
```
- HandlerMethodArgumentResolver가 풀네임이다

**동작방식**
- ArgumentResolver의 supportsParameter()를 호출해서 해당 파라미터를 지원하는지 체크하고, 지원하면 resolveArgument()를 호출해서 실제 객체를 생성한다. 
- 이렇게 생성된 객체가 컨트롤러 호출시 넘어가는 것이다.

- ReturnValueHandler
	- HandlerMethodReturnValueHandler가 풀네임
	- ArgumentResolver와 비슷한데 이것은 응답 값을 변환하고 처리한다.

## HTTP 메시지 컨버터 위치
![](/assets/argument-resolver-loc.png)

- 요청의 경우 @RequestBody를 처리하는 ArgumentResolver가 있고, HttpEntity를 처리하는 ArgumentResolver가 있다. 이 ArgumentResolver들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성하는 것이다.

- 응답의 경우 @ResponseBody 와 HttpEntity 를 처리하는 ReturnValueHandler가 있다. 여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.

- 스프링 MVC는 @RequestBody, @ResponseBody가 있으면 RequestResponseBodyMethodProcessor(ArgumentResolver), HttpEntity가 있으면 HttpEntityMethodProcessor(ArgumentResolver)를 사용한다.








