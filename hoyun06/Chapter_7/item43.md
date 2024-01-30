### Item43 : 람다보다는 메서드 참조를 사용하라

###### 메서드 참조
이전 item을 보면 람다가 익명 클래스보다 훨씬 간결함을 알 수 있다.  
그런데 자바에는 이보다 더 간결하게 함수 객체를 생성하는 방법이 있는데 그것이 바로 `메서드 참조`다.

```java
// 람다
map.merge(key, 1, (count, incr) -> count + incr);
```
```java
// 메서드 참조
map.merge(key, 1, Integer::sum);
```
람다식을 사용한 코드에선 두 수를 받아 그 합을 반환하는 함수 객체를 넘겼다.  
자바 8부터는 모든 wrapper 클래스(Integer, Boolean, Double ...)에 위 람다식과 동일한 기능을 제공하는
정적 메서드를 제공하므로 람다식을 메서드 참조로 수정할 수 있다. 훨씬 간결하지 않은가?

메서드 참조는 대부분 그에 대응되는 람다보다 간결하므로 메서드 참조를 잘 활용하면 좋다. 람다를 메서드 참조로 바꾸는 방법은
우선 람다식에 해당되는 메서드를 새로 정의하고 메서드 참조 방식으로 호출하면 된다.

###### 메서드 참조 유형

| 메서드 참조 유형  |           예시           |                        대응되는 람다                         |
|:----------:|:----------------------:|:------------------------------------------------------:|
|     정적     |   Integer::parseInt    |              str -> Integer.parseInt(str)              |
| 한정적(인스턴스)  | Instant.now()::isAfter | Instant then = Instant.now();<br/>t -> then.isAfter(t) |
| 비한정적(인스턴스) |  String::toLowerCase   |                str -> str.toLowerCase()                |
|  클래스 생성자   |   TreeMap<K, V>::new   |                () -> new TreeMap<K, V>                 |
|   배열 생성자   |       int[]::new       |                  len -> new int[len]                   |
