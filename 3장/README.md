# 3장: 모든 객체의 공통 메서드

final 이 아닌 Object 메서드들을 언제 어떻게 재정의해야하는지 다룬다.
Comparable.compareTo의 경우 Object의 메서드는 아니지만 성격이 비슷하여 이
번 장에서 함께 다룬다.

## Object 클래스

객체를 만들 수 있는 구체 클래스지만 기본적으로 상속해서 사용하도록 설계되어있다.

final 이 아닌 메서드 (equals, hashCode, toString, clone, finalize) 는 모듀 재정의 (overriding) 을 염두해 두고 설계되었다.

- 재정의시 지켜야 하는 일반 규약이 명확히 정의되어 있다.
- 일반 규약에 맞게 재정의하지 않는다면 대상 클래스가 이 규약을 준수한다고 가정하는 클래스 (HashMap, HashSet 등)를 오동작하게 만들 수 있다.
