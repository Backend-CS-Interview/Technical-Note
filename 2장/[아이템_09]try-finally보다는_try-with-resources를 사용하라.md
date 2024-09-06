기존 방식인 ` try-finally`는 자원을 닫는 중에 예외가 발생하면 원래 예외를 덮어버릴 수 있어 문제가 발생할 수 있다. `try-with-resources` 를 사용하면 자원을 안전하게 닫을 수 있으며, 자원을 닫는 중 발생한 예외도 부가 예외로 처리되므로 원래 예외를 잃지 않습니다.

## 1. try-finally 방식의 문제점

자바에는 close를 호출해 직접 닫아줘야 하는 자원이 있다.

- ex) InputStream, OutputStream, java.sql.Connection 등

```java
public static String inputString() throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    String result = br.readLine(); // IOException 이 발생할 수 있다.
    br.close();
    return result;
}
```

위 코드는 BufferedReader를 사용해준 뒤에 close를 호출해 직접 닫아주는 코드다. 하지만 위 방법대로 close를 호출하게 되면 문제가 발생할 수 있다.

br.readLine() 에서 IOException 이 발생하면 그 즉시 메서드가 종료되는데, br.close() 를 호출되지 않으므로, 스트림이 닫히지 않고 남게 된다.

이처럼 BufferedReader 와 같은 자원을 사용할 때, 자원이 닫히지 않으면 시스템 리소스를 차지한 상태로 남아, 메모리 누수나 파일 핸들 잠금 등의 문제가 발생할 수 있다.

따라서 예외가 발생하더라도 자원을 닫을 수 있도록 해줘야 하는데, 아래는 try-finally 문을 사용해서 close 처리를 해준 예시다.

```java
public static String inputString() throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

finally 블록은 try, catch 블록이 끝난 뒤 실행할 로직을 정의해 주는 블록이다. 따라서 IOException이 발생하게 되더라도 상위 메서드로 IOException 객체를 던져준 뒤 finally 메서드를 종료하게 된다.

close를 여러 번 호출해야 하는 상황이 오면 아래처럼 만들 수 있겠지만, 코드가 굉장히 지저분해지게 된다.

또, 아래 코드는 결함이 존재하는데 다음과 같다.

기기에 물리적인 문제가 생길 경우 br.readline(), br.close() 가 실패할 것이다. 이때 br.readLine() 에서 IOException 이 발생하지만, 자바는 기본적으로 하나의 예외만 상위 호출스택으로 전달하기 때문에 두 번째 예외가 발생하면 첫 번째 예외를 잃어버리게 된다.

여기서는 br.clode() 가 실패해서 발생한 예외가 상위로 전달되고, br.readline() 에서 발생한 예외가 무시될 것이다.이는 디버깅을 복잡하게 만들고, 문제가 발생한 원인을 파악하기 어렵게 만든다.

두 번째 예외 대신 첫 번째 예외를 기록하도록 코드를 수정할 수 있지만 코드가 너무 지져분해져서 이 책의 저자의 말에 따르면 그렇게까지 하는 경우는 별로 없다고 한다.

```java
public static void inputAndWriteString() throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    try {
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
        try {
             String line = br.readLine(); // 예외 발생 가능
            bw.write(line);
        } finally {
            bw.close();
        }
    } finally {
        br.close();
    }
}
```

아래 예제 코드를 실행시켜보면 main의 catch 블록에서 예외를 printStackTrace 호출한 결과가 아래처럼 나온다.

```
실행 결과

java.lang.NullPointerException
    at Application.check(Application.java:14)
    at Application.main(Application.java:4
```

finally 블록에서 던져진 NullPointerException을 catch 하고 try 블록의 IllegalArgumentException은 무시된 것을 볼 수 있다.

```java
public class Application {

    public static void main(String[] args) {
        try {
            check();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void check() {
        try {
            throw new IllegalArgumentException();
        } finally {
            throw new NullPointerException();
            // 예외는 관련 없이 임의로 선택했다.
        }
    }
}
```

정리하자면 try-finally는 가독성을 해칠 가능성이 높으며, 예외 처리 로직을 작성하기에 미묘한 결점이 존재한다.

## 2. try-with-resources 로 해결

이러한 try-finally 방식의 단점을 보완하기 위해 자바 7 버전부터는 try-with-resources가 도입되었다.

> **try-with-resources 동작 방식**  
> try 구문에서 자원을 선언하면 자원은 자동으로 close() 된다.  
> 자원을 처리하는 중 예외가 발생하고, 닫는 중에 예외가 발생하더라도 두 번째 예외는 부가 예외(Suppressed Exception) 으로 처리된다.  
> 원래 발생한 예외는 상위 호출 스택으로 전달되고, 자원을 닫는 중 발생한 예외 또한 무시되지 않고 기록된다.  
> Throable.getSuppressed() 메서드를 통해 부가예외를 확인할 수 있다.
>
> ```java
>    public static void main(String[] args) {
>         try {
>            inputAndWriteString();
>        } catch (IOException e) {
>            // 원래 발생한 예외
>            System.out.println("Main exception: " + e);
>
>            // 자원을 닫는 중 발생한 부가 예외
>            for (Throwable suppressed : e.getSuppressed()) {
>                System.out.println("Suppressed exception: " + suppressed);
>            }
>        }
>    }
> ```

````
try-with-resources를 사용하기 위해서는 AutoCloseable 인터페이스를 구현한다.

```java
public interface AutoCloseable {

    void close() throws Exception;
}
````

AutoCloseable 인터페이스는 close 메서드 하나만을 정의해 놓은 간단한 인터페이스이며, 자바 라이브러리와 서드파티 라이브러리의 수많은 클래스와 인터페이스는 이미 AutoCloseable을 구현하거나 확장해두었다.

아래는 inputString 메서드를 try-with-resources 방식으로 재구성한 코드다.

```java
public static String inputString() throws IOException {
    try (BufferedReader br = new BufferedReader(new InputStream(System.in))) {
        return br.readLine();
    }
}
```

위의 try-finally 와 비교해 보았을 때

- 가독성이 향상되었다.
- 예외가 발생했을 때 디버깅 하기에도 더 편리하다.
  - 아까 가정한 상황처럼 inputString 메서드의 readLine과 close 모두에서 예외가 발생하는 경우, close(물론 코드 상으로는 보이지 않지만) 호출 시 발생하는 예외는 숨겨지고 readLine의 예외가 기록된다.
  - 이렇게 숨겨진 예외는 무시되는 것이 아니라, suppressed 상태가 되어 stackTrace 시 숨겨졌다는 메시지로 출력된다. suppressed 상태가 된 예외는 자바 7부터 도입된 getSuppressed 메서드를 통해 가져와서 사용할 수 있다.

다음의 테스트 코드를 보자.

```java
public class Application {

    public static void main(String[] args) {
        try {
            check();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void check() throws Exception {
        try (IllegalArgumentExceptionThrower thrower = new IllegalArgumentExceptionThrower()) {
            throw new NullPointerException();
        }
    }

    //
    static class IllegalArgumentExceptionThrower implements AutoCloseable {
        @Override
        public void close() throws Exception {
            throw new IllegalArgumentException();
        }
    }
}
```

```
실행 결과

java.lang.NullPointerException
    at Application.check(Application.java:17)
    at Application.main(Application.java:9)
    Suppressed: java.lang.IllegalArgumentException
        at Application$IllegalArgumentExceptionThrower.close(Application.java:24)
        at Application.check(Application.java:16)
        ... 1 more
```

직접 throw 한 NullPointerException이 catch되어 출력되며, 이 때 close 메서드에서 던지는 IllegalArgumentException은 Suppressed: 태그 뒤로 출력되는 것을 볼 수 있다.

## try-with-resources와 catch

try-with-resources 구조 역시 기존 try-finally 처럼 catch를 병용해서 사용할 수 있다.

```java
public static String inputString() {
    try (BufferedReader br = new BufferedReader(new InputStream(System.in))) {
        return br.readLine();
    } catch (IOException e) {
        return "IOException 발생";
    }
}
```

close를 통해 회수해야 하는 자원을 다룰 때는 try-finally를 사용하는 대신 **반드시** try-with-resources를 사용하자. 보다 가독성 좋으며, 쉽고 정확하게 자원을 회수할 수 있다.
