왜 프로덕션 코드에서 System.out.println() 말고 Logger나 Log4j 같은 로깅 프레임워크를 사용해야 할까? 
아래와 같은 이유에서이다.
- 로그레벨
- 성능
- ?
- ?

1. 로그 레벨을 사용한 정보 분리

여러 로깅프레임워크는 로그 레벨에 따라 디버깅 정보를 로그하게끔 해준다. 로깅 프레임워크에서 제공하는 로그 레벨은 아래와 같다.
FATAL
ERROR
WARN
INFO
DEBUG
TRACE
ALL

이러한 레벨을 사용하여 언제 어디서 어떤 정보를 프린트할지 쉽게 필터링 할 수 있다.
```java
logger.trace("Trace log message");
logger.debug("Debug log message");
logger.info("Info log message");
logger.error("Error log message");
logger.warn("Warn log message");
logger.fatal("Fatal log message");
```

반면 System.out.println()은 이러한 레벨별 출력이 불가능하다. 오직 System.out.println을 사용한 인포메이션 로그, System.err.println를 사용한 에러 로그 2가지로만 분리가 가능하다. 


2. 성능

위에서 얘기했지만 System.err.println() 메서드는 로그 레벨 분리가 안되고, 이로인해 필터링이 불가능하다. 바꿔 말해 모든 정보를 다 출력해야만 된다는 소리다. 로깅 프레임워크를 사용하면, 예를 들어 프로덕션 서버 상에서는 DEBUG 정보를 보고 싶지 않다면 설정에서 바꿔주기만 하면 원하는 정보만 골라서 로깅할 수 있다.

println()는 콘솔에 결과물을 출력하게 해주는 메서드이다. 
https://media.geeksforgeeks.org/wp-content/uploads/20191126171503/println1.png

위의 그림에서 보면 out은 PrintStream 타입의 인스턴스이며 이 PrintStream 클래스는 io 패키지 내에 있다. 즉 println()은 I/O 작업이며 이는 I/O 시스템콜을 호출하여 커널모드에서 작업한다는 의미이다. 당연히 시간이 많이 드는 작업이며 성능에 좋지 않다. 
게다가 println()은 synchronized 메서드이다.
```java
public void println(String x) {
	synchronized (this) {
	  print(x);
	  newLine();
}
```
즉 멀티스레드 환경에서 사용시 해당 모니터락객체(아마 out? 좀 더 조사 필요)를 점유한 스레드 외의 다른 스레드는 대기해야만 하고 위에서 말했듯 출력작업이 커널모드에서 실행되기에 점유시간도 상대적으로 길 것이다. 

System.out.println()은 위에서 설명한 바와 같이 시스템에 많은 오버헤드가 발생하기 때문에 느린 작업이다. 















