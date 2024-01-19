
# benefits (MSA 프로젝트)

<h2 id="list">목차</h2>

- <a href="#intro">마이크로 서비스 구성도</a>
- <a href="#service">서비스 핵심 기능</a>
- <a href="#auth">API 인증(Authentication) 및 인가(Authorization)</a>
- <a href="#spec">공통 응답 스펙</a>
- <a href="#cicd">CI/CD 파이프라인 구성도</a>
- <a href="#docker-hub">Docker Hub</a>
- <a href="#real">현재 가용 중인 서비스 구성도</a>
- <a href="#swagger">현재 서비스 중인 스웨거 API 문서 목록</a>
- <a href="#swagger-cors">스웨거를 위한 CORS 허용 (스웨거 도메인)</a>
- <a href="#front-cors">프론트엔드(SPA)를 위한 COSR 허용 (로컬 도메인)</a>
- <a href="#erd">ERD 다이어그램</a>
- <a href="#erd-relation">ERD 관계 맵핑</a>

<br>

<h2 id="intro">마이크로 서비스 구성도</h2>

![](https://velog.velcdn.com/images/develing1991/post/53bd6ec2-2868-406b-8332-3b1530f98f7f/image.png)

### Micro service - docker-compose 구성 목록 (이미지 파일)
도커 네트워크 : benefits-network ( Subnet: 172.18.0.0/16, Gateway: 172.18.0.1 )  
- zipkin:latest
- wurstmeister/zookeeper:latest
- wurstmeister/kafka:latest
- prometheus:latest
- grafana:latest
- rabbitmq:latest
- mysql:8.0.33
- completed0728/naming-server:1.0
- completed0728/config-server:1.0
- completed0728/gateway-service:1.0
- completed0728/user-service:1.0
- completed0728/product-service:1.0
- completed0728/review-service:1.0
- completed0728/seller-service:1.0
- completed0728/supervisor-service:1.0

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

<br>


<h2 id="service">서비스 핵심 기능</h2>

### user-service 예시
- 모든 서비스는 `gateway-service`의 `IP`를 확인하고 해당 `IP`에서 들어오는 요청만 허용합니다.
  
![](https://velog.velcdn.com/images/develing1991/post/3aa65189-ec24-42be-a638-3721203103f1/image.png)

<br>

### order-service, product-service
- `order-service`와 `product-service`는 대량의 트래픽 동시성 처리를 위해  
카프카 큐잉 메시지 브로커를 사용했습니다.  

- `order-service`에서 주문을 받고 orders 테이블에 주문의 상태를 `ORDER`로 등록합니다.  
카프카 토픽으로 `order`란 이름의 토픽과 주문에 대한 데이터를 생산합니다.  

- `product-service`에서는 `order`란 이름의 토픽을 소비합니다.  
`order`토픽의 주문 데이터를 가져와서 products 테이블에 해당 상품의 수량을 업데이트 합니다.  
업데이트 전에 수량을 먼저 조회 하고 수량이 부족하면 `orderReject`라는 이름의 토픽에 해당 주문의 반려시킬 고유 값 데이터를 포함하여 생산합니다.  
수량 감소 업데이트도 진행하지 않습니다.  

- `order-service`에서 `orderReject`라는 이름의 토픽을 소비합니다.  
`orderReject`토픽의 주문에 대한 고유 값 데이터로 주문을 조회 하고 해당 주문의 상태를 `REJECT`상태로 업데이트 합니다.

![](https://velog.velcdn.com/images/develing1991/post/b2ab3c9d-8f82-4f1c-80cf-7a589c21f91b/image.png)


<br>

### config-server

- `keytool`을 사용해서 RSA 알고리즘의 키 쌍을 생성하고 각 각의 서명 값을 추출하여 공개키와 비밀키로 사용할 각 각의 키에 인증서를 넣어 사용할 키 쌍을 구성 했습니다.   
- 그리고 나서 `config-server`에서 불러오는 설정 평문 데이터 값은 `비밀키의 공개키 인증서`로 암호화 하였습니다.  
- `config-server`에서 갖고 있는 RSA 알고리즘의 `공개키`를 통해 복호화 합니다.
- `config-server`에서 가져오는 설정 값들이 변경 되면 `Spring Clous Bus`와 `Rabbitmq`를 조합한 `/actuator/refreshbus`API를 POST 요청하여 갱신 합니다.

![](https://velog.velcdn.com/images/develing1991/post/65232943-b199-4201-82c4-c34b089e6d2d/image.png)

<br>

### review-service

- 특정 주문에 대한 리뷰 작성은 `Feign Client` 통신을 이용하여 orders 테이블에 review의 id를 업데이트 하는 방식으로 구성 했습니다.

![](https://velog.velcdn.com/images/develing1991/post/c338836a-120c-41b3-8783-851b54039105/image.png)

<br>

### 모니터링 분산 추적 zipkin

- 마이크로 서비스의 요청 또는 통신 간의 장애 발생 지점 등을 모니터링 하기 위해 zipkin을 사용했습니다.

![](https://velog.velcdn.com/images/develing1991/post/114f2778-99ad-4db7-92c7-ed7909793abf/image.png)

<br>

### 모니터링 지표 수집 Micrometer, Prometheus, Actuator
- `Micrometer`, `Prometheus`, `Actuator`로 gateway-service로의 요청 횟수, 특정 API 요청 횟수와 같은 지표 데이터들을 수집합니다.

![](https://velog.velcdn.com/images/develing1991/post/afcbc49b-2fde-48de-a6a0-bc2c311ef262/image.png)

<br>

### 모니터링 시각화 Grafana 
- Micrometer, Prometheus, Actuator로 수집한 지표 데이터를 바탕으로  `Grafana`로 시각화 했습니다.

![](https://velog.velcdn.com/images/develing1991/post/66516e69-42ab-4e4b-b882-67730b68bb8d/image.png)

<br><br>



<h2 id="auth">API 인증(Authentication) 및 인가(Authorization)</h2>

### 인증 방식 JWT - ( user-service, seller-service, supervisor-service - login )
- JWT을 사용해 인증과 인가하는 방식을 사용했습니다.
- `user-service`, `seller-service`, `supervisor-service`는 토큰 발행 시에  
페이로드의 권한 역할을 지정하는 `role`이라는 이름의 키 값에 각 각 `USER`, `SELLER`, `SUPERVISOR`로 발행 합니다.

![](https://velog.velcdn.com/images/develing1991/post/b30f4ae2-711b-4231-86e2-5fceb82b196b/image.png)

<br>

### 인가 방식 1 - ( gateway-service - filter )

- 로그인을 제외한 모든 요청과 허용은 `gateway-service`에서만 이루어지므로  
  `gateway-service`에서 토큰이 유효한가를 검증함과 동시에 요청 된 `API`에 해당 토큰이 적절한 권한을 가지고 있는지도 페이로드를 통해 확인합니다.
  
![](https://velog.velcdn.com/images/develing1991/post/aff84d24-6ca5-4810-9b9a-903f5c09fc66/image.png)


<br>

### 인가 방식 2 - ( 각 마이크로 서비스 ) 예시 - user-service
- 각 각의 마이크로 서비스에서는 

  만약 유효한 토큰임과 적절한 권한을 가진 토큰이라는 것이 `gateway-service`에서 검증 되었다 하더라도  
  자신의 정보를 조회, 수정, 삭제하는 종류의 API같은 경우에는 오직 자신만 가능해야 하므로
  
  각 서비스에 등록된 `AOP` 페이로드를 검증 하는 로직을 통해 자신 이외의 요청은 차단됩니다. (`@UserSelfRole` 어노테이션 하단 이미지 예시)
  
  모든 API가 해당 검증 로직을 거쳐야 하는 것이 아니므로 `AOP`기능을 사용했습니다.  

![](https://velog.velcdn.com/images/develing1991/post/d62e4713-633c-474a-bfe6-527297e3d834/image.png)


<br><br>


<h2 id="spec">공통 응답 스펙</h2>

- 공통 응답 스펙 입니다.
```javascript
{
    "result": {
        "result_code": 200,		// 응답 코드 (외부코드, 내부코드로 관리)
        "result_message": "성공", 	// 기본 성공 메시지
        "result_description": "성공" 	// 직접 커스텀할 설명
    },
    "data": null, 			// 데이터
    "pagination": null 			// 페이지네이션
}
```

![](https://velog.velcdn.com/images/develing1991/post/ef16f964-7c0a-42f8-b4ee-c1c785df78c6/image.png)


<br><br>


<h2 id="cicd">CI/CD 파이프라인 구성도</h2>

- 배포 자동화를 위해 파이프라인을 구성 했습니다.

![](https://velog.velcdn.com/images/develing1991/post/c708b244-8431-4910-bc66-c7ec2edcee51/image.png)

<br>

### CI/CD - docker-compose 구성 목록
도커 네트워크 : cicd-network ( Subnet: 172.19.0.0/16, Gateway: 172.19.0.1 )  
- jenkins-server: completed0728/jenkins-server:1.0
- ansibler-server: completed0728/ansible-server:1.0
- delivery-docker-server: completed0728/docker-server:1.0
- deploy-docker-server: completed0728/docker-server:1.0

<br>

### jenkins-server
- jenkis-serve에 등록한 마이크로 서비스(job) 목록입니다.

![](https://velog.velcdn.com/images/develing1991/post/65e6bf56-0e50-4772-bd94-37fbc21e8200/image.png)

<br>

### 파이프라인 동작 흐름 (예시 user-service)
1. jenkins-server에서 `user-service`(job)을 직접 빌드 버튼 클릭 하거나    
또는  깃 허브 `user-service`리포지토리의 `deploy` 브랜치에 코드 커밋, 푸시(Poll SCM)가 발생 하면    
	-> 깃 허브의 `deploy` 이름의 브랜치에서 코드를 읽어오고 빌드를 진행 합니다.  
  -> 빌드 된 결과물(jar, source code)을 `delivery-docker-server`로 전달 합니다.  
    
2. `jenkins-server`에서 `ansible-server`로 배포를 진행할 `user-service`의 playbook.yml을 실행합니다.
3. `user-service/delivery-playbook.yml` 실행합니다. (image build, push, prune)
4. `user-service/deploy-playbook.yml` 실행합니다. (image pull, container run)

![](https://velog.velcdn.com/images/develing1991/post/966b2f09-c1e5-4770-b467-da929c91be17/image.png)

<br><br>


<h2 id="docker-hub">Docker Hub</h2>

- 도커 허브에 업로드 된 이미지 목록  
`CI/CD` 관련: jenkins-server, ansible-server, deploy-server  
`마이크로 서비스` 관련: <span style="color:red;">config-server를 제외한</span> 모든 마이크로 서비스 이미지  
 -> config-server에는 구성 정보 파일을 읽어 들이는 공개키와 깃 허브 토큰이 포함되어 있어서 제외 하였습니다.

- 도커 허브 리포지토리 주소: https://hub.docker.com/repositories/completed0728  
`CI/CD` 관련 이미지들은 현재 사용 중인 컨테이너들을 다시 이미지화 하지 않았습니다.  
각 각의 서버 구성에 맞게 패키지가 설치되는 초기 상태 이미지 입니다.  
 -> jenkins-server에도 해당 깃 허브에 접근할 수 있는 권한의 토큰이 포함되고 docker-server에는 도커 허브의 이미지를 push, pull권한의 토큰이 포함되기 때문입니다.

### Dockerfile 예시 (docker-server)
```yml
FROM ubuntu

ENV TZ=Asia/Seoul

ENV container docker

RUN apt-get update && apt-get upgrade -y
RUN apt install -y init systemd
# RUN apt install -y build-essential  
RUN apt install -y vim curl
RUN apt install -y sudo wget 
RUN apt install -y net-tools iputils-ping

# SSH
RUN apt install -y openssh-server
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN echo 'root:benefits' | chpasswd

# Docker: https://docs.docker.com/engine/install/ubuntu/
# Add Docker's official GPG key:
RUN sudo apt-get install -y ca-certificates curl gnupg
RUN sudo install -m 0755 -d /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
RUN sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
RUN echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
RUN apt-get update
RUN sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
# Docker

# apt-get cache clean
RUN apt-get clean autoclean
RUN apt-get autoremove -y
RUN rm -rf /var/lib/{apt,dpkg,cache,log}

# home 으로 초기화
WORKDIR /root

EXPOSE 22

VOLUME ["/sys/fs/cgroup"]

#ENTRYPOINT ["/usr/sbin/init" "systemctl" "start" "sshd"]
CMD ["/usr/sbin/init" "systemctl" "start" "sshd"]
```
<br><br>

<h2 id="real">현재 가용 중인 서비스 구성도</h2>

- AWS EC2 비용 관련 문제로 현재 로컬 PC에서 서비스중 입니다.
- AWS Router53 비용만을 청구하고 있습니다.

![](https://velog.velcdn.com/images/develing1991/post/f3249ecb-dc13-4027-be59-3579414e1fba/image.png)

<br><br>

<h2 id="swagger">현재 서비스 중인 스웨거 API 문서 목록</h2>

user-service: https://benefits.completed0728.site/user-service/swgger-ui/index.html

order-service: https://benefits.completed0728.site/order-service/swgger-ui/index.html

product-service: https://benefits.completed0728.site/product-service/swgger-ui/index.html

review-service: https://benefits.completed0728.site/review-service/swgger-ui/index.html

seller-service: https://benefits.completed0728.site/seller-service/swgger-ui/index.html

supervisor-service: https://benefits.completed0728.site/supervisor-service/swgger-ui/index.html

<br><br>

<h2 id="swagger-cors">스웨거를 위한 CORS 허용 (스웨거 도메인)</h2>

- gateway-service(CORS): 스웨거 도메인이 gateway-service 도메인 자체이므로  
  해당 도메인의 CORS 작업은 없습니다.  
- 각 마이크로 서비스들의 CORS  
user-service, order-service,  
product-service, review-service,  
seller-service, supervisor-service 들의 CORS 설정은    
`https://benefits.completed0728.site` 도메인, `http://localhost:3001` 도메인을 허용 했습니다.

![](https://velog.velcdn.com/images/develing1991/post/f04c9f8b-3e9b-4a49-ac79-58ff94befd53/image.png)

<br><br>


<h2 id="front-cors">프론트엔드(SPA)를 위한 COSR 허용 (로컬 도메인)</h2>

- gateway-service(CORS): http://localhost:3001  
게이트웨이는 `http://localhost:3001` 도메인 CORS 허용 했습니다.
- user-service, order-service,  
product-service, review-service,  
seller-service, supervisor-service 들의 CORS 설정은   
`http://localhost:3001` 도메인을 CORS 허용 했습니다.  
- localhost:3001번으로 실행 후 API요청
- 요청 API 주소 예시(GET):  https://benefits.completed0728.site/user-service/open-api/users/1

#### 요약: 프론트엔드(SPA) localhost에서 테스트 하시려면 3001번 포트로 실행 하시고 API를 요청 하시면 됩니다.

![](https://velog.velcdn.com/images/develing1991/post/3c54bff4-eb2c-46d1-8cec-cddf071436f1/image.png)

<br><br>

<h2 id="erd">ERD 다이어그램</h2>

- 데이터베이스를 각 서비스의 기능과 가장 관련된 기준으로 나누었습니다.

![](https://velog.velcdn.com/images/develing1991/post/0601d52a-c4ab-4ed1-b89f-f745036d67b8/image.png)


<h2 id="erd-relation">ERD 관계 맵핑</h2>

- 관계의 강결합 적인 문제를 고려하여 데이터베이스에는 실제로는 맵핑을 하지 않았으며 논리적인 맵핑 관계 입니다.

![](https://velog.velcdn.com/images/develing1991/post/df90c347-6e66-4002-8f5a-a3628a1cee86/image.png)

- ORM의 Entity 관계에서만 맵핑을 진행 하였습니다.

![](https://velog.velcdn.com/images/develing1991/post/58db0275-e8b3-4566-824c-ec504ba64b7d/image.png)

