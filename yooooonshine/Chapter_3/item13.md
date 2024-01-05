# clone 재정의는 주의해서 진행하라
---
## cloneable이란 무엇일까?
`cloneable`은 자바 8부터 사용가능하게 된 인터페이스이다.
`Object.clone()`메서드를 사용하기 위해서는 clone을 할 클래스는 `cloneable`인터페이스를 implements해야 한다. 그럼 clone메서드를 통해 객체 복사가 가능하다.
`Clone`메서드를 사용하는 클래스가 `Cloneable`인터페이스를 implements하지 않았다면, `CloneNotSupportedException` 예외가 터진다.

`Cloneable`인터페이스를 상속한 클래스는 `clone`메서드를 public으로 `@Override`해주어야 한다.

## clone메서드는 어디에 있을까?
clone메서드는 `Object`에 구현되어 있다. `Cloneable`인터페이스에는 `clone`메서드가 구현되어 있지 않다. 
## 그럼 `cloneable`인터페이스를 왜 쓰는걸까?
`cloneable`인터페이스는 단지 이 클래스가 clone가능한 객체임을 명시하는 역할을 위해 쓰는 것이다.

## 문제점
다만 이와 같이 `cloneable`인터페이스에 `clone`이 구현되어 있는 게 아니라, `Object`에 `protected`로 구현되어 있기에, 외부 객체에서 `clone`메서드를 호출할 수 없다.
## 그럼에도 불구하고 cloneable은 자주 사용된다.
Cloneable 방식은 널리 쓰이고 있어, `clone`메서드를 잘 동작하게끔 해주는 구현 방법과, 언제 그렇게 해야하는 지 등을 알아두는 것이 좋다.

## `Cloneable`인터페이스는 무슨일을 할까?
방금도 말했다시피 `clone`메서드도 구현되어 있지 않지만, 이는 `Object.clone`메서드가 동작하는 방식을 결정한다.
즉 `Cloneable`인터페이스를 구현한 클래스가 `clone`을 호출하면, 그 객체들의 필드들을 하나하나 복사한 객체를 반환한다.
반면 `Cloneable`인터페이스를 구현하지 않은 클래스가 호출하면, `CloneNotSupportedException`을 터트린다.

즉 이건 기존의 인터페이스 역할(하는 일들을 정의한 것)과는 다르게 상위 클래스의 메서드 동작방식을 결정한다.
이는 좋은 방법이 사실 아니다.

## 예시
```java
public class PhoneNumber implements Cloneable extends 상위클래스명{
	...

	@Override
	public PhoneNumber clone() {
		try {
			return (PhoneNumber) super.clone();
		} catch (CloneNotSupportedException e) {
			throw new AssertionError();
		}
	}
}
```

만약 상위 클래스에서 `clone`을 완벽하게 구현하였다면, 이는 정상적으로 작동할 것이다.
또한 `super.clone()`에서 복사되어 `return`되는 객체는 상위 클래스에 속하므로, 형변환을 해줌으로써 사용자가 형변환하지 않도록 하였다.

참고로 여기서 `try-catch`문으로 `CloneNotSupportedException`을 받아준 이유는 에러가 나면, `CloneNotSupportedException`예외가 터지기에 이를 `AssertionError`로 바꿔준 것이다.
## clone은 항상 조심해야 한다.
```java
public class stack {
	private Object[] elements;
	private int size = 0;
}
```
위와 같은 클래스의 `clone`을 생각해보자.
`size`변수같은 경우에는 정상적으로 복제되겠지만, `elements`의 경우 객체의 주소값들을 가지고 있으므로, 복제를 하더라도 같은 객체원소들을 공유하는 꼴이 되므로, 프로그램이 이상하게 동작하거나 `NullPointerException`을 던질 것이다.

따라서 `clone`은 생성자와 유사하게, 원본 객체에 아무런 해를 끼지지 않으면서, 복제된 객체의 불변식을 보장해야 한다.

## 배열을 복제할 때 원본 객체에 해를 끼지치 않는 방법
이는 배열 또한 배열의 clone식을 호출하는 것이다.
```java
@Override public Stack clone() {
	try {
		Stack result = (Stack) super.clone();
		result.elements = elements.clone();
		return result;
	} catch (CloneNotSupportedException e) {
		throw new AssertionError();
	}

}
```
위와 같이 배열인 `elements`의 clone을 호출하며 넣어준다.

참고로 배열에 존재하는 `clone`은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환하기에 형변환이 필요없다.

## clone할 클래스의 배열은 final이면 안된다.
위와 같이 `elements`를 재할당해야 하는 경우에, `elements`가 `final`이면 필드에 새로운 값을 할당할 수 없기에 `final`을 사용하면 안 된다.
이는  *가변 객체를 참조하는 필드는 final로 선언하라*는 일반 용법과 충돌한다.
## 연결 리스트의 경우 깊은 복사가 필요하다.
단순히 복사만 한다면, 결국 새로 복사된 리스트가 기존의 리스트를 가리키게 된다.
## 결론적으로 `cloneable`을 사용하는 것보다 가능하면 복제 생성자, 팩토리를 이용하자
즉 생성자에 값을 넣어줌으로써 복제를 하던가, 팩터리를 통해 복제하는 방법이, `cloneable`의 위험성을 감수하는 것보다 훨 이득이다.