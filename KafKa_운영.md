
### Kafka standalone 설치
- http://yehongj.tistory.com/m/post/48

wget https://archive.apache.org/dist/kafka/0.8.1/kafka_2.10-0.8.1.tgz

동작 확인
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

### Kafka 모니터링
- http://epicdevs.com/22
- Kafka는 기본적으로 JMX 인터페이스를 제공하기 때문에 Kafka에서 제공하는 MBean(managed bean)들의 값을 모니터링할 수 있다. 하지만 JMX 툴보다는 Kafka 전용으로 개발된 모니터링 툴을 사용하는 것이 훨씬 간편하고, 중요한 정보들을 좀 더 직관적으로 파악할 수 있기 때문
- Kafka Offset Monitor : http://quantifind.github.io/KafkaOffsetMonitor/
   ```
   #!/bin/bash
   java -cp KafkaOffsetMonitor-assembly-0.2.0.jar \
        com.quantifind.kafka.offsetapp.OffsetGetterWeb \
        --zk 172.21.4.201:2181,172.21.4.202:2181,172.21.4.203:2181/kafka-1 \
        --port 8080 \
        --refresh 1.minutes \
        --retain 1.day &

   zk the ZooKeeper hosts
   port on what port will the app be available
   refresh how often should the app refresh and store a point in the DB
   retain how long should points be kept in the DB
   dbName where to store the history (default 'offsetapp')
   ```


#. test dev00  (ssh -i ids-dev.pem ids@175.126.56.165 )
모니터링 : http://175.126.56.165:8080/#/



### Kafka Cluster
- 분산으로 설치된 경우는 zookeeper  ip만 여러 개 적어주면된다.
- http://tfcloud.blogspot.kr/2013/07/apache-kafka-0.html
- http://epicdevs.com/20

1:175.126.56.165:9092,2:175.126.56.166:9092 # Logstash 분산으로 설치하는 경우
175.126.56.165:2181,175.126.56.166:2181 # zookeper 클러스터일때

#### zookeeper 서버 실행
- /tmp/zookeeper/myid :유일한 숫자로지정
   - zookeeper의 myid설정밑 zookeeper Cluster를 위한 추가설정 / Cluster구축시에 myid의값과 server.#의 값이 값아야 한다.
- /config/zookeeper.properties 설정
   ```
   # the directory where the snapshot is stored.
   dataDir=/tmp/zookeeper          # 이위치밑에 myid파일을 만들고 그 안에 유니크한 숫자값을 넣어야한다
   # the port at which the clients will connect
   clientPort=2181
   # disable the per-ip limit on the number of connections since this is a non-production config
   maxClientCnxns=0

   ###### Added ############  # 인스턴스 간 통신을 할 때 필요한 설정을 추가
   #initLimit=10
   #syncLimit=3
   #server.1=175.126.56.165:2888:3888   #  server.#에서 #은 인스턴스의 ID
   #server.2=175.126.56.166:2888:3888
   ```


#### kafka 서버 실행
- /config/server.properties
```
############################# Server Basics #############################
broker.id=0
############################# Socket Server Settings #############################
port=9092
host.name=localhost
socket.request.max.bytes=104857600

############################# Log Basics #############################
log.dirs=/tmp/kafka-logs      # 로그파일 저장위치 (로그파일이 안 생기거나 기동이 제대로 안되면 확인)
num.partitions=1
num.recovery.threads.per.data.dir=1
############################# Log Flush Policy #############################

############################# Log Retention Policy #############################
log.retention.hours=24              # 메시지의 수명. 수명이 지나면 메시지가 삭제
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000

############################# Zookeeper #############################
#zookeeper.connect=localhost:2181  
zookeeper.connect=175.126.56.165:2181,175.126.56.166:2181/cluster01  # 클러스터구축시

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=6000

############################ Added  #############################
auto.create.topics.enable=true
delete.topic.enable=true
default.replication.factor=2
message.max.bytes=1000000
```

(1) Zookeeper 서버실행
- bin/zookeeper-server-start.sh config/zookeeper.properties

(2) Kafka 서버실행
- bin/kafka-server-start.sh config/server.properties

#### Topic
   ```
   %> bin/kafka-topics.sh --zookeeper localhost:218/cluster01 --create --topic alarm_service --partitions 1 --replication-factor 1
   %> bin/kafka-topics.sh --zookeeper localhost:218/cluster01 --create --topic alarm_system --partitions 1 --replication-factor 1

   %> bin/kafka-topics.sh --zookeeper localhost:218/cluster01 --create --topic access_log --partitions 3 --replication-factor 2
   %> bin/kafka-topics.sh --zookeeper localhost:218/cluster01 --delete --topic access_log
   %> bin/kafka-topics.sh --zookeeper localhost:218/cluster01 --describe --topic access_log
   %> bin/kafka-topics.sh --zookeeper localhost:218/cluster01 --list
   ```

#### producer & consumer test 실행
   ```
   %> bin/kafka-console-producer.sh --broker-list 175.126.56.165:2181,175.126.56.166:2181 --topic access_log
   %> bin/kafka-console-consumer.sh --zookeeper 175.126.56.165:2181,175.126.56.166:2181 --topic test --from-beginning
   %> bin/kafka-console-consumer.sh --zookeeper 175.126.56.165:2181,175.126.56.166:2181 --topic crash_log
   ```

#### Partition

- Partition
   ```
   %> bin/kafka-topics.sh --zookeeper 175.126.56.165:2181,175.126.56.166:2181/cluster01 --alter --topic access_log --partition 40  //(파티션추가)
   ```

- Reassign Partition
   ```
   %> bin/kafka-reassgin-partitions.sh --zookeeper localhost:2181/cluster01 --topics-to-move-json-file topics-to-move.json --broker-list "0,1"  --generate //
   cat topics-to-move.json
   {
    "topics" : [
    	{ "topic" : "access_log" }
    ],
    "version" : 1
   }

   %> bin/kafka-reassgin-partitions.sh --zookeeper localhost:2181/cluster01 --reassignment-json-file expand-cluster-reassignment.json --execute
   cat expand-cluster-reassignment.json
   {
      "partitions" : [
         { "topic" : "access_log", "partition" : 0, "replicas" : [1] },
         { "topic" : "access_log", "partition" : 2, "replicas" : [1] },
         { "topic" : "access_log", "partition" : 1, "replicas" : [0] }
      ],
      "version" : 1
   }

   %> bin/kafka-reassgin-partitions.sh --zookeeper localhost:2181/cluster01 --reassignment-json-file increase-replication-factor.json --execute
   {
      "partitions" : [
         { "topic" : "access_log", "partition" : 0, "replicas" : [0,1] },
         { "topic" : "access_log", "partition" : 1, "replicas" : [0,1] },
         { "topic" : "access_log", "partition" : 2, "replicas" : [0,1] }
      ],
      "version" : 1
   }
   ```


### kafka 설정
- Apache Kafka Configuration : http://kafka.apache.org/documentation.html#configuration
- zookeeper.properties
- server.properties
- producer.properties
- consumer.properties
