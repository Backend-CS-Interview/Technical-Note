abstract로 만들어도 상속 받아서 인스턴스를 만들 수 있다. 따라서 인스턴스화를 막기 위해 private 생성자를 사용하라

## 1. static 메서드와 static 필드만을 담은 클래스

### 1.1. `java.lang.Math`, `java.util.Array` 등 기본 타입 값이나 배열 관련 메서드를 모아놓는 경우

```java
public class Math {
    private Math() {}

    // 정적 메서드: 두 수의 합을 반환
    public static int add(int a, int b) {
        return a + b;
    }

    // 정적 메서드: 배열의 최대값을 반환
    public static int max(int[] array) {
        int max = array[0];
        for (int num : array) {
            if (num > max) {
                max = num;
            }
        }
        return max;
    }
}
```

### 1.2. `java.util.Collectons 처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드(혹은 팩터리) 를 모아두는 경우

List 인터페이스를 구현한 불변 리스트를 생성하는 정적 팩터리 메서드

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public final class ListFactory {

    // 인스턴스 생성 불가
    private ListFactory() {}


    // 불변 리스트를 반환하는 정적 메서드
    public static <T> List<T> createImmutableList(T... elements) {
        List<T> list = new ArrayList<>();
        Collections.addAll(list, elements);
        return Collections.unmodifiableList(list); // 불변 리스트로 반환
    }
}
```

### 1.3. final 클래스와 관련한 메서드들을 모아놓을 때

final 클래스를 상속해서 하위 클래스에 메서드를 넣는 것은 불가능하다. 이때 final 클래스와 관련한 메서드들을 모아놓을 때 사용한다.

```java
import java.util.UUID;

public final class UUIDUtils {

    // 인스턴스 생성 불가
    private UUIDUtils() {}

    // UUID 생성
    public static String generateUUID() {
        return UUID.randomUUID().toString();
    }

    // UUID 문자열이 유효한지 확인하는 메서드
    public static boolean isValidUUID(String uuid) {
        try {
            UUID.fromString(uuid);
            return true;
        } catch (IllegalArgumentException e) {
            return false;
        }
    }
}

// 사용 예시
public class Main {
    public static void main(String[] args) {
        String uuid = UUIDUtils.generateUUID();
        System.out.println("Generated UUID: " + uuid);  // UUID 출력
        System.out.println("Is valid UUID: " + UUIDUtils.isValidUUID(uuid));  // true
        System.out.println("Is valid UUID: " + UUIDUtils.isValidUUID("invalid-uuid"));  // false
    }
}

```

> Arrays를 사용할 때 `Arrays arrays = new Arrays()`로 생성하지 않는다. `Arrays.asList(배열)` 식으로 바로 사용한다.
> 내부 메소드를 보면 전부 static으로 선언되어 있고 생성자가 private로 선언된 것을 확인할 수 있다.

```java
public class Arrays {
    private Arrays() {}

    public static void sort(int[] a) { ... }
    public static void parallelSort(byte[] a)
    // ...
}
```

## 2. private 생성자를 추가하여 인스턴스가 생성되는 상황을 방지하자.

클래스를 의도치 않게 인스턴스화 할 수 있게 만들지 말자.

- 정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 만든게 아니지만, 생성자를 명시하지 않으면 컴파일러가 자동으로 매개변수를 받지 않는 public 기본 생성자를 만들어준다.

추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.

- 하위 클래스로 만들어 인스턴스화 할 수 있게 때문이다.
- 오히려, 상속해서 쓰라는 뜻으로 오해할 수 있으므로 지양한다.

private 생성자만 붙여주면 실수로라도 생성자를 호출하지 않도록 해준다.

- 어떤 환경에서도 클래스가 인스턴스화되는 것을 막아준다.
- 생성자가 존재하는데, 호출할 수 없기 때문에 직관적이지 않으므로 적절한 주석을 다는 것이 좋다.
- 상속을 불가능하게 한다.
  - 상속한 경우에 명시적이든 암묵적이든 상위 클래스의 생성자를 호출해야 하는데, 이 클래스의 생성자가 private이라 호출이 막혔기 때문에 상속을 할 수 없다.
