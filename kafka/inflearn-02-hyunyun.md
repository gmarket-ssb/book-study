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

### Producer Java 클라이언트 API 내부 동작원리


