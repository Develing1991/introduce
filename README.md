
## Micro-service 구성도
![](https://velog.velcdn.com/images/develing1991/post/53bd6ec2-2868-406b-8332-3b1530f98f7f/image.png)

### Micro service - docker-compose 구성 목록
네트워크 : benefits-network (172.18.0.1 ~ )
- zipkin
- zookeeper
- kafka
- prometheus
- grafana
- rabbitmq
- mysql
- naming-server
- config-server
- gateway-service
- user-service
- product-service
- review-service
- seller-service
- supervisor-service

<br>

## 스웨거 문서 목록 (API)
user-service: https://benefits.completed0728.site/user-service/swgger-ui/index.html
order-service: https://benefits.completed0728.site/order-service/swgger-ui/index.html
product-service: https://benefits.completed0728.site/product-service/swgger-ui/index.html
review-service: https://benefits.completed0728.site/review-service/swgger-ui/index.html
seller-service: https://benefits.completed0728.site/seller-service/swgger-ui/index.html
supervisor-service: https://benefits.completed0728.site/supervisor-service/swgger-ui/index.html

<br>

## CI/CD Pipeline 구성도
![](https://velog.velcdn.com/images/develing1991/post/c708b244-8431-4910-bc66-c7ec2edcee51/image.png)

### CI/CD - docker-compose 구성 목록
네트워크 : cicd-network (172.19.0.1 ~ )
- jenkins-server
- ansible-server
- deploy-server

<br>

## Docker Hub
- 모든 이미지들은 Dockerfile로 빌드 하였고 전부 수작업으로 작성 했습니다.
- 도커 허브에 업로드 된 이미지 목록
`CI/CD` 관련: jenkins-server, ansible-server, deploy-server
`Micro-service` 관련: 모든 Micro-service 이미지
<span style="color:red;">config-server 제외 (Github 인증 토큰이 있으므로)</span>

- 도커 허브 리포지토리 주소: https://hub.docker.com/repositories/completed0728

<br>

## 실제 가용중인 서비스 구성도
- AWS EC2 비용 관련 문제로 현재 로컬 PC에서 서비스중 입니다.
- AWS Router53 비용만 청구하고 있습니다.

![](https://velog.velcdn.com/images/develing1991/post/f3249ecb-dc13-4027-be59-3579414e1fba/image.png)

<br>

## 스웨거를 위한 CORS 허용 (스웨거 도메인)
- gateway-service(CORS): 스웨거 도메인이 gateway-service 도메인 자체이므로
해당 도메인의 CORS 작업 없음
- 각 마이크로 서비스들의 CORS 
user-service, order-service, 
product-service, review-service, 
seller-service, supervisor-service (CORS): 
`https://benefits.completed0728.site` 도메인, `http://localhost:3001` 도메인 CORS 허용

![](https://velog.velcdn.com/images/develing1991/post/f04c9f8b-3e9b-4a49-ac79-58ff94befd53/image.png)

<br>

## 프론트를 위한 COSR 허용 (로컬 도메인)
- gateway-service(CORS): http://localhost:3001
게이트웨이는 `http://localhost:3001` 도메인 CORS 허용
- user-service, order-service, 
product-service, review-service, 
seller-service, supervisor-service (CORS):
-> 각 마이크로 서비스들 `http://localhost:3001` 도메인 CORS 허용
- 로컬 프론트엔드 3001번으로 실행 후 API요청
- 요청 API 주소 예시(GET):  https://benefits.completed0728.site/user-service/users/1
![](https://velog.velcdn.com/images/develing1991/post/3c54bff4-eb2c-46d1-8cec-cddf071436f1/image.png)

<br>

## ERD 다이어그램
![](https://velog.velcdn.com/images/develing1991/post/0601d52a-c4ab-4ed1-b89f-f745036d67b8/image.png)

## ERD 관계 맵핑
![](https://velog.velcdn.com/images/develing1991/post/df90c347-6e66-4002-8f5a-a3628a1cee86/image.png)
