## 요약 정리

### 재정의하기 전 생각해 보아야 할 것들

- 꼭 필요한 경우가 아니라면 재정의하지 말자.
- 그래도 필요하다면 핵심필드를 빠짐 없이 비교하며 다섯 가지 규약을 지키자.
- 어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다. 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 더 싼 필드를 먼저 비교하자.
- 객체의 논리적 상태와 관련 없는 필드는 비교하면 안 된다.
- 핵심 필드로부터 파생되는 필드가 있다면 굳이 둘다 비교할 필요는 없다. 편한 쪽을 선택하자.
- equals를 재정의할 땐 hashCode도 반드시 재정의하자(아이템 11)
- equals의 매개변수 입력을 Object가 아닌 타입으로는 선언하지 말자.

### equals 좋은 재정의 방법

```java
@Override
public boolean equals(final Object o) {
    // 1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    if (this == o) {
        return true;
    }

    // 2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    if (!(o instanceof Point)) {
        return false;
    }

    // 3. 입력을 올바른 타입으로 형변환 한다.
    final Point p = (Point) o;


    // ... 이후 생략
}
```

4.입력 개체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.

모든 필드가 일치하면 true 를, 하나라도 다르면 false 를 반환한다.

float와 double을 제외한 기본 타입 필드는 == 연산자로 비교하고, 참조 타입 필드는 각각의 equals 메서드로 비교한다.

float와 double 필드는 각각 정적 메서드인
Float.compare(float, float)와 Double.compare(double, double)로 비교한다

- Float.equals 와 Double.equals 메서드를 대신 사용할 수 있지만 오토박싱을 수반할 수 있으니 성능상 좋지 않다.

배열 필드는 원소 각각을 앞서의 지침대로 비교한다. 배열의 모든 원소가 핵심 필드라면 Arrays.equals 메서드들 중 하나를 사용한다.

```java
private int[] favoriteNumbers; // 배열 필드

Arrays.equals(favoriteNumbers, person.favoriteNumbers); // 재정의하는 equals 에서 비교할 때
```

<br>
5.null 값도 정상 값으로 취급하는 참조 타입 필드는 정적 메서드인 Object.equals(Object, Object) 로 비교해 NullPointException 발생을 예방한다.

- Objects.equals() 메서드는 두 객체를 비교할 때 둘 다 null이면 true, 하나만 null이면 false, 둘 다 null이 아니면 해당 객체들의 equals() 메서드를 호출해 비교한다.

```java
public class Car {
    private String model;
    private String manufacturer; // 이 필드는 null일 수 있음

    public Car(String model, String manufacturer) {
        this.model = model;
        this.manufacturer = manufacturer;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Car)) return false;
        final Car car = (Car) o;
        // model과 manufacturer 비교 시, manufacturer는 null일 수도 있음
        return Objects.equals(model, car.model) &&
               //
               Objects.equals(manufacturer, car.manufacturer);
    }
    ...
}
```

### 검토하기

equals를 다 구현했다면 세 가지만 자문해보자

- 대칭적인가?
- 추이성이 있는가?
- 일관적인가?

자문에서 끝내지 말고 단위 테스트를 작성해 돌려보자

- > AutoValue 프레임워크를 사용하면 테스트를 작성하지 않아도 된다.
  > 클래스에 애너테이션 하나만 추가하면 AutoValue가 이 메서드들을 알아서 작성해주며, 직접 작성하는 것과 근본적으로 똑같은 코드를 만들어줄 것이다. 인텔리제이 IDE도 같은 기능을 제공하지만 생성된 코드가 AntoVale 만큼 깔끔하거나 읽기 좋지는 않다 또한 IDE는 나중에 클래스가 수정된 걸 자동으로
  > 알아채지는 못하니 테스트 코드를 작성해둬야 한다

<br>

## 1.equals 메서드 개념

### 1.1 기본적으로 정의되어있는 equals 메서드

equals() 메서드는 객체 간의 논리적 동등성 (logical equality) 를 비교하기 위해 사용되는 메서드다.

Object 클래스에 정의된 메서드로 **기본적으로는 참조 비교**를 수행한다.
즉, 두 객체가 동일한 메모리 주소를 참조하고 있는지 확인한다.
따라서, 기본적으로는 같은 인스턴스여야만 equals() 메서드가 true 를 반환한다.

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

따라서 별도의 재정의 없이 사용하면 **논리적인 비교가 아닌** 두 객체가 **메모리상 동일한 객체인지를 확인**하게 된다.

### 1.2 equals() 메서드를 재정의하는 이유

많은 클래스에서는 객체간의 논리적 동등성을 비교하는 것이 더 중요하므로, 두 객체가 서로 다른 메모리 주소를 참조하고 있더라도, 그 내용이 같으면 equals() 메서드가 true 를 반환하도록 하는 것이 더 합리적이다.

따라서, 논리적으로 동등한 객체를 판단하기 위해서는 equals() 메서드를 override 해야 한다.

String 클래스를 예시로 들면 equals() 메서드를 재정의해 문자열의 내용을 비교하도록 구성되어 있다. 문자열 리터럴 자체가 같으면 true 를 반환하도록 되어 있는 것이다.

```java
String str1 = new String("hello");
String str2 = new String("hello");

System.out.println(str1.equals(str2));  // true
```

## 2. equals() 메서드의 재정의

### 2.1 equals() 메서드를 재정의할 때

객체의 식별성(object identity = 두 객체가 물리적으로 같은가?) 이 아닌 논리적 동등성을 비교해야 하는데, 상위 클래스의 equals 가 논리적 동등성을 비교하도록 재정의되지 않았을 때 equals() 메서드를 재정의한다.

주로 **값 클래스**가 이에 해당한다. equals 가 논리적 동치성을 확인하도록 재정의해두면, 그 인스턴스는 값을 비교하길 원하는 프로그래머의 기대에 부응함은 물론 Map 의 키와 Set 의 원소로 사용할 수 있게 된다.

값 클래스라 해도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스(아이템1) 이라면 equals 를 재정의하지 않아도 된다.

- ex) enum 은 값 클래스라고 해도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하므로 equals 를 재정의하지 않아도 된다.
- 논리적으로 같은 인스턴스가 2개 만들어지지 않으니 논리적 동등성과 객체 식별성이 사실상 같은 의미가 된다.

### 2.2 equals 메서드를 재정의할 때 따라야 하는 규약 (요약)

null이 아닌 모든 참조 값 x, y, z에 대해,

- 반사성:
  - x.equals(x) == true
- 대칭성:
  - y.equals(x) == true 이면 x.equals(y) == true
- 추이성:
  - y.equals(z) == true, x.equals(z) == true 이면 x.equals(y) == true
- 일관성:
  - x.equals(y)를 반복해도 값이 변하지 않는다.
- null-아님:
  - x.equals(null) == false

<br>

## 3. equals 를 재정의 하지 않는 것이 최선인 경우

### 3.1. 각 인스턴스가 본질적으로 고유하다.

값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스(Thread)

### Thread

```java
@Override
public boolean equals(Object obj) {
    if (obj == this)
        return true;

    if (obj instanceof WeakClassKey) {
        Object referent = get();
        return (referent != null) && (referent == ((WeakClassKey) obj).get());
    } else {
        return false;
    }
}
```

### 3.2. 인스턴스의 '논리적 동등성(=동치성)'을 검사할 일이 없다.

equals를 재정의하여 각 인스턴스가 같은 정규표현식을 나타내는지 검사할 수 있지만,
그럴 일이 없다면 굳이 재정의할 필요가 없다.(Pattern)

```java
void patternTest() {
    final Pattern P1 = Pattern.compile("//.*");
    final Pattern P2 = Pattern.compile("//.*");

    System.out.println(P1.equals(P1)); // true
    System.out.println(P1.equals(P2)); // false
    System.out.println(P1.pattern()); // //.*
    System.out.println(P1.pattern().equals(P1.pattern())); // true
    System.out.println(P1.pattern().equals(P2.pattern())); // true
}
```

### 3.3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.

Set의 구현체들은 모두 AbstractSet이 구현한 equals를 상속 받아서 쓴다.
그 이유는 서로 같은 Set인지 구분하기 위해서 사이즈, 내부 값들을 비교하면 되기 때문이다.

```java
public boolean equals(Object o) {
    if (o == this) {
        return true;
    }

    if (!(o instanceof Set)) {
        return false;
    }

    Collection<?> c = (Collection<?>) o;
    if (c.size() != size()) { // 사이즈 비교
        return false;
    }
    try {
        return containsAll(c); // 내부 인스턴스 비교
    } catch (ClassCastException | NullPointerException unused) {
        return false;
    }
}
```

그런데 그로 인해서 이런 일도 생긴다.

```java
void setTest() {
    Set<String> hash = new HashSet<>();
    Set<String> tree = new TreeSet<>();
    hash.add("Set");
    tree.add("Set");

    System.out.println(hash.equals(tree)) // true;
}
```

### 3.4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

```java
@Override
public boolean equals(Object o){
    throw new AssertionError(); // 호출 금지
}
```

<br>

## 4.

### 4.1. 반사성

- null이 아닌 모든 참조 값 x에 대해 x.equals(x)를 만족해야 한다.
- 자기 자신과 같아야 한다.

<br>

### 4.2. 대칭성

- null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면, y.equals(x)가 true를 만족해야 한다.
- 서로에 대한 동치 여부가 같아야 한다.

```java
public final class CaseInsensitiveString {

    private final String str;

    public CaseInsensitiveString(String str) {
        this.str = Objects.requireNonNull(str);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return str.equalsIgnoreCase(((CaseInsensitiveString) o).str);
        }

        if (o instanceof String) { // 한 방향으로만 작동한다.
            return str.equalsIgnoreCase((String) o);
        }
        return false;
    }
}

void symmetryTest() {
    CaseInsensitiveString caseInsensitiveString = new CaseInsensitiveString("Test");
    String test = "test";
    System.out.println(caseInsensitiveString.equals(test)); // true
    System.out.println(test.equals(caseInsensitiveString)); // false
}
```

<br>

### 4.3. 추이성

- null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고,
  y.equals(z)가 true이면 x.equals(z)도 true이다.

```java
public class Point {

    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return this.x == p.x && this.y == p.y;
    }
}
```

```java
public class ColorPoint extends Point {

    private final Color color;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
        if (!(o instanceof ColorPoint)) {
            return o.equals(this);
        }

        // o가 ColorPoint이면 색상까지 비교한다.
        return super.equals(o) && this.color == ((ColorPoint) o).color;
    }
}
```

```java
void transitivityTest() {
    ColorPoint a = new ColorPoint(2, 3, Color.RED);
    Point b = new Point(2, 3);
    ColorPoint c = new ColorPoint(2, 3, Color.BLUE);

    System.out.println(a.equals(b)); // true
    System.out.println(b.equals(c)); // true
    System.out.println(a.equals(c)); // false
}
```

<br>

#### 4.3.1. 추이성(무한 재귀)

```java
public class SmellPoint extends Point {

    private final Smell smell;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
        if (!(o instanceof SmellPoint)) {
            return o.equals(this);
        }

        // o가 ColorPoint이면 색상까지 비교한다.
        return super.equals(o) && this.smell == ((SmellPoint) o).smell;
    }
}

void infinityTest() {
    Point cp = new ColorPoint(2, 3, Color.RED);
    Point sp = new SmellPoint(2, 3, Smell.SWEET);

    System.out.println(cp.equals(sp));
}
```

![](stackOverflowError.png)

<br>

#### 4.3.2. 추이성(리스코프 치환 원칙)

만약 추이성을 지키기 위해서 Point의 equals를 각 클래스들을 getClass를 통해서
같은 구체 클래스일 경우에만 비교하도록 하면 어떨까?

```java
@Override
public boolean equals(Object o) {
    // getClass
    if (o == null || o.getClass() != this.getClass()) {
        return false;
    }

    Point p = (Point) o;
    return this.x == p.x && this.y = p.y;
}
```

이렇게 되면 동작은 하지만, 리스코프 치환원칙을 지키지 못하게 된다.
이는 모든 객체 지향 언어의 동치관계에서 나타나는 근본적인 문제이며,
객체 지향적 추상화의 이점을 포기하지 않는 한 불가능하다.

<br>

### 4.3.3. 추이성(상속 대신 컴포지션(아이템 18)을 사용해라)

상속 대신 컴포지션을 사용해라(아이템 18)

```java
public class ColorPoint2 {

    private Point point;
    private Color color;

    public ColorPoint2(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return this.point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return this.point.equals(cp) && this.color.equals(cp.color);
    }
}
```

<br>

#### 4.3.4. 추이성(추상 클래스)

추상 클레스의 하위 클래스에서라면 equals의 규약을 지키면서도 값을 추가할 수 있다.

- 태그 달린 클래스보다는 클래스 계층구조를 활용하라(아이템 23)
  이와 같은 구조에서는 상위 클래스를 직접 인스턴스로 만드는 것이 불가능 하기 때문에
  지금까지 이야기한 문제들은 모두 일어나지 않는다.

<br>

### 4.4. 일관성

- null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

```java
@Test
void consistencyTest() throws MalformedURLException {
    URL url1 = new URL("www.xxx.com");
    URL url2 = new URL("www.xxx.com");

    System.out.println(url1.equals(url2)); // 항상 같지 않다.
}
```

java.net.URL 클래스는 URL과 매핑된 host의 IP주소를 이용해 비교하기 때문에 같은
도메인 주소라도 나오는 IP정보가 달라질 수 있기 때문에 반복적으로 호출할 경우 결과가
달라질 수 있다.

따라서 이런 문제를 피하려면 equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산을 수행해야 한다.

<br>

### 4.5. null-아님

- null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false

`o.equals(null) == true`인 경우는 상상하기 어렵지만, 실수로
`NullPointerException`을 던지는 코드 또한 허용하지 않는다.

이러한 `Exception`을 막기 위해서 여러가지 방법이 존재하는데, 그중 하나는 null인지 확인을 하고
`false`를 반환하는 것이다. 하지만 책에서는 다른 방법을 추천하고 있다.

```java
@Override
public boolean equals(Object o) {
    // 우리가 흔하게 인텔리제이를 통해서 생성하는 equals는 다음과 같다.
    if (o == null || getClass() != o.getClass()) {
        return false;
    }

    // 책에서 추천하는 내용은 null 검사를 할 필요 없이 instanceof를 이용하라는 것이다.

    // instanceof는 두번째 피연산자(Point)와 무관하게 첫번째 피연산자(o)거 null이면 false를 반환하기 때문이다.
    if (!(o instanceof Point)) {
        return false;
    }
}

```
