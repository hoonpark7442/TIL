# package란?

- 연관된 클래스끼리 묶는 기법
- 마치 디스크 상의 폴더와 같은 역할
	- ex) 사진 폴더는 사진 파일을 저장

- 패키지 이름 짓기
	- 패키지 이름 중복 최소화 해야함
	- 보통 회사 도메인 명을 패키지 이름에 사용. 단, 역순
		- ex) lookingood의 경우
		```java
			pacakge com.lookingood.<패키지이름>;
		```
- 패키지 사용하기
	- 외부패키지 안에 들어있는 클래스 사용하려면
		```java
			import java.util.Random;
			import java.util.*;
		```
	- 웬만하면 위의 코드처럼 쓰길 권장. 가독성때문.

- java.lang
	- 자바에서 제공하는 기본 패키지 중 하나
	- 모든 .java 파일에 자동으로 제공되는 패키지
	- System.out.println() 에서 System도 java.lang 안에 있는 클래스 중 하나