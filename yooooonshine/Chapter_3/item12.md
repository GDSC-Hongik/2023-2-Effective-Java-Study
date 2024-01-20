# toString을 항상 재정의하라
---
근데 toString이 뭔데?
모든 클래스의 최상위 클래스로 항상 Object라는 클래스가 존재하며,
`extends Object`를 명시하지 않아도, java는 모든 클래스를 자동으로 `Object`하위 클래스로 자동 설정한다.

![[Pasted image 20231219145611.png]]
위는 Object 클래스에 있는 toString 메서드를 가져온 것이다.
보면 기본적으로 `클래스 이름@hashCode의 16진수 숫자`를 리턴한다.
이는 아무 의미 없는 값이다.

다음은 String 클래스의 toString메서드이다.
![[Pasted image 20231219150619.png]]
보면 return this로 재정의되어 있는 것을 볼 수 있다.
이를 이용하여 출력해보면
```java
public class Main {  
    public static void main(String[] args) throws IOException {  
        String a = "me";  
        System.out.println(a.toString());  
    }  
}
```
이의 결과로 me가 출력되는 것을 볼 수 있다.
분명히 this를 return했는데 어떻게 me가 출력될까?
여기서 a객체의 this는 a객체 자체를 의미하며, String 클래스에서 객체는 문자열 자체를 나타낸다. 따라서 return this를 하면 "me"가 나오는 것이다.

참고로 객체 자체를 출력하려하면 자동으로 toString메서드가 호출된다.
따라서 여기서는 `System.out.println(a);`코드를 실행해도 똑같은 결과를 얻을 수 있다.
## toString이 뭔지 알았으니 왜 toString을 항상 재정의해야 하는 지 알아보자
```java
public class phoneNumber extends Object{  
    public int first;  
    public int second;  
    public int third;  
  
    @Override  
    public String toString() {  
        return first + "-" + second + "-" + third;  
    }  
}
```
Object클래스의 toString은 기본적으로 `클래스 이름@hashCode의 16진수 해시코드`를 반환하므로 아무 의미 없는 값을 반환한다.

하지만 `toString`의 일반규약을 따르면 "간결하면서 사람이 읽기 쉬운 형태의 유익한 정보"를 반환해야 한다.
따라서 010-1234-5678 등의 유익한 정보를 담기 위해 toString을 재정의 하는 것이 좋다.
또한 `toString`의 규약은 "모든 하위 클래스에서 이 메서드를 재정의하라"고 되어 있다.

즉
* toString은 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야한다.
* toString에 대해 모든 하위 클래스에서 이 메서드를 재정의하라고 규칙이 정해져 있다.
라는 이유를 통해 toString을 항상 재정의하라 말할 수 있다.

## toString을 재정의 할 때 고려 사항
### 1. toString은 그 객체가 가진 주요 정보를 모두 반환하는 게 좋다.
즉 주요 정보를 모두 담지 않았을 때, 서로 다른 두 객체가 같다고 생각되게 끔 출력이 될 수 도 있기 때문이다.

예를 들어
```java
public class phoneNumber extends Object{  
    public int first;  
    public int second;  
    public int third;  
  
    @Override  
    public String toString() {  
        return first + "-" + third;  
    }  
}
```
이와 같이 `toString()` 메서드를 특정 정보를 빼서 만든다면, 010-1111-1111과 010-2222-1111을 같다고 생각하게 될 수 있다.

### 2.toString을 구현할 때면 반환값의 포맷을 문서화할지 정해야 한다.
즉 010-1111-1111을 받았을 때 자동으로 객체로 만들어주며, 객체 또한 자동으로 010-1111-1111 형식으로 만들어 주게 할 수 있다.
하지만 이는 한번 이렇게 정적 팩터리 메서드나, 생성자로 만들어 주면, 추후 평생 이 포맷을 따라야 하고, 포맷이 변경되면 큰 혼란을 야기할 수 있다.

따라서 포맷을 사용한다면 주석을 통해 포맷에 대해 의도를 명확히 밝혀야 한다.