### Item80 : 스레드보다는 실행자,태스트,스트림을 애용하라

###### 실행자 프레임워크(Executor framework)
java.util.concurrent 패키지 내부의 인터페이스를 기반으로 하는 태스크 실행 기능을 담당한다.
이 실행자 프레임워크를 이용하면 '작업 큐' 사용이 굉장히 쉬워진다.

`작업 큐` : 클라이언트가 요청한 작업(태스크)을 백그라운드 스레드를 이용하여 비동기로 처리하는 클래스.

```java
ExecutorService exec = Executors.newSingleThreadExecutor();
```
위 코드를 통해 '작업 큐'를 하나 생성한 것이고
```java
exec.execute(runnable);
```
위 메서드를 통해 해당 '작업 큐'에게 수행할 작업을 넘긴 것이다.

이외에도 `ExecutorService` 에서 제공하는 메서드는 꽤나 많다.
```java
public interface ExecutorService extends Executor {

    void shutdown();

    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();

    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> Future<T> submit(Callable<T> task);

    <T> Future<T> submit(Runnable task, T result);
	
    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```
만약 생성한 '작업 큐'를 여러 개의 스레드가 처리하길 바란다면 다른 정적 팩터리 메서드를 이용하여
다른 ExecutorService 를 반환받아 사용하면 된다.

규모가 작은 프로그램이나 프로젝트의 경우
```java
    /**
     * Creates a thread pool that creates new threads as needed, but
     * will reuse previously constructed threads when they are
     * available.  These pools will typically improve the performance
     * of programs that execute many short-lived asynchronous tasks.
     * Calls to {@code execute} will reuse previously constructed
     * threads if available. If no existing thread is available, a new
     * thread will be created and added to the pool. Threads that have
     * not been used for sixty seconds are terminated and removed from
     * the cache. Thus, a pool that remains idle for long enough will
     * not consume any resources. Note that pools with similar
     * properties but different details (for example, timeout parameters)
     * may be created using {@link ThreadPoolExecutor} constructors.
     *
     * @return the newly created thread pool
     */
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, 
            TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
    }
```
`newCachedThreadPool()` 정적 팩터레 메서드를 이용하면 된다. 이를 통해 생성된 '작업 큐'의 경우
필요할 때마다 새로운 스레드를 생성하여 작업을 수행한다.

하지만 대규모 서비스의 경우 클라이언트가 요청하는 작업마다
스레드를 생성하여 작업을 수행하려고 한다면 과부하가 걸릴 것이다. 
```java
    /**
     * Creates a thread pool that reuses a fixed number of threads
     * operating off a shared unbounded queue.  At any point, at most
     * {@code nThreads} threads will be active processing tasks.
     * If additional tasks are submitted when all threads are active,
     * they will wait in the queue until a thread is available.
     * If any thread terminates due to a failure during execution
     * prior to shutdown, a new one will take its place if needed to
     * execute subsequent tasks.  The threads in the pool will exist
     * until it is explicitly {@link ExecutorService#shutdown shutdown}.
     *
     * @param nThreads the number of threads in the pool
     * @return the newly created thread pool
     * @throws IllegalArgumentException if {@code nThreads <= 0}
     */
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<Runnable>());
    }
```
이때는 `newFixedThreadPool(int nThreads)` 메서드를 이용하여 고정된 스레드 개수만 사용하는 '작업 큐'를 생성하여 사용할 수 있다.

###### 스레드와 실행자 프레임워크의 차이점
저자는 웬만해선 스레드를 직접 사용하는 행위는 지양해야 한다고 한다. 그 이유는 다음과 같다.
- 스레드는 작업 단위와 해당 작업을 수행하는 메커니즘을 모두 스레드가 담당한다
- 실행자 프레임워크에선 작업 단위는 '태스크'라는 추상화된 개념으로 구분되며 실제 '태스크'를 실행하는 메커니즘은 `ExecutorService`가 수행한다

이때 '태스크'에는 두 종류가 있다.
- Runnable
- Callable
  - `Runnable`과 비슷하나 결과값을 반환하고 임의의 예외를 던질 수 있다는 특징이 있다.

###### 포크-조인 태스크
자바 7이 도입되면서 실행자 프레임워크에선 '포크-조인 태스크'를 지원하기 시작했다.  
위에서 말한 것처럼 '포크-조인 태스크' 는 작업 단위를 나타내는 것이고 이를 실제 처리하는 것은 
'ForkJoinPool'이라는 ExecutorService를 구현한 하위 클래스다.
```java
public class ForkJoinPool extends AbstractExecutorService {
    ...
}
```
'AbstractExecutorService' 클래스가 'ExecutorService'를 구현하므로 'ForkJoinPool'도 'ExecutorService'를 구현한다고 할 수 있다.  
'포크-조인 태스크'는 그보다 더 작은 단위의 태스크로 나뉠 수 있고 'ForkJoinPool'의 스레드들은 이 작은 단위의 태스크들을 처리하고 만약 자신의 
작업을 모두 수행하면 다른 스레드가 처리해야 할 태스크들도 가져와 처리한다.  
이를 통해 높은 생산성 + 낮은 지연 시간을 달성한다고 한다.  
실제로 예전 item에서 봤던 병렬 Stream의 경우 이 'ForkJoinPool'을 이용하여 구현되었다고 한다.