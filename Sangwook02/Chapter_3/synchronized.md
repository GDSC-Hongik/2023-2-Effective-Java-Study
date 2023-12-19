# synchronized

클래스들의 최상위 층에는 Object 클래스가 있다.  
우리 책에서는 equals(), hashCode(), toString() 메서드를 위주로 설명하지만 
이외에도 Object 클래스가 가지고 있는 특징에 대해 알아볼 필요가 있다.  

[java.lang의 Object 클래스](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html)
의 명세를 보면, 우리 책에서 다루지 않는 것들이 두 가지 존재한다.  
notify()와 wait()이다.  
이들에 대하여 공부하던 중 이 두 메서드는 thread 환경에서 사용됨을 알게 되었고, 
반드시 synchronized 블록 내에서 호출되어야 한다는 것을 알게되었다.  
그러나, 여전히 synchronized에 대해 잘 알지 못하기 때문에 주제로 선정하게 되었다.  

자바에는 여러 동기화 기법이 있는데 4가지 주요 동기화 방법에 대해 소개한다.
### synchronized method
복수의 thread에서 하나의 인스턴스에 접근하는 상황을 고려해보자.  
```java
public static void main(String[]args) {
    Common common = new Common();
    
    Thread thread1 = new Thread(() -> {
        common.run("1번 thread");
    });
    
    Thread thread2 = new Thread(() -> {
        common.run("2번 thread");
    });
    
    thread1.start();
    thread2.start();
}
```
이런 경우 Common 클래스의 run() 메서드가 아래와 같이 synchronized 메서드라면, 접근한 순서대로 실행된다.  
```java
public class Common {
    public synchronized void run(String message) {
        System.out.println(message);
    }
}
```
이렇게 되면 접근 순서에 따라 "1번 thread"가 먼저 출력되고, "2번 thread"가 이어서 출력되는 것이다.  
그렇다면, lock의 범위가 어디까지인지 알아볼 필요가 있다.  
만약 thread2에서 synchronized가 붙지 않은 common의 메서드를 호출한다면, synchronized가 붙은 메서드가 실행되는 동안 동시에 실행되므로 인스턴스 자체에 락이 걸린다기 보다, synchronized가 붙은 메서드간에 락을 공유한다고 이해하면 될 것 같다.

### static synchronized method
위의 synchronized 메서드를 잘 이해했다면, static synchronized 메서드는 쉽게 이해할 수 있다.
```java
public class Common {
    public static synchronized void run(String message) {
        System.out.println(message);
    }
}
```
이전의 synchronized 메서드가 인스턴스 내에서 synchronized가 붙은 메서드끼리 락을 공유했다면, static은 다른 static 메서드들이 그렇듯이 
다른 인스턴스에서 이 메서드에 접근하고 있다면 락을 공유하므로 접근할 수 없다.
### synchronized block
method와 다른 점이 있다면, synchronized block에는 인자를 넘길 수 있다.  
인자로는 인스턴스를 넘길 수도 있고, 클래스를 넘길 수 있는데 이 두가지의 차이를 알아보자.  
결국 이것도 락의 범위에 관한 것이긴 하다.  
```java
public class Common {
	public void synchronizedBlock(String message) {
		System.out.println(message + "블록 진입 전");
		synchronized(this) {
			System.out.println(message + "블록 내부");
		}
		System.out.println(message + "블록 밖으로 나옴");
	}
}
```
만약 인자로 Common.class를 넘기면, 클래스 자체에 lock 걸리기 때문에 static synchronized block과 같은 효과를 낼 수 있다.

### static synchronized block
static synchronized block은 static synchronized method와 달리 lock 객체를 지정할 수 있고, 
범위를 제한할 수 있다는 점이다.