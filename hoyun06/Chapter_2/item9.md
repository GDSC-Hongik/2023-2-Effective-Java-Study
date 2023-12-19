### Item9 : try-finally 보다는 try-with-resources를 사용하라

###### AutoClosable의 close를 직접 호출하여 닫아줘야 하는 자원들
- 입출력 스트림  
ex) OutputStream, InputStream ..
- DB 커넥션  
ex) java.sql.Connection

위와 같은 자원들의 회수는 개발자가 까먹기 쉽다.  
그래서 이전 item에서 언급한 finalizer를 안전망으로 
사용하긴 하지만 성능 면에서나 안정성 면에서나 쉽사리 믿을 수 없다.

###### 전통적으로 사용해온 자원 회수 방법
예전부터 주로 사용하는 방법은 try-finally였다.
```java
static String firstLineOfFile(String path) {
    BufferedReader br = new BufferedReader(new FileReader(path));    
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```
위 경우 readLine 메서드의 정상적인 종료 여부와는 상관없이 항상 finally 문이 실행되면서
close 메서드가 잘 실행된다.  
하지만 여기서 회수해야 할 자원이 하나만 더 추가되면 코드가 갑자기 난잡해진다.
```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```
회수해야 할 자원이 딱 한 개 더 늘었는데 코드가 굉장히 지저분한 것을 알 수 있다.

###### try-finally 방식의 문제점
try-finally는 코드가 지저분해진다는 단점 말고도 문제점이 또 있다.  

**예외 확인이 어렵다**
  - try-finally문에서 exception은 try문과 finally문 둘 중 어느 곳에서나 발생할 수 있다.
  - firstLineOfFile 메서드를 실행하는 컴퓨터에 물리적인 문제가 있다고 가정하자.  
  그럼 readLine 메서드도 실행에 실패할 것이고 close 메서도드 당연히 실행에 실패할 것이다.  
  하지만 이 경우 두 번째 exception만이 콘솔 로그에 찍히게 되면서 첫 번째 exception은 그대로 묻히게 된다.  
  이는 디버깅 시에 문제의 시발점 파악을 어렵게 만든다.

###### try-with-resources
위에서 설명한 try-finally가 가지는 문제점을 해결한 것이 바로 `try-with-resources`다.

try-with-resources 구조를 사용하기 앞서 먼저 확인해야 하는 사항이 있다.  
이 구조를 사용하고자 하는 자원의 클래스가 `AutoClosable` 인터페이스를 구현하는지 확인해야 한다.

AutoClosable 인터페이스는 void형 반환 타입을 가지는 `close()` 메서드 딱 하나만 가지고 있는 인터페이스다.
요즘 대부분의 자바 라이브러리와 외부 라이브러리에는 AutoClosable을 구현하는 클래스가 많이 선언되어 있다고 한다.

아래 코드는 firstLineOfFile 메서드를 try-with-resources 구조로 다시 작성한 것이다
```java
public static String firstLineOfFile(String path) throw IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```
그리고 아래 코드는 copy 메서드를 try-with-resources 구조로 다시 작성한 것이다
```java
static void copy(String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src); OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0)
		out.write(buf, 0, n);
	}
}
```
- 확실히 try-finally를 사용할 때보다 코드가 간결하다.  
- 그리고 try(...) 에서 선언된 객체들에 대해선 try문 종료 시 자동으로 close() 메서드를 호출해준다.
- 그리고 만약 try문과 close() 메서드 모두에서 예외가 발생하더라도 close 메서드보다 try문에서 발생한 예외가 우선 순위가 높으므로
태초의 원인이 무엇인지 파악하기 쉽다.  
그렇다고 해서 close 메서드에서 발생한 예외를 확인하지 못하는 것도 아니다. Throwable에 포함된 getSuppressed 메서드를 통해
코드 상에서 해당 예외를 확인할 수 있는 방법도 제공한다.

try-with-resources 구조에서도 당연히 catch문을 사용할 수 있다.
```java
static String firstLineOfFile(String path, String defaultVal) {
	try (BufferedReader br = new BufferedReader(new FileReader(path))) {
		return br.readLine();
	} catch (IOException e) {
		return defaultVal; // 파일을 열지 못하거나 제대로 읽지 못한 경우 디폴트 값 반환한다.
	}
}
```
위 코드에서는 똑같이 catch문을 사용하여 파일 열람 및 읽기 도중에 발생하는 예외 처리 또한 가능케 한다.

## 핵심 정리
    꼭 회수해야 하는 자원을 다룰 때는 try-finally 대신 try-with-resources를 사용하자 