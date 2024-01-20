# Item 33. 타입 안전 이종 컨테이너를 고려하라

컨테이너 대신 키를 매개변수화하고 컨테이너에 값을 넣거나 뺄 때마다 키를 함께 제공하는 방식을 
**타입 안전 이종 컨테이너 패턴**이라 한다.
```java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```
- Class 객체를 매개변수화하여 key로 활용
- Favorites에는 일반적인 Map과 달리 여러 타입 저장 가능

### favorites의 타입은 Map<Class<?>, Object>
비한정적 와일드카드 타입이라 맵 안에 아무것도 넣을 수 없지 않나?
- 맵이 와일드카드인 것이 아니라 키가 와일드카드임
- 다양한 타입을 지원할 수 있는 이유임

### Favorites의 제약
1. Class 객체를 제네릭이 아닌 로 타입으로 넘기면 타입 안전성이 깨짐
2. 실체화 불가 타입에는 사용 불가  
   String이나, String[ ]같은 타입은 저장 가능, List<String>은 불가능