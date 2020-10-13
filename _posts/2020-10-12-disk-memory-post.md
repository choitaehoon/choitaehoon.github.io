---
title: 인스타그램과 같은 SNS 플랫폼은 어떻게 많은 트래픽을 처리할까? - 3편(Disk database vs In-Memory database)
date: 2020-10-12 14:15:00 
---

2편에서 Session Storage를 구축하여 데이터 정합성 이슈를 해결할 수 있었고 하나의 서버가 장애 나더라도 다른 서버에서 연속적으로 제공할 수 있었습니다.
3편에서 세션 저장소는 어떠한 데이터베이스가 적합한지 알아보겠습니다.

### 디스크 기반 데이터베이스는 세션 데이터를 처리하기에 너무 느려!

![image](https://user-images.githubusercontent.com/33123391/95710640-0ac44b00-0c9c-11eb-8968-8eb47e7fafb0.png)

디스크 기반의 데이터베이스는 mysql, oracle, mssql등 여러 종류가 존재합니다. 이러한 데이터베이스는 전원이 꺼지더라도 저장된 정보가 그대로 보존되며 용량이 크다는 장점이 있지만
다른 저장소와 달리 속도가 매우 느리다는 단점이 존재합니다. 이러한 단점을 극복하고자 SSD를 많이 사용하지만 Ram에 비해서는 역시 속도가 떨어집니다.
위의 그림을 살펴보면 Disk와 In-Memory의 TPS는 뚜렷하게 차이 납니다. 

Session Storage는 세션 데이터에 대한 정보를 가져오기 위해 Select 작업이 잦습니다. 만일 디스크 기반 데이터베이스를 선택하게 된다면 
장치 간의 속도 차이로 병목현상이 발생할 수 있습니다. 이러한 이유로 Session Storage는 In-Memory Database가 더 적합하다고 생각했습니다.

### 인 메모리 데이터베이스는 괜찮을까?

인 메모리 기반의 데이터베이스는 디스크가 아닌 주 메모리에 데이터를 보유하고 있어 응답 속도가 빠르고 데이터의 구조를 유연하게 변동 가능합니다. 하지만 컴퓨터 전원이 꺼지면
데이터가 손실이 발생할 수 있습니다. 

**그렇다면 인 메모리 데이터베이스는 부적합하지 않을까?**

![image](https://user-images.githubusercontent.com/33123391/95737512-c304ea00-0cc2-11eb-90b6-faac9bd8cae2.png)

결론부터 말하자면 인 메모리 데이터베이스는 Failover, Sharding 방식을 제공하여 데이터 손실을 방지합니다. 
또한 세션은 반영구적으로 사용하기 때문에 인 메모리 데이터베이스는 적합합니다.

그렇다면 어떠한 In-Memory Database를 선택해야 하는지 다음 편에서 살펴보겠습니다. 

참고 문서
* https://devopedia.org/in-memory-database
* https://blog.intelligentbee.com/2017/05/22/emulating-redis-failover-docker/