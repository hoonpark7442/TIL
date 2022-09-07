# 객체란?

- 일반적으로 실제세계에서의 객체는 우리 주변에 존재하는 사물, 동물 등을 의미한다. 그리고 이 모든 객체는 공통적으로 상태와 행동을 갖고 있다.
- 소프트웨어 객체 역시 개념적으로 실제 세계 객체와 유사하다. 객체는 상태를 변수에 저장하고 메소드를 통해 행동을 나타낸다. 
- 메소드는 객체의 내부 상태를 관리/제어 하고 객체간의 커뮤니케이션에 있어서 가장 주된 메카니즘으로서의 역할을 한다. 
- 이렇게 내부 상태를 숨기고 모든 상호작용을 객체의 메소드를 통해 수행되도록 하는 것을 데이터 캡슐화라고 한다. 
- 객체의 특성
1. 모듈화: 객체 소스코드는 독립적으로 쓰여지고 유지
2. 은닉화: 객체 메소드를 통해서만 상호작용함으로써 내부 구현의 세부사항은 외부세계로부터 숨길 수 있음
3. 재사용성: 다른 사람이 만든 객체를 내가 가져다 쓰는 것이 가능
4. 연결성 및 디버깅 용이성: 특정 객체에 문제 발생시 단순히 애플리케이션에서 해당 객체 제거 후 다른 객체를 대체해서 사용 가능
