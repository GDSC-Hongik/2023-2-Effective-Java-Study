# Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라

정적 팩토리와 생성자의 경우 선택적 매개변수를 적용할 때 어려움이 있다.

서비스 초창기에는 아주 작은 유저 클래스가 존재할 수 있다. 1) 이름 2) 이메일 두 멤버만 가지고 있을 때, 객체를 생성하는 것은 별 문제가 없다. 서비스가 커져서, 유저 클래스에 다양한 정보를 저장하고 싶을 수 있다. 이제는 나이, 성별, 거주지, 직업, 직장, 학교, 취미, 프로필 사진, 관심사 등등을 저장한다. 하지만 이 정보들은 유저가 가입할 때 선택적으로 기입할 수 있는 정보들이다. 이 정보들에 대하여 우리는 간단하게 전체 필드에 대한 생성자를 만든 수 인자로 null을 전달하는 방식을 통해 간단하게 해결할 수 있다.

```java
public User(
    String name,
    String email,
    int age,
    Gender gender,
    String address,
    JobType job,
    String company
) {
    // any validate logic goes here (name, email)
    this.name = name;
    this.email = email;
    this.age = age;
    this.gender = gender;
    this.address = address;
    this.job = job;
    this.company = company;
}
```

이제 이름과 이메일, 그라리고 성별만 입력한 유저의 인스턴스를 생성하고 싶다면, 아래와 같이 null을 전달하면 된다.

```java
User user = new User(
    "김철수",
    "test@email.com",
    null,
    Gender.MALE,
    null,
    null,
    null
)
```

아뿔싸! 하지만 원시 타입의 경우 null을 할당할 수 없다. 따라서 위 코드를 호출하면 예외가 발생한다. 이를 해결하려면 인자에 대하여 null 여부를 체크하여 예외를 던질 수 잇다.

```java
public User(
    String name,
    String email,
    int age,
    Gender gender,
    String address,
    JobType job,
    String company
) {
    if (age == null) {
        throw new IllegalArgumentException("age must not be null");
    }
    // any validate logic goes here (name, email)
    this.name = name;
    this.email = email;
    ...
}
```

만약 직업을 작성하지 않는 경우 이제 null이 아닌 `JobType.NONE` 을 할당하고 싶다면 어떻게 해야 할까? 클라이언트가 해당 인자를 전달하도록 하는 것보다는, 생성자에 해당 로직을 구현하는 쪽이 더 좋을 것이다.

```java
public User(
    String name,
    String email,
    int age,
    Gender gender,
    String address,
    // JobType job,
    String company
) {
    this(name, email, age, gender, address, JobType.NONE, company);
}

public User(
    String name,
    String email,
    int age,
    Gender gender,
    String address,
    JobType job,
    String company
) {
    // any validate logic goes here
    if (job == null) {
        throw new IllegalArgumentException("job must not be null");
    }
    this.name = name;
    this.email = email;
    this.age = age;
    this.gender = gender;
    this.address = address;
    this.job = job;
    this.company = company;
}
```

이렇게 JobType을 받는 생성자와 받지 않는 생성자를 분리하는 방식으로 해결할 수 있다. 하지만 이런 식으로 필드 초기화 로직이 필요할 때마다 생성자를 추가하는 것은 번거로운 일이다. 심지어 로직이 추가될 때마다 생성자의 가짓수는 2배씩 증가한다. 이제는 gender 필드에 대하여 null을 허용하지 않고 기본적으로 `Gender.NONE` 을 할당하고 싶다면, 총 4개의 생성자가 존재할 수 있다. 상당히 까다로운 일이다.

또한 이 사례를 통해서 null을 넘기는 것이 그리 바람직한 방법은 아니라는 것 역시 알 수 있다. null을 허용하게 되면 gender 필드에 기본값을 null이 아닌 `Gender.NONE` 으로 설정하겠다는 정책을 지키기 어렵다. 이를 위해서 다시 null 검증 로직을 추가해주고... 번거로움이 있다. 클라이언트는 null을 넘기지 않도록, 선택적 매개변수가 존재하는 경우에만 해당 매개변수를 받는 생성자를 사용하도록 하는 쪽이 좋다.


```java
// 이름, 이메일만 받는 경우
public User(
    String name,
    String email
) {
    // any validate logic goes here
    this(name, email, 0, Gender.NONE, null, JobType.NONE, null);
}

// 이름, 이메일, 나이를 받는 경우 (null이 아니라고 가정)
public User(
    String name,
    String email,
    int age 
) {
    // any validate logic goes here
    this(name, email, age, Gender.NONE, null, JobType.NONE, null);
}
```

이런 느낌이다. 전체 생성자의 파라미터에 null을 넘기는 것보다는 훨씬 신뢰성 있게 객체를 생성할 수 있다.

이걸 점층적 생성 패턴이라고 한다.
역시 생성자가 너무너무 많아진다는 문제가 있다.
차라리 생성자가 많아지는 걸 감수하고 하나의 생성자에 파라미터에 null을 허용하는 방식을 쓰는 게 낫지 않을까 하는 생각도 든다.

또 다른 문제도 있다. 생성자가 많아지면 파라미터 순서를 실수하기 쉽다는 거다. 자바는 함수 파라미터를 넘길 때 positional argument 방식을 사용하기 때문에 파라미터가 많은 경우 실수하기 쉽다는 문제가 있다. 번외로 형제자매 격의 코틀린은 named argument 방식을 지원하기 때문에 이런 문제가 발생하지 않는다.

```kotlin
fun add(
    a: Int, 
    b: Int
) {
    return a + b
}
```

### 자바빈즈 패턴

자바빈즈 패턴은 간단하게 말하자면 기본 생성자로 객체를 생성하고 setter로 값을 업데이트하는 방식이다. 위에서 언급된 문제는 해결할 수 있겠지만 객체지향에서는 너무너무 좋지 않은 방식이다. 

- 각 필드에 대해서 final을 사용할 수 없기 때문에 불변 객체를 만들 수 없다.
    - 일관성이 깨진다는 것 역시 비슷한 맥락이다.
- 캡슐화가 깨진다.
    - 가령 주문 객체에 대하여 `orderAmount` 와 `discountAmount` 를 받아서 `this.totalPaymentAmount = orderAmount - discountAmount` 와 같이 필드에 값을 할당하는 로직이 있을 수 있다. setter를 사용하면 클라이언트에게 해당 로직이 그대로 노출되므로 아주아주 좋지 않은 방식이다.
- 파라미터가 많은 경우 클라이언트의 코드가 setter 로직으로 범벅이 되어버린다.


### 빌더 패턴

빌더 패턴을 쓰면 모든 문제를 해결할 수 있다. 빌더 패턴은 클래스 내부의 정적 클래스로 존재하면서, `build()` 가 호출되기 전까지 깨진 일관성을 격리하는 / 혹은 클라이언트가 넘긴 값들을 임시로 저장하는 "버퍼" 역할을 한다. 그리고 `build()` 가 호출되면 버퍼링된 값을 생성자로 한번에 넘겨서 일관성을 보장할 수 있게 해준다. 나아가, 빌더 클래스는 클래스 내부의 정적 클래스로 선언되기 때문에 위에서 언급된 캡슐화 문제를 해결할 수 있다.

직접 구현해보자. 추가로, 위에서 언급했던 결제 예시도 함께 구현해보자. 유저는 총주문금액과 총할인금액, 총결제금액을 가지고 있다. 이때 총결제금액은 총주문금액에서 총할인금액을 뺀 값이다.

```java

```java
public class User {

    private final String name;
    private final String email;
    private final int age;
    private final Gender gender;
    private final String address;
    private final JobType job;
    private final String company;
    // 결제 예시
    private final Money totalOrderAmount;
    private final Money totalDiscountAmount;
    private final Money totalPaymentAmount;

    public static class Builder {
        // 빌더 버퍼에 저장할 필수 매개변수
        private final String name;
        private final String email;
        // 빌더 버퍼에 저장할 선택적 매개변수: 기본값으로 초기화
        private final int age = 0;
        private final Gender gender = Gender.NONE;
        private final JobType job = JobType.NONE;

        public Builder(String name, String email) {
            // validate not null
            this.name = name;
            this.email = email;
        }

        public Builder age(int age) {
            // validate not null
            this.age = age;
            return this;
        }

        ...


        public User build() {
            // 빌더 버퍼에 저장된 값으로 생성자 호출
            return new User(this);
        }
    }

    // 빌더 객체를 받는 생성자는 private으로 선언하여 외부에서 호출할 수 없도록 한다.
    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
        this.gender = builder.gender;
        ...
        this.totalPaymentAmount = builde.totalOrderAmount
            .minus(builder.totalDiscountAmount);
    }
}
```

검증 로직을 Builder에서 구현한 이유는 외부에서 호출된 빌더 생성자의 초입에서 예외를 잡을 수 있기 때문이다. 물론 원한다면 private User 생성자에서 검증 로직을 작성할 수도 있다. 롬복을 사용한 빌더 패턴의 경우 내부 정적 빌더 클래스를 생성할 때 따로 검증 로직을 지정할 수 없으므로 기반이 되는 생성자에 검증 로직을 작성해줄 수밖에 없다.

이제 이렇게 호출할 수 있다.

```java
// 김철수, test@email.com, 20, Gender.MALE, 주소 null, JobType.NONE, ...
User user = new User
    .builder("김철수", "test@email.com")
    .age(20)
    .gender(Gender.MALE)
    .build();
```

객체를 생성할 때 "순서"가 존재하는 경우 유용하게 사용할 수 있다. item 1에서 언급한 예시다.

```java

라면클래스 라면 = 라면클래스
        .조리시작(신라면)
        .물넣기(1000)
        .스프넣기(스프.매운맛)
        .면넣기(면.기본면)
        .끓이기(10분)
        .조리완료();

```

### 번외 : 정적 팩토리 메서드와 빌더 중 어떤 것을 사용해야 할까?

그것은 유우비트의 "내일부터 바로 써먹는 클린코드" 영상을 보면 된다.

https://youtu.be/PAjcpYrwRa0?si=xlAk7onO-HI993yq

결론부터 말하자면 내부에서는 빌더를 사용하고 외부에는 정적 팩토리 메서드를 사용한다. 그 이유는 item 1과 item 2에 작성한 내용을 잘 읽어보면 알 수 있다.