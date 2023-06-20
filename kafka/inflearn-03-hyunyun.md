# Java 기반 Consumer 구현 실습 및 내부 매커니즘 이해

1. 브로커의 Topic 메세지를 읽는 역할을 수행
2. Consumer는 group.id를 가지는 Consumer Group에 소속되어야 함.
3. Consumer Group의 여러개의 Consumer들은 토픽 파티션 별로 분배됨

![image](https://github.com/gmarket-ssb/book-study/assets/106303141/f74fe50d-f94a-4fb5-891d-67e4dea08604)

## subscribe, poll, commit 로직

* subscribe: 읽어들이려는 토픽 등록
* poll: 브로커의 토픽 파티션에서 메세지를 가져옴
* commit: 다음에 읽을 offset 위치를 기재

![image](https://github.com/gmarket-ssb/book-study/assets/106303141/c3acc474-addb-49ae-b1c2-39ec36fe6e90)

## 내장 객체 및 Heartbeat 

### 내장객체
1. Fetcher
2. ConsumerClientNetwork
> 메세지를 Fetch 및 Poll 수행

#### KafkaCounsumer.poll(1000)으로 수행 시

![image](https://github.com/gmarket-ssb/book-study/assets/106303141/b917a402-8daa-4859-894c-68223123ee58)

* Linked Queue에 데이터가 있을 경우 Fetcher는 데이터를 가져오고 반환하며 poll() 수행 완료
* ConsumerNetworkClient는 비동기로 게속 브로커의 메세지를 가져와서 Linked Queue에 저장
* Linked Queue에 데이터가 없을 경우 1000ms 까지 Broker에 메세지 요청 후 poll() 수행 완료

### Heartbeat
* KafkaConsumer는 Heart Beat Thread가 를 생성하며 Consumer의 정상적인 활동을 Group Coordinator에 보고
* Group Coordinator는 주어진 시간동안 Heart Beat를 받지 못하면 Consumer들의 Rebalance를 수행

![image](https://github.com/gmarket-ssb/book-study/assets/106303141/762caf52-8daa-4d9a-8458-b0f156ecc24d)


## Java 기반 Consumer 구현

```Java
public static void main(String[] args) {

    String topicName = "simple-topic";

    Properties props = new Properties();
    props.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.56.101:9092");
    props.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    props.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    props.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "group-01");

    KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<String, String>(props);
    kafkaConsumer.subscribe(List.of(topicName));

    while (true) {
        ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration.ofMillis(1000));
        for (ConsumerRecord record : consumerRecords) {
            logger.info("record key:{}, record value:{}, partition:{}",
                    record.key(), record.value(), record.partition());
        }
    }

    //kafkaConsumer.close();

}
```

![image](https://github.com/gmarket-ssb/book-study/assets/106303141/1a9cc0b1-fafb-48c7-aac4-14656baa9fd6)

## Wakeup을 이용하여 Consumer 효과적으로 종료하기

* KafkaConsumer.wakeup() 메서드를 통해 WakeupException 을 throw 할 수 있음.
* try catch 구문을 통해 WakeupException이 발생하면 close()를 수행하는 원리

```Java
  //main thread
  Thread mainThread = Thread.currentThread();
  
  //main thread 종료시 별도의 thread로 KafkaConsumer wakeup()메소드를 호출하게 함.
  Runtime.getRuntime().addShutdownHook(new Thread() {
      public void run() {
          logger.info(" main program starts to exit by calling wakeup");
          kafkaConsumer.wakeup();

          try {
              mainThread.join();
          } catch(InterruptedException e) { e.printStackTrace();}
      }
  });

  try {
      while (true) {
          ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration.ofMillis(1000));

          for (ConsumerRecord record : consumerRecords) {
              logger.info("record key:{},  partition:{}, record offset:{} record value:{}",
                      record.key(), record.partition(), record.offset(), record.value());
          }
      }
  }catch(WakeupException e) {
      logger.error("wakeup exception has been called");
  }finally {
      logger.info("finally consumer is closing");
      kafkaConsumer.close();
  }
```

