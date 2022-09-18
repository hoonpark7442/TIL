https://www.baeldung.com/cs/deep-vs-shallow-copy 번역

# 1. 소개

이 아티클에서는 정확한 메모리를 공유하고 적절한 복사본을 만듦으로써 서로 다른 코드 영역 간에 동일한 데이터를 안전하게 공유할 수 있는 몇 가지 방법을 살펴볼 예정이다.

# 2. References vs Values

자바와 같은 많은 언어에서 대부분의 변수는 실제 값을 저장하지 않고 값에 대한 참조 또는 포인터를 저장한다. 

이러한 방식은 몇 가지 이점을 갖는다. 예를 들어 변수를 전달할 때 큰 값 대신 작은 참조값만 전달한다. 

또한 많은 다른 변수들이 메모리에서 정확히 동일한 값을 가리키게 할 수도 있다.
이는 두 변수 모두 정확히 동일한 데이터를 바라볼 수 있다는 것을 의미하기 때문에 유용할 수 있다. 그러나 둘 중 하나가 변경되면 다른 하나는 자동으로 동일한 변경 내용을 보게 된다. 두 변수는 항상 동일하다.

이는 객체를 사용할 때만 적용된다. int나 바이트와 같은 기본형 타입은 항상 정확한 값으로 저장되고 전달되며 값에 대한 참조를 하지 않는다. 가장 큰 기본형 타입(long)은 일반적으로 참조할 때와 같은 양의 메모리이고 기본형은 항상 불변하므로 어찌됐든 변경할 수 없기 때문에 문제없다.

# 3. What Is a Shallow Copy?

어떤 경우에는 값의 복사본을 만들어 두 개의 서로 다른 코드 조각이 동일한 값의 다른 복사본을 바라보길 원할 수 있다. 이를 통해 하나를 다른 것과 다르게 조작할 수 있다.

이 작업을 수행하는 가장 간단한 방법은 객체를 얕게 복사하는 것이다. 즉 원본과 동일한 필드 및 동일한 값의 복사본을 모두 포함하는 새 객체를 만든다.

![shallow_copy_example_1](/assets/shallow_copy_example_1.webp)
[이미지출처](https://www.baeldung.com/cs/deep-vs-shallow-copy)<br/>

비교적 간단한 객체의 경우 이 작업은 잘 작동한다. 그러나 객체에 다른 객체가 포함되어 있으면 해당 객체에 대한 참조값만 복사된다. 이는 다시 말해 두 복사본이 메모리의 동일한 값에 대한 참조를 포함한다는 것을 의미하며, 이는 다음과 같은 장단점을 수반한다.

![shallow_copy_example_2](/assets/shallow_copy_example_2.webp)
[이미지출처](https://www.baeldung.com/cs/deep-vs-shallow-copy)<br/>

이 예제에서 원본과 복제본에는 동일한 숫자 리스트를 가리키는 def 필드가 있다. 둘 중 하나가 리스트를 변경한다면 다른 하나 역시 동일한 변경 내용을 볼 수 있다. 그러나 원본의 복사본을 만들었기 때문에 기본 데이터가 여전히 그들 간에 공유된다는 것은 서프라이즈한 일이며 이로 인해 우리 코드에 예기치 않은 버그가 발생할 수 있다.

# 4. What Is a Deep Copy?

이에 대한 대안은 깊은 복사를 수행하는 것이다. 여기서는 각 필드를 원본에서 복사본으로 복사하지만, 참조만 복사하는 대신 깊은 복사를 한다.

![deep_copy_example_1](/assets/deep_copy_example_1.webp)
[이미지출처](https://www.baeldung.com/cs/deep-vs-shallow-copy)<br/>

그러면 새 복사본이 원본의 정확한 복사본이지만 다른 복사본에 변경 사항이 반영되지 않도록 연결되지 않는다.

# 5. Immutability vs Copying

데이터 복사본을 만드는 주요 이점은 두 개의 서로 다른 코드가 간섭없이 데이터에 대해 작동할 수 있다는 것이다. 각각 정확히 동일한 리스트가 지정된 두 개의 코드가 있고 한 개 가 해당 리스트에서 아이템을 제거하면 다른 한 개도 해당 변경 내용을 볼 수 있다. 리스트를 복사한다는 것은 하나에 대한 변경사항이 다른 하나에 표시되지 않는다는 것을 의미한다.

그러나 객체를 복사하는 데 비용이 많이 들 수 있다. 객체 구조가 복잡할수록 비용이 더 많이 들 수 있다. 그리고 어떤 경우에는 복사가 불가능할 수도 있다. 예를 들어 객체가 일부 컴퓨터 메모리 대신 네트워크 소켓이나 파일 핸들 같은 물리적 리소스를 나타내는 경우이다.

하지만 다른 대안이 있다. 만약 우리의 객체가 불변하다면(즉 값은 절대 변경될 수 없다), 다른 코드 조각들 간에 정확히 동일한 값을 공유하는 것에 대한 위험이 훨씬 적다. 만약 우리가 우리의 리스트를 다른 코드 조각으로 넘긴다면, 그리고 우리가 그것이 결코 변하지 않을 것이라고 보장할 수 있다면, 우리는 이것이 안전할 것이라는 것을 안다.

하지만 특히 중첩된 구조에서 불변한 코드를 작성하는 것이 항상 쉬운 것은 아니다. 예를 들어 우리가 getter만 있고 setter가 없는 객체를 가지고 있다하면, 해당 필드는 절대 변경할 수 없다. 이 객체는 그 자체로 불변이지만 만약 그 필드들 중 어느 하나라도 mutable하다면 동일한 문제가 발생할 수 있다.

```java
class Immutable {
	private final List<String> names = new ArrayList<>();

	public List<String> getNames() {
		return names;
	}
}
```

이 예에서는 객체에서 이름 필드를 변경할 수 없다. 항상 같은 리스트를 가리킬 것이다. 하지만 여기서 무슨 일이 일어날까?

```java
var immutable = new Immutable();
var immutable2 = immutable;

immutable.getNames().add("Hoon");
```

이름 필드는 변경할 수 없지만 새 항목을 삽입할 수 있다. 그리고 이 항목은 immutable과 immutable2 둘 모두에 의해 동시에 보여진다. 둘 다 같은 메모리를 가리키고 있기 때문이다.

# 6. Copy-on-Write
생략

# 7. Summary

여기서는 코드의 여러 영역 간에 데이터를 공유할 수 있는 몇가지 방법을 확인했으며 한 영역이 다른 영역에 실수로 영향을 미치지 않도록 이 작업을 수행할 수 있는 몇 가지 방법을 살펴보았다.








