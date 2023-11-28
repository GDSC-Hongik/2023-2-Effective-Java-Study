# Item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

정적 메서드 및 필드로만 구성된 클래스는 인스턴스화 되어서는 안된다. 이를 정적 유틸리티 클래스라고 한다. 정적 유틸리티 클래스를 남용하게 되면 유지보수가 크게 어려워진다.

정적 유틸리티에 대한 유혹이 커지는 상황들이 있다. 서로 다른 클래스에 존재하는 중복 로직을 추출하는 경우이다. 

우리가 구매한 일반 로또와, 추첨 대상이 되는 당첨 로또가 존재한다고 하자. 둘을 각각 `Lotto` 와 `WinningLotto` 라고 하겠다. `Lotto` 는 6자리 숫자를 가지고, `WinningLotto` 는 6자리 숫자에 더하여 보너스 숫자 1개를 추가적으로 가진다. `Lotto` 와 `WinningLotto` 에는 모두 6자리 숫자가 존재하고, 이 6자리 숫자는 중복되면 안된다는 규칙이 존재한다.

```java
public Lotto(List<Integer> numbers) {
    validateNumbersDistinct(numbers);
    this.numbers = numbers;
}

public WinningLotto(List<Integer> numbers, int bonusNumber) {
    validateNumbersDistinct(numbers);
    validateBonusNumber(numbers, bonusNumber);
    this.numbers = numbers;
    this.bonusNumber = bonusNumber;
}
```

`validateNumbersDistinct` 메서드는 중복된 메서드이지만, `Lotto` 와 `WinningLotto` 에서 모두 사용되는 메서드이다. 이때 유지보수를 위해 중복을 제거하고 싶다면, 아래와 같이 정적 유틸리티 클래스를 만들 수 있다.

```java

public class LottoValidator {
    public static void validateNumbersDistinct(List<Integer> numbers) {
        if (numbers.size() != new HashSet<>(numbers).size()) {
            throw new IllegalArgumentException("중복된 숫자가 존재합니다.");
        }
    }
}

public Lotto(List<Integer> numbers) {
    LottoValidator.validateNumbersDistinct(numbers);
    this.numbers = numbers;
}

public WinningLotto(List<Integer> numbers, int bonusNumber) {
    LottoValidator.validateNumbersDistinct(numbers);
    validateBonusNumber(numbers, bonusNumber);
    this.numbers = numbers;
    this.bonusNumber = bonusNumber;
}
```

괜찮지 않나? 싶은 생각이 들 수도 있겠지만, 만약 `WinningLotto` 에서 numbers를 검증하는 로직이 변경되어, `LottoValidator` 의 `validateNumbersDistinct` 로직에 변경이 발생한다고 하자. 이때, `Lotto` 는 이 검증기를 사용하므로, 변경이 쉽게 파급된다.

조금 현실과 동떨어진 이야기이지만, 정적 유틸리티를 지양해야 하는 이유는 전역적으로 사용되는 메서드가 변경될 경우 프로그램은 변경을 감당할 수 없기 때문이다. 개인적으로는 스프링 등의 컨테이너 기술을 사용하는 경우, 컨테이너에서 관리되는 싱글턴으로 사용할 것을 적극 권장한다. 정적 유틸리티는 테스트하기도 어렵다. 하지만 컨테이너의 DI를 사용하면 이러한 경우에도 쉽게 테스트가 가능하다. 

정적 유틸리티는 악인가? 에 대해서는 다양한 관점이 있다고 본다. 상황에 따라서 적절한 선택을 내릴 수 있어야 한다.