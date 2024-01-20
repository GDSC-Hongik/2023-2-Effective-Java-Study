# Item 27. 비검사 경고를 제거하라

제네릭 사용  
=> 컴파일러가 다양한 경고를 줌.  

### 예시
```java
Set<Lark> exaltation = new HashSet();
```
이 한 줄의 코드는 HashSet의 타입 매개변수를 지정하지 않아서 컴파일러가 경고를 준다.  
다이아몬드 연산자(<>)를 사용해 간단히 경고를 제거할 수 있다.  
비검사 경고를 모두 제거하면 타입 안전성을 확보할 수 있다.  
=> ClassCastException이 발생할 일이 없음.

### 경고를 제거할 수 없다면?
- @SuppressWarnings("unchecked")를 달아 경고를 숨기자.
  - 한계: 경고는 사라지지만 런타임에 ClassCastException 발생 가능.
  - 가능한 좁은 범위에 적용