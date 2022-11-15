### 코드
```java
@Override
public Optional<User> findByEmail(String email) {
    return userDb.values().stream()
            .filter(user -> user.getEmail().equals(email))
            .findFirst();
}
```

### 리뷰
이렇게 하면 findByEmail 의성능은 어떻게될까요?
리스트를 모두 뒤져야하니 O(n)이 걸릴것 같습니다.
email 조회 성능을 올리려면 어떻게 해야될지 고민해보고 수정해봅시다


### 수정 코드
findByEmail 의성능
인덱스용 해시맵을 새로 만들어 email을 키값으로, id를 밸류로 저장할 수 있도록 했습니다. UserRepositoryImpl클래스의 save() 메서드에서 유저를 해시맵(userDb)에 저장할 때 인덱스용 해시맵에도 추가적으로 데이터를 저장하도록 수정했습니다.

```java
userDb.put(user.getId(), user);
userEmailIndex.put(user.getEmail(), user.getId());
```
findByEmail() 메서드에서 이메일 조회할 때에는 전처럼 스트림을 사용하여 풀스캔하지 않고, 해시맵에서 파라미터로 들어온 이메일을 키값으로 하여 O(1)의 시간복잡도로 id를 가져온 후 해당 id로 userDb해시맵에서 유저 정보를 조회하도록 하였습니다.

