# Item 30. 이왕이면 제네릭 메서드로 만들라

```java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```
이 코드에서는 두 가지 경고가 발생한다.
1. 생성자에 대한 경고
2. addAll에 대한 경고

메서드 선언에서의 세 집합(입력 2, 반환 1)의 원소 타입을 타입 매개변수로 명시하고, 
메서드 안에서도 이 타입 매개변수만 사용하도록 수정하면 된다.

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```
타입 매개변수 목록은 반환 타입 앞에 온다.  
위의 코드에서는 <E>가 타입 매개변수 목록이고, 반환 타입은 Set<E>이다.  
- 경고 없는 컴파일
- 타입 안전성
- 편리성

### 사용 예시
위의 제네릭 메서드를 사용한 예시이다.
```java
public static void main(String[] args) {
    Set<String> guys = Set.of("톰", "딕", "해리");
    Set<String> stooges = Set.of("래리", "모에", "컬리");
    Set<String> aflCio = union(guys, stooges);
    System.out.println(aflCio);
}
```
모두 같은 타입(String)만 담을 수 있다.

## 제네릭 싱글턴 팩터리
하지만, 종종 여러 타입을 사용해야하는 경우가 있다.  
제네릭은 컴파일타임에만 타입이 유지되고, 런타임에는 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화할 수 있다.  
이를 이용하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다.  
=> 이를 제네릭 싱글턴 팩터리라고 한다.

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```
- UnaryOperator<Object> 타입인 IDENTITY_FN을 UnaryOperator<T>로 형변환
- 값을 수정하지 않고 그대로 반환하므로 T가 어떤 타입이든, UnaryOperator<T>를 사용해도 안전하다.

## 재귀적 타입 한정
자기 자신이 들어간 표현식을 사용해 타입 매개변수의 허용 범위를 한정
- 모든  E는 자신과 비교할 수 있어야 한다는 것
```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty()){
        throw new IllegalArgumentException("컬렉션이 비어 있습니다.");
    }
    
    E result = null;
    for (E e : c) {
        if (result == null || e.compareTo(result) > 0) {
            result = Objects.requireNonNull(e);
        }
    }
    return result;
}
```