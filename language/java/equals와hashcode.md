## equals와 hashCode
`equals()`와 `hashCode()`는 모든 Java 객체의 부모 클래스인 Object 클래스에 정의되어있다.

* equals() : 두 객체의 **내용이 같은지** 확인하는 메서드
* hashCode() : 두 객체가 **같은 객체인지** 확인하는 메서드

### equals()
```text
public boolean equals(Object obj) {
  return (this == obj);
}
```
그러나 Object 클래스의 equals() 메서드는 기본적으로 ==으로 구현되어있다.
따라서 값이 같은지 비교하기 위해서는 equals() 메서드를 override 해야한다.

### hashCode()
hashCode() 메서드는 실행 중(Runtime) heap에 저장된 객체의 메모리 주소와 연관된 객체의 유일한 int 값을 반환한다.

### 재정의의 필요성
equals를 재정의한 클래스에는 hashCode도 반드시 재정의해야한다.
* `equals()`의 값이 true이면 `hashCode()`도 동일한 값을 가져야 한다.
* 반대로 `equals()`의 값이 false이면 `hashCode()`는 같아도 되고 달라도 된다.

hashCode를 함께 재정의하지 않으면 코드가 예상과 다르게 작동할 수 있다.
HashSet, HashMap, HashTable은 다음과 같은 방법으로 두 객체가 동등한지 비교한다.

<img width="695" height="502" alt="Image" src="https://github.com/user-attachments/assets/2a30db0b-7751-425b-9804-c885b402f02a" />

Hash 기반 컬렉션은 먼저 hashCode()로 비교 후 그 다음 equals() 비교한다.

예시 상황
```java
class User {
    String name;

    User(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object o) {
        return o instanceof User && this.name.equals(((User)o).name);
    }
    // hashCode() 재정의 안함!
}
```

```java
User u1 = new User("belluga");
User u2 = new User("belluga"); // equals() true, hashCode() 다름

Set<User> set = new HashSet<>();
set.add(u1);
set.add(u2); // ❗ 들어가버림 → 중복 허용됨
```
hashCode()가 다르면 HashSet에 중복된 데이터가 계속 들어간다.


```java
Map<User, String> map = new HashMap<>();
map.put(u1, "developer");

map.get(u2); // ❗ null → 못 찾음!
```
key 객체의 hashCode()가 다르니 찾을 수가 없다.