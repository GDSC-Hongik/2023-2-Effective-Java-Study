# 태그 달린 클래스보다는 클래스 계층구조를 활용하라

## 태그 달린 클래스

- 두 가지 이상의 의미를 표현하는 경우 사용
- 현재 표현하는 의미를 태그 값으로 알려줌

### 단점

```
⛔ 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다
```

- 쓸데없는 코드↑ : 열거 타입 선언, 태그 필드, switch문
- 가독성↓ : 여러 구현이 한 클래스에 혼합돼 있음
- 메모리 사용↑
- 필드를 final로 선언하는 경우 → 쓰지 않는 필드를 초기화하는 불필요한 코드↑
- 또 다른 의미를 추가하는 경우 → 코드 수정해야함
- 인스턴스의 타입만으로는 그 의미를 알 길이 없음

## 클래스 계층구조

### 작성 방법

- 계층구조의 루트가 될 **추상 클래스** 정의
- 태그 값에 따라 동작이 달라지는 메서드 → 루트 클래스의 **추상 메서드**로 선언
- 동작이 일정한 메서드 → 루트 클래스의 **일반 메서드**로 추가
- 하위 클래스에서 공통으로 사용하는 **데이터 필드** → 루트 클래스로 올림
- **구체 클래스**를 의미별로 하나씩 정의
    - 각자의 의미에 해당하는 데이터 필드 삽입

### 장점

- 쓸데없는 코드 사라짐
- 관련없는 데이터 필드 제거
    - 살아남은 필드는 모두 final
- 필드 초기화 및 추상 메서드 구현 여부 → 컴파일러가 확인해줌
- 독립적으로 계층구조 확장 및 함께 사용 가능
- 변수의 의미 명시 및 제한 가능
- 특정 의미만 매개변수로 받을 수 있음
- 타입 사이의 자연스러운 계층 관계 반영
    - 유연성↑
    - 컴파일타임 타입 검사 능력↑