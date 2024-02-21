# Item 60. 정확한 답이 필요하면 float과 double은 피하자

float과 double은 근사치를 제공하므로, 정확한 값이 아님.  
정확한 값을 요구하는 경우, BigDecimal이나 int, long을 사용하자.  

### BigDecimal 단점
- 기본 타입보다 쓰기 불편
- 느림.

### 대안
- int와 long을 사용
- 소수점을 직접 관리해야 하는 점이 단점