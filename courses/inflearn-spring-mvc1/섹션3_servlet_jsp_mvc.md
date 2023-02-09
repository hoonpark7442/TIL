## 회원 도메인 모델 및 리포지터리
(생략)

## 서블릿으로 회원 관리 웹 앱 만들기

- 서블릿으로 회원등록 html 폼 제공해보자
```java
@WebServlet(name = "memberFormServlet", urlPatterns = "/servlet/members/new-form")
public class MemberFormServlet extends HttpServlet {

	@Override
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		response.setContentType("text/html");
		response.setCharacterEncoding("utf-8");
		PrintWriter w = response.getWriter();
		w.write("<!DOCTYPE html>\n" +
		"<html>\n" +
		"<head>\n" +
		"    <meta charset=\"UTF-8\">\n" +
		"    <title>Title</title>\n" +
		"</head>\n" +
		"<body>\n" +
		"<form action=\"/servlet/members/save\" method=\"post\">\n" +
		"    username: <input type=\"text\" name=\"username\" />\n" +
		"    age:      <input type=\"text\" name=\"age\" />\n" +
		" <button type=\"submit\">전송</button>\n" + "</form>\n" +
		"</body>\n" +
		"</html>\n");
	}
}
```
- `MemberFormServlet`은 단순히 회원 정보를 입력할 수 있는 HTML Form을 만들어서 응답한다.

- HTML Form에서 데이터 입력 후 전송 누르면 실제 회원 데이터가 저장되도록 해보자.

```java
@WebServlet(name = "memberSaveServlet", urlPatterns = "/servlet/members/save")
public class MemberSaveServlet extends HttpServlet {
	
	private MemberRepository memberRepository = MemberRepository.getInstance();

	@Override
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		System.out.println("MemberSaveServlet.service");
		String username = request.getParameter("username");
		int age = Integer.parseInt(request.getParameter("age"));

		Member member = new Member(username, age);
		System.out.println("member = " + member);
		memberRepository.save(member);

		response.setContentType("text/html");
		response.setCharacterEncoding("utf-8");

		PrintWriter w = response.getWriter();

		w.write("<html>\n" +
		"<head>\n" +
		" <meta charset=\"UTF-8\">\n" + "</head>\n" +
		"<body>\n" +
		"성공\n" +
		"<ul>\n" +
		"    <li>id="+member.getId()+"</li>\n" +
		"    <li>username="+member.getUsername()+"</li>\n" +
		" <li>age="+member.getAge()+"</li>\n" + "</ul>\n" +
		"<a href=\"/index.html\">메인</a>\n" + "</body>\n" +
		"</html>");
	}
}        
```

- 저장된 모든 회원 목록 조회 기능
(코드 생략)
- `MemberListServlet`은 다음 순서로 동작
	- `memberRepository.findAll()`을 통해 모든 회원 조회
	- 회원목록 HTML을 for 루프를 통해 회원수만큼 동적 생성 후 응답

## 템플릿 엔진으로
(생략)

## 서블릿과 JSP의 한계
- 서블릿으로 개발 시 뷰 화면을 위한 HTML을 만드는 작업이 코드에 섞여서 지저분하다.
- JSP를 사용함으로써 HTML 작업을 깔끔하게 처리 가능 했으나 JSP 코드에 리포지토리 등 다양한 코드가 모두 JSP에 노출되게 된다.
- 즉 JSP가 너무 많은 역할을 한다. 

### MVC 패턴의 등장
- 비즈니스 로직은 서블릿 처럼 다른 곳에서 처리하고 JSP는 목적에 맞게 뷰를 그리는 일에 집중하도록 하자. 
- 이러한 고민에서 MVC 패턴이 등장했다.

## MVC 패턴 개요

### 너무 많은 역할
- 하나의 서블릿이나 JSP 만으로 비즈니스 로직과 뷰 렌더링 까지 모두 처리하게 되면 너무 많은 역할을 하게 되고 유지보수가 어려워진다.

### 변경의 라이프 사이클
- UI를 일부 수정하는 일과 비즈니스 로직을 수정하는 일은 각각 다르게 발생할 가능성이 매우 높고 대부분 서로에게 영향을 주지 않는다. 즉 둘 사이의 변경의 라이프 사이클이 다르다.
- 이렇게 변경의 라이프 사이클이 다른 부분을 하나의 코드로 관리하는 것은 유지보수에 좋지 않다.

### 기능 특화
- JSP같은 뷰 템플릿은 화면을 렌더링 하는데 최적화 되어 있기에 이 부분의 업무만 담당하는 것이 효과적이다.

### Model View Controller
- MVC 패턴은 지금까지 학습한 것 처럼 하나의 서블릿이나 JSP로 처리하던 것을 컨트롤러와 뷰라는 영역으로 서로 역할을 나눈 것을 말한다.
	- 컨트롤러: HTTP 요청 받아서 파라미터 검증하고 비즈니스 로직을 실행. 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다.
	- 모델: 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고 화면을 렌더링 하는 일에 집중 할 수 있다.
	- 뷰: 모델에 담겨있는 데이터를 사용하여 화면을 그린다.

> 컨트롤러에 비즈니스 로직을 둘 수도 있지만 이리되면 컨트롤러가 너무 많은 역할을 담당한다. 그래서 일반적으로 비즈니스 로직은 서비스라는 계층을 별도로 만들어서 처리한다. 그리고 컨트롤러는 비즈니스 로직이 있는 서비스를 호출하는 역할을 담당한다. 참고로 비즈니스 로직을 변경하면 비즈니스 로직을 호출하는 컨트롤러의 코드도 변경될 수 있다. 


## MVC 패턴 적용
- 서블릿을 컨트롤러로, JSP를 뷰로 사용하여 MVC 패턴을 적용해보자.
- 모델은 HttpServletRequest 객체를 사용한다. request는 내부에 데이터 저장소를 가지고 있는데 `request.setAttribute()`, `request.getAttribute()`를 사용하면 데이터를 보관, 조회할 수 있다.

(코드생략)

### (참고) redirect vs forward
- 리다이렉트는 실제 클라이언트에 응답이 나갔다가 클라이언트가 리다이렉트 경로로 다시 요청한다. 따라서 클라이언트가 인지할 수 있고 URL 경로도 실제로 변경된다. 반면 포워드는 서버 내부에서 일어나는 호출이기에 클라이언트가 전혀 알 수 없다.


## MVC 패턴 한계

### MVC 컨트롤러의 단점

### 포워드 중복
- 뷰로 이동하는 코드가 항상 중복 호출되어야 한다. 물론 이 부분을 메서드로 공통화해도 되지만 해당 메서드도 항상 직접 호출해야 한다.
```java
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```

### ViewPath에 중복
```java
String viewPath = "/WEB-INF/views/new-form.jsp";
```
- prefix: ` /WEB-INF/views/`
- suffix: `.jsp`

- 만약 jsp가 아닌 타임리프같은 다른 뷰로 변경한다면 전체 코드를 다 변경해야 한다. 

### 사용하지 않는 코드
- `HttpServletRequest request, HttpServletResponse response` 이 코드를 사용할 때도 사용하지 않을 때도 있다.
- 또한 `HttpServletRequest`, `HttpServletResponse`를 사용하는 코드는 테스트 케이스를 작성하기도 어렵다.

### 공통 처리가 어렵다.
- 기능이 복잡해질수록 컨트롤러에서 공통으로 처리해야 하는 부분이 점점 더 많이 증가할 것이다. 단순히 공통 기능을 메서드로 뽑으면 될 거 같지만 결과적으로 해당 메서드를 항상 호출해야 하고 실수로 호출하지 않으면 문제가 될 것이다. 그리고 호출하는 것 자체도 중복이다.

### 정리하면 공통 처리가 어렵다는 문제가 있다.
- 이 문제를 해결하려면 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다. 소위 수문장 역할을 하는 기능이 필요하다. **프론트 컨트롤러 패턴**을 도입하면 이런 문제를 깔끔히 해결할 수 있다.
- 스프링 MVC의 핵심도 바로 이 프론트 컨트롤러에 있다.














