# 적시에 방어적 복사본을 만드라

```
💡 클라이언트가 불변식을 깨뜨리려 혈안이 되어 있다고 가정하고 방어적으로 프로그래밍해야 한다.
```

## 불변 객체를 만들 때 필요하다

```
💡 예전에 작성된 낡은 코드들을 대처하자
```

- `Date` : 가변

```java
public final class Period {
	private final Date start;
	private final Date end;

	// ...
}
```

### 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사해야 한다

- 방어적 복사본을 만들고 → 이 **복사본**으로 유효성 검사
    - TOCTOU 공격 : 검사시점/사용시점 공격
- 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 **clone**을 사용해서는 안 된다
    - 악의를 가진 하위 클래스의 인스턴스 반환 가능성

```java
public Period(Date start, Date end) {
	this.start = new Date(start.getTime());
	this.end = new Date(end.getTime());

	if (this.start.compareTo(this.end) > 0)
		throw new IllegalArgumentException(this.start + "가 " + this.end + "보다 늦다.");
}
```

### **접근자**가 가변 필드의 방어적 복사본을 반환하게 하라

- 생성자와 달리 방어적 복사에 clone을 사용해도 된다
    - 신뢰할 수 없는 하위 클래스가 아니기 때문
    - 그러나 일반적으로 생성자나 정적 팩터리를 쓰는 게 좋다

```java
public Date start() {
	return new Date(start.getTime());
}

public Date end() {
	return new Date(end.getTime());
}
```

## 내부 객체를 클라이언트에 건네주기 전에 반드시 심사숙고해야 한다

- 안심할 수 없다면 원본을 노출하지 말고 방어적 복사본을 반환해야 한다
    - 길이가 1 이상인 배열은 무조건 가변이다
    - 배열의 불변 뷰를 반환하는 대안도 있다
- 되도록 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어든다

## 방어적 복사를 생략해도 되는 상황

- 복사 비용이 너무 큰 경우
- 해당 클래스와 그 클라이언트가 상호 신뢰할 수 있을 때
    - 통제권을 이전받아 더 이상 직접 수정하는 일이 없다고 약속하는 경우
    - 그 사실을 확실히 문서에 기재해야 함
- 불변식이 깨지더라도 그 영향이 오직 호출한 클라이언트로 국한될 때
    - 래퍼 클래스 패턴