
# benefits (MSA 프로젝트)


- <a href="#_1">Micro-service 구성도</a>
  - <a href="#1_1-">Micro service - docker-compose 구성 목록 (이미지 파일)</a>
  - <a href="#1_2">서비스 설명</a>
- <a href="#2">API 권한 및 보안 관련 설명</a>
- <a href="#3">공통 응답 Spec 구조</a>
- <a href="#4">CI/CD Pipeline 구성도</a>
  - <a href="#4_1">CI/CD - docker-compose 구성 목록</a>
- <a href="#5">Docker Hub</a>
  - <a href="#5_1">Dockerfile 예시</a>
- <a href="#6">현재 가용중인 서비스 구성도</a>
- <a href="#7">현재 서비스 중인 스웨거 API 문서 목록</a>
- <a href="#8">스웨거를 위한 CORS 허용 (스웨거 도메인)</a>
- <a href="#9">프론트엔드(SPA)를 위한 COSR 허용 (localhost 도메인)</a>
- <a href="#10">ERD 다이어그램</a>
- <a href="#11">ERD 관계 맵핑</a>

<br>

## Micro-service 구성도
  
![](https://velog.velcdn.com/images/develing1991/post/53bd6ec2-2868-406b-8332-3b1530f98f7f/image.png)

### Micro service - docker-compose 구성 목록 (이미지 파일)
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

### 서비스 설명
- 모든 서비스는 gateway-service의 IP를 확인하고 해당 IP에서 들어오는 요청만 허용합니다.
  
![](https://velog.velcdn.com/images/develing1991/post/3aa65189-ec24-42be-a638-3721203103f1/image.png)

<br>

- order-service와 product-service는 대용량 트래픽 관련하여  
상품 수량 감소 동시성 처리를 위해 카프카 큐잉 메시지 브로커를 사용했습니다.  
주문을 받고 ORDER 상태로 등록하며 수량이 부족하면 REJECT상태로 업데이트 됩니다.

![](https://velog.velcdn.com/images/develing1991/post/b98bcea0-ed2d-43f1-8c55-eaaea6ca3cb7/image.png)

<br>

- config-server에서 불러오는 공통 설정 데이터는 암호화 하였습니다. (`{cipher}`)  
config-server에서 갖고 있는 RSA 알고리즘의 공개키를 통해 복호화 됩니다.

![](https://velog.velcdn.com/images/develing1991/post/65232943-b199-4201-82c4-c34b089e6d2d/image.png)

<br><br>

## API 권한 및 보안 관련 설명
- JWT 인증 방식을 사용했습니다.
- 토큰의 권한은 발급 시 페이로드의 ROLE키 값에 `USER`, `SELLER`, `SUPERVISOR`로 발행 합니다.

  모든 요청과 허용은 `gateway-service`에서만 이루어지고
  
  `gateway-service`에서 토큰 자체를 검증함과 동시에 해당 토큰이 요청 된 API에 적합한 권한을 가지고 있는지도 페이로드를 통해 확인합니다.
  
![](https://velog.velcdn.com/images/develing1991/post/aff84d24-6ca5-4810-9b9a-903f5c09fc66/image.png)

<br>
  
- 각각의 마이스로 서비스에서는 (USER-SERVICE를 예시)  
  
  만약 토큰이 검증이 되었다 하더라도 자신의 정보를 조회, 수정, 삭제하는 API같은 경우에는 오직 자신만 가능해야 하므로
  
  각 서비스에 등록된 `AOP`기능을 통해 자신 이외의 요청은 차단됩니다.
  
![](https://velog.velcdn.com/images/develing1991/post/d62e4713-633c-474a-bfe6-527297e3d834/image.png)



<br><br>

## 공통 응답 Spec 구조
```javascript
{
    "result": {
        "result_code": 200,
        "result_message": "성공",
        "result_description": "성공"
    },
    "data": null,
    "pagination": null
}
```

<br><br>

## CI/CD Pipeline 구성도
![](https://velog.velcdn.com/images/develing1991/post/c708b244-8431-4910-bc66-c7ec2edcee51/image.png)

<br>

### CI/CD - docker-compose 구성 목록
네트워크 : cicd-network (172.19.0.1 ~ )
- completed0728/jenkins-server:1.0
- completed0728/ansible-server:1.0
- completed0728/deploy-server:1.0

<br>

### jenkins-server
![](https://velog.velcdn.com/images/develing1991/post/65e6bf56-0e50-4772-bd94-37fbc21e8200/image.png)

<br>

### 파이프라인 동작 흐름
- 각 ssh 서버 접근은  
접근하고자 하는 서버에는 RSA public key를 배포하고  
접속을 시도 하는 서버는 자신의 RSA private key로 인증하여 접속 합니다.  
1. jenkins-server에서 빌드 버튼 클릭, 또는 Github의 deploy 브랜치의 java코드 커밋, 푸시(Poll SCM)가 발생하면  
	-> Github의 deploy 이름의 브랜치에서 코드를 가져와서 빌드합니다.  
  -> 빌드 된 결과물(jar, source code)을 delivery-docker-server로 전달합니다.  
    
2. jenkins-server에서 ansible-server로 배포를 진행할 마이크로 서비스의 playbook.yml을 실행합니다.
3. delivery-playbook.yml 실행합니다. (image build, push, prune)
4. deploy-playbook.yml 실행합니다. (image pull, container run)

![](https://velog.velcdn.com/images/develing1991/post/966b2f09-c1e5-4770-b467-da929c91be17/image.png)

<br><br>

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
<br><br>

## 현재 가용중인 서비스 구성도
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

## 스웨거를 위한 CORS 허용 (스웨거 도메인)
- gateway-service(CORS): 스웨거 도메인이 gateway-service 도메인 자체이므로  
  해당 도메인의 CORS 작업은 없습니다.  
- 각 마이크로 서비스들의 CORS  
user-service, order-service,  
product-service, review-service,  
seller-service, supervisor-service (CORS):  
`https://benefits.completed0728.site` 도메인, `http://localhost:3001` 도메인 CORS 허용

![](https://velog.velcdn.com/images/develing1991/post/f04c9f8b-3e9b-4a49-ac79-58ff94befd53/image.png)

<br><br>

## 프론트엔드(SPA)를 위한 COSR 허용 (localhost 도메인)
- gateway-service(CORS): http://localhost:3001  
게이트웨이는 `http://localhost:3001` 도메인 CORS 허용
- user-service, order-service,  
product-service, review-service,  
seller-service, supervisor-service (CORS):  
-> 각 마이크로 서비스들 `http://localhost:3001` 도메인 CORS 허용  
- localhost:3001번으로 실행 후 API요청
- 요청 API 주소 예시(GET):  https://benefits.completed0728.site/user-service/open-api/users/1

### 요약: 프론트엔드(SPA) localhost에서 테스트 하시려면 3001번 포트로 실행 하시고 api 요청 하시면 됩니다.

![](https://velog.velcdn.com/images/develing1991/post/3c54bff4-eb2c-46d1-8cec-cddf071436f1/image.png)

<br><br>

## ERD 다이어그램
![](https://velog.velcdn.com/images/develing1991/post/0601d52a-c4ab-4ed1-b89f-f745036d67b8/image.png)

## ERD 관계 맵핑
![](https://velog.velcdn.com/images/develing1991/post/df90c347-6e66-4002-8f5a-a3628a1cee86/image.png)

