## 주문 전 테이블 확인 및 주문 수량 확인

<br>

### 상품 테이블

- 한 주문당 구매할 상품  
	4번 상품(길럭시북 그래파이트 15.6인치 코어 i5 - 512GB) - 2개  
	5번 상품(김럭시북9 그려파이트 14인치 코어 16GB - 2TB - 인도우11 PRO) - 1개
  
```javascript
// 하나의 주문
{
    "order": {
        "user_id": 1,
        "amount": 4515000.0000
    },
    "deliveryAddress": {
        "receiver": "김수령",
        "address1": "주소1",
        "address2": "주소2",
        "zipcode": "123-23",
        "phone": "01022223333",
        "receive_type": "DOOR",
        "receive_message": "문 앞"
    },
    "orderProducts" : [
        {
            "product_id": 4, // 4번 상품
            "seller_id": 2,
            "qty": 2,        // 수량 2개
            "unit_price": 1178000.0000,
            "total_price": 2356000.0000
        },
        {
            "product_id": 5, // 5번 상품
            "seller_id": 2,
            "qty": 1,        // 수량 1개
            "unit_price": 2159000.0000,
            "total_price": 2159000.0000
        }
    ]
}
```

- 위 주문을 200건 동시에 요청할 계획 입니다.

![](https://velog.velcdn.com/images/develing1991/post/44b24925-e11d-4a3c-ad4a-285e7799cc08/image.png)

<br>

### 주문 테이블

- 주문 테이블은 현재 어떤 주문도 받지 않은 상태 입니다.  

![](https://velog.velcdn.com/images/develing1991/post/b93707cc-a46a-4350-8267-72cdc586fd30/image.png)

<br>

### 테스트 전에 기대하는 결과

한 주문당 4번의 상품을 2개씩 주문하니 

- 4번의 상품이 재고가 0개가 되는 시점은 30개 - (2개 * 15번 주문) = `재고 0개`   
	-> 즉, 15개의 주문이 들어왔을 때 `4번` 상품의 재고가 `0`개, 상태가 `SOLD_OUT` 되고    
	재고 수량은 더 이상 마이너스로 줄어들지 않기를 기대합니다.  

- 4번의 재고가 0개가 되는 시점에   
	5번 상품은 20개 - (1개 * 15번 주문) = `5개`   
    	-> 즉, 하나로 묶인 주문이니 4번의 재고가 부족할 때 주문 자체가 불가능 해야 하므로  
        `5번` 상품의 재고는 `5개`의 재고가 남을 것으로 기대합니다.

<br><br>

## 테스트 코드 실행

- 스레드 풀 200개를 준비 합니다.

- 동시 요청 할 200건의 태스크를 준비 합니다.  
	(한 개의 태스크 당 1건의 주문 - 4번 상품 2개, 5번 상품 1개 )

- 200건의 동시 요청 **실행** 합니다.

![](https://velog.velcdn.com/images/develing1991/post/d6b2b57e-ba21-424c-a1dd-f5142f62a2ab/image.png)

<br><br>

## 결과 진행 상황 확인

<br>

### 상품 테이블

- `4번` 상품 재고 `0`개, 상태 `SOLD_OUT` - 기댓 값과 일치 합니다.  
- `5번` 상품 재고 `5`개  - 기댓 값과 일치 합니다.  

![](https://velog.velcdn.com/images/develing1991/post/bcf80392-d269-400e-af89-a029690bde22/image.png)

<br>

### 주문 테이블 들어온 전체 주문 수

- **select count(*) form orders.orders;**

- `200`건의 요청이  한 번에 들어왔을 때

- 주문 요청 자체에서   
	상품의 상태가 `SOLD_OUT`임을 확인 한 시점 `121`개  
	즉 `121`개 지점 부터는 더 이상 요청 자체를 받지 않았습니다.  

![](https://velog.velcdn.com/images/develing1991/post/7c505e17-d0a9-4d21-b3a7-768be48ba7a0/image.png)

<br>

### 121건 주문의 처리 상태 확인

- **select count(*) form orders.orders;**

- 동시 다발로 들어온 주문 `121`건 중  
	실제 주문 받을 `15`건을 제외 하고   

- 나머지 `106`건의 주문은 재고가 부족했기 때문에  
	카프카에 의해 반려 처리 중인 것을 확인합니다.  

![](https://velog.velcdn.com/images/develing1991/post/58ef9839-f97f-40e3-b356-6a2f88b9d0a1/image.png)

<br>

### 상품 서비스에서 orderReject 토픽을 생산하는 과정

- 해당 상품이 소진 되었기 때문에 `orderReject`토픽을 생산합니다.

![](https://velog.velcdn.com/images/develing1991/post/47c14bc4-f8ab-46a5-9d2d-92a5b360a4f0/image.png)

<br>

### 주문 서비스에서 orderReject 토픽을 소비하는 과정

- `orderReject`토픽을 소비합니다. 해당 주문 건을 찾아 주문 상태를 `REJECT`로 변경 합니다.
  
![](https://velog.velcdn.com/images/develing1991/post/4f046a68-ed06-4a76-b822-9386594ba2a1/image.png)

<br><br>

## 최종 결과 정리

- `200`건의 동시 요청 중 상품 상태가 `SOLD_OUT` 임을 확인 한 시점 부터 요청을 받지 않았으며   
그 사이엔 `121`건의 요청이 들어 왔습니다.  

1. 최종 결과 주문을 정상적으로 받아야 하는 건수는 `15`건 입니다.  

2. 최종 결과 재고 부족으로 주문을 카프카에 의해 반려 처리되어야 할 건수는 `106`건 입니다.

<br>

### 15건 을 제외한 나머지 주문 106건은 전부 반려 처리 되었음을 확인합니다.

![](https://velog.velcdn.com/images/develing1991/post/7df3217a-fc49-4b42-abdb-8722e47842aa/image.png)


![](https://velog.velcdn.com/images/develing1991/post/2f954d11-9323-40c9-992b-7d8d23c1f8b1/image.png)

