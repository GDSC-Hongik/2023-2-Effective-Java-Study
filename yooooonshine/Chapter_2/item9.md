# try-finally보다는 try-with-resources를 사용하라
---
자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야하는 자원이 존재한다.
* `InputStream`
* `OutputStream`
* `java.sql.Connection`
등이 있다.

이러한 자원은 close를 명시적으로 호출해줘야 하지만, 클라이언트가 놓치기 쉬워 finalized가 존재한다.
하지만 finalized는 믿을만 하지 못하므로 try-finally를 사용한다.
## try-finally란?
```java
static String firstLineOfFine(String path) throws IOException {
	BuffereReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
		br.close;
	}
}
```
즉 위와 같이 br.readLine()을 시도하나, 에러와 무관하게 반드시 마지막에 br.close를 실행시키는 것이다.

하지만 이는 문제를 내포하고 있다.
예를 들어 기기의 물리적인 문제가 존재하여, `br.readLine()`에서 예외를 던지면, 우선적으로 finally의 `br.close();`가 호출될 것이다. 하지만 기기는 문제를 가지고 있으므로, `br.close();` 또한 예외를 호출할 것이다.
그럼 이를 받는 `IOException`은 첫번째의 예외는 확인하지 않고 두 번째 예외만 throw하므로, 최초의 원인인 `br.readLine()`에서의 예외를 확인할 수 없게 되는 것이다.


이러한 문제들을 자바 7의 try-with-resources가 해결해주었다.
## try-with-resources란?
try with resource란 `try()`에 명시된 자원에 대하여, try가 끝났을 때 자동으로 자원을 해제해주는 기능이다. try에서 선언된 객체가 AutoCloseable을 구현했을 경우에, JAVA는 try구문이 끝났을 때 `close()` 메서드를 호출해준다.

예를 보면서 이해해보자
```java
private static class MyResource implements AutoCloseable {
    public MyResource() {
    }

	public void doSomething(){
		System.out.println("난 뭔가를 하고 있어");
	}

    @Override
    public void close() throws Exception {
        System.out.println("MyResource 회수할께");
    }
}

public static void main(String args[]) {
    try (
        MyResource myResource = new MyResource();
    ) {
        myResource.doSomething();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
위 예시를 보면 `AutoCloseable`을 implements하여 `close()` 메서드를 `@Override`해주었다.
아래의 `main()`메서드에서 `MyResource`자원을 try문에서 할당해주었고, try문이 끝나면 자동으로 `MyResource`에 있는 `close()`메서드가 호출된다. 따라서 이 `close()` 메서드에 자원을 해제하는 기능을 추가해주면 된다.

위 try-finally 구문을 try-with-resource 구문으로 바꿔주면 아래와 같다
```java
static String firstLineOfFine(String path) throws IOException {
	try (
	BuffereReader br = new BufferedReader(new FileReader(path));
	) {
		return br.readLine();
	}
}
```

이를 통해 여러 장점을 얻을 수 있다.
* try-finally에서 첫 번째 예외가 안 보였던 반면에,  try-with-resource구문에서는 예외가 발생하여도 finally()에 존재했던 예외가 아닌,`br.readLine()`에서 발생한 예외가 IOException으로 throw 된다. 이 외의 예외는 숨겨지고, 이는 `Throwable`에 존재하는 `getSuppressed`메서드를 통해 확인할 수 있다.
* 코드가 간결해진다.
