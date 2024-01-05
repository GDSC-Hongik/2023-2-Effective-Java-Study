### Item11 : equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스에서는 hashCode도 재정의해야 한다.  
그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를
HashMap이나 HashSet같은 컬렉션의 원소로 사용할 때 문제가 발생한다.

###### Object 명세에 나와 있는 규약
>- equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode 도 변하면 안 된다.
(애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없음)
>- equals가 두 객체가 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환한다.
→ 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
>- equals가 두 객체를 다르다고 판단했더라도, hashCode는 꼭 다를 필요는 없다.
하지만, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

hashCode를 잘못 재정의할 경우 문제가 되는 것은 두 번째에 적혀 있는 규약이다.  
이전 item에서 봤듯이 equals 메서드는 물리적으로 다른 객체라도 논리적 동치에 의해 동일하다고 판단을 내릴 수 있다.
그렇게 때문에 equals에 의해 동일한 객체라고 판단할 경우 hashCode값도 동일한 값을 반환해주는 것이 옳다.  
하지만 Object의 hashCode 메서드는 물리적 동치인 경우에만 같은 hashCode값을 반환하므로 재정의 시 주의해야 한다.

###### hashCode를 재정의하지 않을 경우 발생하는 문제 
이전 item에서 본 PhoneNumber 클래스를 이용해서 확인하고자 한다.  
PhoneNumber 클래스는 `equals`는 규약에 맞게 재정의했지만
`hashCode`는 재정의하지 않았다.  
그래서 다음과 같이 HashMap에 인스턴스를 넣고 get으로 해당 인스턴스를 찾으려 하면 `null`이 반환된다.
```java
Map<PhoneNumber, String> map = new HashMap<>();
map.put(new PhoneNumber(031,123,5678), "Sponge Bob");
map.get(new PhoneNumber(032,123,5678)); // null 반환
```
- 논리적으로 같은 두 객체이지만 hashCode를 재정의하지 않았기 때문에 서로 다른 hashCode값을 반환한다. 그래서 엉뚱한 해시 버킷에서 해당
인스턴스를 찾기에 null이 반환된다.
- 설령 두 객체가 같은 해시 버킷에 들어갔다고 하더라도 HashMap 자체가 서로 다른 hashCode를 가지는 인스턴스에 대해서는 동치 비교를 하지 않도록 구현됐기에
마찬가지로 null이 반환된다.

위 문제는 PhoneNumber 클래스에 hashCode 메서드를 재정의 해주면 해결된다.

###### hashCode 잘못된 예시
처음에 언급한 규약들을 준수하는 hashCode 메서드를 정의하려고 하면 사실 굉장히 간단하다.
```java
@Override 
public int hashCode() {
    return 42;
}
```
- 위 hashCode는 모든 인스턴마다 같은 값을 반환한다. 그리고 놀랍게도 이는 모든 규약을 만족한다..!  
- 하지만 이렇게 되면 같은 해시 버킷에 모든 객체가 담기게 되므로 해시 테이블의 성능이 급격하게 나빠진다. 이는 해시 테이블이 마치 연결 리스트처럼 동작하게 한다.

> 좋은 hashCode 메서드는 각 인스턴스마다 서로 다른 값을 반환해야 한다.

###### 좋은 hashCode 메서드 구현하는 법
1. int 변수인 result를 선언한 후 값을 c로 초기화한다.
이 때, `c는 해당 객체의 첫번째 핵심 필드를 단계 2.i방식으로 계산한 해시코드`이다.
여기서 핵심 필드는 equals 비교에 사용되는 필드를 말한다.
2. 해당 객체의 나머지 핵심 필드인 f 각각에 대해 다음 작업을 수행한다.
   1. 해당 필드의 해시코드 c 를 계산한다.
      - 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본타입의 박싱 클래스다(ex.Integer, Short ...).
      - 참조 타입 필드면서, 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출하여 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다.
      - 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
   2. 단계 2.i에서 계산한 해시코드 c로 result를 갱신한다.
   result = 31 * result + c;
3. result를 반환한다.

이때 주의해야 할 점은 equals 메서드에서 비교하지 않은 필드에 대해서는 계산을 하면 안된다. 자칫하면 hashCode의 두 번째 규약을 어길 위험이 있다.

위 글만 읽어서는 굉장히 이해하기가 어렵다. 이는 예시 코드를 보면 비교적 이해가 쉽다.
```java
@Override
public int hashCode() {
    int result = Integer.hashCode(areaCode);
    result = 31 * result + Integer.hashCode(prefix);
    result = 31 * result + Integer.hashCode(lineNum);
    return result;
}
```
메서드를 보면 필수 필드 중 첫 번째 필드의 hashCode값을 result에 저장한다.  
그 뒤에 각 필수 필드에 대해 hashCode값을 구하고
그 값을 이용하여 result를 업데이트하는 방식이다.  
이때 `31 * result` 라는 식은 필드를 곱하는 순서에 따라 서로 다른 해시값이 나오게 해주는 방법이라고 한다.
그래서 클래스에 비슷한 필드가 여러 개 있더라도 매번 다른 해시값을 생성해낼 수 있다.

###### 흠.. 직접 구현하기 귀찮은데요?
그렇다면 `Object` 클래스에서 제공하는 정적 메서드인 `hash` 메서드를 이용하자.  
이 메서드는 위에서 구현한 것과 비슷한 방식으로 해시값을 생성해서 반환 해준다.  
**다만, 속도는 조금 느리다.**  
매개변수로 받는 객체들을 저장할 배열도 생성해야 하고, 그중에 기본 타입이 있다면 wrapper 클래스로 Boxing도 해줘야 하기 때문이다.`
```java
@Override
public int hashCode() {
    return Objects.hash(lineNum,prefix,areaCode);
}
```

###### 불변 클래스인데 인스턴스의 해시 코드 계산 비용이 큰 경우..
이때는 한번 해시 코드를 생성하여 캐싱한 뒤에 지속적으로 재사용하는 것이 좋다. 이때 사용하는 방법 2가지가 있다
- 해당 인스턴스 생성 시점에 해시 코드값 계산 후 캐싱
```java
public class PhoneNumber {
    private final short areaCode, prefix, lineNum;
    private final int hashCode;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = (short) areaCode;
        this.prefix = (short) prefix;
        this.lineNum = (short) lineNum;

        int result = Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        hashCode = result;
    }

    @Override
    public int hashCode() {
         return hashCode;
    }
}
```
- 해당 인스턴스의 hashCode 메서드 호출 시점에 지연 초기화(lazy initialization) 후 캐싱
```java
private int hashCode; // 0으로 자동 초기화

@Override
public int hashCode() {
    int result = hashCode;
    if(result == 0) {
        int result = Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        hashCode = result;
    }
    return result;
}
```
지연 초기화 방식을 사용하는 경우 thread 안정성을 고려하여 구현해야 한다.

###### 주의사항
- 성능을 위해 필수 필드를 계산 과정에서 제외하면 안된다.
  - 필드 개수가 적어지면 당연히 속도는 빨라지겠지만 그만큼 반환되는 해시값의 품질이 떨어져 해시 테이블의 성능이 굉장히 떨어진다.
  - 만약 특정 필수 필드를 제외하고 계산을 했는데 마침 해당 필드가 특정 해시값 영역에 몰린 인스턴스들을 더 넓은 범위의 해시값으로 분산시켜주는 역할을 맡고 있었다면
  성능 면에서 오히려 손해를 보게 된다.

## 핵심 정리
    equals를 재정의할 때는 hashCode도 반드시 재정의하자. 처음 언급한 규약들을 준수하고  
    서로 다른 인스턴스에 대해서는 되도록이면 서로 다른 해시값을 반환하도록 구현하자.