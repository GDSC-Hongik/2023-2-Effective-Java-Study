### Item78 : 공유 중인 가변 데이터는 동기화해 사용하라

###### synchronized 키워드
`synchronized`는 해당 키워드가 사용된 메서드 혹은 코드 블럭이 한 번에 한 스레드에 의해서만 실행되도록 
설정하는 키워드다.  
그래서 많은 개발자들이 '동기화'는 '배타적 실행'이 가능하도록 하는 수단이라고 생각한다. 이 말도 맞지만 '동기화'는 이외에도
다른 역할도 수행한다.

###### 동기화의 기능
동기화는 `배타적 실행`을 가능하게 하고 그와 더불어 스레드 간의 안정적인 통신을 가능하게 해준다.
즉, 동기화 없이는 한 스레드에서 만든 변화를 다른 스레드에서 확인할 수 없을 수도 있다는 뜻이다.  
사실 long, double 을 제외한 변수의 값을 읽는 작업은 '원자적'이다. 다시 말해 '동기화'없이 여러 스레드에서 
동일 변수의 값을 수정하는 중이더라도 해당 변수의 값을 읽어오면 항상 '수정이 완전히 반영된 값'을 읽어옴을 보장한다는 뜻이다.

###### '원자적' 작업의 오해
하지만 이 '원자적'특성을 오해해선 안된다. 물론, 이 특성에 의해 깨지지 않은 온전한 값을 읽어오긴 하지만 그렇다고 해서 그 읽어온 값이
가장 최근에 반영된 수정본이라는 보장은 없다.   
즉, 해당 값이 스레드 A 에 의해 먼저 수정되고 그 바로 뒤에 스레드 B에 의해 수정되었어도 값을 읽어오면
A에 의해 수정된 '온전한 값'이 반환될 수 있다는 뜻이다. A에 의해 수정된 값도 결국 '`수정이 완전히 반영된 값`'이긴 하니 말이다.  
하지만 이건 우리가 원하는 결과가 아니다. 우리는 특정 변수의 값을 읽어올 때 가장 최근 수정본의 값을 원한다. 그렇기 때문에 '원자적'특성만을 믿고 
코드를 작성하면 안되고 반드시 '동기화'가 필요하다.

###### 동기화 관련 팁
- 특정 클래스 내부 필드가 가변이라 '동기화'가 필요한 상황이라 가정하자.  
이 경우 해당 필드의 '쓰기 작업을 수행하는 메서드'와 '읽기 작업을 수행하는 메서드' 모두에 동기화를 적용해야 한다. 쓰기 작업에만 동기화를 적용하는 것만으로는 
충분하지 않다.
    ```java
    public class StopThread {
        private static boolean stopRequested;
        
        // 가변 변수의 쓰기 작업에도 동기화 적용
        private static synchronized void requestStop() {
            stopRequested = true;
        }
  
        // 가변 변수의 읽기 작업에도 동기화 적용
        private static synchronized boolean stopRequested() {
            return stopRequested;
        }
        
        public static void main(String[] args) throws InterruptedException {
            Thread backgroundThread = new Thread(() -> {
                int i = 0;
                while (!stopRequested())
                    i++;
            });
            backgroundThread.start();
            
            TimeUnit.SECONDS.sleep(1);
            requestStop();
        }
    }
    ```
- 매번 메서드 레벨의 동기화 보다 조금 더 간편하고 빠른 방법은 해당 가변 변수에 `volatile` 키워드를 이용하는 것이다.  
`volatile` 키워드는 해당 변수에 가장 최근에 기록된 값을 가져옴을 보장한다. 하지만 이 `volatile` 키워드를 배타적 실행과는 아무런 상관은 없다.
  ```java
  public class StopThread {
      private staic volatile boolean stopRequest;
      
      public static void main(String[] args) throws InterruptedException {
          Thread backgroundThread=  new Thread(() -> {
              int i  = 0;
              while (!stopRequested)
                  i++;
          });
          backgroundThread.start();
          
          TimeUnit.SECONDS.sleep(1);
          stopRequest = true;
      }
  }
  ```
  이와 같이 코드를 작성하면 명시적인 synchronized 키워드 없이도 매번 stopRequest 변수의 최신 값을 가져옴을 보장한다.
- 다만, 가변 변수의 값을 읽는 작업과 쓰는 작업이 공존하는 코드에서는 volatile 키워드를 조심해서 사용해야 한다.
  ```java
  private static volatile int nextSerialNumber = 0;
  
  public static int generateSerialNumber() {
      return nextSerialNumber++;
  }
  ```
  위 메서드의 경우 가변 변수 nextSerialNumber 에 '++' 연산자를 사용했다.  
  '++'연산자는 우선 변수의 값을 읽고 그 뒤에 값을 증가하는 로직으로 분리된다. 스레드 A에서 generateSerialNumber()를 실행하여 
  nextSerialNumber 를 우선 읽어온 뒤에 그 값을 증가시키려고 하는 순간에 스레드 B에서 generateSerialNumber()를 실행하여 nextSerialNumber 변수의 값을 읽게 되면
  아직 스레드 A에서 쓰기 작업을 수행하지 않았기 때문에 스레드 A에서 읽은 값과 동일한 값을 읽게 된다.
  위에서 말했듯이 `volatile` 키워드는 배타적 실행을 보장하지 않기 때문에 발생하는 문제다.  
  이 경우 generateSerialNumber 메서드를 `synchronized` 로 선언하면 해결된다.
- `java.util.concurrent.atomic 패키지`에는 `락`을 명시하지 않아도 스레드 안전한 프로그래밍을 할 수 있게 해주는 클래스들을 제공한다.(ex. AtomicLong)

## 핵심 정리
    가변 데이터는 단일 스레드 환경에서만 사용하는 것이 좋다. 
    하지만 만약 멀티 스레드 환경에서 사용해야 한다면 반드시 동기화를 시키자.  


## 동기화
이번 item에서도 배웠지만 멀티 쓰레드 환경에서 가변 데이터를 공유하는 경우
`동기화`가 필요하다.   
`동기화`가 지니는 의미는 2개가 있다
- 쓰레드 간의 통신
- 코드의 배타적 실행

이번 발표에선 `배타적 실행`면에 초점을 맞추고자 한다.

###### 모니터(monitor)
자바에서는 쓰레드로 하여금 `락`을 통해 특정 메서드 혹은 코드 블럭에 대한 독점적인
실행권을 가질 수 있도록 한다.
그렇다면 이 `락`이라는 건 정확히 무엇을 의미하는 걸까? 
이는 자바의 모니터와 연관이 있다.

자바의 '모든' 객체는 자신과 연관된 '모니터'라는 것을 가진다.  
쓰레드들이 해당 객체에 어떻게 엑세스하는지 모니터링을 하기 때문에 '모니터'라고 불린다고 한다. 

쓰레드는 이 '모니터'를 대상으로 lock/unlock 과 같은 행위를 할 수가 있는데 특정 '모니터'에 대한
lock 작업을 성공적으로 수행했다면 그것이 곧 해당 '모니터'와 연관된 객체에 대한 `락` 획득을 의미한다.  
당연히 그 이름에 맞게 오직 한 쓰레드만이 특정 '모니터'에 대한 lock 작업을 수행할 수 있다.

참고: https://bgpark.tistory.com/161

```java
public class Test {
    private Car car = new Car();
	
    public void doSomething() {
        synchronized (car) {
          ...
        }
    }
}
```
그래서 위 `synchronized` 블록에서 일어나는 일은..
1. 쓰레드가 doSomething 메서드 실행을 하려고 함
2. `synchronized` 키워드를 발견하고 그 대상 객체인 car 객체의 래퍼런스값을 먼저 계산함
3. 그리고 해당 car 객체의 '모니터'에 대한 lock 작업을 시도함
4. 성공적으로 획득했다면 코드를 계속 실행 
   - `synchronized` 블록이 (정상적이든 비정상적이든)종료되면 '모니터'에 대한 unlock 작업 수행 
5. 만약 lock 작업에 실패했다면 성공할 때까지 메서드 실행을 중지했다가 lock 작업을 수행한 뒤에 메서드 마저 실행 

참고
- https://happy-coding-day.tistory.com/entry/JAVA-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80
- https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.1
- https://www.artima.com/insidejvm/ed2/threadsynch.html