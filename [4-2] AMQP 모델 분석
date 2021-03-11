# AMQP

### What is AMQP 0-9-1?

- Advanced Message Queuing Protocol
- 클라이언트 어플리케이션들이 미들웨어 메세징 브로커와 통신할 수 있도록 하는 메세징 프로토콜

### Brokers and Their Role

- 메세지 브로커들은 publisher(== producer)로부터 메세지를 전달받음
- 위 메세지를 consumer에게 라우팅해줌

### AMQP 0-9-1 Model in Brief

![image](https://www.rabbitmq.com/img/tutorials/intro/hello-world-example-routing.png)

- message는 exchange로 publish됨
    - message를 publish할 때, 다양한 속성(메타 데이터)를 지정할 수 있음
    - 일부는 브로커에서 사용가능하나, 나머지는 메세지를 받는 어플리케이션에서 사용
- exchange들은 바인딩되어 message 복사본들을 큐에 분배시킴
- 브로커는 큐에 있는 consumer에게 메세지를 전달하거나 소비자가 요청시 큐에서 메세지를 가져옴
    - **메시지 승인 개념**
        - 네트워크가 불안정하거나 응용 프로그램이 메시지 처리 실패했을 시 사용할만한 개념
        - message가 consumer에 전달되면, consumer는 자동 또는 응용프로그램 개발자가 (해당 작업) 선택 시 이를 Broker에 알림
        - message가 사용중이라고 확인되면, Broker는 큐에서 message를 완전히 제거함
- message를 라우팅할 수 없는 상황에서
    - `message가 publisher에게 **반환** 또는 **삭제**`
    - `dead letter queue`에 배치

### AMQP 0-9-1 is a Programmable Protocol

---

## Exchanges and Exchange Types

Exchange는 메세지를 가져와 0개 이상의 큐로 라우팅함

라우팅 알고리즘은 바인딩이라는 exchange type 및 규칙에 따라 다름

### **exchange type**

- **`Default exchange`**
    - 브로커가 미리 선언한 이름이 없는 교환
    - 생성되는 모든 큐는 **큐 이름과 동일한 라우팅키를 사용하여 자동 바인딩됨**
        - 메세지를 대기열에 직접 전달하는 것처럼 보임
- **`Direct exchange`**
    - **메시지 라우팅키를 기반으로 큐에 메시지를 전달**
    - 메시지의 유닛 케스트 라우팅에 이상적
    - 큐는 라우팅 키 K를 사용하여 Exchange에 바인딩됨
    - 라우팅 키가 R인 메시지가 도착하면 Exchange는 K=R인 경우 큐로 라우팅함
    - 라운드 로빈 방식으로 여러 워커에게 작업을 배포하는데 사용됨
    - 메시지는 큐가 아닌 consumer로 balanced 맞춰 분배

    ![image](https://www.rabbitmq.com/img/tutorials/intro/exchange-direct.png)

- **`Fanout exchange`**
    - 메시지를 바인딩된 **모든 큐로 라우팅**하며 라우팅 키는 무시됨
        - N개의 대기열이 팬아웃 Exchange와 바인딩된 경우, 새 메시지가 publish되면 해당 메시지의 사본이 N개의 큐에 전달됨
        - 브로드 캐스트 라우팅에 이상적임
    - 멀티 플레이어 온라인 게임에서 리더 보드 업데이트 또는 글로벌 이벤트에 사용 가능
    - 실시간 모바일 클라이언트에 점수 업데이트
    - 그룹채팅

    ![image](https://www.rabbitmq.com/img/tutorials/intro/exchange-fanout.png)

- **`Topic exchange`**
    - 메시지 라우팅키와 큐를 바인드하는데, **사용된 패턴간의 일치**를 기반으로 라
- **`Headers exchange`**
    - 라우팅 키 속성을 무시 → 헤더 속성에서 가져옴
    - 헤더 값이 바인딩 시 지정되었던 값과 같으면 메시지와 일치하는 것으로 간주
    - 둘이상의 헤더를 사용하여 해결 가능
        - → 이때 헤더가 하나만 일치해도 되는지, 모두 일치해야 하는지 "x-match" 옵션을 알아야함
        - `any` 또는 `all` 두 가지 옵션이 있음

**다음과 같은 주속성이 있음**

- Name
- Durability (broker 재시작 시에도 exchange가 유효하게 하는 옵션)
- Auto-delete (마지막 큐가 바인딩 해제되면 exchange가 삭제됨)
- Arguments

---

## Queues

- 응용 프로그램에서 사용하는 메시지를 저장
- 대기열은 일부 속성을 exchange와 공유하나 몇가지 추가 속성도 존재함
    - 이름
    - Durable
        - 하나의 연결에서만 사용되며, 해당 연결이 닫히면 대기열 삭제
    - Auto-delete
        - 최종 Consumer가 구독을 취소하면 대기열이 삭제됨
    - Arguments
        - 메시지 TTL, 대기열 길이 제한 등과 같은 플러그인
- 어플리케이션은 queue 이름을 선택하거나 브로커에게 이름 생성을 요청할 수 있음
- 영구 대기열의 메타 데이터는 디스크에 저장되고, 임시 대기열의 데이터는 메모리에 저장됨

## Bindings

- 메시지를 큐로 라우팅하기 위해 Exchange에서 사용하는 규칙
- 메시지를 큐로 라우팅하려면 큐를 Exchange에 바인딩해야함
    - 일부 Exchange 유형에서 사용하는 라우팅 키 속성이 있음

---

## Consumer

- `push API`
    - Message가 전달되도록 구독
    - 어플리케이션이 특정 큐에서 메시지를 사용하는데 관심이 있음을 나타내야함
    - == 소비자 등록 → 대기열에 가입
    - 한 큐는 둘 이상의 consumer를 갖거나 독점 소비자를 등록할 수 있음
- `pull API`
    - 권장되지 않음

---

## Message

### Message Acknowledgements

- 메시지를 처리하지 못하거나 충돌이 생길 수 있음
- Broker가 Queue에서 메시지를 제거하는 타이밍
    1. 브로커가 응용프로그램에 메시지를 보낸후 
        - `basic.deliver` 또는 `basic.get-ok` 메소드 사용
        - 자동 확인 모델
    2. 어플리케이션이 `acknowledgement`을 다시 보냈을 때
        - 명시 확인 모델
        - 명시 모델을 사용하면, 어플리케이션이 승인을 보낼 시간을 선택가능
            1. 메시지를 수신한 직후
            2. 메시지 처리 직전
            3. 데이터 저장소에 저장한 후
            4. 메시지를 완전히 처리한 후
- 소비자가 승인을 보내지 않고 죽으면, 브로커는 이를 다시 소비자에게 전달함
- 소비자가 없는 경우, 동일한 대기열에 소비자가 등록될 때까지 기다림
- 메시지 처리 실패 또는 처리 할수 없을 시,
    - 브로커에게 메시지를 버리거나 다시 대기열에 넣도록 요청할 수 있음
    - `basic.reject` 를 이용해 거부할 수 있음

### Prefetching Messages

- 여러 Consumer가 대기열 공유 시, 승인을 보내기 전에 각 소비자가 한번에 보낼 수 있는 메시지를 지정

→ 로드 밸런싱 또는 메시지가 배치로 publish되는 작업일 경우 처리량을 향상 시킬 수 있음

### Message Attributes and Payload

> Content type <br>
> Content Encoding <br>
Routing Key <br>
Delivery Mode (persistent or not) <br>
Message priority <br>
Message Publishing timestamp <br>
Expiration period <br>
Publisher Application ID <br>

- 일부 속성은 브로커에서 사용하나 대부분 이를 수신하는 응용프로그램에서 해석함
- 메시지 속성은 publish 당시 설정됨
- 브로커는 페이로드를 검사하거나 수정하지 않음
    - 메시지는 속성만 포함하고 페이로드는 없을 수도 있음
- 메시지는 영구 메시지로 publish 될 수도 있으므로 브로커가 메시지를 디스크에 유지함
- 서버 재시작 시, 시스템은 수신된 지속성 메시지가 손실되지 않도록 함
    - 메시지 자체의 지속성에 따라 다름
    - 단, 메시지를 지속적으로 서빙할 때 성능의 영향(저장소 또는 비용적 측면)이 있으므로 속성을 잘 지정할 것

[참고링크](https://www.rabbitmq.com/tutorials/amqp-concepts.html#amqp-model)
