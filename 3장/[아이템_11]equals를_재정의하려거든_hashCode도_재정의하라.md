> **"equlas(Object) 두 객체가 같다고 판단했으면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다."**

즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");

m.get(new PhoneNumber(707, 867, 5309)); // "제니"가 아닌, null이 출력!!!!
```
- hashcode를 재정의하지 않았기 때문에 논리적 동치인 두 객체가 서로 다른 해시코드를 반환한다.
<center><img src="https://blog.kakaocdn.net/dn/sMxPO/btrNAyvpegC/UPc1EBboouzu0WZnc9bUfk/img.png" width="60%" ></center>

## hashCode 재정의 방법
### 사용 금지
```java
@Override
public int hashCode() {
    return 32;
}
```
- 모든 인스턴스의 hashCode가 동일하다.
- 해시 테이블의 수행 시간이 O(1) 에서 O(N) 으로 느려진다.
    - 해시 테이블의 모든 객체가 같은 버킷에 저장되기 때문에 매번 해시 충돌이 발생한다. 해시 테이블은 일반적으로 각 버킷에 연결 리스트를 사용하여 충돌을 해결한다. 동일한 해시 값을 가진 객체들은 해당 버킷의 연결 리스트에 저장이 되는데, 객체를 찾기 위해 리스트를 탐색할 경우 탐색시간이 O(n)으로 늘어난다. 

### hashCode 재정의 요령
1. `int result = c`
    - c는 해당 객체의 첫번째 핵심 필드를 2단계로 계산한 해시코드
    - 핵심 필드 : equlas 비교에 사용되는 필드
2. 나머지 핵심 필드에 대해 다음을 반복한다. (해시코드 c 계산 후, result 갱신)
    1. 기본 타입 필드라면 `Type.hashCode(필드)`를 수행
        - Type은 해당 기본 타입의 박싱 클래스
    2. 참조 타입 필드라면 이 필드의 hashCode를 재귀적으로 호출, 계산이 복잡해질 것 같으면 표준형을 만들어서 그 표준형의 hashCode를 호출
        - 필드의 값이 null이면 0을 사용한다. 
    3. 배열 필드라면 핵심 원소 각각을 2번의 방식으로 계산&갱신, 배열에 핵심 원소가 하나도 없으면 0을 사용, 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용
    4. 계산한 c로 result를 갱산한다. 
        - `result = 31 * result + c`
        - 31인 이유 : 31 * i = (i << 5) - i 
            - 시프트 연산과 뺄셈으로 최적화 가능
3. result를 반환한다.


#### 전형적인 hashCode 메서드
```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

#### Objects.hash 사용
```java
@Override
public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```
- 입력 인수를 받기 위한 배열 생성, 입력 중 기본타입이 있다면 박싱&언박싱 수행
- 따라서, 속도 느림 

#### 클래스가 불변이고 해시코드를 계산하는 비용이 크다면
- 매번 새로 계산하기보다는 캐싱을 고려한다.
- 해시의 키로 사용되지 않는경우라면 hashCode가 처음 불릴 때 계산하는 지연 초기화를 적용한다. 
```java
private int hashCode; // 0으로 초기화

@Override
public int hashCode() {
    int result = hashCode;
    if(result == 0){
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```

### 주의 사항
- 파생 필드(다른 필드로부터 계산해 낼 수 있는 필드)는 해시코드 계산에서 제외한다.
- equals 비교에 사용되지 않는 필드는 계산에서 '반드시' 제외한다. 
- hashCode 값의 생성 규칙을 API 사용자에게 자세히 공표하지 마라.