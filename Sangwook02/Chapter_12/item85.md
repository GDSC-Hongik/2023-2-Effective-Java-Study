# Item 85. 자바 직렬화의 대안을 찾으라

직렬화 -> 공격 범위가 넓고 게속해서 넓어지므로 방어가 어려움  
ObjectInputStream의 readObject 메서드를 호출하면 거의 모든 타입의 인스턴스를 만들 수 있음  
=> 무적 생성자

신뢰할 수 없는 바이트 스트림을 역직렬화 하는 것부터가 위험에 노출되는 것이므로 아무것도 역직렬화하지 않는 것이 좋음.  
=> JSON, Protocol Buffers 등을 사용하자

만약, 자바 직렬화를 완전히 피할 수 없다면 신뢰할 수 없는 데이터는 절대 역직렬화하지 않도록 해야 함.  
-> 화이트 리스트(신뢰 가능한 것들의 목록)를 관리하여 역직렬화를 허용할 클래스를 제한 가능