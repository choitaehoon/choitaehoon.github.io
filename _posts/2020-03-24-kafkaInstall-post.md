---
title: "kafka 설치"
date: 2020-03-24 14:15:00 
---

kafka 설치

서버 운영을 3대 이상으로 해야 완벽한 카프카 클러스터가 되지만 테스트를 하기 위해 1대의 local 장비로만 구성했습니다. 환경은 mac, iterm2로 진행 했습니다. 

혹시 다수의 장비로 카프카 클러스터 설치 및 설정을 잡고 싶은 분은 https://blog.voidmainvoid.net/325를 들어가면 쉽게 구성 할 수 있습니다. **이 글도 또한 저 블로그를 참고 했습니다.**

#### /etc/hosts 설정 하기

/etc/hosts를 살펴보면 localhost가 default가 기본으로 잡혀 있다. 1대의 local 장비로만 구성 하면 따로 설정 안해도 되지만 몇대의 장비를 구성 하면 ip를 넣고 그 ip를 사용할 이름을 적어 주면 된다.

![images1](https://user-images.githubusercontent.com/33123391/77389176-c0e64700-6dd5-11ea-91ff-ce33305e3a6b.png)

#### zookeeper 설치

```java
wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz
```

다운 받은 후 에 아래의 명령으로 압축을 풀면 됩니다.

```java
tar xvf zookeeper-3.4.12.tar.gz
```

Zookeeper의 Configuration을 설정 해야 합니다. zookeeper 내부의 conf 폴더에 zoo.cfg파일을 생성 하고 아래와 같은 설정 파일을 넣어 줍니다.

```java
tickTime=2000
dataDir=zookeeper가 설치 된 곳
clientPort=2181
initLimit=20
syncLimit=5
server.1=localhost:2888:3888
```



이제 zookeeper를 실행 하면 됩니다.

```java
./bin/zkServer.sh start
```



#### kafka 설치

아래의 그림에 보이는 명령어로 설치 해줍니다.

```java
wget https://archive.apache.org/dist/kafka/2.1.0/kafka_2.11-2.1.0.tgz
```

또한 아래 명령어를 통해 압축을 풀어 줍니다.

```java
tar xvf kafka_2.11-2.1.0.tgz
```

Kafka  실행 하기 위해서 zookeeper와 같이 설정 해줘야 합니다. 폴더는 config/server.properties 입니다.

```java
broker.id=0
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://localhost:9092
zookeeper.connect=localhost:2181/test
```

zookeeper 마지막에 /test를 넣는 이유는 zookeeper의 root node 가 아닌 child node에 카프카 정보를 저장하므로 유지보수에 이득이 있습니다.

kafka를 실행 하는 명령어는 아래와 같습니다.

```java
./bin/kafka-server-start.sh ../config/server.properties
```



#### producer, consumer 테스트

```java
./bin/kafka-topics.sh --create --zookeeper localhost:2181/test \
  --replication-factor 1 --partitions 1 --topic test
```



명령어를 통해 테스트 해볼 수 있지만 저는 springboot로 값이 잘 들어 가는지 확인 해보겠습니다.

아래의 그림은 Producer 코드 입니다.

![producer](https://user-images.githubusercontent.com/33123391/77390272-d5780e80-6dd8-11ea-8200-3e6fcb048bbc.png)

아래의 그림은 consumer 코드 입니다.

![consumer](https://user-images.githubusercontent.com/33123391/77390313-f50f3700-6dd8-11ea-912d-38fbc331c95d.png)

그리고 실행 시켜 봤습니다. 잘 소비 되는 것을 아래의 그림 처럼 살펴 볼 수 있습니다.

![image4](https://user-images.githubusercontent.com/33123391/77390427-45869480-6dd9-11ea-9bba-b3a5ccee4774.png)



