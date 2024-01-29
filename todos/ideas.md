todo list

- 기능 추가에 따른 유즈케이스, 요구사항정의서 업데이트
- redis (auth, cache, transaction, single thread) - slave, master sentinel  
    -> 로그아웃 기능 user-service, seller-service, supervisor-service 로그아웃 시 redisTemplate을 통해 redis-server의 redis에
          Set 토큰 값 key, value, ttl은 decode token AccessExpiredAt, ttlRefreshExpiredAt - 각각 )  
    -> 검증기능 gateway + redisTemplate (토큰 검증 앞 단에 get redis + token ) 확인 로직을 추가
    -> 로그아웃을 했다면 redis에 해당 키 토큰이 존재하니 invalid한 token으로 응답 처리
    
  ttl: accessExpireAt,
    -> product default sorting low price   
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
