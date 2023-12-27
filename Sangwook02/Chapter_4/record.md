# [record](https://docs.oracle.com/en/java/javase/17/language/records.html)

record는 자바에서 제공하는 클래스의 종류이다.  
간단하게 데이터를 저장하고 전달하기 위한 목적으로 사용 가능하다.  
```java
public record Rectangle(Integer width, Integer height) {
}
```
위는 width와 height를 필드로 가지는 record의 예시이다.  
일반 class와 다른 구조를 가지고 있는 것을 확인할 수 있는데, 이렇게만 해도 자동으로 getter와 AllArgsConstructor가 생성된다.
```java
// .class 파일
public record Rectangle(Integer width, Integer height) {
    public Rectangle(Integer width, Integer height) {
        this.width = width;
        this.height = height;
    }

    public Integer width() {
        return this.width;
    }

    public Integer height() {
        return this.height;
    }
}
```

- 모든 클래스가 final이다.  
final이기 때문에 값을 변경할 수 없다.  
- 모든 필드의 값을 반환하는 toString이 자동으로 생긴다.
- equals와 hashcode도 자동으로 생성된다.
- getter의 이름은 필드의 이름이다.
