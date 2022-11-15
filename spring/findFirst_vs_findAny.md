- 둘 다 스트림에서 요소를 찾는 메서드이다.
- findAny는 스트림에서 가장 먼저 탐색되는 요소를 리턴하고
- findFirst는 조건에 일치하는 요소들 중 가장 순서가 앞에 있는 요소를 리턴(물론 정렬되어 있지 않다면 -HashSet과 같은 컬렉션- 첫 번째 요소라는 것이 의미 없으니 findAny와 같이 작동)
- 스트림을 직렬로 처리할 때 둘은 동일한 요소를 리턴하며 차이점이 없다.
- 하지만 병렬 처리시 findFirst는 스트림 순서 고려하여 가장 앞에 있는 요소를 리턴한다.
- 반면 findAny는 멀티스레드에서 스트림을 처리할 때 가장 먼저 찾은 요소를 리턴한다.
- 병렬로 처리하면(아래와 같이 parallel메서드 넣어서) findFirst와 findAny 차이점을 볼 수 있다.

```java
List<String> lst1 = Arrays.asList("Jhonny", "David", "Jack", "Duke", "Jill","Dany","Julia","Jenish","Divya");
List<String> lst2 = Arrays.asList("Jhonny", "David", "Jack", "Duke", "Jill","Dany","Julia","Jenish","Divya");

Optional<String> findFirst = lst1.stream().parallel().filter(s -> s.startsWith("D")).findFirst();
Optional<String> fidnAny = lst2.stream().parallel().filter(s -> s.startsWith("J")).findAny();

System.out.println(findFirst.get()); //Always print David
System.out.println(fidnAny.get()); //Print Jack/Jill/Julia :behavior of this operation is explicitly nondeterministic
```


findAny는 스트림에서 가장 먼저 탐색되는 요소를 리턴하고
findFirst는 조건에 일치하는 요소들 중 가장 순서가 앞에 있는 요소를 리턴합니다(물론 정렬되어 있지 않다면 -HashSet과 같은 컬렉션- 첫 번째 요소라는 것이 의미 없으니 findAny와 같이 작동합니다).

병렬로 처리할 시 findFirst는 언제나 같은 값을 보장할 수 있으나 findAny보다 성능 면에서 불리합니다. 
반면 findAny는 병렬 처리시 성능 면에서 이점이 있으나 언제나 같은 값 리턴하는 것을 보장하지 않습니다. 성능이 중요하고 조건만 충족해도 문제 없다면 findAny를 사용합니다.

```java
@Override
public Optional<User> findByEmail(String email) {
    return userDb.values().stream()
            .filter(user -> user.getEmail().equals(email))
            .findFirst();
}
```
여기서 findFirst() 메서드를 사용한 이유는 성능보다는 정확한 값을 찾는게 더 중요하다고 판단했기 때문입니다.