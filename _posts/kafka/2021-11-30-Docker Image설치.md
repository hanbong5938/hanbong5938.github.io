---
layout : post
title : "Docker Image이용한 설치"
category : Kafka
---

### **Kafka-Docker 레파지토리 클론**

```
git clone https://github.com/wurstmeister/kafka-docker
```

### **docker-compose.yml 파일 수정**

분산 환경은 고려하지 않고 우선 싱글 사용한다.

docker-compose.yml 파일을 이용하는 것이 아니라, docker-compose-single-broker.yml를 이용

로컬 호스트 사용 위해 docker-compose-single-broker.yml 수정

```
...
KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
...
```

### **docker-compose 실행**

```
docker-compose -f docker-compose-single-broker.yml up -d
```

-f 옵션은 docker-compose.yml 다른 이름을 가진 docker-compose를 실행시킬 경우 사용

### **로컬에 Kafka 설치**

```
$ wget http://mirror.navercorp.com/apache/kafka/2.4.1/kafka_2.12-2.4.1.tgz
$ tar xzvf kafka_2.12-2.4.1.tgz
```

### **TOPIC 생성**

```
$ cd kafka_2.12-2.4.1
$ bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test_topic
```

### **Producer 실행**

```
$ bin/kafka-console-producer.sh --topic test_topic --broker-list localhost:9092
```

위와 같이 실행시키고 나면 메세지 전송 대기 상태로 변경된다.

메세지를 타이핑하기 전에 Consumer를 띄워보자

### **Consumer 실행**

Consumer는 터미널창을 하나 새로 띄어 아래와 같이 실행시켜준다.

```
$ bin/kafka-console-consumer.sh --topic test_topic --bootstrap-server localhost:9092 --from-beginning
```

### **Producer에서 메세지 전송 후 Consumer에서 확인**

다시 Producer를 실행시킨 터미널 창으로 돌아가서 아무거나 타이핑 해본다.

그러면 Consumer 터미널 창에서 해당 메세지를 받은 것을 확인할 수 있다.
