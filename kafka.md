# KAFKA

### Kafka broker

- 실행된 카프카 애플리케이션 서버 중 1대
- 3대 이상의 브로커로 클러스터 구성
- 주키퍼와 연동 → 현재는 KRaft를 사용하여 메타데이터 저장
- N개 중 1대는 컨트롤러 기능을 수행
    - 담당 파티션 할당을 수행한다.

### Record

```java
new ProducerRecord<String, String>("topic", "key", "message");

ConsumerRecords<String, String> records = consumer.poll(1000);
for(ConsumerRecord<String, String> record: records) {
...
}

```

- 직렬화와 역직렬화하여 사용
- StringSerializer, ShortSerializer를 사용함.

### Topic & Partition

- 메세지 분류 단위
- n개의 파티션 할당 가능
- 각 파티션마다 고유한 오프셋을 가짐
- 메시지 처리 순서는 파티션 별로 유지 관리됨.

### Producer & Consumer

1. 프로듀서는 레코드를 생성하여 브로커로 전송
2. 전송된 레코드는 파티션에 신규 오프셋과 함께 기록됨
3. 컨슈머는 브로커로 부터 레코드를 요청하여 가져감(polling)
- Consumer가 11번 오프셋까지 가져갔다는 것은 0~10번까지는 이미 가져갔다는 것을 의미한다. 동일한 데이터도 가져갈 수 있다.

### Kafka log and segment

- 실제로 메시지가 저장되는 파일시스템 단위
- 메시지가 저장될 때는 세그먼트파일이 열려있음
- 세그먼트는 시간 또는 크기 기준으로 닫힘
- 세그먼트가 닫힌 이후 일정 시간에 따라 삭제 또는 압축

1. 그룹 내 파티션보다 컨슈머가 개수가 더 많으면 문제가 생긴다.
2. 컨슈머에 문제가 생기면 리밸런스를 하여 파티션 컨슈머 할당을 재지정한다.
3. 컨슈머에 따라 그룹이 달라지면 같은 파티션을 사용할 수 있다.

엘라스틱서치 → 로그 실시간 확인용. 시간순 정렬

하둡 → 대용량 데이터 적재.

하둡에 문제가 생겨서 적재지연이 발생하더라도 엘라스틱서치는 문제가 생기지 않기 때문에 컨슈머에 문제가 생기지 않음

### 리더 파티션, 팔로워 파티션

- replication_factor로 생성된 파티션은 팔로워, 원본은 리더이다.
- In-Sync Replica → 리더, 팔로워가 레코드가 모두 복제되어 sync가 맞는 상태
- 만약 장애가 나면 → unclean.leader.election.enable : true이면 유실이 되어도 처리를 한다.

### 서버 장애에 대응하는 로직

- 장애 허용(Fault-tolerant)은 매우 중요하다.

### 카프카 용어

- Broker: 카프카 애플리케이션 서버 단위
- Topic: 데이터 분리 단위. 다수 파티션 보유
- Partition: 레코드를 담고 있음. 컨슈머 요청시 레코드 전달
- Offset: 각 레코드당 파티션에 할당된 고유 번호
- Consumer: 레코드를 polling하는 애플리케이션
    - Consumer group: 다수 컨슈머 묶음
    - Consumer offset: 특정 컨슈머가 가져간 레코드 번호
- Producer: 레코드를 브로커로 전송하는 애플리케이션
- Replication: 파티션 복제 기능
    - ISR: 리더+팔로워 파티션의 sync가 된 묶음
- Rack-awareness: Server rack 이슈에 대응

### Kafka client

- Consumer 등이 모두 카프카 클라이언트임

### Kafka Streams

- 데이터를 변환하기 위한 목적으로 사용하는 API
- Stateful 또는 Stateless와 같이 상태기반 스트림 처리 가능

### Kafka Connect

- 미리 제공된 데이터를 import, export할 수 있음.
- 코드 없이 configuration으로 데이터를 이동하는 것이 목적

### Kafka Mirror maker

- Topic및 Record를 전부 복제하는 것이 목표이다.

### Confluent inc.

- Kafka opensource. kafka 만든 사람이 만든 회사

```bash
export KAFKA_HEAP_OPTS="-Xmx400m -Xms400m"
// Kafka heap 메모리 조정

-Xmx6g -Xms6g -XX:MetaspaceSize=86m -XX:+UseG1GC
-XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:G1HeapRegionSize=16M
-XX:MinMetaspaceFreeRatio=50 -XX:MaxMetaspaceFreeRatio=80
// 링크드인에서 테스트한 최적의 자바 옵션 추천값
// 60개의 브로커, 5만개의 파티션, Replication-factor2로 구성시
// 300MB/sec inbound, 1GB/sec outbound 보장
// 환산: 1TB/hour inbound, 3TB/hour outbound
```

### server.properties

- broker.id: 정수로 된 브로커 번호. 클러스터 내 고유번호로 지정
- listeners: kafka 통신에 사용되는 host:port
- advertised.listeners: kafka 클라이언트가 접속할 host:port
- log.dirs: 메시지를 저장할 디스크 디렉토리. 세그먼트가 저장됨
- log.segment.bytes: 파일 크기
- log.retention.ms: 메시지 보존 지정
- zookeeper.connect: 주키퍼 위치

### Producer

- 카프카로 데이터(K-V) 전송
- ProducerRecord 객체를 생성
- Java Kafka-client 제공

### Record Key

- 역할 : 메시지를 구분하는 구분자 역할
- 특징
    - 동일 키, 동일 파티션 적재 by Default partitioner
        - 순서를 보장하므로 state machine으로 사용 가능
        - 역할에 따른 컨슈머 할당 적용 가능
        - key = 주문 / value = 키보드 → 주문처리하는 컨슈머 애플리케이션
    - 레코드 값을 정의하는 구분자

### Record Value

- 역할: 실질적으로 전달하고 싶은 데이터
- String, ByteArray, Int 등 제한없이 보낼 수 있음
- CSV, JSON, Object 등 서비스에 맞게 사용할 수 있음
- Confluentinc/schema-registry → 포맷을 정할 수 있음

### Producer acks

- acks=0 → 속도가 가장 빠름, 유실 가능성이 높음
- acks=1 → 속도는 보통, 유실 가능성이 있음
- acks=all 또는 -1 → 속도는 느리고 유실 가능성은 없음

### acks = 0

- 프로듀서가 브로커와 소켓연결을 맺어 보낸 즉시 성공으로 간주
- 브로커가 정상적으로 받아서 리더 파티션에 저장했는지 알 수 없음
- 팔로워 파티션에도 저장되었는지 알 수 없음

### acks = 1

- 프로듀서가 보낸 메시지가 리더 파티션에 정상 저장되었는지 확인
- 팔로워 파티션에 저장되었는지 모름
- 리더 파티션에 저장되고 해당 브로커가 죽으면 손실될 수 있음

### acks = all 또는 -1

- 프로듀서가 보낸 메시지가 리더, 팔로워 파티션에 정상 저장되었는지 확인(ISR 상태)

### Producer options

- 필수 옵션
    - bootstrap.servers: 카프카 클러스터에 연결하기 위한 브로커 목록
    - key.serializer: 메시지 키 직렬화에 사용되는 클래스
    - value.serializer: 메시지 값을 직렬화 하는데 사용되는 클래스
- 선택 옵션
    - acks: 레코드 전송 신뢰도 조절(리플리카)
    - comression.type: snappy, gzip, lz4중 하나로 압축하여 전송
    - retries: 클러스터 장애에 대응하여 메시지 전송을 재시도하는 회수
    - buffer.memory: 브로커에 전송될 메시지의 버퍼로 사용될 메모리 양
    - batch.size: 여러 데이터를 함께 보내기 위한 레코드 크기
    - [linger.ms](http://linger.ms): 현재의 배치를 전송하기 전까지 기다리는 시간
    - [client.id](http://client.id): 어떤 클라이언트인지 구분하는 식별자

### Consumer

- 데이터를 가져가는(polling) 주체
    - 브로커가 보내는 게 아님
- commit을 통해 consumer offset을 카프카에 기록
- Java Kafka-client를 제공
- 데이터를 저장 → FileSystem, Object Storage, Hadoop …

### Consumer commit

- enable.auto.commit=true
    - 일정 간격, poll() 메소드 호출시 자동 commit
    - commit 관련 코드를 작성할 필요 없음. 편리함.
    - 속도가 가장 빠르고 중복 또는 유실이 발생할 수 있음
- 리밸런싱 발생시 중복 처리 됨
- 커밋을 뒤쪽까지 했는데 컨슈머가 처리하다가 유실될 수 있음
- 오토 커밋을 사용하되, 컨슈머가 죽지 않도록 잘 돌봐준다 → 말이 안됨.

- enable.auto.commit=false
    1. commitSync() : 동기 커밋
        1. ConsumerRecord 처리 순서를 보장함.
        2. 가장 느림 → 커밋 완료까지 blocking
        3. poll() 메소드로 반환된 ConsumerRecord의 마지막 offset 커밋
        4. Map<TopicPartition, OffsetAndMetadata> 을 통해 오프셋 지정 커밋 가능
    2. commitAsync() : 비동기 커밋
        1. 동기 커밋보다 빠름
        2. 중복이 발생할 수 있음
        3. ConsumerRecord 처리 순서를 보장하지 못함

```java
while(true) {
	ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
	for(ConsumerRecord<String, String> record: records) {
		System.out.println(record.value());
	}
	try {
		consumer.commitSync();
	} catch(CommitFailedException e) {
		System.err.println("commit failed");
	}
}

Map<TopicPartition, OffsetAndMetadata> offset = new HashMap<>();
offset.put(new TopicPartition(record.topic(), record.partition()), null);
try {
	consumer.commitASync(offset);
} catch(CommitFailedException e) {
	System.err.println("commit failed");
}
```

### Consumer rebalance

- 컨슈머 그룹의 파티션 소유권이 변경될 때 일어나는 현상
- 리밸런스를 하는 동안 일시적으로 메시지를 가져올 수 없음
- 리밸런스 발생 시 데이터 유실/중복 발생 가능성 있음
    - commitSync() 또는 추가적인 방법(unique key)으로 데이터 유실, 중복 방지
- 리밸런스는 consumer.close() 또는 consumer 세션이 끊겼을 때 발생

```java
configs.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 3000);
configs.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 10000);
// heartbeat 3초마다 보냄, 10초의 세션 타임아웃동안 heartbeat가 왔는지 확인
// 안왔으면 리밸런싱
```

- 리밸런스 리스너는 리밸런스 발생에 따른 offset commit, 시간 측정을 통한 컨슈머 모니터링 때 동작

### Consumer wakeup

- 정상동작 예시
    1. poll() 호출 → records 100개 반환 : offset 101~200번 
    2. records loop 구문 수행
    3. record.value() System print
    4. offset 200 커밋
    5. poll() 호출
    6. 반복
- SIGKILL로 인한 중복 처리 발생 시
    1. poll() 호출 → 마지막 커밋된 오프셋이 100, records 100개 반환, 오프셋 101~200
    2. records loop() 구문 수행
    3. record.value() system 150번 오프셋 프린트 중 SIGKILL 호출
        1. 151~200번은 처리는 안됨.
    4. offset 200 커밋 불가
        1. 브로커에서는 100번 오프셋이 마지막 커밋
        2. 컨슈머 재시작, 다시 오프셋 101부터 처리 시작, 101~150번은 중복 처리
- wakeup()을 통한 graceful shutdown은 필수
    - SIGTERM을 통한 shutdown signal로 kill하여 처리한 데이터 커밋 필요
    - SIGKILL(9)는 프로세스 강제 종료로 커밋 불가 → 중복, 유실 발생

### Consumer thread 전략

1. 1 프로세스 + 1 스레드
    1. 간략한 코드
    2. 프로세스 단위 실행 및 종료
    3. 다수의 컨슈머 실행 시 다수의 프로세스 실행

```java
cat consumer.conf
{"topic":"click_log", "group.id":"hadoop-consumers"}

java -jar one-process-one-consumer.jar --path consumer.conf
```

1. 1 프로세스 + n 스레드(동일 컨슈머 그룹)
    1. 복잡한 코드
    2. 스레드 단위 실행 및 종료
    3. 스레드간 간섭 주의
    4. 다수의 컨슈머 실행 시 다수의 스레드 실행
2. 1 프로세스 + n 스레드(다수의 컨슈머 그룹)

### Consumer lag

- 컨슈머 마지막 커밋 오프셋이 117이고 토픽의 마지막 오프셋이 123이면 컨슈머 랙은 6이다.
- 프로듀서가 더 많은 양을 보낼 수록 컨슈머 랙이 많이 발생한다.
- 컨슈머의 상태를 나타내는 지표.
- consumer.metrics()를 통해 확인할 수 있는 지표
    - records-lag-max: 토픽의 파티션 중 최대 랙
- 애플리케이션 자체를 죽어버리면 랙 수집에 문제점이 생긴다.
    - 구현하는 컨슈머마다 지표를 수집하는 로직 개발 필요
    - 컨슈머 랙 최대 값만 알 수 있음
    - 외부 모니터링 애플리케이션, confluent platform, kafka burrow(opensource) 등
- kafka burrow
    - OK: 정상
    - ERROR: 컨슈머가 polling을 멈춤
    - WARNING: 컨슈머가 polling을 수행하지만 lag이 늘어남

## Kafka 적용

### 1. Kafka 동작 방식

![kafka.png](KAFKA%20015d10b0aa57457a9c4bed2646e56df9/kafka.png)

### Kafka Producer

- Kafka Producer는 Event(Record)를 Kafka Cluster로 스트리밍한다.
- 레코드는 K-V 페어로 전달한다.
    - Key와 Value는 모두 serialize 해야 한다.

### Kafka Consumer

- Kafka Consumer는 Event(Record)를 Kafka Cluster에서 받는다.
- 그룹을 지정해 줄 수 있다.
- 레코드를 K-V 페어로 전달 받는다.
    - Key와 Value 모두 deserialize 해야 한다.

### Kafka Cluster

- Kafka Cluster는 3개 이상의 Broker로 이뤄져 있다.
- 만약 3개 미만이라면(보통 1개) 단일 브로커라고 한다.
- 그 중 하나는 Controller로 수행하며, 담당 파티션을 할당한다.
    - 여러 개의 브로커 중 컨트롤러를 선정해야 하는데 이를 카프카 3.4 전까지는 zookeeper가 선출하고, 3.0 이후부터는 카프카 자체 컨센서스 알고리즘인 KRaft를 사용하여 선출한다.
    - zookeeper는 2008년에 hadoop의 서브 프로젝트로 시작했으며, controller인 브로커가 실패했을 때 failover 알고리즘을 사용하여 active NameNode를 선택한다.
    - KRaft는 failover를 거의 즉각적으로 만들며, zookeeper와 kafka의 의존성을 없애고 확장성을 높여준다.

### Zookeeper

- Zookeeper가 여러 개로 구성되어 있다면 Zookeeper Ensemble이라고 한다.
- 파일 시스템에 저장되는 파일을 znode라고 한다.
    - Persistent znode: Default type으로 Zookeeper에서 삭제되기 전까지 남아있는다.
    - Ephemeral znode: znode가 만들어진 세션이 연결이 끊기면 삭제된다. 또한 자식 노드를 가질 수 없다.
    - Sequential znode: ID와 같은 sequential한 데이터를 만들기 위해 사용된다.
- Zookeeper Atomic Broadcast(ZAB) protocol을 사용하여 요청들을 leader server로 보낸다.
- Zookeeper는 클라이언트로부터의 모든 업데이트에 순차적인 동시성과 원자성을 제공한다.
- Controller election
    - 각 브로커는 ephemeral node를 만들려고 시도한다.
    - 처음 ephemeral node를 만든 브로커는 controller로 선정되며, controller epoch을 가진다.
- Cluster membership
    - Zookeeper 인스턴스에 브로커가 연결되면, ephemeral znode가 group znode 밑에 생성된다. 만약 브로커가 실패하면, ephemeral znode가 삭제된다.
    - 다시 말하자면, Zookeeper 내부의 같은 그룹의 모든 client는 ephemeral znode를 볼 수 있고, 보다가 ephemeral znode가 사라짐에 따라 membership의 여부를 알 수 있다.
- Topic configuration
    - Kafka는 Zookeeper 내부의 각 토픽에 대한 configuration set을 유지한다. 이 설정은 topic 별 일 수도 있고 global 일 수도 있다.
    - 또, 현재 존재하는 토픽의 리스트나 partition, replica의 개수도 저장한다.
- Access Control Lists(ACLs)
    - Kafka는 모든 topic의 ACL을 유지한다.
    - topic에 어떤 혹은 누가 읽기, 쓰기를 허용할 지 결정한다.
    - consumer의 consumer group도 저장한다.
- Quotas
    - Kafka는 client가 이용할 수 있는 브로커 자원을 컨트롤한다.
    - Zookeeper에 Quotas로 저장된다.

### KRaft

![event-driven-consensus-internals.png](KAFKA%20015d10b0aa57457a9c4bed2646e56df9/event-driven-consensus-internals.png)

- Zookeeper 대신에 자체적으로 metadata quorum을 컨트롤하도록 대체했다.
- KIP-500에 나온 컨센서스 프로토콜이다.
- quorum controller는 metadata가 정확하게 quorum controller들 사이에 복제되도록 보장한다.
- quorum controller는 그 상태를 event-sourced storage model(상태 변환 이벤트를 순서에 맞게 저장하는 모델)을 사용하여 저장한다.
- 이 상태를 저장하기 위해 사용되는 이벤트 로그는 로그가 무한정 쌓이지 않게 하기 위해 주기적으로 요약된 snapshot이다.
- quorum에 있는 다른 컨트롤러는 active controller를 따라 event에 응답한다.
- 예를 들어, 한 노드가 partitioning event를 위해 멈춘다면, 놓친 event들을 이벤트 로그를 확인하는 것만으로 따라잡을 수 있다.
- 이런 event-driven이라는 특성 덕분에 quorum controller는 active가 되기 전까지 Zookeeper로부터 상태를 로딩할 필요가 없어진다.
- 만약 leadership이 바뀌면, 새로 active된 controller는 메모리에 커밋된 metadata record를 모두 가지고 있다.
- KRaft를 사용한 클러스터는 아파치 카프카 3.3 버전부터 가능하다.

### Kafka Topic

![streams-and-tables-p1_p4.png](KAFKA%20015d10b0aa57457a9c4bed2646e56df9/streams-and-tables-p1_p4.png)

- Kafka Topic은 이벤트의 로그와 같다.
- 각 토픽은 0개 혹은 여러 개의 producer와 consumer를 가진다.
- 그리고 각 토픽은 1개 혹은 여러 개의 partition을 가진다.
- 토픽은 복제될 수 있는데, 복제된 토픽을 follower, 원본 토픽을 leader라고 한다.

### docker-compose.yml

```yaml
zookeeper:
    image: confluentinc/cp-zookeeper:7.0.0
    hostname: zookeeper
    container_name: zookeeper
    environment:
      ZOOKEEEPER_SERVER_ID: 1
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_INIT_LIMIT: 5
      ZOOKEEEPER_SYNC_LIMIT: 2
    ports:
      - "22181:2181"

  kafka1:
    image: confluentinc/cp-kafka:7.0.0
    hostname: kafka1
    container_name: kafka1
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka1:19092,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
    depends_on:
      - zookeeper

  kafka2:
    image: confluentinc/cp-kafka:7.0.0
    hostname: kafka2
    container_name: kafka2
    ports:
      - "9093:9093"
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka2:19093,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
    depends_on:
      - zookeeper

  kafka3:
    image: confluentinc/cp-kafka:7.0.0
    hostname: kafka3
    container_name: kafka3
    ports:
      - "9094:9094"
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka3:19094,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9094
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
    depends_on:
      - zookeeper

  kafdrop:
    image: obsidiandynamics/kafdrop
    restart: "no"
    ports:
      - "9000:9000"
    environment:
      KAFKA_BROKER_CONNECT: "kafka1:19092"
    depends_on:
      - kafka1
      - kafka2
      - kafka3
```

- zookeeper 한 개에 Kafka broker가 3개가 클러스터로 붙은 상황이다.
