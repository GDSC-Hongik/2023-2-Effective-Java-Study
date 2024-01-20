### Item27 : 비검사 경고를 제거하라

###### 제네릭을 사용하면서 생기는 컴파일러 경고들
1. 비검사 형변환 경고
2. 비검사 메서드 호출 경고
3. 비검사 매개변수화 가변인수 타입 경고
4. 비검사 변환 경고

###### 비검사 경고 제거의 필요성
비검사 경고를 모두 제거하면 `타입 안전성`이 보장된다.  
즉, 런타임에 ClassCastException이 발생할 일이 없다는 뜻이다.  
만약 비검사 경고를 제거할 순 없지만 타입 안전성에 대한 강한 확신이 있는 경우 `@SuppressWarnings("unchecked")`를 사용하여
해당 경고를 표시하지 않도록 설정할 수 있다.  

###### @SuppressWarning("unchecked") 어노테이션
이 어노테이션은 비검사 경고를 띄우지 않도록 설정할 수 있는 어노테이션이다. 적용 가능한 범위는 개별 지역변수 선언부터 클래스 선언부까지로 굉장히 방대하다.
그렇지만 항상 `선언`에 적용해야 함을 잊지 말아야 한다.   
그리고 가능한 좁은 범위에 설정하는 것이 바람직하다. 예를 들어 클래스 선언부에 어노테이션을 적용하면
해당 클래스에서 발생하는 경고를 모두 무시하므로 좋지 않은 방법이다.

###### 예시
```java
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        return (T[]) Arrays.copyOf(elements, size, a.getClass());
    }
    
    System.arraycopy(elements, 0, a, 0, size);
    
    if (a.length > size) {
        a[size] = null;
    }
    return a;
}
```
위 메서드는 ArrayList 클래스에서 가져온 toArray 메서드다. 위 메서드를 컴파일하면
copyOf 메서드에서 타입 캐스팅에 관한 비검사 경고가 발생한다. 이를 제거하기 위해 @SuppressWarning 어노테이션을 붙이면 다음과 같다.
```java
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elementData, 
    size, a.getClass()); 
        return result
    }
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size) {
        a[size] = null;
    }
    return a;
}
```
result라는 지역 변수를 하나 선언하여 해당 선언부에 @SuppressWarnings을 붙이면 된다.

###### 어노테이션 사용 시 지켜야 할 점
해당 어노테이션을 사용하면 항상 주석을 남겨서 왜 해당 경고를 suppress 했는지 설명하는 것이 협업 차원에서 바람직하다.

## 핵심 정리
    비검사 경고는 무시하지 말고 최대한 제거하자. 만약 타입 안전성이 확실하게 보장되지만 해당 경고를 제거할 방법이 없는 경우
    @SuppressWarnings("unchecked") 어노테이션을 사용하여 숨기자. 그리고 주석을 남겨 어떤 근거로 해당 검사를 숨겼는지 명시하자.