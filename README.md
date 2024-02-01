# benefits (MSA 프로젝트)

<h2 id="list">목차</h2>

- <a href="#intro">🗿 소개</a>

- <a href="#micro-structure">📔 마이크로 서비스 구성도</a>
- <a href="#service">🧑‍💻 서비스 핵심 기능</a>
- <a href="#test">🥕 kafka 동시 주문 테스트</a>
- <a href="#auth">🔐 API 인증(Authentication) 및 인가(Authorization)</a>
- <a href="#spec">📝 공통 응답 스펙</a>
- <a href="#cicd">🔅 CI/CD 파이프라인 구성도</a>
- <a href="#docker-hub">💿 Docker Hub</a>
- <a href="#real">📗 서비스의 전체 구성도</a>
- <a href="#swagger">✅ 현재 호스팅 중인 스웨거 API 문서 목록</a>
- <a href="#swagger-cors">🟡 스웨거를 위한 CORS 허용 (스웨거 도메인)</a>
- <a href="#front-cors">🟡 프론트엔드(SPA)를 위한 COSR 허용 (로컬 도메인)</a>
- <a href="#erd-relation">📚 ERD 다이어그램 (논리적 연관 관계 맵핑)</a>
- <a href="#dependencies">🛠️서비스 개발에 사용 된 핵심 의존성 목록</a>
- <a href="#after">📌 이 후 업데이트 사항</a>


<br><br><br><br>

<h2 id="intro">🗿 소개</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

### 프로젝트 소개

**이 프로젝트 비즈니스 모델은 B2C입니다.**  

**Java언어 기반의 SpringBoot Application을 사용 했습니다.**  

**MicroService Architecture(MSA)로 구성한 프로젝트 입니다.**  

<br>

### 서비스 목록  
`user-service`  

`order-service`  

`product-service`  

`review-service`  

`seller-service`  

`supervisor-service`  

<br>

### 사용자 모델

서비스를 이용할 수 있는 **사용자의 모델과 권한**은  

**관리자(Supervisor), 판매자(Seller), 사용자(User)** 로 구성 됩니다.

<br>

**관리자(Supervisor)**  
> 카테고리, 브랜드를 등록할 수 있습니다.  
상품을 등록할 수 있는 최소 단위인 아이템(인벤토리)을 생성할 수 있습니다.  
API의 인가 관련 테스트의 번거로움을 덜고자 현재 관리자는 모든 API기능 동작 권한을 갖고 있습니다.

<br>

**판매자(Seller)**  
> 판매자의 계정은 관리자만이 등록할 수 있습니다.    
판매자는 로그인을 할 수 있습니다.  
판매자는 관리자가 등록한 아이템을 기반으로 아이템을 조회하여 상품을 등록할 수 있습니다.  
판매자가 처음 등록한 상품의 상태는 REGISTERED(등록)입니다.  
판매자가 상품을 판매 하고자 할 때 SELL(판매중) 또는 SOLD_OUT(품절), UNREGISTERED(해지)로 변경할 수 있습니다.    
판매자는 자신이 등록한 상품의 주문 요청 건을 조회할 수 있습니다.  
판매자는 주문 요청 된 상품 건의 상태가 ORDER(주문) 건에 한정 하여 ACCEPT(수락), REJECT(거절/반려)로 변경 할 수 있습니다.  
판매자는 사용자가 주문한 상품 CANCELING(주문 취소 요청) 건에 한하여 CANCEL(취소 - 확정) 상태로 변경할 수 있습니다.  
상품의 주문 상태가 CANCEL(취소 - 확정)되면 사용자의 주문 요청으로 감소한 해당 상품의 재고 주문한 수량 만큼 다시 증가 합니다.

<br>

**사용자(User)**  
> 사용자는 회원가입과 로그인을 할 수 있습니다.  
회원가입과 동시에 사용자 프로필은 기본적으로 생성 됩니다.  
사용자는 프로필을 수정할 수 있습니다.  
사용자는 주소를 등록할 수 있습니다. (주소는 최대 3개 제한)  
사용자는 상품을 조회 하고 주문을 할 수 있습니다.   
사용자는 상품 상태가 SELL(판매중), SOLD_OUT(품절)인 상품을 조회할 수 있습니다.   
사용자는 상품 상태가 SELL(판매중) 상품만 주문할 수 있습니다.  
사용자는 배송지 테이블의 배송의 상태가 DELIVERY(배송중)이지 않은 개별 상품 건에 대하여 CANCELING(주문 취소 요청)을 할 수 있습니다.   
주문이 완료된 상품 건에 한정 하여 리뷰를 작성할 수 있습니다.  

<br><br>

👇 **권한 관련한 기능들은 각 문서 상단에서 콤보박스로 확인하실 수 있습니다.** 👇

<br>

![](https://velog.velcdn.com/images/develing1991/post/16eb9dab-10b8-44a4-b774-065c94940d8b/image.png)

<br><br><br>

<h3> ⛔ 개인정보가 기입 될 수 있는 기능 제공 제외 </h3>

개인정보 수집 및 보호 관련한 문제가 발생할 수 있을 점을 고려하여   

**현재 사용자, 판매자, 관리자 계정의 등록 및 수정 등의 개인 정보 수집 가능성이 있는 기능의 Writable한 API 기능은 제공하지 않습니다.**  

해당 기능의 제공을 제외한 점 양해의 말씀을 드립니다.🙈    

<br>

**👉 테스트가 가능한 별도의 테스트 계정이 제공 됩니다.**  

<a href="#swagger">✅ 현재 호스팅 중인 스웨거 API 문서 목록</a>

<br>

**유저 테스트 계정**    

```javascript
{
  "email": "testUser1@aaa.com",
  "password": "uP@sswr0d!11"
}
```
```javascript
{
  "email": "testUser2@bbb.com",
  "password": "uP@sswr0d!22"
}
```
```javascript
{
  "email": "testUser3@ccc.com",
  "password": "uP@sswr0d!33"
}
```

![](https://velog.velcdn.com/images/develing1991/post/55a38ac5-726a-4dfd-ba7c-4ff47bb59f32/image.png)

<br>

**판매자 테스트 계정**  

```javascript
{
  "email": "testSeller1@aaa.com",
  "password": "slP@sswr0d!11"
}
```
```javascript
{
  "email": "testSeller2@bbb.com",
  "password": "slP@sswr0d!22"
}
```
![](https://velog.velcdn.com/images/develing1991/post/f3928207-8a30-4475-a6a6-1c76ce41577b/image.png)

<br>

**관리자 테스트 계정**    

```javascript
{
  "email": "testSuper1@aaa.com",
  "password": "spP@sswr0d!11"
}
```
![](https://velog.velcdn.com/images/develing1991/post/4c7b1502-adda-431f-bea0-422371036922/image.png)


<br><br><br>

<h2 id="micro-structure">📔 마이크로 서비스 구성도</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

![](https://velog.velcdn.com/images/develing1991/post/53bd6ec2-2868-406b-8332-3b1530f98f7f/image.png)

<br>

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
- completed0728/order-service:1.0
- completed0728/product-service:1.0
- completed0728/review-service:1.0
- completed0728/seller-service:1.0
- completed0728/supervisor-service:1.0

<br><br><br>

<h2 id="service">🧑‍💻 서비스 핵심 기능</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

### API 요청에 대한 허용

- 모든 마이크로 서비스는 `gateway-service`의 `IP`를 확인하고 해당 `IP`에서 들어오는 요청만 허용합니다.

👇 **예시) user-service(SecurityConfig)** 👇

![](https://velog.velcdn.com/images/develing1991/post/f84e179f-f6fc-4436-a677-b31143ade323/image.png)


<br>

### 주문과 상품

-  주문 서비스: `order-service`, 상품 서비스: `product-service`
  
- `order-service`와 `product-service`는 주문 처리 과정에서 트래픽의 동시 요청 처리를 위해
  
  	즉, 상품 재고 수량 처리의 정합성을 위해 카프카 큐 메시지 브로커를 사용했습니다.  

- `order-service`에서 주문을 받아 주문 데이터를 `orders` 테이블에 등록하며 주문의 상태를 `ORDER`로 등록합니다.
  
	카프카 토픽으로 `order`란 이름의 토픽과 주문에 대한 데이터를 생산합니다.  

- `product-service`에서는 `order`란 이름의 토픽을 소비합니다.
  
	`order`토픽의 주문 데이터를 가져와서 `products` 테이블에 해당 상품의 수량을 업데이트 합니다.

	업데이트 전에 수량을 먼저 조회 하고 수량이 부족하면 `orderReject`라는 이름의 토픽에 해당 주문의 반려시킬 고유 값 데이터를 포함하여 생산합니다.

	수량 감소 업데이트도 진행하지 않습니다.  

- `order-service`에서 `orderReject`라는 이름의 토픽을 소비합니다.
  
	`orderReject`토픽의 주문에 대한 고유 값 데이터로 주문을 조회 하고 해당 `orders` 테이블의 주문 데이터의 상태를 `REJECT`상태로 업데이트 합니다.  


<br>

**Orders table**  

![](https://velog.velcdn.com/images/develing1991/post/1831df72-9889-409f-88b6-da431aecb3f4/image.png)

<br>

**Products table**  

![](https://velog.velcdn.com/images/develing1991/post/fc5593d8-801f-43be-8e9d-c61d8858892a/image.png)


<br><br>

### 주문에 대한 상품의 재고 감소 처리 (하나의 주문, 여러 개 상품)

<br>

- **주문**
  
	`4`번 상품 수량 `2`개, `5`번 상품 수량 `1`개 주문

![](https://velog.velcdn.com/images/develing1991/post/6732c67f-9a6b-4b36-9ef3-5bc844ec774d/image.png)

<br><br>

### 주문 취소 처리에 대한 상품의 재고 증가 처리 (개별 상품 취소)

<br>

- **사용자 주문 취소 요청(CANCELING)**

	하나의 주문에서 개별 상품 `4`번 상품만 주문 취소 요청

 	상품 주문 번호 `1`번 취소 요청, 재고 수량 변화 없음

<br>
  
![](https://velog.velcdn.com/images/develing1991/post/fb389b3a-5611-4e5d-8232-00f512aa27ad/image.png)

<br>

- **판매자 주문 취소(CANCEL)**

	상품 주문 번호 `1`번 취소 확정  

	`4`번 상품 재고 수량 `2`개 증가(복구)

<br>

![](https://velog.velcdn.com/images/develing1991/post/223b3b7e-64f0-44d9-803b-ba6bbfac8539/image.png)


<br><br>

### config-server

- `keytool`을 사용해서 RSA 알고리즘의 `공개키`, `비밀키`를 생성 했습니다.
  
- `config-server`에서 불러오는 설정 평문 데이터 값은 `비밀키가 갖고 있는 공개키 인증서`로 암호화 하였습니다.  
- `config-server`에서 갖고 있는 RSA 알고리즘의 `공개키`를 통해 복호화 합니다.  
- `config-server`에서 가져오는 설정 값들이 변경 되면 새로운 구성 정보로 업데이트 하기 위하여
  
  	`Spring Clous Bus`와 `Rabbitmq`를 조합한 `Event Bus` 기능을 사용해서
  
  	`/actuator/refreshbus`API를 POST 요청하여 데이터를 갱신 합니다.  

<br>

![](https://velog.velcdn.com/images/develing1991/post/fc53779d-eb64-465e-b873-f4bddaae19b1/image.png)

<br>

### review-service

- 특정 주문에 대한 리뷰 작성은 `Feign Client` 통신을 이용하여 `orders` 테이블에 `review`의 `id`를 업데이트 하는 방식으로 구성 했습니다.

<br>

![](https://velog.velcdn.com/images/develing1991/post/c338836a-120c-41b3-8783-851b54039105/image.png)

<br>

### 모니터링 분산 추적 zipkin

- 마이크로 서비스의 요청 또는 통신 간의 장애 발생 지점 등을 모니터링 하기 위해 `zipkin`을 사용했습니다.

<br>

![](https://velog.velcdn.com/images/develing1991/post/114f2778-99ad-4db7-92c7-ed7909793abf/image.png)

<br>

### 모니터링 지표 수집 Micrometer, Prometheus, Actuator

- `Micrometer`, `Prometheus`, `Actuator`를 이용하여 `gateway-service`로의 요청 되는 횟수 또는 특정 API 요청 횟수와 같은 지표 데이터들을 수집합니다.

<br>

![](https://velog.velcdn.com/images/develing1991/post/afcbc49b-2fde-48de-a6a0-bc2c311ef262/image.png)

<br>

### 모니터링 시각화 Grafana 

- `Micrometer`, `Prometheus`, `Actuator`를 이용하여 수집한 지표 데이터를 바탕으로 `Grafana`로 시각화 하였습니다.

<br>

![](https://velog.velcdn.com/images/develing1991/post/66516e69-42ab-4e4b-b882-67730b68bb8d/image.png)

<br><br><br>


<h2 id="test">🥕 kafka 동시 주문 테스트</h2> 

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>
- 테스트 결과 👉 https://github.com/benefits-inc/introduce/blob/main/test/KafkaTestMulti200.md


<br><br><br>

<h2 id="auth">🔐 API 인증(Authentication) 및 인가(Authorization)</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

### 인증 방식 JWT - ( user-service, seller-service, supervisor-service - login )
- **Json Web Token(JWT)** 을 사용해 인증과 인가하는 방식을 사용 하였습니다.
- `user-service`, `seller-service`, `supervisor-service`는 토큰 발행 시에  

	페이로드의 권한 역할을 지정하는 `role`이라는 이름의 키 값에 각 각 `USER`, `SELLER`, `SUPERVISOR`로 발행 합니다.

  	토큰 발행에 사용 되는 토큰의 시크릿 키는 모두 다르게 구성 됩니다.

![](https://velog.velcdn.com/images/develing1991/post/b30f4ae2-711b-4231-86e2-5fceb82b196b/image.png)

<br>

### 인가 방식 - ( gateway-service - filter )

-  모든 요청의 허용은 `gateway-service`에서만 이루어 집니다.
    
  	`gateway-service`에서 토큰이 유효한가를 검증함과 동시에  
  
   	요청 된 `API`에 해당 토큰이 적절한 권한을 가지고 있는지도 페이로드를 통해 확인 합니다.  

   	이 때 만약 페이로드의 값을 완벽히 조작 하더라도  
   
   	토큰 발행에 사용 된 시크릿 키가 모두 다르므로 시그니쳐의 부분이 일치하지 않기 때문에 유효하지 않은 토큰으로 간주합니다.  
  
![](https://velog.velcdn.com/images/develing1991/post/aff84d24-6ca5-4810-9b9a-903f5c09fc66/image.png)


<br>

### 인가 방식 - ( 각 마이크로 서비스 ) 예시 - user-service
- 만약 유효한 토큰임과 적절한 권한을 가진 토큰이라는 것이 `gateway-service`에서 검증 되었다 하더라도
    
  	자신의 정보를 조회, 수정, 삭제하는 종류의 API같은 경우에는 오직 자신만 가능해야 하므로 해당 서비스에서는 추가 검증을 진행합니다.
    
  	각 서비스에 등록된 `AOP` 페이로드를 검증 하는 로직을 통해 자신 이외의 요청은 차단 됩니다. (하단 이미지 예시)
  
  	모든 API가 해당 검증 로직을 거쳐야 하는 것이 아니므로 `AOP`기능과 어노테이션을 활용 하였습니다.  

![](https://velog.velcdn.com/images/develing1991/post/d62e4713-633c-474a-bfe6-527297e3d834/image.png)

<br>

👇 **본인 이외의 사용자가 자신의 정보를 조회하려 할 때의 예시입니다. `auth-user/users/{id}`** 👇

<br>

![](https://velog.velcdn.com/images/develing1991/post/218070b9-6c18-4e42-bc00-8aa020d022f1/image.png)

<br> 

**또한 다른 사용자를 제한적인 정보로 조회할 수 있는 별도의 `open-api/users/{id}` API가 제공 됩니다.**

<br><br><br>

<h2 id="spec">📝 공통 응답 스펙</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

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


<br><br><br>


<h2 id="cicd">🔅 CI/CD 파이프라인 구성도</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

- 배포 자동화를 위해 파이프라인을 구성 하였습니다.
- `jenkins-server`: 지속적 통합(CI) 및 지속적 배포(CD) 오픈 소스 도구로 `jenkins`를 사용하였습니다.
  
- `ansible-server`: 오픈 소스 IaC(Infrastructure as Code) 솔루션인 `ansible`을 사용 하였습니다.
  
  	-> `delivery`와 `deploy`에 대한 명령을 실행 하는 `playbook.yml` 파일을 가지고 있습니다.
  
- `docker-server`: CD 개념을 담당하는 `delivery`와 `deploy` 도커 서버 입니다.

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

### 파이프라인 동작 흐름 - 예시) user-service
1. jenkins-server에서 `user-service`(job)을 직접 빌드 버튼 클릭 하거나
   
	또는  깃 허브 `user-service`리포지토리의 `deploy` 브랜치에 코드 커밋, 푸시(Poll SCM)가 발생 하면

	-> 깃 허브의 `deploy` 이름의 브랜치에서 코드를 읽어오고 빌드를 진행 합니다.
   
	  -> 빌드 된 결과물(jar, source code)을 `delivery-docker-server`로 전달 합니다.  
    
3. `jenkins-server`에서 `ansible-server`에게 배포를 진행 할 `user-service`의 `playbook.yml`의 실행 명령을 전달 합니다.
4. `ansible-server`에서 `user-service/delivery-playbook.yml` 실행합니다. (image build, push, prune)
5. `ansible-server`에서 `user-service/deploy-playbook.yml` 실행합니다. (image pull, container run)

![](https://velog.velcdn.com/images/develing1991/post/966b2f09-c1e5-4770-b467-da929c91be17/image.png)

<br><br><br>

<h2 id="docker-hub">💿 Docker Hub</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

- 도커 허브에 업로드 된 이미지 목록
  
	`CI/CD` 관련: `jenkins-server`, `ansible-server`, `deploy-server`  

	`마이크로 서비스` 관련: `config-server`를 제외한 모든 마이크로 서비스 이미지 

 	-> `config-server`에는 구성 정보 파일을 읽어 들이는 공개키와 깃 허브 토큰이 포함되어 있어서 제외 하였습니다.

- 도커 허브 주소: https://hub.docker.com/search?q=completed0728
  
	현재 구동 중인 `CI/CD` 컨테이너들을 다시 이미지화 하지는 않았습니다.  

	즉, 각 각의 서버 구성에 맞게 패키지만 설치되는 초기 상태 이미지 입니다.  

	 -> `jenkins-server`에도 해당 깃 허브에 접근할 수 있는 권한의 토큰이 포함되어 있기 때문입니다.
  
  	 -> `docker-server`에는 이미지를 push, pull권한의 도커 허브의 토큰이 포함되기 때문입니다.

<br>

### delivery-docker-server와 deploy-docker-server에서 사용 된 Dockerfile 예시

- 초기 root 계정: root
- 초기 root 패스워드: benefits

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

<br><br><br>


<h2 id="real">📗 서비스의 전체 구성도</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

- AWS의 EC2 비용 관련 문제로 현재 로컬 PC에서 서비스중 입니다.
  
- AWS의 Router53 비용만을 청구하고 있습니다.

<br>

![](https://velog.velcdn.com/images/develing1991/post/c6b3b07c-10ab-4bf3-ba24-2d72f2c2c92b/image.png)

<br><br><br>


<h2 id="swagger">✅ 현재 호스팅 중인 스웨거 API 문서 목록</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

user-service: https://benefits.completed0728.site/user-service/swagger-ui/index.html

order-service: https://benefits.completed0728.site/order-service/swagger-ui/index.html

product-service: https://benefits.completed0728.site/product-service/swagger-ui/index.html

review-service: https://benefits.completed0728.site/review-service/swagger-ui/index.html

seller-service: https://benefits.completed0728.site/seller-service/swagger-ui/index.html

supervisor-service: https://benefits.completed0728.site/supervisor-service/swagger-ui/index.html


<br><br><br>


<h2 id="swagger-cors">🟡 스웨거를 위한 CORS 허용 (스웨거 도메인)</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

- gateway-service(CORS): 스웨거 도메인이 gateway-service 도메인 자체이므로 해당 도메인의 CORS 작업은 없습니다.
  
- 각 마이크로 서비스
  
	user-service, order-service, product-service, review-service, seller-service, supervisor-service 들의 CORS 설정은

	`https://benefits.completed0728.site` 도메인 주소를 허용 하였습니다. 

<br>

![](https://velog.velcdn.com/images/develing1991/post/f04c9f8b-3e9b-4a49-ac79-58ff94befd53/image.png)


<br><br><br>


<h2 id="front-cors">🟡 프론트엔드(SPA)를 위한 COSR 허용 (로컬 도메인)</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

- gateway-service는 `http://localhost:3001` 도메인 주소를 허용 하였습니다.

- 각 마이크로 서비스
   
  	user-service, order-service, product-service, review-service, seller-service, supervisor-service 들의 CORS 설정은   

	`http://localhost:3001` 도메인 주소를 허용 하였습니다. 

#### 프론트엔드(SPA) localhost에서 테스트 하시려면 3001번 포트로 실행 하시고 API를 요청 하시면 됩니다.

- 요청 API 주소 예시(GET):  https://benefits.completed0728.site/user-service/open-api/users/1

<br>

![](https://velog.velcdn.com/images/develing1991/post/3c54bff4-eb2c-46d1-8cec-cddf071436f1/image.png)


<br><br><br>

<h2 id="erd-relation">📚 ERD 다이어그램 (논리적 연관 관계 맵핑)</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

- 데이터베이스를 각 서비스의 기능과 가장 관련된 기준으로 나누었습니다.
  
- 관계의 설정 시 강결합 문제를 고려하여 데이터베이스에는 실제로는 연관 관계를 설정 하지 않았습니다.

<br>

![](https://velog.velcdn.com/images/develing1991/post/df835156-9d15-41cd-9e48-2ba0a68ecc60/image.png)

<br>

- ORM의 Entity 관계에서만 설정을 진행 하였습니다.

<br>

![](https://velog.velcdn.com/images/develing1991/post/58db0275-e8b3-4566-824c-ec504ba64b7d/image.png)

<br><br><br>

<h2 id="dependencies">🛠️서비스 개발에 사용 된 핵심 의존성 목록</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

**서비스 마다 특징이 부각 되는 의존성 목록 입니다.**

### 공통
- springboot 3.2.1
- jdk 17
- gradle 8.5

<br>
  
### naming-server

- org.springframework.cloud:spring-cloud-starter-netflix-eureka-server

<br>

### config-server

- org.springframework.cloud:spring-cloud-config-server

<br>

### gateway-service
- org.springframework.cloud:spring-cloud-starter-gateway

<br>

### user-service
- org.springframework.boot:spring-boot-starter-web
- org.springframework.boot:spring-boot-starter-security
- org.springframework.boot:spring-boot-starter-data-jpa
- org.springframework.boot:spring-boot-starter-validation
- io.jsonwebtoken:jjwt-api
- io.jsonwebtoken:jjwt-impl
- io.jsonwebtoken:jjwt-jackson
- com.mysql:mysql-connector-j
- 그 외 - (prometheus, zipkin, etc...)

<br>

### user-service, order-service, product-service
- org.springframework.kafka:spring-kafka
- 그 외 - (web, jpa, connector-j, security, prometheus, zipkin, etc...)

<br>

### review-service
- org.springframework.cloud:spring-cloud-starter-openfeign
- 그 외 - (web, jpa, connector-j, security, prometheus, zipkin, etc...)


<br><br><br>

<h2 id="after">📌 이 후 업데이트 사항</h2>

<h4 align="right">
	<a href="#list">목차로 이동</a>
</h4>

- <del>사용자 주문 취소와 판매자 주문 반려(거절)에 대한 재고 증감 - 동시성 처리 `orderCancel`</del>
- 현재 redis관련 학습중 입니다.
- user-service를 제외한 다른 마이크로 서비스들 미구현 된 AOP 검증 로직 추가
- 공통 코드 common-service로 분리 메이븐 업로드 또는 jar로 패키징 후 import로 사용
- 인증 관련 기능 로직 auth-service로 분리
- oauth 추가
- payment-service 추가
- redis
- Resilience4J circuitbreaker 추가
- 사용자 주문 취소 건 알림을 위한 SSE(Server Sent Events)기능 추가
- CI/CD 추가적인 빌드 검증 sonarqube-server 추가
- 배포 서버 docker-server에서 k8s-server로 이관 해보기 (pods, deployments, services 개념 적용)
- 구체적인 프론트 기획/개발 - 마주하는 장애 대응 해보기
