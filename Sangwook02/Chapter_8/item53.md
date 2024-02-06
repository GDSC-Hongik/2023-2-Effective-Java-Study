# Item 53. 가변인수는 신중히 사용하라

### 가변인수
아래와 같이 가변인수를 활용할 수 있다.
```java
int add(int... args) {
    int sum = 0;
    for (int arg : args) {
        sum += arg;
    }
    return sum;
}
```

### 가변인수가 1개 이상이라면?
0개가 들어온다면 예외가 발생하므로 주의해야함.
```java
int minimum(int first, int... args) {
    int min = first;
    for (int arg : args) {
        if (arg < min) {
            min = arg;
        }
    }
    return min;
}
```
첫 매개변수만 따로 받아줌으로써 해결 가능

### 성능이 중요하다면?
만약 성능이 중요하고, 대부분의 경우 3개 이하의 매개변수를 받는다면?  
3개까지의 메서드는 가변인수를 사용하지 않는 것이 좋다.  
가변 인수는 배열을 사용하므로 성능에 영향을 주기 때문.