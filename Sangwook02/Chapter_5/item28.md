# Item 28. 배열보다는 리스트를 사용하라

## 배열 vs 제네릭 타입
- 배열은 공변인데 제네릭은 불공변임.
  - 공변: 함께 변한다.
    - ex) Super의 하위인 Sub라면, Super[]의 하위인 Sub[]
  - Long 타입에 String을 넣는다면 배열(공변)에서는 런타임에서 알아챌 수 있는데 이건 너무 늦음.
- 배열은 실체화됨
  - 배열: 런타임에도 담기로 한 원소의 타입을 알고 있음

## 실체화 불가 타입
- E, List<E> 와 같은 타입
- 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입
- 배열로 형변환할 때 발생하는 비검사 형변환 경고는 배열 대신 컬렉션을 사용하면 제거 가능

```java
public class Chooser {
    private final Object[] choiceArray;
    
    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }
   
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```
배열을 사용한 위의 코드를 제네릭을 사용한 아래의 코드로.
```java
public class Chooser<T> {
  private final List<T> choiceList;

  public Chooser(Collection<T> choices) {
    choiceArray = new ArrayList<>(choices);
  }

  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceList.get(rnd.nextInt(choiceList.size()));
  }
}
```