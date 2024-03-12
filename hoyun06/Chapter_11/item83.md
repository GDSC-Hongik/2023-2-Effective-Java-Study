### Item83 : 지연 초기화는 신중히 사용하라

###### 지연 초기화
지연 초기화는 필드의 초기화 시점을 해당 필드가 실제로 사용되는 시점까지 미루는 기법이다.  
물론 이 방법은 객체의 생성 비용은 줄이지만 해당 필드의 접근 비용은 증가시킨다. 그리고 대부분의 지연 초기화는
최적화 수단으로 사용되는데 그렇다고 해서 항상 성능 개선이 있는 것도 아니다.  
그럼에도 지연 초기화를 사용해야 하는 경우에는 다음과 같이 사용하자.

###### 인스턴스 필드의 지연 초기화
```java
private FieldType field;

private synchronized FieldType getField() {
    if (field == null) {
        field = computeFieldValue();
    }
    return field;
}
```

###### 정적 필드의 지연 초기화
```java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static Fieldtype getField() {
    return FieldHolder.field;
}
```
자바에선 static 클래스 혹은 static 필드라고 해서 프로그램 시작 시 무조건 메모리 로딩이 완료되며
초기화가 진행되는 것이 아니다. 실제 해당 클래스가 인스턴스화 되거나 아니면 내부 필드가 사용되는 시점에
jvm 메모리에 해당 클래스 혹은 필드가 로딩되며 초기화도 진행된다.  
이 매커니즘을 이용하여 static 필드가 실제 사용되는 시점에 초기화가 이뤄지도록 한 것이
위 코드 예시에서 사용한 `지연 초기화 홀더 클래스` 방식이다.

###### 인스턴스 필드의 지연 초기화 - 이중검사 관용구
인스턴스 필드를 지연 초기화해야 하는 경우에 사용할 수 있는 방법이 또 있는데 바로 
`이중검사 관용구`를 사용하는 것이다.
```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result != null) {
        return result;
    } 
    synchronized(this) {
        if (field == null) {
            field = computeFieldValue();
        }
        return field;
    }
}
```
이 관용구는 이름 그대로 `이중 검사`를 실시한다.  
첫 번째 검사에서는 `락`없이 필드를 검사한다. 만약 해당 필드가 null이 아니라면
그대로 필드를 반환한다. 이때 해당 필드에는 `volatile` 이 적용되었으므로 가장 최근에
쓰여진 값이 반환됨이 보장된다.  
만약, 첫 번째 검사를 통과하지 못하면 이번엔 `락`을 가지고 재검사를 실시한다. 그럼에도 필드가 null인 경우에만
초기화를 실시하고 해당 값을 반환한다.

위 방법의 변형으로 `단일검사 관용구`라는 것도 있는데 이것도 이름 그대로 이해하면 된다.
```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result == null) {
        field = result = computeFieldValue();
    }
    return result;
}
```
여기선 검사를 한 번만 실행하고 만약 필드가 null이라면 바로 초기화를 진행한다. 따라서, 초기화가 여러 번
진행될 수도 있다는 특징이 있다. 이 경우에도 필드에는 `volatile`이 적용되었기에 가장 최근에 쓰여진 값이 반환됨은 보장된다.

이상으로 설명한 위 방식들은 객체 참조 필드 및 기본 타입 필드 모두에 적용 가능하다.  
기본 타입의 경우 null 대신 0을 사용하면 된다.

## 핵심 정리
    대부분의 상황에선 일반 초기화가 지연 초기화 보다 훨씬 낫다.
    하지만 지연 초기화를 해야 하는 경우 올바른 방법을 사용하자