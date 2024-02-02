todo list

- 기능 추가에 따른 유즈케이스, 요구사항정의서 업데이트
- redis 구성 - slave, master sentinel (디테일 스터디 진행중)      
- redis 사용 - auth, cache, transaction 등등..
    - 이전 적용 했던 내용 정리  
    -> 로그아웃 기능 user-service, seller-service, supervisor-service 로그아웃 시 redis-server의 redis에   
          access_token, refresh_token 키, 밸류 형태로 밀어넣기. 넣을 때 ttl 옵션은 각 토큰 decode 해서 expired 시간 셋팅 - 1시간, 12시간   
  
    -> 검증기능 gateway에 redisTemplate (토큰 검증 앞 단에 get redis + token ) 확인 로직을 추가    
    -> 로그아웃을 했다면 redis에 해당 키 토큰이 존재하므로 invalid한 token으로 응답 처리   
    -> 불필요한 토큰이니 ttl 타임이 지나면 redis에서는 알아서 값이 삭제 됨    

    - 생각 해볼 만한 것들  
    -> 빠른 응답이 필요한 조회 성 로직  
    -> kafka와의 연계  
  
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
- 프론트 기획/개발

### 물류센터 관련
예를 들어 쿠x같이 물류센터가 여러 군데 나뉘어 있을 수 있기 때문에  
특정 지역에서 찜 해놨던 상품을   
다른 지역에서 주문을 하려 앱을 열면 해당 상품을 주문할 수 없던 이유..  
해당 지역의 물류센터에는 해당 상품이 없거나 혹은 상품의 재고가 없음  
이런 케이스도 있을 수 있으니 테이블에 상품에 배송 가능한 물류 창고 로케이션 필드도 추가하고  
사용자의 현재 접속 위치도 받아서 select 해야 겠다.   
