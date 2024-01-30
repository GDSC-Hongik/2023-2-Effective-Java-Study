# Item 43. 람다보다는 메서드 참조를 사용하라

람다보다도 간결한 방법  
=> 메서드 참조(method reference)

```java
map.merge(key, 1, (count, incr) -> count + incr);
```

이 코드를 보면, count와 incr는 큰 역할 없이 공간만 차지  
두 값을 더한다는 의미를 가지는 것을 메서드 참조로 대체하면 더 간결해짐.
```java
map.merge(key, 1, Integer::sum);
```
<=> 종종 람다식의 변수명이 더 의미가 있을 때도 있음.  
람다는 길지만 메서드 참조보다 가독성이나 유지보수의 면에서 간단할 수도.

### 메서드 참조보다 람다가 간결한 케이스
```java
public class GoshThisClassIsHumongous {
    //...
    service.execute(GoshThisClassNameIsHumongous::action);
}
```

위와 같은 경우에는 람다식을 사용하는 편이 간단해진다.
```java
public class GoshThisClassIsHumongous {
    //...
    service.execute(() -> action());
}
```

### 메서드 참조의 유형
1. 정적 메서드를 참조하는 메서드 참조
2. 수신 객체를 특정하는 한정적 인스턴스 메서드 참조
3. 수신 객체를 특정하지 않는 비한정적 인스턴스 메서드 참조
4. 클래스 생성자를 가리키는 메서드 참조
5. 배열 생성자를 가리키는 메서드 참조
