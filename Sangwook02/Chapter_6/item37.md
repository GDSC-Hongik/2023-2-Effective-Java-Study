# Item 37. ordinal 인덱싱 대신 EnumMap을 사용하라

## EnumMap
- EnumMap은 키가 열거 타입인 Map 구현체
- ordinal을 사용하는 것에 비해 안전하고 이해하기 쉬움
- 내부에서 배열을 사용하므로, Map의 타입 안전성과 배열의 성능을 모두 얻음

아래와 같이 사용할 수 있음.
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


public class Main {
    public static void main(String[] args) {
        Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
        for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
            plantsByLifeCycle.put(lc, new HashSet<>());
        }

        for (Plant p : garden) {
            plantsByLifeCycle.get(p.lifeCycle).add(p);
        }
    }
}
```
