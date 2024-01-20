### Item37 : ordinal 인덱싱 대신 EnumMap을 사용하라

```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
    
    final String name;
    final LifeCycle lifeCycle;
    
    Plant(String name, LifeCycle lifeCycle) {
      this.name = name;
      this.lifeCycle = lifeCycle;
    }
    
    @Override
    public String toString() {
      return name;
    }
}
```
```java
public static void main(String[] args) {
    Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
    
    for (int i=0; i<plantsByLifeCycle.length; i++)
        plantsByLifeCycle[i] = new HashSet<>();
    
    for (Plant p : garden)
        plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
    
    // 결과 출력
    for (int i=0; i<plantsByLifeCycle.length; i++) {
        System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
    }
}
```
위와 같이 ordinal 메서드를 통해 얻은 값을 인덱스로 이용하여 배열에 접근하는 방식은 문제가 있다.
- 제네릭은 배열과 호환되지 않으니 비검사 형변환을 해야 한다
- 마지막 결과 출력에 어떤 상수 카테고리에 속하는지 직접 명시해야 한다
- 인덱스를 통해 배열에 접근하므로 해당 값이 사용해도 안전한 값인지를 보장해야 한다

그래서 위 방법은 효율적이지 않을 뿐만 아니라 위험하다. 다행히 대안은 존재한다.

###### EnumMap
Enum을 키로 사용하는 map이다.
```java
    Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
        new EnumMap<>(Plant.LifeCycle.class);
    
    for (Plant.LifeCycle lc : Plant.LifeCycle.values())
        plantsByLifeCycle.put(lc, new HashSet<>());
    
    for (Plant p : garden)
        plantsByLifeCycle.get(p.lifeCycle).add(p);
    
    System.out.println(plantsByLifeCycle);
```
EnumMap을 사용하면 위에서 ordinal을 사용할 때 발생할 수 있는 문제점이 모두 해결된다.
EnumMap은 다차원 관계를 표현하는 것에 있어서도 엄청난 강점을 가진다. 하지만 우선 배열로 열거 타입의 다차원 관계를 구현한 예시를 보자.

```java
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
      MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
      
      private static final Transition[][] TRANSITIONS = {
         { null, MELT, SUBLIME },
         { FREEZE, null, BOIL },
         { DEPOSIT, CONDENSE, null }
      };
      
      public static Transition from(Phase from, Phase to) {
         return TRANSITIONS[from.ordinal()][to.ordinal()];
      }
    }
}
```
보다시피 2차원 배열로 관계를 유지하며 Phase의 ordinal을 기준으로 2차원 배열에서의 상전이를 표현했다.
그래서 행과 열이 같은 경우 null로 처리했음을 알 수 있다.

```java
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
      MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
      BOILD(LIQUID, GAS), CONDENSE(GAS, LIQUID),
      SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);
      
      private final Phase from;
      private final Phase to;
      
      Transition(Phase from, Phase to) {
         this.from = from;
         this.to = to;
      }
      
      private static final Map<Phase, Map<Phase, Transition>>
         m = Stream.of(values()).collect(groupingBy(t -> t.from,
            () -> new EnumMap<>(Phase.class),
            toMap(t -> t.to, t -> t, 
               (x, y) -> y, () -> new EnumMap<>(Phase.class))));
               
      public static Transition from(Phase from, Phase to) {
         return m.get(from).get(to);
      }
    }
}
```
이 코드는 훨씬 안정적이고 안전하다. 하지만 map을 초기화하는 코드가 이해하기 쉽지는 않다.
하지만 쉽게 이해하면 된다.  
첫 번째 groupingBy는 외부 Map의 key를 세팅하는 부분이다.
그 다음에 나오는 toMap같은 경우 외부 map의 value인 map을 세팅하는 것이다. 이때 외부 map의 key는
상전이 이전 상태를, 외부 map의 value인 map은 상전이 이후의 상태와 상전이를 각각 key,value로 가지는 map이다.

만약 이 상황에서 새로운 phase인 PLSMA를 추가한다고 할 때  
배열로 구현한 경우 phase 에 상수 1개, transition에 상수 2개 그리고 9개짜리 배열을 16개짜리 배열로 교체해야 한다.

EnumMap으로 구현한 경우 phase 에 상수 1개, transition에 상수 2개만 새로 추가하면 문제없다.
코드는 조금 더 복잡해 보이더라도 성능면에서 그리고 안정성면에서 확실한 우위를 점함을 알 수 있다.

## 핵심 정리
    ordinal()을 이용한 인덱싱을 이용하기 보단 EnumMap을 이용하자. 열거 타입의 다차원 관계를 
    표현하는 경우에 더 효과를 볼 수 있다.