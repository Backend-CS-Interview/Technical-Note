자바에서 제공하는 객체소멸자는 2개입니다. 하나는 finalizer이고 다른 하나는 cleaner입니다.

finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요합니다. 오동작, 낮은 성능도 문제의 원인이 됩니다. cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측 불가에 느리고 일반적으로 불필요합니다. 또한 둘 모두 가비지 컬렉터가 객체를 수거할 때 실행되어, 즉시 수행된다는 보장이 없습니다. 따라서 제때 실행되어야 하는 작업을 하면 안됩니다.

성능 문제도 동반합니다. try-with-resources로 가비지 컬렉터가 수거할 때까지 12ns걸린 반면, finalizer는 550, cleaner는 500ns가 걸렸습니다. 안전망 방식으로는 빠르긴 했지만 안전망 설치로 인해 성능이 5배 느려집니다.

보안문제도 발생합니다. 생성자나 직렬화 과정에서 예외가 발생하면, 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있는데, 이는 있어서는 안됩니다. 이 finalizer가 자신의 참조를 정적 필드에 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있습니다. final 클래스는 누구도 하위클래스를 만들 수 없으니 final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일 하지 않는 finalizer 메서드를 만들고 final로 선언하면 됩니다.

```java
public class SecureClass {
    
    // 아무 일도 하지 않는 finalize 메소드, 그리고 이를 final로 선언
    @Override
    protected final void finalize() throws Throwable {
        // 의도적으로 아무 작업도 수행하지 않음
    }
    
    // 다른 클래스의 기능들
    public void doSomething() {
        System.out.println("Important work done.");
    }

    public static void main(String[] args) {
        SecureClass secure = new SecureClass();
        secure.doSomething();
    }
}
```

그렇다면 finalizer나 cleaner 대신 무엇을 하면 될까요? AutoCloseable을 구현 후 클라이언트가 인스턴스를 다 쓰면 close 메서드를 호출하면 됩니다(이 때 try-with-resources를 사용하면 됩니다). 각 인스턴스는 자신이 닫혔는지 추적하는 것이 좋습니다. 즉 close 메서드에서 객체가 더 이상 유효하지 않음을 기록하고 다른 메서드는 이 필드를 검사하여 닫힌 후 불렀으면 IllegalStateException을 던지는 것입니다.

그럼 이것은 언제 사용하는가 궁금할 것입니다. 첫번째는 close 메소드를 호출하지 않는 것에 대한 안전망 역할입니다.  자원 회수를 늦게라도 하는 것이 낫다는 것입니다. 두번째는 네이티브 피어(native peer)와 연결된 객체에서입니다. 네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 의미합니다. 다시 말하면 C나 C++과 같은 언어에서 생성되고 관리되는 객체입니다. 이러한 객체는 Java의 관리 하에 있지 않기 때문에, GC의 대상이 되지 않습니다. 또한, try-with-resources 구문의 경우 Java 내부의 자원을 자동으로 해제해 주는 기능이기 때문에 네이티브 피어 객체의 경우 Cleaner를 이용하여 직접 객체를 회수해야 합니다.

>네이티브 피어 객체란 Java가 아닌 네이티브 코드(C, C++, 혹은 어셈블리어 등) 사이에 존재하는 중간다리 역할을 하는 객체입니다. 주로 JNI(Java Native Interface)를 사용할 때 등장하는 개념입니다. gradle, 그리고 maven을 사용하기에 최근에는 잘 사용되지 않는다고 합니다.
>


https://www.youtube.com/watch?v=6kNzL1bl1kI
(위 링크는 java에서 finalizer 공격에 대한 영상입니다.)
