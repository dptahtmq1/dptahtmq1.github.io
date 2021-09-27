# Apache Flink

# Kafka Connector

[https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/connectors/kafka.html](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/connectors/kafka.html)

## Kafka Consumer

### DeserializationSchema

Flink에서 기본적으로 제공하는 schema들

- TypeInformationSerializationSchema : Flink의 TypeInformation을 기반으로 schema 적용. flink에서 write, read 둘 다 할 때 유용함.
- JsonDeserializationSchema(JSONKeyValueDeserializationSchema) : JSON을 ObjectNode로. `metadata` field에 offset/partition/topic 정보가 담겨 있음
- AvroDeserializationSchema : Avro 포맷

deserialization 실패했을 때 null 을 리턴해야 해당 record를 건너뛸 수 있음. 그게 아니면 fault tolerance 로직에 의해 계속 재시도 하게 됨.

### Kafka consumers start position

- setStartFromGroupOffsets : committed offset 그대로 사용
- setStartFromEarliest
- setStartFromLatest
- setStartFromTimestamp
- setStartFromSpecificOffsets
    - partition-offset이 명시되지 않으면 fallback:default 로 동작

### Fault Tolerance

Flink checkpointing이 켜져 있으면 주기적으로 checkpoint를 기록함. job이 실패하면 최근 checkpoint에서 값을 가져와서 re-consume.

checkpointing이 꺼져 있으면 주기적으로 zookeeper에 offset을 저장함.

### Discovery

dynamic하게 kafka partition 추가를 감지할 수 있음. default로 꺼져 있고, 감지 주기도 설정 가능함.

마찬가지로 topic도 dynamic하게 감지 가능함. consumer 생성 시에 정규식으로 패턴을 넣을 수 있음.

### Offset committing behaviour

kafka에 저장하고 싶을 때는 어떻게 하는가?

kafka offset에 저장하긴 하는데, fault tolerance를 위해 kafka itself에 의존하지 않음. 커밋된 오프셋은 모니터링 목적.

checkpointing disabled인 경우 : kafka client의 기능에 의존

checkpointing enabled인 경우 : checkpoint가 완료됐을 때 그 값을 commit함. 이는 checkpoint와 committed offset이 일관성 있음을 보장할 수 있음. setCommitOffsetsOnCheckpoints 이 값 설정을 통해 offset committing을 on/off 할 수 있음

## Kafka Producer

### SerializationSchema

object → binary data

각 record마다 serialize method가 호출되어 ProducerRecord 생성하고 kafka에 기록

`KafkaSerializationSchema` 를 쓰면 key 지정할 수 있음 : [https://stackoverflow.com/questions/65248097/send-key-in-flink-kafka-producer](https://stackoverflow.com/questions/65248097/send-key-in-flink-kafka-producer)

### Producer and Fault Tolerance

FlinkKafkaProducer011 (FlinkKafkaProducer for Kafka >= 1.0.0 versions)

checkpointing enabled면 얘가 exactly-once delivery 보장할 수 있음

그리고 semantic parameter로 설정할 수도 있음

- Semantic.NONE
- Semantic.AT_LEAST_ONCE (default)
- Semantic.EXACTLY_ONCE

# Checkpointing

[https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/stream/state/checkpointing.html](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/stream/state/checkpointing.html)

Flink의 모든 function 및 operator는 stateful 할 수 있음. state의 fault tolerant를 위해 flink는 checkpoint 가 필요함. 이를 통해 state 및 stream position을 복구하여 failure-free execution을 달성할 수 있음.

## Prerequisites

- persistent data source: message queue 또는 file system
- state를 저장할 persistent storage: 전형적인 distributed filesystem(HDFS, S3 등)

## Enabling and Configuring

default는 disabled. `enableCheckpointing(n)` 으로 enable 할 수 있음.

## State Backend

consistent snapshots을 어디에 저장할거냐? 

default : state는 task manager의 in-memory, checkpoint는 job manager의 in-memory.

[https://ci.apache.org/projects/flink/flink-docs-release-1.11/ops/state/state_backends.html](https://ci.apache.org/projects/flink/flink-docs-release-1.11/ops/state/state_backends.html) 여기에 관련 내용 있음.

KDA에서는 어떻게 설정할 수 있을까? configure에 옵션 있었던 것 같음. snapshot enable 어쩌고..(snapshot은 savepoint 랑 동일한 것. 다른 내용임)

[https://docs.aws.amazon.com/kinesisanalytics/latest/java/reference-flink-settings.title.html](https://docs.aws.amazon.com/kinesisanalytics/latest/java/reference-flink-settings.title.html)

여기 보면 RocksDBStateBackend를 사용한다고 함(S3 기반). 다른거 설정 못함.

여기는 기본적으로 CheckpointingEnabled가 default true네..

## Savepoints

consistent image: streaming job의 execution state

savepoint를 통해 stop-and-resume, fork, update job 등을 할 수 있음.

- binary files on stable storage(HDFS, S3 등)
- meta data file : stable storage의 모든 파일 가리키는 포인터
-