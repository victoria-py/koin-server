# Kombu [RabbitMQ with Python]

→ `kombu` 라이브러리를 기반으로 rabbitmq 큐에 원하는 (mcard 버닝이란) 테스크를 퍼블리시 하면, 큐에 mcard_burn이라는 키와 데이터, 리퀘스트 아이디가 인큐됨

### Kombu란?

> 파이썬 메시징 라이브러리
메시지 브로커를 쉽게 사용할 수 있는 솔루션

1. 다양한 메시지 브로커를 지원함
2. 자동으로 인코딩과 정규화를 해줌
3. 메시지 전송과 관련하여 예외처리가 잘 되어있음
4. 커넥션이나 채널 오류가 있을 때에도 정상적으로 동작하도록 구현되어있음
5. amqplib의 불편한점들이 수정됨
6. carrot을 사용하고 있는 프로젝트를 쉽게 포팅할 수 있음

- **`rabbitmq`** 에서 Exchange란?
    - Queue에 전송되기 전에 거쳐가는 라우터
    - 어떤 방식으로 메세지를 전달하냐에 따라 type이 나뉨
        1. fanout
            - bind된 모든 큐에 메시지를 전달
        2. direct
            - routing_key를 설정하여 같은 routing_key를 갖은 큐에만 선택적으로 메시지를 전달
        3. topic
            - direct type과 비슷하나 대신 routing_key에 패턴을 설정할 수 있음
            - 패턴에는 * 또는 #을 사용하는데, *은 모든 단어, #은 공백을 포함한 모든 단어
            - 단어의 구분은 .으로 함
        4. header
            - key와 value를 설정하여 선택적으로 메시지를 전달
- **`Producer`**
    - producer가 메시지를 publish하는 순서
    - exchange 선언 → queue 선언 → producer 선언 → publish 메시지 전달
    - publish할 때, declare 옵션에 따라 exchange, queue의 존재 여부를 확인하여 생성을 하거나 에러를 발생할 수 있음
        - rabbitmq에 exchange, queue가 있다면 그대로 사용 가능
        - 존재하는 exchange나 queue라면 바로 사용 가능하다는 뜻인듯
        - 코드상에서는 exchange나 queue 명을 쓰면 바로 그 녀석을 사용할 듯

            QueueMixin을 정의함으로 인해, option값으로 기본 세팅을 맞칠 수있음 → QueueMixin을 사용함으로 인해, 큐의 기본 세팅을 할 수 있음

            아래는 koin_server에서 사용하는 큐 퍼블리셔들임

            ```python
            QUEUE_PUBLISHERS = {
                'mcard.mint': _publisher_default_options,
                'mcard.single_burn': _publisher_default_options,
                'mcard.single_mint': _publisher_default_options,
            }
            ```

    - Exchange
        - `option` 중 **durable**
            - True로 하면, rabbitmq 서버가 죽어서 재시작하여도 유지됨
            - 디스크를 사용하냐, 메모리를 사용하냐의 차이
    - Queue
        - 위 durable이라는 옵션은 동일
        - **auto_delete**
            - 하나 이상의 소비자가 연결된 후에 모든 소비자가 연결이 끊어지면 큐가 삭제됨
            - rabbitmq에 exchange나 queue가 없을 때에도 자동으로 만들어짐
- **`Consumer` [Reciever]**
    - exchange → queue → binding 정보 업데이트
- **`channel`**
    - 하나의 물리적인 connection 안의 논리적인 connection
    - 비동기 작업을 할때 consumer(worker)를 프로세스나 쓰레드 형태로 여러개를 생성하여 작업을 처리하게 되는데, 이때 각 프로세스나 쓰레드도 물리적인 connection을 맺으면 자원 낭비 &&작업처리의 일관성도 떨어질 것
        - 한 워커가 물리적 연결이 끊기면, 그 워커는 계속 에러를 발생
    - 한 개의 물리적인 connection으로 여러 worker를  관리하는 것이 효율적

[참고링크](https://pygirl.tistory.com/1)
