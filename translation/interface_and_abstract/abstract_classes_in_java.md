https://www.baeldung.com/java-abstract-class 번역

# 1. Overview

contract를 이행/구현할 때 구현의 일부를 나중에 완료하도록 하는 경우가 많다. 추상 메서드를 통해 이를 쉽게 달성할 수 있다.

이 아티클에서는 추상 클래스의 기본 사항 및 어떤 경우에 도움이 될 수 있는지 알아볼 예정이다.

# 2. Key Concepts for Abstract Classes

추상 클래스를 언제 사용해야되는지 알아보기 전에 몇가지 특징들을 살펴보자.

- 클래스 키워드 앞에 abstract라는 제어자를 넣어 추상클래스를 정의한다.
- 추상 클래스는 서브클래스가 될 수 있으나 인스턴스화 할 수 없다.
- 클래스가 하나 이상의 추상 메서드를 정의하는 경우 해당 클래스는 반드시 추상클래스로 선언되어야 한다.
- 추상 클래스는 추상과 구체적인 메서드 모두를 선언할 수 있다.
- 추상 클래스에서 파생된 서브클래스는 모든 기본 클래스의 추상 메서드를 구현하거나 그 자체가 추상클래스여야 한다.

이러한 개념을 더 잘 이해하기 위해 간단한 예를 만들어보자. 

기본 추상 클래스가 보드게임의 추상 API를 정의하도록 해보자.

```java
public abstract class BoardGame {

    //... field declarations, constructors

    public abstract void play();

    //... concrete methods
}
```

그런 다음 play 메서드를 구현하는 서브클래스를 만들자.

```java
public class Checkers extends BoardGame {

    public void play() {
        //... implementation
    }
}
```

# 3. When to Use Abstract Classes

이제 인터페이스와 구체적인 클래스보다 추상클래스를 선호해야하는 몇 가지 일반적인 시나리오를 분석해보자.

- 여러 연관된 서브클래스가 공유하는 몇 가지 공통 기능을 한 곳에 캡슐화(코드재사용)하려 할 때
- 서브클래스가 쉽게 확장하고 세분화할 수 있는 API를 부분적으로 정의해야 할 때
- 서브클래스가 protected 접근제어자가 있는 하나 이상의 공통 메서드 또는 필드를 상속해야 할 때

이러한 모든 시나리오는 완전한 상속기반 개방/폐쇄 원칙을 준수하는 좋은 예이다.

또한 추상 클래스의 사용은 암묵적으로 기본 타입과 서브 타입을 다루기 때문에 다형성을 활용하기도 한다.

코드 재사용은 클래스 계층 내의 "is-a" 관게가 보존되는 한 추상클래스를 사용해야 하는 매우 강력한 이유가 된다.

# 4. A Sample Hierarchy of File Readers 

추상 클래스의 기능을 더욱 명확히 이해하기 위해 다른 예를 살펴보자.

### 4.1. Defining a Base Abstract Class

여러 유형의 파일 리더기를 사용하고 싶다면 파일 리딩을 위한 공통적인 내용을 캡슐화하는 추상 클래스를 만들 수 있다.

```java
public abstract class BaseFileReader {
    
    protected Path filePath;
    
    protected BaseFileReader(Path filePath) {
        this.filePath = filePath;
    }
    
    public Path getFilePath() {
        return filePath;
    }
    
    public List<String> readFile() throws IOException {
        return Files.lines(filePath)
          .map(this::mapFileLine).collect(Collectors.toList());
    }
    
    protected abstract String mapFileLine(String line);
}
```

필요한 경우 서브클래스가 액세스할 수 있도록 filePath를 protected로 만들었다. 더 중요한 것은 아직 완성안된채 남겨둔 것이 있단 것이다. 바로 어떻게 파일 내용에서 실제로 텍스트 한 줄을 실제로 parse하는가이다.

우리의 계획은 간단하다. 구체적인 클래스는 각각 파일 경로를 저장하거나 파일을 살펴볼 수 있는 그들만의 특별한 방법을 가지고 있지 않지만, 각 라인을 변환하는 것에 대해서는 각각 특별한 방법을 가지고 있다.

언뜻 보기에 BaseFileReader는 불필요해 보일 수 있다. 그러나 깔끔하고 쉽게 확장할 수 있는 설계의 기반이다. 이를 통해 각자 고유한 비즈니스 로직에 집중할 수 있는 다양한 버전의 파일 리더기를 쉽게 구현할 수 있다.

### 4.2. Defining Subclasses

가장 일반적인건 아마도 파일의 내용을 소문자로 변환하는 파일리더기일 것이다.

```java
public class LowercaseFileReader extends BaseFileReader {

    public LowercaseFileReader(Path filePath) {
        super(filePath);
    }

    @Override
    public String mapFileLine(String line) {
        return line.toLowerCase();
    }   
}
```

또 다른 하나는 대문자로 변환하는 리더기이다.

```java
public class UppercaseFileReader extends BaseFileReader {

    public UppercaseFileReader(Path filePath) {
        super(filePath);
    }

    @Override
    public String mapFileLine(String line) {
        return line.toUpperCase();
    }
}
```

이 간단한 예에서 볼 수 있듯이 각 서브클래스는 파일 리딩의 다른 것들을 신경쓸 필요 없이 각자의 고유한 동작에만 집중할 수 있다.

### 4.3. Using a Subclass

마지막으로 추상클래스에서 상속받은 클래스를 사용하는 것은 다른 구체적인 클래스와 다르지 않다. 

(생략)

# 5. Conclusion

이 아티클을 통해 자바에서 추상 클래스의 기본과 추상화를 달성했고 공통 구현을 한 곳에서 캡슐화하는데 사용할 시기에 대해 배울 수 있었다.
