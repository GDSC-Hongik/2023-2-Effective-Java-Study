### Item17 : 변경 가능성을 최소화하라

###### 불변 클래스
인스턴스의 내부 값을 수정할 수 없는 클래스  
불변 인스턴스에 저장된 값은 프로그램이 종료되면서 해당 객체가 파괴되는 순간까지 유지된다.

###### 불변 클래스 설계를 위한 5가지 규칙
1. 객체의 상태를 변경할 수 있는 메서드(변경자)를 제공하지 않는다.
   - 흔히 말하는 setter 메서드를 포함한 다른 업데이트 메서드들을 뜻한다.
2. 클래스 확장을 막는다.
   - final 키워드를 이용하여 클래스를 선언하면 해당 클래스는 확장 불가능하다.
3. 모든 필드를 final로 선언한다.
4. 모든 필드를 private으로 선언한다.
5. 클래스 본인을 제외하곤 외부에서 가변 필드에 접근할 수 없게 한다.
   - 클래스 내부에 가변 객체를 가리키는 참조 필드를 가지고 있다면 클라이언트 코드에서 절대로 해당 참조를
   접근할 수 없게 해야 한다. 가변 객체를 반환해야 할 경우 방어적 복사를 한 뒤 반환하자.
   
###### 불변 클래스 예시
```java
public final class Complex {
    private final double re;
    private final double im;

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```
- 위 클래스를 보면 복소수에 대한 사칙연산 메서드를 제공한다.  
- 해당 메서드들의 내부 구현을 보면 **피연산자 자체를 수정해서 반환하는 것이 아니라** **연산을 적용한
새로운 인스턴스를 생성하여 반환함**을 알 수 있다.  
- 이처럼 메서드를 피연산자에 적용하지만 피연산자의 상태는 변하지 않는 패턴을 `함수형 프로그래밍`이라고 한다.  
- 또한, 피연산자의 상태가 변하지 않음을 명시적으로 표현한 것이 메서드 이름이다. 이름을 보면 add, sub와 같은 동사가 아닌
plus, minus와 같은 전치사를 사용했다.

불변 클래스는 절대로 변경될 수 없기 때문에 해당 인스턴스가 여러 쓰레드에서 동시 사용되더라도 안전하다(Thread-safe).
즉, 불변 클래스는 Thread-safe한 클래스 설계를 위한 가장 쉬운 방법이라고 할 수 있다.

###### 불변 클래스의 이점
1. 정적 팩터리를 통해서 캐싱된 객체를 반환할 수 있다.
   - 어차피 인스턴스는 변하지 않을 것임을 알기 때문에 자주 사용되는 인스턴스는 캐싱했다가 여러 클라이언트 코드에서 공유하도록 할 수 있다.
2. 복사 메서드를 선언할 필요가 없다.
   - clone, 복사 생성자등 인스턴스를 복제하는 메서드를 제공할 필요가 없다. 어차피 복사해도 다 똑같기 때문이다.
3. 불변 클래스 인스턴스끼리는 내부적으로 데이터를 공유할 수 있다.
   - BigInteger 클래스는 내부적으로 부호와 수의 절대값을 분리해서 유지하는데 수의 절대값은 int 배열로 유지한다. negate라는 메서드는 기존 숫자에서 부호만 반대인 수를 반환한다.
   이때, 기존 숫자에서 부호만 바꾸고 수의 절대값이 담긴 배열은 공유하는 새 인스턴스를 반환하도록 한다. 어차피 불변 객체이기 때문에 값을 공유해도 문제가 되지 않는다.
4. 불변 객체는 Map의 키, Set의 원소로 사용하기 적합하다.
5. 불변 객체는 그 자체로 실패 원자성을 보장한다.
   - `실패 원자성`: 메서드 실행 중 예외가 발생하여 로직 실행이 중단되더라도 객체는 메서드 실행 전과 같이 유효한 상태를 유지해야 한다.

###### 불변 클래스의 단점
- 값이 다르다면 항상 새로운 인스턴스를 생성해야 한다.
   - 위 예제에서 언급했던 BigInteger 클래스의 경우 기존의 값과 오로지 1비트만 달라지는 인스턴스가 필요한 경우에도 새로운 인스턴스를 반환한다.
   만약 객체 생성 비용이 큰 클래스라면 성능 문제로 이어질 수 있다.

###### 불변 클래스 설계를 위한 또 다른 방법
위에서 불변 클래스를 만들려면 final 클래스로 선언하여 상속을 막으라고 했다.  
이 방법 대신 모든 생성자를 private/package-private으로 바꾸고 public 정적 팩터리를 제공하는 방법도 사용 가능하다.
```java
public final class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

  	public static Complex valueOf(double re, double im){
  		return new Complex(re, im);
  	}
}
```
이렇게 하면 final 클래스가 아니기 때문에 필요한 경우 위 클래스를 상속하는 package-private 클래스를 생성해서 내부적으로 사용할 여지가 생긴다(생성자를 package-private으로 선언한 경우).  
하지만 이 경우에도 오로지 같은 패키지 내부의 클래스에서만 상속이 허용되므로 비교적 안전하다고 볼 수 있다.

그리고 위에서 모든 필드를 final로 선언하라고 했다.  
하지만 성능 개선을 위한 예외 사항이 있다. 계산 비용이 비싼 작업에 대한 결과물은 해당 메서드가 처음 실행될 때 계산을 한 뒤에 그 값을 final이 아닌 필드에 캐싱해뒀다가 이후에 재사용할 수 있다.

###### 정리
- 클래스에 getter 메서드가 있다고 해서 setter를 만들지 말자.
- 웬만해선 불변 클래스로 설계하자. 특히나 단순한 값 클래스에 대해선 더더욱 말이다.
- 특별한 목적이 있지 않는 한 모든 필드는 private final로 선언하자.
- 특별한 목적이 있지 않는 한 생성자와 정적 팩터리를 제외한 수정 메서드는 public으로 제공하지 말자.