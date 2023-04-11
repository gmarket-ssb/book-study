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




