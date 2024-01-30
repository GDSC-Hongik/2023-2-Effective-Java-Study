# @Override 애너테이션을 일관되게 사용하라

```
💡 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자
```

## @Override

- **메서드** 선언에만 달 수 있음
- 상위 타입의 메서드 **재정의**
- 컴파일 오류의 보완재 역할

### 클래스의 메서드 재정의

```java
@Override
public boolean equals(Object o) {
	if (!(o instanceof Bigram))
		return false;
	Bigram b = (Bigram) o;
	return b.first == first && b.second == second;
}
```

### 인터페이스의 메서드 재정의

- 디폴트 메서드 지원