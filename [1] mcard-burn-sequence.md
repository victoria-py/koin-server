# Koin-Server 정복하기

## Mcard 삭제 시퀀스
![image](https://t1-beta.daumcdn.net/blockadmintest/victoria_test/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202021-03-05%20%EC%98%A4%ED%9B%84%203.19.11.png)

### 구성요소

1. **mcard_single_burn_queue**
2. **blockchain-node**
    - 블록체인 노드에는 토큰만 존재
    - MCard Model에서 token_id에 매칭되는 값
3. **transaction_proccess_queue**
4. **transaction-db**
    - 트랜잭션 기록용 데이터 베이스
5. **mcard-db**
    - 블록체인 노드에는 단순 토큰만 관리되는데, 카드 데이터들이 이 디비에 관리되고 있음

### 플로우

1. 클라이언트 → Koin-Server로의 요청
    - 202 Accepted 응답을 보낼 때, 요청에 대한 트랜잭션 ID를 반환해야하기 때문에 빈껍데기의 트랜잭션을 생성함
    - 해당 요청 작업을 mcard_single_burn_queue에 enqueue
    - transaction-db에 트랜잭션 생성 후 클라이언트에 해당 id 리턴
2. **mcard_single_burn_queue [worker : mcard_single_burner]**
    - Queue에 들어간 Task를 mcard_single_burner가 받아서 작업함
3. **mcard_single_burner**
    - **blockchain-node**에 burn-transaction을 날림
    - transaction-db에 관련 transaction 상태를 업데이트함
4. **transaction-tracer**
    - blockchain-node로부터 transaction을 읽어옴
    - tracer가 해당 transaction 작업이 완료되면, **transaction_process_queue**에 Enqueue함
5. **transaction_processor**
    - Task를 받아서 **transaction-db**에 해당 트랜잭션 상태 업데이트
    - **mcards-db**에서 카드 데이터 제거
    - 트랜잭션 콜백 있을 시, Callback_queue에 Enqueue
    - TMS도, 해당 queue에 Enqueue