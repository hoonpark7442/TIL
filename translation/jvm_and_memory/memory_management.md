https://deepu.tech/memory-management-in-programming 번역

# 파트1 메모리 관리 소개

메모리 매니지먼트는 소프트웨어 애플리케이션이 컴퓨터 메모리에 접근하는 방식을 컨트롤하고 조정하는 프로세스이다.

### What is it?

소프트웨어가 컴퓨터 상의 OS에서 돌아갈 때 컴퓨터의 램에 접근해야 하는데 이는
- 실행되어야하는 바이트코드를 로드하기 위해
- 프로그램에 의해 사용되는 데이터 밸류나 데이터 구조를 저장하기 위해
- 그로그램 실행하기 위해 요구되는 런타임 시스템을 로드하기 위해
위 3가지 등을 위해서이다.

소프트웨어 프로그램이 메모리를 사용할 때 바이트코드를 로드하기 위한 공간 외에 2가지 영역이 있는데, 스택과 힙이다.

### 스택

스택은 스태틱 메모리 할당을 위해 사용되며 LIFO 방식이다.
- 스택이므로 데이터를 저장하고 꺼내는게 굉장히 빠르다. 룩업타임이 없기 대문이다.
- 하지만 이러한 이유로 스택에 저장되는 데이터는 유한하고 정적(static)해야만 한다(데이터 크기를 컴파일 타임에 이미 알고 있어야 한다)
- 함수의 실행 데이터가 스택프레임으로 저장되는 곳이다(그래서 이것은 실제 실행 스택이다). 각 프레임은 해당 기능에 필요한 데이터가 저장되는 공간의 블록이다. 예를들어 함수가 새 변수를 선언할 때마다 스택의 맨위 블록에 푸시된다. 그런다음 함수가 종료될 때 마다 맨 위 블록이 삭제되므로 해당 함수에 의해 스택에 푸시된 모든 변수가 삭제된다. 
- 멀티 스레드 애플리케이션은 스레드마다 스택을 갖는다.
- 스택의 메모리 관리는 심플하고 직관적이며 OS에 의해 행해진다.
- 스택에 저장되는 전형적인 데이터는 지역변수(밸류 타입 혹은 기본형, 기본형 상수), 포인터, 펑션프레임이다.
- 스택오버플로우가 발생되는 곳이며 이는 힙과는 달리 스택의 사이즈에 한계가 있기 때문이다.
- 대부분의 언어에서 스택에 저장할 수 있는 값 크기에 제한이 있다.

### 힙

힙은 동적메모리 할당에 사용되며 스택과 달리 프로그램은 포인터를 사용해 힙에서 데이터를 룩업해야 한다.
- 이러한 룩업 작업으로 인해 스택보다 느리지만 대신 더 많은 데이터를 저장할 수 있다.
- 이는 동적 사이즈를 가진 데이터를 여기에 저장할 수 있단 뜻이다.
- 힙은 스레드가 공유하는 영역이다.
- 동적인 특성으로 인해 힙은 관리하기 까다롭고 이로인해 대부분의 메모리 관제 문제가 발생하는 곳이며, 자동 메모리 관리 솔루션이 시작되는 곳이다.
- 힙에 저장되는 전형적인 데이터는 전역변수, 객체와 같은 참조타입, 스트링, 맵, 그리고 복잡한 자료구조 등이다.
- 만약 애플리케이션이 할당된 힙보다 더 많은 메모리를 사용하려 할 때 OOM이 발생할 수 있는 곳이다.
- 일반적으로 힙에 저장할 수 있는 값의 크기에는 제한이 없다. 물론 애플리케이션에 할당되는 메모리 양에는 상한선이 있다.

### 왜 중요한가?

하드디스크와 달리 램은 무한하지 않다. 만약 프로그램이 메모리 해제 없이 계속 메모리를 소비한다면 OOM 이슈가 발생한다든지 심하면 OS에 영향을 미칠 수도 있다.
따라서 소프트웨어 프로그램은 램을 마음대로 사용할 수 없다. 다른 프로그램과 프로세스의 메모리 부족을 초래할 수 있기 때문이다.
그래서 대부분의 프로그래밍 언어는 자동 메모리 관리법을 제공한다. 그리고 메모리 관리를 말할 때 대부분은 힙 메모리 관리를 뜻한다.

### Garbage Collection
사용하지 않는 메모리 할당을 해제하여 힙 메모리를 자동으로 관리한다. GC는 현대 언어에서 가장 일반적인 메모리 관리 중 하나이며 이 프로세스는 종종 특정 간격으로 실행되므로 일시중지 시간이라는 마이너한 오버헤드가 발생할 수 있다.

- Mark & Sweep GC
	- Tracing GC라고도 알려져 있다. 일반적으로 이 알고리즘은 2개의 페이즈로 이루어져있는데, 먼저 아직 참고되고 있는 객체를 alive 라고 마크하고, 다음 단계에서 alive하지 않은 객체의 메모리를 해제한다.



