# Item 11. equals를 재정의하려거든 hashCode도 재정의하라

Object 명세에 따르면, equals가 같다고 판단한 두 객체는 같은 hashCode를 반환해야 한다.  
즉, 논리적으로 동일한 객체는 같은 hashCode를 반환해야 한다.  

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");

m.get(new PhoneNumber(707, 867, 5309));
```

해시코드를 재정의하지 않으면 get으로 찾으려 해도 "제니"가 반환되지 않고 null이 반환된다.  
두 new PhoneNumber(707, 867, 5309)가 논리적 동치임에도 다른 해시코드를 반환하기 때문이다.  
아래는 전형적인 hashCode 메서드의 예시이다.
```java
@Override
public int hashCode(){
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```