### Item12 : toString을 항상 재정의하라

###### Object가 제공하는 toString
Object 클래스에 정의된 toString 메서드는 의미있는 데이터를 반환한다고 보긴 어렵다.
```
NyPizza@3b07d329
```
- 이는 실제로 Object가 제공하는 toString을 이용해서 NyPizza라는 객체를 출력한 것이다.  
- 보다시피 `클래스_이름@16진수_해시코드`의 형태로 반환되며 해당 피자에 대한 상세 정보를 알 수가 없다.

###### toString 메서드 사용처
개발자가 직접 toString 메서드를 호출하지 않아도 특정 인스턴스를 
- println, printf
- 문자열 연결 연산(+)
- assert 구문  

에 넘기면 자동으로 toString 메서드가 호출된다. 

###### toString을 재정의해야 하는 이유
toString 일반 규약에
- '간결하면서도 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다'고 나와 있다.  
- '모든 하위 클래스에서 toString을 재정의하라'고 나와있기도 하다.

toString을 재정의하면 로그를 출력할 때도 훨씬 도움이 된다.
```java
NyPizza@3b07d329 // toString 재정의하지 않은 경우
{name: NYPizza, radius: 3} // toString 재정의한 경우
```

###### toString 재정의 시 반환 형식의 문서화?
**특정 클래스에서 toString 메서드를 재정의 하고 그 반환 형식을 문서화해야 할까?**

전화번호나 행렬과 같은 `값타입`이라면 **문서화하는 것이 좋다**.  
문서화하는 것 자체가 표준을 정하는 것이고 이렇게 문서화를 하면 사람이 직접 읽을 수 있기 때문이다.  
그리고 문서화를 했다는 것은 웬만해선 표준이 바뀔 일이 없다는 뜻이므로 사용자 입장에선 toString 메서드에서 반환되는 데이터를 파싱한 뒤 추가적으로 가공하여 본인만의 객체로 저장할 수도 있다.

만약 문서화를 하기로 결정했다면 
- toString 반환 형식 -> 객체 
- 객체 -> toString 반환 형식

으로의 변환을 지원하는 정적 팩터리 혹은 생성자를 제공하는 것이 좋다.

`생성자 예시`
```java
public class Pizza {

    private int size;

    public Pizza(int size) {
        this.size = size;
    }

    // format은 {size: 100}의 형식을 준수한다고 가정.
    public Pizza(String format) {
        this.size = extractSize(format);
    }

    private int extractSize(String format) {
        int index = format.indexOf("size: ");
        if (index == -1)
            throw new AssertionError("잘못된 형식입니다.");

        String size = format.substring(index + 6, format.length() - 1);
        return Integer.parseInt(size);
    }
}
```
`정적 팩터리 예시`
```java
public class Pizza {

    private int size;

    public Pizza(int size) {
        this.size = size;
    }
    
    public static Pizza from(String format) {
        int index = format.indexOf("size: ");
        if (index == -1)
            throw new AssertionError("잘못된 형식입니다.");

        String size = format.substring(index + 6, format.length() - 1);
        return new Pizza(Integer.parseInt(size));
    }
}
```

###### 문서화의 단점
toString 반환 형식을 문서화하면 단점도 생긴다.
- 형식을 한 번 문서화해서 정의하면 이후에 바꾸기가 어렵다.
  - 사용자들은 문서를 바탕으로 반환 데이터를 파싱하고, 새로운 객체도 만들고, 영속 데이터를 만들어서 저장할 것이다.
  이런 상황에서 섣불리 toString의 반환 형식을 바꾸게 된다면 굉장히 많은 문제가 발생할 것이다.  
  문서화라는 것은 결국에는 표준을 세우는 것이라고 봐도 무방하기에 그 표준에 얽매일 수도 있다.

###### 문서화 여부와 관계없이 지켜야 하는 사항
**toString이 반환한 값에 포함된 정보들을 조회할 수 있는 api들을 제공해야 한다.**

이는 toString에 포함된 필드들의 getter를 제공하라고 이해해도 될 것 같다. 만약 getter를 제공하지 않는다면
사용자들은 어쩔 수 없이 toString의 반환 결과를 파싱해서 사용할 수밖에 없는데 이는 매우 비효율적이다.


## 핵심 정리
    Object가 제공하는 toString을 재정의하자. 만약 슈퍼 클래스가 적절한 toString을 재정의했다면 그것을 재사용하자.