# Item 73. 추상화 수준에 맞는 예외를 던지자

저수준 예외를 처리하지 않고 바깥으로 전하면 안 됩니다.  
무책임하게 던져버리면 메서드의 맥락과 맞지 않는 예외를 잡게 될지도 모르기 때문.

상위 계층에서는 저수준 예외를 잡으면 자신의 수준에 맞게 바꿔서 던져야 함.  
이걸 예외 번역이라고 하는데 아래는 예시 코드..

```java
try {
    ... // 저수준 추상화
} catch (LowerLevelException e) {
    throw new HigherLevelException(...);
}
```