### Item87 : 커스텀 직렬화 형태를 고려해보라

이전 item에서도 언급했듯이 `Serializable`을 구현하면서 기본 직렬화를 사용하는 클래스를 릴리즈하면
다음 릴리즈에서 내부 구현을 바꾸는 것이 어렵다. 한 번 공개한 직렬화 형태는 지속적으로 지원해야 하기 때문이다.  
그렇다면 기본 직렬화 형태는 어떤 형태일까


###### 기본 직렬화 형태
객체의 기본 직렬화 형태는 해당 객체를 루트 객체로 하는 객체 그래프의 물리적 형태를 
인코딩한 것이다.  
즉, 해당 객체 내부에 포함된 데이터 + 해당 객체로부터 접근할 수 있는 모든 객체 + 객체 그래프의 물리적 위상(topology)까지
포함한다는 뜻이다.  

하지만 '이상적인' 직렬화 형태는 해당 객체의 '물리적' 형태를 나타내는 것이 아니라 해당 객체의 '논리적' 형태를 
나타내야 한다.

###### 논리적 형태와 물리적 형태가 같은 경우
아래 클래스를 보자
```java
public class Name implements Serializable {
    private String firstName;
    private String middleName;
    private String lastName;
}
```
이름은 논리적으로(미국식 이름) `성`, `이름`, `중간 이름`으로 이뤄져 있다.
위 `Name` 클래스는 이름의 논리적 형태를 그대로 물리적 형태로 나타낸 것이다.  
이 경우 논리적 형태와 물리적 형태가 일치하므로 기본 직렬화 형태를 사용함에 있어 무리가 없다.

###### 논리적 형태와 물리적 형태가 다름에도 기본 직렬화 형태를 사용할 때 존재하는 문제점
- 공개 API가 현재의 내부 표현 방식에 영원히 묶이게 된다
  - 다음 릴리즈에 해당 클래스의 내부 구현 방식을 설령 바꾸더라도 기존에 공개한 직렬화 형태 지원을 위해 기존 코드를 계속 놔둬야 한다
- 너무 많은 공간을 차지할 수 있다
  - 기본 직렬화 형태를 사용하면 해당 클래스의 물리적 형태까지 외부로 노출하게 된다. 이는 내부 구현을 불필요하게 노출하는 것임과 동시에 직렬화된 결과의 크기가
  너무 커지게 된다는 문제가 있다
- 직렬화하는 데 오랜 시간이 걸릴 수 있다
  - 기본 직렬화 형태는 해당 객체 그래프를 일일이 순회하며 그 물리적 위상을 파악해야 한다
- 스택 오버플로우가 발생할 수 있다
  - 기본 직렬화 형태에서 객체 그래프를 순회할 땐 재귀 형태로 순회하므로 지속된 호출로 인한 스택 오버플로우가 발생할 수 있다
  
###### 직렬화 사용할 때 주의할 점
- 직렬화 대상이 아닌 필드에 대해선 'transient' 한정자를 이용하자
  - defaultWriteObject 메서드를 호출하면 'transient'가 붙지 않은 필드에 대해선 기본적으로 직렬화가 진행되므로 명심하자
- 기본 직렬화 사용 시 'transient' 키워드가 붙은 필드들은 기본 값으로 세팅된다(정수형은 0, boolean형은 false, 참조형은 null)
  - 만약 이러한 점이 내부 불변식을 깨뜨린다면 readObject 메서드를 선언하고 defaultReadObject 메서드를 실행한 뒤 해당 필드값들을 직접 초기화하면 된다
- 기본 직렬화든 커스텀 직렬화든 객체의 전체 상태를 읽는 메서드에 적용하는 동기화 매커니즘을 직렬화에도 적용해야 한다
  - 해당 클래스 내부의 모든 메서드가 동기화된 상태라면 직렬화 메서드도 다음과 같이 동기화가 되어야 한다
  ```java
  private synchronized void writeObject(ObjectOutputStream s) {
      s.defaultWriteObject();
  }
  ```
- 기본 직렬화든 커스텀 직렬화든 '직렬화 버전 UID'를 명시적으로 부여하자
  - 명시적으로 부여하면 런타임에 이 값을 계산하는 시간이 줄어들기 때문에 약간의 성능상 이점을 챙길 수 있다.
  - 해당 클래스를 처음 작성하는 거라면 다음과 같이 아무 값이나 '직렬화 버전 UID'로 설정해도 무리가 없다.
  ```java
  private static final long serialVersionUID = <아무 long 값>
  ```
  - 만약 기존에 '직렬화 버전 UID'를 부여하지 않았던 구버전 클래스와의 호환성을 유지하면서 수정을 하고 싶은 경우
  구버전 클래스에서 자동 생성되어 사용되던 값을 사용해야 한다.

## 핵심 정리
    기본 직렬화는 해당 클래스의 물리적 구조와 논리적 구조가 일치할 때만 사용하자.
    나머지 경우엔 적절한 커스텀 형태의 직렬화를 사용하자.
    해당 클래스의 직렬화 형태를 일단 릴리즈하고 나면 그 이후엔 수정하기가 어렵다.
    