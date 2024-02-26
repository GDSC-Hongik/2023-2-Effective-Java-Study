# 또 너냐?

오늘 자정에 배포를 앞둔 지뎃씨 플랫폼 api를 일찌감치 다 만들어두고 유튜브를 보고 있을 때, 
프론트 개발자님이 500 에러가 뜬다고 하셔서 로그를 까보니 NPE가 발생하고 있었습니다.  
이런 상황이 세 번이나 발생해버려서 NPE에 대한 경각심이 잔뜩 충전되었습니다.

## 우선 NPE란?
NPE란 Null Pointer Exception의 약자로, 
가장 흔하게 발생하는 Exception 중 하나이다.  
NPE는 null 값을 갖는 참조 변수로 객체 접근 연산자인 도트(.)를 사용했을 때 발생한다.  
객체 참조가 없는 상태에서 객체에 접근하려 했기 때문에 발생하는 것이다.

예를 들자면 아래와 같은 상황이다.
```java
public class NPE {
    public static void main(String[] args) {
        String str = null;
        System.out.println(str.length());
    }
}
```

## NPE 처리 방법
NPE를 예방하는 방법은 크게 세 가지가 있다.
1. try-catch문 사용  
2. 직접 null인지 체크
3. Optional 클래스 사용


### try-catch문 사용
NPE를 처리하겠다는 의지를 노골적으로 보여주는 방법이다.  
일반적으로 java 문법 책에서는 예외 처리를 위해 try-catch를 사용할 것을 권한다.  
권한다기 보다는 가장 기본적이고 대부분의 프로그래밍 언어에서 사용 가능한 방법이기 때문에 
많이 소개된다.
```java
public class NPE {
    public static void main(String[] args) {
        String str = null;
        try {
            System.out.println(str.length());
        } catch (NullPointerException e) {
            System.out.println("NPE 발생");
        }
    }
}
```
하지만, 이 방법은 NPE를 예방 가능한 방법이 아니라 처리하는 방법이기도 하고 try-catch를 
너무 많이 쓰면 코드 가독성도 떨어지고 장황해진다.  
즉, NPE 처리는 가능하나 권장하는 방법은 아니라는 것이다.

### 직접 null인지 체크
그렇다면 NPE를 예방하는 방법은 뭐가 있을까?  
우선, null인지 직접 체크하는 방법이 있다.  
이렇게 하면, null인지 먼저 체크한 후에 여부에 따라 다른 처리를 할 수 있다.  
위의 try-catch보다 선진적인 방법으로 판단된다.  
```java
public class NPE {
    public static void main(String[] args) {
        String str = null;
        if (str != null) {
            System.out.println(str.length());
        }
    }
}
```
그렇지만, 하나 문제가 있다.  
안 예쁘고 코드가 길어지므로 approve를 받지 못할 가능성이 높다.  
Optional 클래스를 사용하면 approve까지 쟁취할 수 있을 것이다!

### Optional 클래스 사용
null을 직접 확인하는 방식으로 짠 코드를 보자.  
member.getDepartment()에서 한 단계 더 들어가기 때문에 다른 필드와 달리 null 체크가 필요하다.
```java
public record MemberFindAllResponse(
        Long memberId,
        String studentId,
        String name,
        String department) {
    
        public static MemberFindAllResponse of(Member member) {
            String department = null;
            if (member.getDepartment() != null) {
                department = member.getDepartment().getDepartmentName();
            }
            return new MemberFindAllResponse(
                member.getId(),
                member.getStudentId(),
                member.getName(),
                department);
        }
}
```
정적 팩토리 메서드에서 굉장히 투박하게 null인지 직접 확인하고 있다.  
이를 Optional을 사용하면 훨씬 우아해지고 approve도 받을 수 있다. 
```java
public record MemberFindAllResponse(
        Long memberId,
        String studentId,
        String name,
        String department) {
    
        public static MemberFindAllResponse of(Member member) {
            return new MemberFindAllResponse(
                member.getId(),
                member.getStudentId(),
                member.getName(),
                Optional.ofNullable(member.getDepartment())
                    .map(Department::getDepartmentName)
                    .orElse(null));
        }
}
```
null을 뱉어낼 수도 있는 메서드에 Optional.ofNullable을 사용해 NPE를 예방하고 있다.  
사실 Optional을 사용하는 방법은 무의식적으로 service단에서 많이 사용하고 있다.  
```java
Member member = memberRepository.findById(memberId)
    .orElseThrow(() -> new CustomException(MEMBER_NOT_FOUND));
```
이렇게 service에서 repository를 호출할 때 findById와 같은 메서드가 Optional을 
반환하고 있기 때문에 Optional이라는 표현이 코드에 드러나지는 않지만 자연스럽게 NPE를 예방하고 있었던 것이다.