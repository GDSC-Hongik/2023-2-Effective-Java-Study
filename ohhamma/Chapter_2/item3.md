# private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 싱글턴

- singleton
- **인스턴스**를 오직 **하나**만 생성할 수 있는 클래스
- ex) 무상태 객체(함수), 시스템 컴포넌트(설계상 유일)
- **BUT** 클라이언트 테스트 어려워짐

## 싱글턴 생성 방식

- 생성자 → `private`
- 유일한 인스턴스에 접근 가능 수능 → `public static` 멤버

### public static 멤버가 final 필드

- private 생성자 → Elvis.INSTANCE 초기화 시 한 번만 호출
- **BUT** 권한 있는 클라이언트의 리플렉션 API 사용 → private 생성자 호출 가능
    - 방어 : 생성자에서 두 번째 객체가 생성되려 할 때 예외 던지기

```java
public class Elvis {
		public static final Elvis INSTANCE = new Elvis();
		private Elvis() { ... }

		public void leaveTheBuilding() { ... }
}
```

- 장점
    - 해당 클래스가 싱글턴 → **API**에 명백히 드러남
    - **간결함**

### 정적 팩터리 메서드를 public static 멤버로 제공

- Elvis.getInstance → 항상 같은 객체의 참조 반환

```java
public class Elvis {
		private static final Elvis INSTANCE = new Elvis();
		private Elvis() { ... }
		public static Elvis getInstance() { return INSTANCE; }

		public void leaveTheBuilding() { ... }
}
```

- 장점
    - API 바꾸지 않고도 싱글턴이 아니게 변경 가능
        - 팩터리 메서드가 호출하는 스레드별로 다른 인스턴스 넘겨주도록
    - 정적 팩터리 → **제네릭 싱글턴 팩터리**로 생성 가능
    - 정적 팩터리의 메서드 참조 → supplier로 사용 가능 (Supplier<Elvis>)
- 싱글턴 클래스 직렬화
    - `readResolve()` 필요 : 직렬화된 인스턴스 역직렬화 → 새로운 인스턴스 생성

```java
// 싱글턴임을 보장
private Object readResolve() {
		// 진짜 Elvis 반환 (가짜 Elvis는 GC에게 맡김)
		return INSTANCE;
}
```

### 원소가 하나인 열거 타입 선언

- 간결, 추가 노력없이 직렬화 가능
- 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법

```java
public enum Elvis {
		INSTANCE;

		public void leaveTheBuilding() { ... }
}
```