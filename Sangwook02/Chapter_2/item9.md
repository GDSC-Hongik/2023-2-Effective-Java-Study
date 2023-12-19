# Item 9. try-with-resources를 사용하라

자원을 열고 닫을 때 try-finally를 사용해왔다.  
finally에서 close 메서드를 호출하여 자원의 닫힘을 보장해왔다.  
그러나, 자원을 두 개 이상 사용하는 경우를 고려하면 try-finally는 너무 지저분하고 
경우에 따라 예외가 발생해도 흔적이 남지 않는다.  
try-with-resources를 사용하여 이 문제를 개선할 수 있다.
```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
		br.close();
    }
}
```
이 코드를 try-with-resources를 사용하면 아래처럼 바뀐다.
```java
static String firstLineOfFile(String path) throws IOException {
	try (BufferedReader br = new BufferedReader(new FileReader(path))) {
		return br.readLine();
	}
}
```
코드에는 첫 줄을 읽는다는 주요 로직만 남으므로 짧아지고, 목적이 명확해진다.  
이처럼 회수해야 하는 자원을 다룰 때는 try-with-resources를 사용하는 것이 좋다.