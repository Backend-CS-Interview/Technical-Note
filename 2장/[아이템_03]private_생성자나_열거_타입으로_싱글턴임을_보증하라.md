## 1. 싱글턴이란?

### 1.1 정의

- 인스턴스를 오직 하나만 생성할 수 있는 클래스

스프링 프레임워크는 기본적으로 @Component, @Service, @Repository, @Component 와 같은 어노테이션을 사용하여 빈(Bean)을 등록할 때 싱글턴 범위를 기본으로 설정하므로, 별도로 싱글턴 패턴을 구현하지 않아도 된다.

스프링을 사용하지 않는 경우에 싱글톤이 필요하다면 전통적인 싱글턴 패턴을 구현하여 동일한 인스턴스를 애플리케이션 전체에서 사용할 수 있다.

### 1.2 언제 사용해야 할까?

설계상 유일해야 하는 시스템 컴포넌트

- 애플리케이션 전역에서 로깅 기능을 제공하는 클래스 등
  무상태 객체
- 유틸리티 클래스 등

### 1.3 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어려워질 수 있다.

책에 아래 대목이 있다.

> "클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다. 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수
> 없기 때문이다."

아래는 작성해본 예시 코드다.

인터페이스 없이 단순하게 싱글턴을 구현한 경우

```java
public class SingletonService {
    private static SingletonService instance;

    private SingletonService() {
        // private constructor to prevent instantiation
    }

    public static SingletonService getInstance() {
        if (instance == null) {
            instance = new SingletonService();
        }
        return instance;
    }

    public void doSomething() {
        System.out.println("Doing something...");
    }
}
```

이를 사용하는 클라이언트 코드에서는 SingletonService 클래스의 실제 인스턴스를 직접 참조하고 있다.

```java
public class Client {
    public void performAction() {
        SingletonService service = SingletonService.getInstance();
        service.doSomething();
    }
}
```

테스트 중에 이를 Mock 객체로 대체할 수 없기 때문에 doSomething() 메서드가 호출되는지 여부를 확인하거나 그 동작을 테스트에서 제어하기 어렵다.

- 의존성 주입을 받는다고 하더라도 결국 받는 타입이 SingletonService 이므로 추상화 없이 구체적인 클래스에 여전히 의존하게 된다.
- ```java
  // 의존성 주입
  public Client(SingletonService service) {
      this.service = service;
  }
  ```

여기에 인터페이스를 사용해 SingletonService 를 구현하면

```java
public interface Service {
void doSomething();
}

public class SingletonService implements Service {
...위와 동일
}

```

Client 클래스는 인터페이스 Service 를 참조하도록 수정할 수 있다. 테스트시에 실제 SingletonService 대신 Mock 구현을 사용해 Client 를 테스트할 수 있다.

```java
public class Client {
private Service service;

    // 의존성 주입을 통해 Service 인스턴스를 받음
    public Client(Service service) {
        this.service = service;
    }

    public void performAction() {
        service.doSomething();
    }

}

// 테스트용 가짜(Mock) 서비스
public class MockService implements Service {
@Override
public void doSomething() {
System.out.println("Mock service doing something...");
}
}

// 테스트 코드
public class ClientTest {
    public static void main(String[] args) {
        Service mockService = new MockService();
        Client client = new Client(mockService);

        client.performAction();  // "Mock service doing something..." 출력
    }

}
```

## 2. 싱글턴을 만드는 방법

### 2.1. public static final 필드 방식의 싱글턴

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}
```

- private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 딱
- 한 번만 호출된다
- 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
  간결하다.
- public 이나 protected 생성자가 없으므로, 전체 시스템에서 단 하나의 인스턴스임이 보장된다.

### 2.2. 정적 팩터리 메서드 방식의 싱글턴

```java

public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}
```

정적 팩터리 메서드 방식은 위와 마찬가지로 생성자를 private으로 만들지만 인스턴스를 갖는 static final 필드를 private으로 두고 정적 팩터리 메서드에서 이를 반환한다.

책에서는 아래 3가지를 장점으로 들고 있다.

1. Elvis가 항상 같은 객체를 반환하는 것이 아니라 스레드별로 다른 객체를 반환하도록 변경할 수 있다.

   ```java
   public static Elvis getInstance() {
       return ThreadLocal.withInitial(() -> new Elvis()).get(); // 스레드별로 다른 인스턴스 반환
   }
   ```

2. 정적 팩터리를 제네릭 타입으로 만들어 재사용 가능하게 할 수 있다.
   ```java
   public static <T> T getSingleton(Class<T> type) throws Exception {
       return type.getDeclaredConstructor().newInstance();
   }
   ```
3. Elvis::getInstance와 같은 방식으로 메서드 참조를 사용할 수 있다. 예를 들어, 이를 Supplier<Elvis>로 사용할 수 있다.

   ```java
   Supplier<Elvis> supplier = Elvis::getInstance;
   ```

### 2.3. 열거 타입 방식의 싱글턴

대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

- 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.
- 열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있다.

```java
public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() { ... }
}
```

- public 필드 방식과 비슷하지만 더 간결하다.
- 추가 노력 없이 직렬화할 수 있다.
- 심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽하게 막아준다.

<br/>

## 3. 싱글톤 패턴의 문제점 (추가 정리)

유연성이 떨어진다.

- 싱글톤 패턴을 구현하는 코드가 추가적으로 필요하다.
- 클라이언트가 구체 코드에 의존하므로 DIP(의존관계 역전 원칙)를 위반한다고 볼 수 있다.
  - ex) 구체클래스.getInstance()
- 클라이언트가 구체 코드에 의존하므로 OCP(개방-폐쇄 원칙)를 위반할 가능성이 높다.
- 테스트하기 어렵다.
- 내부 속성을 변경하거나 초기화하기 어렵다.
- private 생성자로 자식 클래스를 만들기 어렵다.

> ### 참고
>
> **DIP(Dependency Inversion Principle)**  
> 고수준 모듈은 저수준 모듈에 의존해서는 안 된다. 둘 모두 추상화에 의존해야 한다.
> 추상화는 구체적인 사항에 의존해서는 안 됩니다. 구체적인 사항은 추상화에 의존해야 한다.
> 간단히 말해, DIP는 클래스 간의 의존 관계를 형성할 때 추상화와 인터페이스에 의존하도록 권장한다. 이를 통해 상위 수준의 모듈이 하위 수준의 모듈에 의존하지 않고, 두 모듈 모두 추상화에 의존함으로써 유연하고 확장 가능한 시스템을 구축할 수 있다.
>
> **개방 폐쇄의 원칙(OCP)**  
> 기존의 코드를 변경하지 않으면서, 기능을 추가할 수 있도록 설계가 되어야 한다는 원칙을 말한다.
> 보통 OCP를 확장에 대해서는 개방적(open)이고, 수정에 대해서는 폐쇄적(closed)이어야 한다는 의미로 정의된다.

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
        // private 생성자
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

- Singleton 클래스의 생성자가 private으로 선언된다.
- 외부에서 직접 객체를 생성할 수 없고, getInstance() 메소드를 통해 인스턴스를 얻어야 한다.
- 객체 수정이 필요하거나 대체하기 위해서는 기존 코드를 수정해야하기 때문에 수정에 폐쇄적이지 않다. `== 설계나 구조에 변경이 필요하다는 의미`
