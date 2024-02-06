### Item53 : 가변인수는 신중히 사용하라

###### 가변인수 메서드
가변인수 메서드는 말 그대로 해당 메서드 호출자로부터 가변인 인수를 넘겨받을 수 있는 메서드다. 호출자로부터 인수를 넘겨 받으면 
해당 값들로 이루어진 배열을 생성하고 메서드로 해당 배열을 넘긴다. 즉, 호출될 때마다 배열이 하나 새로 생성되는 것이다.
```java
public int sum(int... args) {
    int sum = 0;
    for (int arg : args) {
        sum += arg;
    }
    return sum;
}
```
위 가변인수 메서드는 인수를 0개 이상 받을 수 있다. 즉, 인수를 받지 않을 수도 있는 것이다.  
하지만 인수를 최소 1개는 꼭 받아야 한다면 어떻게 해야 할까? 이 경우엔 아래와 같이 작성하면 된다.
```java
public int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs) {
        if (arg < min)
            min = arg;
    }
    return min;
}
```
하지만 위에서 언급했듯이 가변인수는 매번 배열을 사용하므로 성능 상의 이점을 챙기려면 걸림돌이 될 수 있다. 이때는 가변인수를 조금 유연하게 사용하면 도움이 된다.  
만약 특정 메서드를 호출할 때 95%의 경우 매개변수가 3개 이하이고 나머지 5%정도만 4개 이상일 경우 가변인수 메서드를 다음과 같이 사용할 수 있다.
```java
public void method() {}
public void method(int m1) {}
public void method(int m1, int m2) {}
public void method(int m1, int m2, int m3) {}
public void method(int m1, int m2, int m3, int... rest) {}
```
이러면 전체 메서드 호출 중 5%만 배열을 생성하니 성능 상 이점을 챙기기 쉬울 것이다.