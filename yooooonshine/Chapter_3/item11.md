# equals를 재정의하려거든 hashCode도 재정의하라
---
equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다. 그렇지 않으면 hashCode의 일반 규약을 어기게 되어 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킨다.

일반 규약
* equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
* equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
* equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

여기서 문제가 되는 것은 규칙2이다. 즉 논리적으로 같은 객체라고 판단하면 hashCode값도 같아야 하지만, equals를 재정의한 상황에서 기본 hashCode는 같다고 판단하는 두 객체도 전혀 다르다고 판단해버린다.

여기서 정확히 hashcode가 무엇일까
이는 객체의 주소값을 변환하여 생성한 객체의 고유한 정수값이다.
즉 한 클래스에 대해 같은 값을 같는 두 인스턴스 또한 hashcode의 값은 주소가 서로 다르니 다르게 나온다.

## 그럼 hashCode 메서드는 어떻게 작성해야 할까?
### 1. 모두 똑같은 해시코드를 반환하게 한다.
```java
@Override
public int hashCode() {
	return 1;
}
```
이는 모두 동일한 해시코드를 반환하므로 논리적으로 같은 객체라고 판단하면 hashCode도 같고, 다른 두 객체라도 hashCode가 다를 필요는 없기에 문제가 되지 않는다.
하지만 이는 `HashMap`의 경우 모두 같은 hash값을 같게 되므로 연결리스트와 동일한 형태로 저장되며, 접근 시간이 `O(1)`이 아닌 `O(n)`이라는 최악의 성능을 갖게 된다.

### 2. 각 타입, 배열, 참조 타입에서 제공하는 hashCode 메서드를 사용한다.
예로
```java
public PhoneNumber {
	int a;
	int b;
	int c;
	public PhoneNumber(int a, int b, int c) {
		this.a = a;
		this.b = b;
		this.c = c;
	}
}
```
의 경우 a,b,c가 같으면 같은 객체라고 equals를 만들 수 있다.
그럼 hashCode의 경우도 a,b,c가 같으면 같은 객체로 판단할 수 있게 재작성해야 한다.
```java
@Override
public int hashCode() {
	int result = Short.hashCode(a);
	result = 31 * result + Short.hashCode(b);
	result = 31 * result + Short.hashCode(c);
	return result;
}
```
이와 같이 타입, 참조 타입, 배열의 hashcode를 사용하며 hashCode를 만들 수 있다.
### 3. Objects 클래스에서 제공하는 hash 메서드를 사용한다.

```java
@Override 
public int hashCode() {
	return Objects.hash(a,b,c);
}
```
위와 같이 코드가 짧아진다는 장점이 존재한다.
다만 이는 속도가 더 느리다.
성능에 민감하지 않다면 사용가능하다.
### Lazy Initialization 방식으로 해시코드를 사용한다.
```java
private int hashCode //0으로 자동 초기화 된다.
@Override public int hashCode() {
	int result = hashCode;
	if(result == 0) {
		//초기화
	}
	return result;
}
```
즉 특정 객체가 반드시 hashCode를 key로 사용한다면, 호출될 때마다 계산하지 않고 미리 hashCode값을 초기화 하는 것도 좋은 방법이며, 항상 초기화되지 않고 처음 호출했을 때만 초기화하도록 Lazy Initialization방식을 사용하면 좋다.