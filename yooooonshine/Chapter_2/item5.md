# 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
대부분의 클래스가 하나 이상의 자원에 의존하는데, 심심찮게 정적 유틸리티 클래스로 구현하는 모습을 볼 수 있다.
```java
	public class spellchcker{
		private static final Lexicon dictionary = ...;
	}
```
이는 dictionary가 언어별로 구분될 수 있고, 특수 어휘용 사전, 테스트용 사전이 존재할 수 있으므로 static final은 옳지 못하다.
또한 단지 final을 빼고, dictionary를 교체하는 로직을 둔다 하더라도 멀티스레딩에서 자유롭지 못하다.

이를 해결하는 방법이 생성자 주입이다!
```java
	public class spellchcker{
		private final Lexicon dictionary = ...;
		public SpellChecker(Lexicon dictionary) {
			this.dictionary = Objects.requiredNonNull(dictionary);
		}
	}
```
위와 같이 생성자를 통해 dictionary를 주입해주면 상황에 따라 원하는 dictionary를 주입해 줄 수 있다.



### 참고 한정적 와일드카드 타입이란?
```java
List<? extends Number> numbers = new Arraylist<>();

```
한정적 와일드 카드 타입이란 여기서 \<? extends Number>를 의미한다. 이는 ? 타입이나, Number라는 클래스 혹은 이의 하위 타입만을 ?로 지정할 수 있다는 뜻이다.