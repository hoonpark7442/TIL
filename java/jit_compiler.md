# JIT 컴파일러란?

- JIT컴파일러는 JRE의 컴포넌트이다.
- jvm에서 컴파일러만큼 성능에 영향 미치는 것이 없다. 그렇기에 중요하다.
- 자바 WORA의 핵심은 바이트 코드이다. 
- 바이트 코드가 적절한 네이티브 인스트럭션으로 변환되는 것은 어플리케이션 속도에 큰 영향을 미친다.
- 바이트 코드는 해석될수도(interpreted), 네이티브 코드로 컴파일될 수도 있다.
- jvm의 표준 구현인 interpreting은 프로그램 실행 속도를 느리게 한다.
- 성능을 향상시키기 위해 JIT 컴파일러는 런타임에 JVM과 상호 작용하고 적절한 바이트 코드 시퀀스를 네이티브 머신 코드로 컴파일한다.
- JIT 컴파일러를 사용하면 하드웨어는 네이티브코드를 실행시킬 수 있는데, 이는 JVM이 동일한 바이트코드 시퀀스를 반복적으로 interpreting하거나 상대적으로 긴 번역 프로세스의 페널티를 발생시키는 것과 반대된다.
- 이는 메소드들이 드물게 실행되지 않는 한, 실행 속도의 성능 향상으로 이어질 수 있다.
- JIT 컴파일러가 바이트 코드를 컴파일 하는데에 걸리는 시간은 전체 실행 시간에 더해지는데, 만약 JIT컴파일러에 의해 컴파일 된 메소드들이 자주 invoke되지 않는다면 오히려 인터프리터가 바이트코드를 실행시키는 것보다 실행 시간에 더욱 악영향을 끼칠 수 있다.
- JIT 컴파일러는 바이트코드를 네이티브 코드로 컴파일링할 때 특정한 최적화를 실행한다. 
- JIT 컴파일러에 의해 수행되는 일반적인 최적화로는 데이터분석, 스택 작업에서 레지스터 작업으로의 변환, 레지스터 할당에 의한 메모리 액세스 감소, 공통의 sub-expression의 제거 등이다.
- JIT 컴파일러가 수행하는 최적화 수준이 높을 수록 실행단계에서 더 많은 시간을 소비한다. 따라서 JIT 컴파일러는 정적 컴파일러에 의해 수행되는 모든 최적화를 수행할 여유는 없다.
