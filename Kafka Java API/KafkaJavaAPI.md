### Kafka Java API
-
- gradle추가
```
compile 'org.apache.kafka:kafka_2.10:0.8.1.0'
```

- 실행
```
(1) zookeeper 서버 실행
bin/zookeeper-server-start.sh config/zookeeper.properties

(2) kafka 서버 실행
bin/kafka-server-start.sh config/server.properties

(3) Topic생성
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

(4) producer 실행
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

(5) consumer 실행
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
```

- /config/server.properties
```
advertised.host.name=127.0.0.1 //추가
```

#### Kafka Producer Java API
- Partitioning: roundRobin
   ```
   public class KafkaProducer implements Partitioner {

   	private Producer<String, String> producer;

   	private Properties props;
   	private ProducerConfig producerConfig ;


   	public KafkaProducer(String brokerList){

   		props = new Properties();
   		props.put("metadata.broker.list", brokerList);
   		props.put("serializer.class", "kafka.serializer.StringEncoder");
   		props.put("request.required.acks","1");

   		producerConfig = new ProducerConfig(props);
   		producer = new Producer<String, String>(producerConfig);
   	}

   	public Producer<String, String> getInstance(){
   		return this.producer;
   	}

   	// Partitioning : RoundRobin
   	private AtomicInteger n = new AtomicInteger(0);
   	@Override
   	public int partition(Object key, int numPartitions) {
           int i = n.getAndIncrement();
           if (i == Integer.MAX_VALUE) {
               n.set(0);
               return 0;
           }
           return i % numPartitions;
       }

   }

   ```

   ```
   KafkaProducer producer = new KafkaProducer("localhost:9092");

	String topic = "test";
	String message = "HelloWorld";

	// simple message
	KeyedMessage<String, String> data = new KeyedMessage<String, String>(topic, message);
	producer.getInstance().send(data);

	// multi message
	List<KeyedMessage<String, String>> messages = new ArrayList<KeyedMessage<String, String>>();
	for (int i = 0; i < 10; i++) {
		messages.add(new KeyedMessage<String, String>("test", "Hello, World!"));
	}		
	producer.getInstance().send(messages);
	producer.getInstance().close();
   ```
#### Kafka consumer Java API
- Simple Consumer API
- High Level Consumer API

   ```
   public class KafkaConsumer {

   	private ConsumerConnector consumer;

   	private Properties props;
   	private ConsumerConfig consumerConfig ;


   	public KafkaConsumer(String brokerList){

   		props = new Properties();
   		props.put("group.id", "test-group");
   		props.put("zookeeper.connect", brokerList);
   		props.put("auto.commit.interval.ms", "1000");

   		ConsumerConfig consumerConfig = new ConsumerConfig(props);
   		ConsumerConnector consumer = Consumer.createJavaConsumerConnector(consumerConfig);
   	}

   	public ConsumerConnector getInstance(){
   		return this.consumer;
   	}
   }
   ```

   ```
   KafkaConsumer consumer = new KafkaConsumer("localhost:9092");
	String TOPIC = "test";
	int NUM_THREADS = 20;

	Map<String, Integer> topicCountMap = new HashMap<String, Integer>();
	topicCountMap.put(TOPIC, NUM_THREADS);
	Map<String, List<KafkaStream<byte[], byte[]>>> consumerMap = consumer.getInstance().createMessageStreams(topicCountMap);
	List<KafkaStream<byte[], byte[]>> streams = consumerMap.get(TOPIC);
	ExecutorService executor Executors.newFixedThreadPool(NUM_THREADS);
	for (final KafkaStream<byte[], byte[]> stream : streams) {
		executor.execute(new Runnable() {
			@Override
			public void run() {
				for (MessageAndMetadata<byte[], byte[]> messageAndMetadata : stream) {
					System.out.println(new
               String(messageAndMetadata.message()));
				}
			}
		});
	}
	try {
		Thread.sleep(60000);
	} catch (InterruptedException e) {
		e.printStackTrace();
	}
	consumer.getInstance().shutdown();
	executor.shutdown();
   ```
