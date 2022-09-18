# 기본 자료형(Primitive type)

- 기본형 타입은 총 8개 타입이 존재하며 기본값이 있어서 null 값은 존재하지 않는다.
- 실제 값을 그대로 저장하며 메모리 구조상 stack에 저장된다.

# 참조 자료형(Reference type)

- 참조형 타입은 기본형 타입을 제외한 모든 타입이며 null 값이 존재한다.
- 실제 객체는 힙 영역에 저장되며 참조타입 변수에는 스택 영역에 있는 실제 객체의 주소값이 저장된다. 즉 객체를 사용할 때 마다 참조변수에 저장된 객체의 주소를 불러와 사용하는 방식.
- 크기가 정해져 있지 않아 프로그램 실행 시 힙 영역에 동적으로 메모리를 할당

