### Item65 : 리플렉션보다는 인터페이스를 사용하라

###### 리플렉션
리플렉션은 Class 정보를 이용하여 해당 클래스의 생성자, 메서드, 필드 정보를 알 수 있다. 그에 더해 해당 클래스 특정 인스턴스의
메서드 호출 혹은 필드값 변경과 같은 작업도 수행할 수 있다.

###### 리플렉션의 단점
- 컴파일타임 타입 검사가 주는 이점을 챙길 수 없다
  - 리플렉션을 이용하여 접근할 수 없는 메소드를 호출하는 경우 런타임 에러가 발생한다
- 리플렉션을 이용하는 코드는 장황해지고 읽기가 어렵다
- 성능이 떨어진다
  - 리플렉션을 통한 메서드 호출은 일반적인 호출보다 11배나 느렸다고 한다

###### 리플렉션 사용 예시
만약 컴파일타임에 어떤 클래스를 사용할 것인지 알 순 없으나 그 클래스를 참조할 수 있는 상위 타입 혹은 인터페이스를 안다면
리플렉션을 이용하여 런타임에 해당 클래스 인스턴스를 생성하고 상위 타입으로 참조하여 사용할 수 있다. 
```java
public static void main(String[] args) {
    Class<? extends Set<String>> cl = null;
    try {
        cl = (Class<? extends Set<String>>) // 비검사 형변환
            Class.forName(args[0]);
    } catch (ClassNotFoundException e) {
        fatalError("클래스를 찾을 수 없습니다.");
    }
    
    Constructor<? extends Set<String>> cons = null;
    try {
        cons = cl.getDeclaredConstructor();
    } catch (NoSuchMethodException e) {
        fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
    }
    
    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch (IllegalArgumentException e) {
        fatalError("생성자에 접근할 수 없습니다.");
    } catch (InstantiationException e) {
        fatalError("클래스를 인스턴스화할 수 없습니다.");
    } catch (InvocationTargetException e) {
        fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
    } catch (ClassCastException e) {
        fatalError("Set을 구현하지 않은 클래스입니다.");
    }
    
    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}
```
이 코드에선 args에 넘겨진 이름을 토대로 Class 정보를 받아온 뒤 리플렉션을 이용하여 생성자를 호출한다. 그리고 반환된 인스턴스를
적절한 인터페이스인 Set으로 받아 일반 Set처럼 사용하고 있다. 이처럼 컴파일타임에 어떤 클래스를 사용할 것인지 모르지만 그 상위 타입은 정해져 있을 땐
위와 같이 코드를 작성하여 사용할 수 있다. `args[0]`에 담긴 데이터에 따라 Set 구현체가 결정된다.

###### 위 코드에서의 단점
- 우선 코드가 굉장히 길다
  - 원래는 생성자 한 번만 호출하면 끝나지만 리플렉션을 사용하니 코드가 굉장히 길어졌다
- 런타임에 발생할 수 있는 exception들이 굉장히 많다
  - 이는 그냥 생성자를 이용하면 모두 컴파일타임에 체크되는 문제들이지만 리플렉션을 사용하니 런타임에 잡아야 한다

## 핵심 정리
    리플렉션은 되도록이면 객체 생성에만 사용하자. 그리고 생성한 객체도 그 상위 타입으로 참조하여
    사용하자