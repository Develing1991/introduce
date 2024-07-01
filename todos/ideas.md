todo list

- f r t
<del>
- 기능 추가에 따른 유즈케이스, 요구사항정의서 업데이트
- <del>redis 구성 - slave, master sentinel ( 로그아웃 토큰 관리, 검증 필터링 )</del>
- user-service를 제외한 다른 마이크로 서비스들 미구현 된 AOP 검증 로직 추가  
- 공통 코드 common-service로 분리 메이븐 업로드 또는 jar로 패키징 후 import로 사용
- 인증 관련 기능 로직 auth-service로 분리
- RESTful에 대한 고민 ing..
- oauth 추가 (add 시퀀스 다이어그램)
- payment-service 추가
- Resilience4J circuitbreaker 추가
- 사용자 주문 취소 건 알림을 위한 SSE(Server Sent Events)기능 추가
- CI/CD 추가적인 빌드 검증 sonarqube-server 추가
- 배포 서버 docker-server에서 k8s-server로 이관 해보기
</del>

### 물류센터 관련
예를 들어 쿠x같이 물류센터가 여러 군데 나뉘어 있을 수 있기 때문에  
특정 지역에서 찜 해놨던 상품을   
다른 지역에서 주문을 하려 앱을 열면 해당 상품을 주문할 수 없던 이유..  
해당 지역의 물류센터에는 해당 상품이 없거나 혹은 상품의 재고가 없음  
이런 케이스도 있을 수 있으니 테이블에 상품에 배송 가능한 물류 창고 로케이션 필드도 추가하고  

또한 사용자의 현재 접속 위경도를 받아서 네이버 지도 같은 map api를 호출해서 가까운 물류센터를 계산해 컨택할 건데   
일정 호출 건이 넘어가면 비용이 들게 되니까..  
redis의 geo활용 사례를 참고해서 위경도 값을 넣어 놓고 관리하는 것도 방법 일듯 하다  
