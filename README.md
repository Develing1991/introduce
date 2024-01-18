
## Micro-service 구성도
- 모든 서비스는 gateway-service의 IP를 확인하고 해당 IP에서 들어오는 요청만 허용합니다.
- order-service와 product-service는 대용량 트래픽 관련하여 상품 수량 감소 동시성 처리를 위해 카프카 큐잉 메시지 브로커를 사용했습니다.
  
![](https://velog.velcdn.com/images/develing1991/post/53bd6ec2-2868-406b-8332-3b1530f98f7f/image.png)

### Micro service - docker-compose 구성 목록 (이미지)
네트워크 : benefits-network (172.18.0.1 ~ )
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

<br>

## 보안 관련 설명
- JWT 인증 방식을 사용했습니다.
- 토큰의 권한은 발급 시 페이로드 값에 `USER`, `SELLER`, `SUPERVISOR`로 발행합니다.

  모든 요청과 허용은 `gateway-service`에서만 이루어지고
  
  `gateway-service`에서 토큰 자체를 검증함과 동시에 해당 토큰이 요청 된 API에 적합한 권한을 가지고 있는지도 페이로드를 통해 확인합니다.
  
- 각각의 마이스로 서비스에서는 (USER-SERVICE를 예시)
  
  만약 토큰이 검증이 되었다 하더라도 자신의 정보를 조회, 수정, 삭제하는 API같은 경우에는 오직 자신만 가능해야 하므로
  
  각 서비스에 등록된 `AOP`기능을 통해 자신 이외의 요청은 차단됩니다.

- 조금 더 디테일한 설명은 각 서비스의 Swagger 문서에 기재 하였습니다.

<br>

## CI/CD Pipeline 구성도
![](https://velog.velcdn.com/images/develing1991/post/c708b244-8431-4910-bc66-c7ec2edcee51/image.png)

### CI/CD - docker-compose 구성 목록
네트워크 : cicd-network (172.19.0.1 ~ )
- completed0728/jenkins-server:1.0
- completed0728/ansible-server:1.0
- completed0728/deploy-server:1.0

<br>

## Docker Hub
- 모든 이미지들은 Dockerfile로 빌드 하였고 전부 수작업으로 작성 했습니다.
- 도커 허브에 업로드 된 이미지 목록  
`CI/CD` 관련: jenkins-server, ansible-server, deploy-server  
`Micro-service` 관련: 모든 Micro-service 이미지  
<span style="color:red;">config-server 제외 (Github 인증 토큰이 있으므로)</span>

- 도커 허브 리포지토리 주소: https://hub.docker.com/repositories/completed0728
### Dockerfile 예시
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
# RUN ssh-keygen -f /etc/ssh/ssh_host_rsa_key -N '' -t rsa
#RUN apt-get install -y openssh-clients

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
<br>

## 실제 서비스 중인 스웨거 문서 목록 (API)
user-service: https://benefits.completed0728.site/user-service/swgger-ui/index.html

order-service: https://benefits.completed0728.site/order-service/swgger-ui/index.html

product-service: https://benefits.completed0728.site/product-service/swgger-ui/index.html

review-service: https://benefits.completed0728.site/review-service/swgger-ui/index.html

seller-service: https://benefits.completed0728.site/seller-service/swgger-ui/index.html

supervisor-service: https://benefits.completed0728.site/supervisor-service/swgger-ui/index.html

<br>

## 실제 가용중인 서비스 구성도
- AWS EC2 비용 관련 문제로 현재 로컬 PC에서 서비스중 입니다.
- AWS Router53 비용만을 청구하고 있습니다.

![](https://velog.velcdn.com/images/develing1991/post/f3249ecb-dc13-4027-be59-3579414e1fba/image.png)

<br>

## 스웨거를 위한 CORS 허용 (스웨거 도메인)
- gateway-service(CORS): 스웨거 도메인이 gateway-service 도메인 자체이므로  
  해당 도메인의 CORS 작업은 없습니다.  
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
