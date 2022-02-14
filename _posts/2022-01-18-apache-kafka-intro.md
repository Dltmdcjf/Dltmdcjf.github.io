---
date: 2021-01-18
title: "Apache Kafka 이해하기"
categories: Kafka
---

# 1. Apache Kafka 이해하기

## 1.1. Apache Kafka의 초당 2백만 Writes 성능
[2 Million Writes Per Second](<https://engineering.linkedin.com/kafka/benchmarking-apache-kafka-2-million-writes-second-three-cheap-machines>)에서 확인하면
저렴한 장비를 사용하여 초당 2백만 건의 write를 할 수 있는 카프카의 성능을 확인할 수 있다.

## 1.2. Apache Kafka vs. RabbitMQ
[Apache Kafka vs. RabbitMQ](<https://www.confluent.io/blog/kafka-fastest-messaging-system/?utm_medium=sem&utm_source=google&utm_campaign=ch.sem_br.nonbrand_tp.prs_tgt.kafka_mt.mbm_rgn.apac_lng.eng_dv.all_con.kafka-rabbitMQ&utm_term=%2Bkafka%20%2Brabbitmq&creative=&device=c&placement=&gclid=CjwKCAjw3_KIBhA2EiwAaAAliiwLNAaE01pbjuDTGvedQ9G9qPJO-fyKJhHvGIxDq0s9RZ3oU2P9IRoCvT0QAvD_BwE>)에서 두 기술간의 Throughput과 Latency를 확인할 수 있다.
여유가 될때 아티클을 꼭 읽어보자. 


## 1.3. Kafka commit log
* Commit log : 추가만 가능하고 변경이 불가능한 데이터 구조. 데이터는 항상 마지막에 추가됨.   
* offset : commit log에서 데이터의 위치를 오프셋 이라고 한다.
  * Kafka Producer가 Write 하는 `LOG-END-OFFSET`
  * Kafka Consumer Group의 Consumer가 Read 하는 `CURRENT OFFSET`
  * Consumer가 Read 한 `CURRENT OFFSET`과 Producer가 write 한 `LOG-END-OFFSET`의 차이를 `Consumer Lag`라고 한다.


## 1.4. Topic과 Partition
![카프카 토픽](/assets/images/2022-01-18-apache-kafka-intro-01.png "Kafka topic, partition and segment")
<center><a href="https://medium.com/@binary10111010/apache-kafka-a-streaming-platform-57e38f8f9bc1">[출처] Topic, partition, Segment</a></center>
* Topic : Kafka 안에서 메세지가 저장되는 장소
* Partition : 하나의 Topic은 최소 한 개 이상의 파티션을 가진다.(Commit log가 결국 한 개의 파티션이다)
  * 카프카 토픽 생성 시 Partition을 지정해 줄 수 있다. 주어진 브로커에 최적화를 적용해 적절히 주어진 파티션들을 배치한다.
  * 토픽 내의 파티션들은 서로 독립적이며 파티션에 저장된 데이터는 변경불가하다.
* Segment : 실제 메세지 파일이 저장되는 단위는 세그먼트이다. 
  * Rolloing Strategy : log.segment.bytes (크기 기반으로 세그먼트를 쪼개기), log.roll.hours (시간 기반으로 세그먼트를 쪼개기)


## 1.5. Kafka Broker & Zookeeper

## 1.5.1 Kafka Broker
* Kafka Broker는 토픽 안에 있는 Partition들을 관리한다. 
* Kafka Broker는 여러 개의 Broker로 구성될 수 있으며 이를 Kafka Cluster로 통칭한다.
* Client가 특정 Kafak Broker에 접속하면 전체 Kafka Cluster에 연결되는 특징을 가진다.
* 장애를 대비해 Kafka 구성 시 최소 3대 이상의 브로커는 구성해야 한다.

## 1.5.2. Zookeeper
* Broker들을 목록과 설정을 유지 및 동기화 해주는 역할의 소프트웨어.
* Zookeepr는 Kafka Broker의 변경사항에 대해 Kafka에 알려준다.
  * 예를 들어, 특정 브로커의 토픽이 삭제 되면, 나머지 브로커들에게 해당 정보를 업데이트 해준다.
* Zookeeper는 Leader와 Follower로 구성된다. Leader에서 Broker에 대한 정보를 가지고 있으며 이를 Follower에 전달한 후 모든 브로커에 전달함으로써 동기화를 한다.
* Zookeeper 역시 최소 3대 혹은 5대로 구성해야 한다. 
  * 그 이유는 Quorum 알고리즘에 의해 의사를 결정하기 위한 종족수가 필요한데 짝수의 경우 2대가 장애로 죽은 경우 살아 남은 브로커와 죽은 브로커가 2대로 같기 때문에 
  의사 결정이 어렵다. 따라서 홀수 대의 브로커가 필요하다.

## 1.6. Producer & Consumer

### 1.5.1 Producer
* Producer의 구조는 Header + (key,value)의 형태로 구성된다.
  * Header : topic, partition, timestamp 등의 메타데이터
* Kakfa Producer는 Serializer를 거친 후 바이트 형태로 토픽에 전송된다.
* 사용자는 key와 value에 대해 Serializer를 특정한 후 send를 보내면 바이트 형태로 메세지가 전송된다.
그리고 partitioner에 의해 어느 partition으로 갈지 정해진 후 해당 토픽의 파티션으로 메세지가 전송된다.
* Partitioner는 키에 대해 해쉬값을 적용한 후 파티션 개수에 대한 나머지 값을 해당 파티션으로 사용한다.
> 만약 key가 null 인 경우 default-partitioner로 round-robin 방식으로 들어온 메세지를 파티션으로 분산시킨다. (2.4 버전이하)
> 2.4 버전 이후 부터는 sticky 정책을 사용하여 하나의 Batch가 닫힐 때까지 하나의 Partition에게 레코드를 보낸다.


<p align="center">
  <img src="/assets/images/2022-01-18-apache-kafka-intro-02.png" height ="30%" width="60%" alt="kafka default partitioner"/>
</p>
<p align="center">
    <a href="https://www.confluent.io/blog/apache-kafka-producer-improvements-sticky-partitioner/">[출처] sticky partitioner</a>
</p>


### 1.5.2. Consumer
* Consumer는 offset을 가지고 있어 이전에 읽었던 내용을 다시 읽는 것을 방지한다.
  * 카프카는 내부 토픽인 __consumer_offsets 이 있다. 이 내부토픽에 각 컨슈머 그룹의 컨슈머들이 앞으로 읽어드릴 offset 정보를 보관한다.
* 파티션은 항상 컨슈머 그룹내의 단 한 개의 컨슈머에 의해서만 사용된다.
* 다른 컨슈머 그룹의 컨슈머들은 서로 독립적으로 동작한다.

#### 1.5.2.1 메세지 순서 보장
* 카프카에서 만약 파티션이 2개 이상일 경우, 전체 메세지에 대한 순서 보장은 어렵다
  * 왜냐하면 파티션끼리는 순서가 보장되서 들어오지만 파티션들은 서로 독립적이기 때문에 전체 메세지에 대한 순서 보장을 하지 못한다.
  * 파티션 1개를 만들 경우, 모든 메세지에 대해 순서를 보장할 수 있지만, 파티션이 한 개 이기 때문에 처리량이 매우 떨어진다.
    * 파티션 입장에서는 무조건 한 개의 컨슈머가 할당된다.
    * 컨슈머 입장에서는 여러 개의 파티션으로 부터 메세지를 당겨 올 수 있다.
  * 만약 동일한 key를 가진 메세지는 동일한 파티션에 전달되기 때문에 이런 경우 키 레벨에 한해 순서가 보장된다. (키와 파티션에 대한 일대일 매핑)
  > 운영중에 절대 파티션 개수를 바꾸면 안된다. 왜냐하면 파티션 개수를 이용해 해쉬값으로 어떤 파티션으로 갈지 결정 하기 때문에 운영중에 바꾸면 메세지가 들어가는
  파티션이 바뀌게 되므로 순서 보장이 안된다.


## 1.7. Replication

* 카프카는 Partition을 복제하여 다른 브로커 상에 복제본을 유지함으로써 장애를 대비한다.
  * Replicas에는 Leader partition과 follow partition으로 구성된다. Leader 파티션에서 Producer, Consumer의 쓰기, 읽기가 진행되며 Follow는 장애 대비용이다.
  * 장애가 발생하면 Leader가 죽고 Follow 중에서 Leader를 선출한다.
  * 만약 한 브로커에 모든 파티션들이 리더로 구성 될 경우 해당 브로커에만 부하가 몰리는 경우가 발생한다. 이를 방지하기 위해 아래와 같은 옵션을 카프카에서 제공
    * `auto.leader.rebalance.enable : enable` : 한 브로커에 몰리지 않게 적절히 리발란싱 적용
    * `leader.imbalance.check.interval.seconds : 300s` : 리더의 불균형 체크 주기 시간
    * `leader.imbalance.per.broker.percentage : 10` : 전체 리더의 10%이상으로 한 브로커에 몰린 경우 리발랜싱 진행

## 1.8. In-Sync Replicas (ISR)
* ISR이란 High Watermark (성공적으로 메세지를 replicas에게 복제한 지점)까지 동일한 replicas의 목록을 의미한다.
  * 쉽게 이야기해서 ISR은 Leader가 장애가 나면 최대한 Leader와 비슷한 메세지 목록을 가지는 Replicas 중에서 새로운 Leader를 선출해야 데이터 유실이 적다. 이때, 이 후보 리스트가 바로 ISR이라고 생각하면 된다.
  * ISR은 `replica.lag.time.max.ms` 로 판단한다. 만약 10000으로 설정되어 있으면 10초 내에 Follow가 Leader로 Fetch 요청을 보내기만 하면 ISR이 되며, 그렇지 못한 브로커의 replicas들은 ISR이 되지 못한다. 
> Consumer는 반드시 Committed 메세지(High watermark)만 읽어 올 수 있다. Leader에 새로운 메세지가 쓰여졌더라도
다른 replicas의 high watermark된 지점까지만 Consumer들이 읽어 올 수 있다. 즉 모든 replicas는 leader든 follow든 관계없이 동일한 committed offset을 가지게 된다.
* 그리고 Broker가 다시 시작할 때 이전의 Committed의 메세지 목록의 유지를 위해 Broker의 모든 파티션의 Committed offset은 replication-offset-checkpoint에 파일로 기록하고 있다.

* 컨슈머는 4가지 오프셋이 있다.
  * Last committed Offset : 실제 Consumer가 읽고 마지막으로 커밋 위치
  * Current position : 현재 진행형으로 읽은 위치
  * High Watermark : 성공적으로 다른 모든 replicas에게 복제한 위치 (가장 최근의 committed offset 지점)
  * Log End Offset : Producer가 마지막으로 쓰기한 위치

<p align="center">
  <img src="/assets/images/2022-01-18-apache-kafka-intro-03.png" height ="40%" width="70%" alt="consumer offsets"/>
</p>
<p align="center">
    <a href="https://rongxinblog.wordpress.com/2016/07/29/kafka-high-watermark/">[출처] Highwater mark</a>
</p>



***** 
## References
