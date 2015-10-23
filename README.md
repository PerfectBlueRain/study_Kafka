### Kafka
- Kafka 쓰임새 및 사용방법
- Kafka 내부구조 & 동작원리 / Zookeeper
   - producer, consumer, broker
   - Topic & Partition
   - Replication

   - 메세징 모델 : Queue / pub-sub모델 : Consumer Group
   - File System을 활용한 고성능 모델

---

- Kafka설치 및 Cluster 운영
- Kafka CLI
- Kafka Java API


---

(1) fault-tolerant
-  장애가 나더라도 동작을 한다는 의미다. Kafka는 이를 위해서 hadoop에서 즐겨 사용하는 Zookeeper를 장애 탐지 및 복구 시스템으로 사용한다. 그런데 ZooKeeper 방식은 production 구성인 경우, 적어도 3대 이상의 잉여 시스템이 필요로 한다. Zookeeper에 연결된 각 시스템은 주기적으로 heart-beat을 주고 받는다. 그러다가 heat-beat을 주지 않는 시스템이 있는 경우, ZooKeeper는 이를 장애라고 판단하여 configuration에 설정된 동일한 기능을 실행하는 시스템으로 hot switching를 하는 것이다.
(2) distributed
-  distribute, 즉 Kafka Cluster는 Kafka가 cluster 구성의 분산 메시징 시스템이란 의미를 보이고 있다. - 분산 처리 시스템들은 서버가 여러대 있고, 보통 이들을 서버 덩어리, 즉 cluster
(3) pub/sub
-  message queue란 기본적으로 저장소이며, 이 저장소에 데이터를 저장하는 부분과, 데이터가 저장되면 이를 끄집어 내어 사용하는 3 부분, 즉 저장소, 데이터 생성자, 데이터 소비자 등 3개의 역할 담당자들이 있게 된다. Kafka Cluster는 저장소의 역할을 한다. 그림에서 Producer가 데이터 생성자의 역할, Consumer가 데이터 소비자의 역할이다. 그리고, Message Queue에서 데이터는 조금 특별한 형태를 가지며, 통상적으로 데이터란 용어 대신 Message란 용어를 사용

-  Message Queue에서 보통 소비자(Consumer)는 자신이 소비할 Message가 있는지를 주기적으로 알아보는 polling 방식을 취하지 않는다. 시스템 자원을 매우 많이 소비하는 방식이기 때문이다. 보통은 메시지 저장소가 메시지가 생기면, Consumer에게 이벤트로 알려준다. 이런 방식을 (3)번의 의미인 pub/sub 즉 publisher와 subscriber 방식
(4) messaging system
- 이에 따라 Kafka는 설치는 간단하지만, 동작을 시키려면 ZooKeeper, 메시지 저장소 역할을 하는 kafka server, 그리고 message를 생성하는 producer, 마지막으로 message를 소비하는 consumer 등 네 가지 부분들이 동작
