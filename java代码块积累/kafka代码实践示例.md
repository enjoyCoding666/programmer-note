### 安装kafka：

Windows安装kafka, 详情见：https://blog.csdn.net/sinat_32502451/article/details/133067851

Linux 安装kafka，详情见：https://blog.csdn.net/sinat_32502451/article/details/133080353



### 添加依赖包：

```
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
            <version>2.1.10.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>2.0.0</version>
        </dependency>
```



### kafka配置：

在实际开发中，会有多种不同的消息，服务器也不一定一样。需要根据不同的需求，进行不同的配置。

* KafkaConfig：

kafka 配置类。如下：

```
@Configuration
@EnableKafka
public class KafkaConfig {

    @Value("${bootstrap.servers:127.0.0.1:9092}")
    private String servers;

    @Value("${batch.size:16384}")
    private Integer batchSize;

    @Value("${buffer.memory:33554432}")
    private Integer bufferMemory;

    @Value("${group.id:myGroup}")
    private String consumerGroupId;

    @Value("${auto.commit.interval.ms:100}")
    private String commitInterval;

    @Value("${session.timeout.ms:15000}")
    private String sessionTimeout;


    /**
     * 想直接操作kafka发送消息可以用 kafkaTemplateService 注入
     */
    @Bean("kafkaTemplateService")
    public KafkaTemplate<String, String> kafkaTemplateService() {
        return new KafkaTemplate<>(producerFactory());
    }

    /**
     * 生产者 factory
     */
    @Bean
    public ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    /**
     * 生产者配置
     * @return
     */
    @Bean
    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>();
        //服务器ip和端口，多个用逗号隔开
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, servers);
        props.put(ProducerConfig.RETRIES_CONFIG, 0);
        //批量处理个数
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, batchSize);
        //等待时间
        props.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, bufferMemory);
        //序列化
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return props;
    }

    /**
     * 消费者 factory
     *
     */
    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }


    /**
     * 消费者配置
     *
     */
    private Map<String, Object> consumerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, servers);
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
        //消费者群组id
        props.put(ConsumerConfig.GROUP_ID_CONFIG, consumerGroupId);
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, commitInterval);
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, sessionTimeout);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return props;
    }

    /**
     * kafka监听器工厂，使用 @KafkaListener 注解时可以指定，比如  containerFactory = "kafkaListenerContainerFactory"
     *
     * @return
     */
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(5);
        return factory;
    }


}

```





### 生产者代码：

* bean对象：

```
public class MyMsg {

    private String id;
    
    private String name;
    
	//忽略getter、setter
}
```





* KafkaConfigProducerService：

```
@Service
public class KafkaConfigProducerService {

    @Resource(name = "kafkaTemplateService")
    private KafkaTemplate<String, String> kafkaTemplateService;


    public void send()  {
        MyMsg myMsg = new MyMsg();
        myMsg.setName("xu");
        myMsg.setId("5678");

        //发送消息
        kafkaTemplateService.send("myTopic", JSON.toJSONString(myMsg));

    }

}
```





### 消费者代码：

```
@Service
public class KafkaConfigConsumerService {


    /**
     * Kafka监听器，可以监听消息。
     * 指定需要监听的 kafka 主题 topics，可以是多个topic.
     * 指定消费者群组 groupId，可以不写.
     * 消费者的配置，在对应的 KafkaConfig类的 containerFactory 里面。
     *
     */
    @KafkaListener(containerFactory = "kafkaListenerContainerFactory",
            topics = {"myTopic"} )
    public void consume(ConsumerRecord<String, String> consumerRecord)  {
        System.out.println("消费者接收到信息,内容为:" + consumerRecord.value());
        System.out.println("偏移量：" +  consumerRecord.offset());

    }


}
```







### 测试结果 ：

调用生产者发送消息，消费者成功接收到消息，类似如下：

```
消费者接收到信息,内容为:{"id":"5678","name":"xu"}
偏移量：1
```








