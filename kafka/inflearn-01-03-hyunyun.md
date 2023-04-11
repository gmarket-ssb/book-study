# 카프카란?
> 실시간, 대용량, 분산처리, 고성능, 오픈소스인 **Event Streams Platform**의 대표 솔루션


### 카프카를 배우는 이유 
- 데이터 파이프라인 구축부터 마이크로서비스까지 다양한 용도로 활용
- 담당 업무 중, 빅스마일데이 랭킹시스템 유지보수에 사용되고 있음 (카프카 처음 봄)


### 실습환경 세팅
- VM머신 설치 후 Linux 이미지 설치
- openjdk11, confluent kafka
- iTerms로 ssh접속
<img width="761" alt="image" src="https://user-images.githubusercontent.com/106303141/231116582-8f7f1faf-06ea-4d23-973f-caa3d67e86bf.png">


#### 카프카 서버 가동 테스트
zookeeper 서버 가동

`zookeeper-server-start $CONFLUENT_HOME/etc/kafka/zookeeper.properties`

kafka 서버 기동

`kafka-server-start $CONFLUENT_HOME/etc/kafka/server.properties`

topic 생성

`kafka-topics --bootstrap-server localhost:9092 --create --topic welcome-topic`

<img width="757" alt="image" src="https://user-images.githubusercontent.com/106303141/231117837-7294ece5-0e5c-4a82-b860-c561633ce087.png">


### 토픽(Topic) 개요

<img width="1149" alt="image" src="https://user-images.githubusercontent.com/106303141/231128615-86819296-c6b1-4242-a5ec-f0698885cd55.png">

- Topic은 **Partition**으로 구성된 일련의 로그 파일
- Topic은 Key&Value구조이며 Value는 어떤 타입의 메세지도 가능하다.
  - 문자열, 숫자값, 객체, Json, Avro, Protobuf 등
- 메시지가 **순차적**으로 물리적인 파일에 write 된다.
- 병렬 성능과 가용성
- 개별 파티션은 immutable 성질의 레코드로 구성된 메세지
- 개별 파티션은 다른 파티션과 독립적이다.


### 병렬 분산 처리(클러스터)

<img width="1235" alt="image" src="https://user-images.githubusercontent.com/106303141/231130780-197c0e95-0567-4a7b-ae81-f186f20c8e08.png">

- Kafka 서버 중 한 대가 죽어버리면 메세지가 사라져버린다. (정합성 X)
- 이를 방지하기 위해, `replication-factor=2`를 사용

<img width="1217" alt="image" src="https://user-images.githubusercontent.com/106303141/231131146-1fff7347-5365-4346-83d6-3b6085f46ab4.png">

- 복제(replication) 영역을 만들어 대비한다.


## kafka-topics 명령어를 이용하여 Topic 생성 및 정보 확인하기 실습

<img width="963" alt="image" src="https://user-images.githubusercontent.com/106303141/231134411-0facab40-3374-4bdd-8161-692d5955641e.png">

> topic명으로 '_'나 '.'을 동시에 사용하지 말라고 권장한다. 
> 지마켓에서는 '_'를 사용하여 구분한다.


## kafka-console-producer와 kafka-console-consumer를 이용한 실습

<img width="953" alt="image" src="https://user-images.githubusercontent.com/106303141/231138107-de6120ad-7538-45b2-a0d3-6b3212b2b339.png">


### Producer와 Consumer간의 Serialized Message 전송

<img width="1133" alt="image" src="https://user-images.githubusercontent.com/106303141/231138666-e9a00989-168c-4f31-8fc7-d220614c5e58.png">

실제로(물리적으로) Partition에는 Byte Array가 저장되어진다고 함.
DB, file, memeory 또한 마찬가지

앞으로 자바코드 Client 실습에서 Serialized를 많이 다루므로 미리 알려주신 듯


## Key값을 가지지 않는 메세지 전송

- Partitioner가 어떤 Partion에 전송되어야 할 지 미리 결정
- 라운드 로빈(kafka 2.4 이하 기본), 스티키 파티션(kafka 2.4 이상 기본) 등의 파티션 정략등이 선택됨
- **전송 순서가 보장되지 않은 채**로 Consumer에서 읽혀질 수 있음
> 전송 순서가 보장되어야 한다면, Partition을 1개로 사용하는 수 밖에 없다.

### Key값을 가지는 메세지 전송

- 특정 Key값을 가지는 메세지는 **특정 파티션으로 고정**되어 전송됨

![image](https://user-images.githubusercontent.com/106303141/231240791-038bf3c8-e9ae-42d4-9f1d-eff84cfa4b0c.png)


## Consumer Group과 Consumer

![image](https://user-images.githubusercontent.com/106303141/231245869-f7664311-3cb5-4b73-8281-d4252ee02e96.png)

- 모든 Consumer는 단 하나의 Consumer Group에 소속된다.
- Partition의 레코드들은 하나의 Consumer에만 할당한다.
- 보통 Partition 수 만큼의 Consumer를 만들어 할당 (분산처리 성능 향상)
- Consumer Group 내에 Consumer 변화(추가 혹은 삭제)가 있으면 **Rebalancing**(조합 변경)이 발생


![image](https://user-images.githubusercontent.com/106303141/231246276-4ddabc2d-9c14-4e86-9f2f-02c0edfc97cf.png)

- 서로 다른 Consumer Group의 Consumer들은 **분리되어 독립적으로 동작**

