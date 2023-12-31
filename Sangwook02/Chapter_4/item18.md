# Item 18. 상속보다는 컴포지션을 사용하라

상속은 캡슐화를 깨뜨린다.  
상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있는데, 
만약 상위 클래스의 동작이 바뀐 것을 하위 클래스에 반영하지 못하면 잘 작동하던 
하위 클래스의 동작에 문제가 생길 수 있는 것이다.  

때문에, 기존의 클래스를 상속하기 보다 새로운 클래스를 만들고 기존 클래스를 private 으로 참조하게 하는 것이 좋다.  
새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 컴포지션이라고 부른다.  
새 클래스의 메서드들은 기존 클래스의 메서드들을 호출하여 반환하며, 이를 forwarding 이라고 한다.  

상속은 반드시 하위 클래스가 상위 클래스의 진짜 하위 타입인 상황(is-a 관계)에서만 사용해야 하자.