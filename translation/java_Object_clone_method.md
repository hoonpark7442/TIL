https://www.digitalocean.com/community/tutorials/java-clone-object-cloning-java 번역

복제는 기존 인스턴스의 복사본을 반환하는 clone() 메서드로 Object. Java Object 클래스의 복사본을 만드는 프로세스이다. Object가 자바의 기본 클래스 이므로 기본적으로 모든 객체가 복제를 지원한다.

만약 clone() 메서드를 사용하려면 java.lang.Cloneable 메이커 인터페이스를 구현해야 한다. 그렇지 않으면런타임에 CloneNotSupportedException 예외가 발생할 것이다. 또한 Object clone 메서드는 protected 메서드이므로 반드시 오버라이드 해야 한다. 아래 예제를 봐보자.

```java
package com.journaldev.cloning;

import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

public class Employee implements Cloneable {
	private int id;
	private String name;
	private Map<String, String> props;

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Map<String, String> getProps() {
		return props;
	}

	public void setProps(Map<String, String> p) {
		this.props = p;
	}

	 @Override
	 public Object clone() throws CloneNotSupportedException {
	 return super.clone();
	 }

}
``` 

우리는 Object clone() 메서드를 사용하고 있으므로 Cloneable 인터페이스를 구현했다. 우리는 슈퍼클래스 clone 메서드를 호출하고 있다.

# Using Object clone() Method

clone() 메서드를 사용하여 인스턴스의 복사본을 만드는 프로그램을 생성해보자.

```java
package com.journaldev.cloning;

import java.util.HashMap;
import java.util.Map;

public class CloningTest {

	public static void main(String[] args) throws CloneNotSupportedException {

		Employee emp = new Employee();

		emp.setId(1);
		emp.setName("Pankaj");
		Map<String, String> props = new HashMap<>();
		props.put("salary", "10000");
		props.put("city", "Bangalore");
		emp.setProps(props);

		Employee clonedEmp = (Employee) emp.clone();

		// Check whether the emp and clonedEmp attributes are same or different
		System.out.println("emp and clonedEmp == test: " + (emp == clonedEmp));
		
		System.out.println("emp and clonedEmp HashMap == test: " + (emp.getProps() == clonedEmp.getProps()));
		
		// Let's see the effect of using default cloning
		
		// change emp props
		emp.getProps().put("title", "CEO");
		emp.getProps().put("city", "New York");
		System.out.println("clonedEmp props:" + clonedEmp.getProps());

		// change emp name
		emp.setName("new");
		System.out.println("clonedEmp name:" + clonedEmp.getName());

	}

}
```

[Output]

```java
// emp and clonedEmp == test: false
// emp and clonedEmp HashMap == test: true
// clonedEmp props: {city=New York, salary=10000, title=CEO}
// clonedEmp name: Pankaj
```

# CloneNotSupportedException at Runtime

Employee 클래스에서 Cloneable 인터페이스를 구현하지 않는다면 위의 프로그램에서 CloneNotSupportedException 런타임 예외를 발생시킬 것이다.

# Understanding Object Cloning

위의 결과를 살펴보고 Object clone() 메서드에서 어떤 일이 일어난건지 알아보자.

- emp and clonedEmp == test: false
	- 즉 emp와 clonedEmp는 동일한 객체를 가리키는 것이 아니라 서로 다른 두 객체이다. 이는 자바 객체 복사 요구 사항과 일치한다.
- emp and clonedEmp HashMap == test: true
	- 따라서 emp 및 clonedEmp 객체 변수는 모두 동일한 객체를 참조한다. 기본 객체 값을 변경할 경우 심각한 데이터 무결성 문제가 발생할 수 있다. 값의 변경 내용은 복제된 인스턴스에도 반영될 수 있다.
- clonedEmp props:{city=New York, salary=10000, title=CEO}
	- 우리는 clonedEmp 속성을 변경하지 않았지만 emp 변수와 clonedEmp 변수 모두 동일한 객체를 참조하고 있기 때문에 변경되었다. 이 문제는 디폴트 Object clone 메서드가 얕은 복사본을 만들기 때문에 발생한다. 복사 프로세스를 통해 완전히 분리된 객체를 생성하려는 경우 문제가 발생할 수 있다. 이로 인해 원하지 않는 결과가 발생할 수 있으므로 clone() 메서드를 올바르게 오버라이드 해야 한다.
- clonedEmp name:Pankaj
	- 우리는 emp 이름을 변경했지만 clonedEmp 이름은 변경되지 않았다. String이 immutable하기 때문이다. 따라서 emp 이름을 설정할 때 this.name = name;에서 새 string이 생성되고 emp 이름 참조가 변경된다. 따라서 clonedEmp 이름은 변경되지 않은 상태로 유지된다. 모든 기본형 타입 변수에서도 유사한 동작을 볼 수 있다. 따라서 우리는 객체에 기본형타입, immutable 변수만 있는 한 자바 객체 기본 복제로도 충분하다.

# Object Cloning Types

객체 복제에는 얕은 복사와 깊은 복사라는 두 가지 유형이 있다. 각 항목을 이해하고 자바 프로그램에서 복사를 구현하는 가장 좋은 방법을 알아보자.

### 1. Shallow Cloning

자바 Object clone() 메서드의 디폴트 구현은 얕은 복사이다. reflection API를 사용하여 인스턴스의 복사본을 만든다. 아래 코드 조각은 얕은 복사 구현을 보여준다.
```java
@Override
 public Object clone() throws CloneNotSupportedException {
 
	 Employee e = new Employee();
	 e.setId(this.id);
	 e.setName(this.name);
	 e.setProps(this.props);
	 return e;
}
```

### 2. Deep Cloning

깊은 복사에서는 필드를 하나씩 복사해야 한다. List, Map 등과 같이 객체를 필드로 갖고 있다면 하나씩 복사하기 위한 코드를 작성해야 한다. 그래서 깊은 복사라 부른다. 깊은 복사를 위해 다음과 같이 Employee clone 메서드를 오버라이드할 수 있다.

```java
public Object clone() throws CloneNotSupportedException {

	Object obj = super.clone(); //utilize clone Object method

	Employee emp = (Employee) obj;

	// deep cloning for immutable fields
	emp.setProps(null);
	Map<String, String> hm = new HashMap<>();
	String key;
	Iterator<String> it = this.props.keySet().iterator();
	// Deep Copy of field by field
	while (it.hasNext()) {
		key = it.next();
		hm.put(key, this.props.get(key));
	}
	emp.setProps(hm);
	
	return emp;
}
```

이 clone() 메서드 구현으로 우리의 테스트 프로그램은 다음과 같이 출력할 것이다.

```java
// emp and clonedEmp == test: false
// emp and clonedEmp HashMap == test: false
// clonedEmp props:{city=Bangalore, salary=10000}
// clonedEmp name:Pankaj
```

대부분의 경우 이것이 우리가 원하는 것이다. clone() 메서드는 원래 인스턴스에서 완전히 분리된 새 객체를 리턴해야 한다. 따라서 프로그램에서 Object clone을 사용하려는 경우 mutable 필드를 관리하여 적절히 오버라이드 해야 한다. 모든 mutable한 필드의 깊은 복사본을 관리하기 위해 객체 상속 계층 끝까지 다 살펴봐야 한다.

# Cloning using Serialization?

깊은 복사를 쉽게 수행하는 한 가지 방법은 직렬화이다. 그러나 직렬화는 비용이 많이 드는 절차이고 당신의 클래스는 반드시 Serializable 인터페이스를 구현해야 한다. 모든 필드와 슈퍼클래스도 반드시 Serializable 인터페이스를 구현해야 한다.

# Using Apache Commons Util

프로젝트에서 이미 Apache Commons Utility 클래스를 사용하고 있으며 클래스가 직렬화 가능한 경우 아래와 같은 방법을 사용해라.

`Employee clonedEmp = org.apache.commons.lang3.SerializationUtils.clone(emp);
`

# Copy Constructor for Cloning

복사 생성자를 정의하여 객체의 복사본을 만들 수 있다. 왜 Object의 clone() 메서드에 의존해야 하는가? 예를 들어 우리는 다음과 같은 코드와 같이 Employee 복사 생성자를 만들 수도 있다.

```java
public Employee(Employee emp) {
	
	this.setId(emp.getId());
	this.setName(emp.getName());
	
	Map<String, String> hm = new HashMap<>();
	String key;
	Iterator<String> it = emp.getProps().keySet().iterator();
	// Deep Copy of field by field
	while (it.hasNext()) {
		key = it.next();
		hm.put(key, emp.getProps().get(key));
	}
	this.setProps(hm);

}
```

직원 객체의 복사본이 필요할 때 마다 `Employee clonedEmp = new Employee(emp);` 를 사용하여 가져올 수 있다. 하지만 복사 생성자를 작성하는 것은, 특히 기본형과 immutable 변수가 많은 경우 지루한 작업이 될 수 있다.

# Java Object Cloning Best Practices

1. 클래스에 기본형과 불변 변수가 있거나 얕은 복사를 원하는 경우에만 Object clone() 메서드를 사용해라. 상속의 경우 Object 레벨까지 당신이 extending 하는 모든 클래스를 반드시 살펴봐야 한다.
2. 클래스에 대부분 mutable한 속성이 있는 경우 복사 생성자를 정의할 수도 있다.
3. 오버라이드된 clone() 메서드에서 super.clone()을 호출하여 Object clone() 메서드를 사용한 다음 mutable한 필드의 깊은 복사를 위해 필요한 내용을 변경한다.
4. 클래스가 직렬화 가능한 경우 직렬화를 사용하여 복제할 수 있다. 하지만 성능에 영향을 미치므로 복제를 위해 직렬화를 사용하기 전 몇가지 벤치마킹을 수행해봐라.
5. 클래스를 상속하고 있고 그 클래스가 깊은 복사를 적절히 사용하여 clone 메서드를 정의했다면 디폴트 clone 메서드를 사용할 수 있다. 예를 들어 Employee 클래스에 다음과 같이 clone 메서드를 올바르게 정의했다.

```java
@Override
public Object clone() throws CloneNotSupportedException {

	Object obj = super.clone();

	Employee emp = (Employee) obj;

	// deep cloning for immutable fields
	emp.setProps(null);
	Map<String, String> hm = new HashMap<>();
	String key;
	Iterator<String> it = this.props.keySet().iterator();
	// Deep Copy of field by field
	while (it.hasNext()) {
		key = it.next();
		hm.put(key, this.props.get(key));
	}
	emp.setProps(hm);

	return emp;
}
```

우리는 자식 클래스를 만들고 아래와 같이 superclass 깊은 복사를 활용할 수 있다.

```java
package com.journaldev.cloning;

public class EmployeeWrap extends Employee implements Cloneable {

	private String title;

	public String getTitle() {
		return title;
	}

	public void setTitle(String t) {
		this.title = t;
	}

	@Override
	public Object clone() throws CloneNotSupportedException {

		return super.clone();
	}
}
```

EmployeeWrap 클래스는 mutable한 속성이 없으며 슈퍼클래스 clone() 메서드 구현을 사용하고 있다. 만약 mutable한 필드가 있는 경우 해당 필드만 깊은 복사해야 한다.

