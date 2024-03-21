# Item 89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

모든 참조 타입 필드를 transient로 선언하면 통제가 가능하지만 열거 타입을 이용하자

```java
// 열거 타입을 이용한 싱글턴
public enum Elvis {
    INSTANCE;
    private String[] favSongs = {"Hound Dog", "Heartbreak Hotel"};
    public void printFavorites() {
        System.out.println(Arrays.toString(favSongs));
    }
}
```