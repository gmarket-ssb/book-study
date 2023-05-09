# Java 기반 Producer 구현 실습 및 내부 매커니즘 이해

> 환경: jdk11, gradle, intellij 

1. Producer 환경 설정(```Properties``` 객체 이용)
2. 환경 설정을 반영한 ```KafkaProducer``` 객체 생성
3. 토픽 명과 메세지(Key, Value)를 입력하여 보낼 메세지인 ```ProducerRecord``` 객체 생성

```java
public class SimpleProducer {
    public static void main(String[] args) {

        String topicName = "simple-topic";

        //KafkaProducer configuration setting
        // null, "hello world"

        Properties props  = new Properties();
        //bootstrap.servers, key.serializer.class, value.serializer.class
        //props.setProperty("bootstrap.servers", "192.168.56.101:9092");
        props.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.56.101:9092");
        props.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        //KafkaProducer object creation
        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<String, String>(props);

        //ProducerRecord object creation
        ProducerRecord<String, String> producerRecord = new ProducerRecord<>(topicName,"hello world 2");

        //KafkaProducer message send
        kafkaProducer.send(producerRecord);


        kafkaProducer.flush();
        kafkaProducer.close();
    }

}
```

### Producer Java 클라이언트

![image](https://github.com/gmarket-ssb/book-study/assets/106303141/b6049fec-a3f6-4ccd-b1ee-692871fd14d1)

1. 기본적으로 Thread간 Async 전송
2. 즉, Main Thread가 ```send()``` 메소를 호출하여도 바로 시작하지 않고, 내부 Buffer에 저장 후 별도의 Thread가 Kafka Broker에 전송하는 방식

> 지마켓 Java + Kafka 가이드: https://wiki.ebaykorea.com/display/DataPlatform/Kafka+Java+Client

### Producer와 브로커와의 메세지 동기화/비동기화 전송

#### Sync(동기 방식)

* Producer가 브로커로부터 Ack 메세지를 받을 때 까지 **대기**함
* ```KafkaProducer.send().get()```

#### Async(비동기 방식)

* Producer가 브로커커에게 무조건 전송
* callback 객체를 인자로 ack 메세지를 producer로 전달 받을 수 있음

![image](https://github.com/gmarket-ssb/book-study/assets/106303141/2d00c2db-cd11-4010-bc04-937e76232da5)

##### callback 호출 예시

```java
//kafkaProducer message send
kafkaProducer.send(producerRecord, (metadata, exception)-> {
    if (exception == null) {
        logger.info("\n ###### record metadata received ##### \n" +
                "partition:" + metadata.partition() + "\n" +
                "offset:" + metadata.offset() + "\n" +
                "timestamp:" + metadata.timestamp());
    } else {
        logger.error("exception error from broker " + exception.getMessage());
    }
});
```

### Key값을 가지는 메세지 전송 구현

1. 특정 Key값을 가지는 메세지는 **특정 파티션으로 고정되어 전송됨**
2. 특정 Key값을 가지는 메세지는 단일 파티션 내에서 전송 순서가 보장

![image](https://github.com/gmarket-ssb/book-study/assets/106303141/676f01b5-d9c1-487d-b4fc-29e72570436c)

##### topic partitions 생성
```
kafka-topics --bootstrap-server localhost:9092 --create --topic multipart-topic --partitions 3
```

##### topic partitions 확인
```
kafka-topics --bootstrap-server localhost:9092 --describe --topic multipart-topic
```

##### 키값 메세지 전송 실행로직
```java
for(int seq=0; seq < 20; seq++) {
    //ProducerRecord object creation
    ProducerRecord<String, String> producerRecord = new ProducerRecord<>(topicName, String.valueOf(seq),"hello world " + seq);
    logger.info("seq:" + seq);
    //kafkaProducer message send
    kafkaProducer.send(producerRecord, (metadata, exception) -> {
        if (exception == null) {
            logger.info("\n ###### record metadata received ##### \n" +
                    "partition:" + metadata.partition() + "\n" +
                    "offset:" + metadata.offset() + "\n" +
                    "timestamp:" + metadata.timestamp());
        } else {
            logger.error("exception error from broker " + exception.getMessage());
        }
    });
}
```
