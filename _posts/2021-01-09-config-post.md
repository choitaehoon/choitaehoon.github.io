---
title: 스프링 클라우드 환경에서 어플리케이션의 설정값들을 어떻게 관리하면 좋을까 - (Spring Cloud Config vs Spring Cloud Netflix vs Spring Cloud Consul)
date: 2021-01-09 14:03:00
---

### 배경

현재 진행하고 있는 프로젝트의 설정 값들을 분리하고자 `Spring Cloud Config`에 대해 찾아 보던 중
`Twelve-Factor App`을 접했습니다. 이 방법론 중 Config 부분을 살펴보면 
`모든 설정 정보는 코드로부터 분리된 공간에 저장되어야 하고
런타임에서 코드에 의해 읽히며 SaaS는 동일한 코드를 여러 환경(운영/개발)에 배포한다.`로 설명되어 있습니다.

현재 프로젝트에서는 애플리케이션 안에 설정 정보가 있으며 배포된 상황에서 설정이 변경된다면 어플리케이션에
함께 패키징되어 있기 때문에 재배포해야 하는 구조입니다 이러한 문제점을 해결하고자 검색해서 알아본 결과 `Spring Cloud Config`,
`Spring Cloud Netflix`, `Spring Cloud Consul`있다는 것을 파악했습니다.

그렇다면 이러한 기술 중 어떤 것이 저희 프로젝트에 알맞을지 파악해보겠습니다.

### Spring Cloud Config

![Untitled Diagram](https://user-images.githubusercontent.com/33123391/104173457-254bc300-5449-11eb-9a2a-8c550462a1f1.png)

`Spring Cloud Config`는 분산 시스템에서 설정파일을 외부로 분리하는 것을 지원해주고 중앙에서
관리할 수 있습니다. 동작 방식은 Git 또는 SVN에 환경설정 정보를 저장하고 Config Server에서 정보를 읽어 온 뒤에
각각의 서버는 Config Server에 접근해서 정보를 읽어옵니다.

##### `Spring Cloud Config` 장점

수 많은 Application의 설정 파일을 한 곳에서 관리 할 수 있으며 만일 설정이 바뀔때 `{도메인}/actuator/refresh`로 API를
호출하면 컴파일을 따로 다시 할필요가 없어 재배포하지 않아도 됩니다. 또한 Spring Boot Application에서 `@EnableConfigServer`
선언하면 쉽게 임베드 할 수 있습니다.

##### `Spring Cloud Config` 단점

만일 설정값을 변경한다면 수동으로 `{도메인}/actuator/refresh` API를 호출해야 바뀐 설정값들을 인식할 수 있으며
Config 서버에 의존하는 Application이 많다면 모든 Application을 API 호출해줘야 하기 때문에
유지보수가 까다로워 집니다.

이러한 문제점을 해결하고자 Spring Cloud에서는 `Spring Cloud Bus`를 제공하여 `Spring Cloud Config`와 함께
사용합니다.

### Spring Cloud Netflix

`Spring Cloud Netflix`는 Spring Boot앱에 대한 Netflix OSS 통합을 제공하고 
`Service Discovery (Eureka), Circuit Breaker (Hystrix), Intelligent Routing (Zuul), Client Side Load Balancing (Ribbon),
Archaius (External Configuration)` 으로 대규모 분산 시스템을 구축할 수 있습니다.

저의 관심사는 `Archaius`이기 때문에 이 모듈을 중점적으로 살펴보겠습니다

![Spring Cloud Netflix](https://user-images.githubusercontent.com/33123391/104430560-2bb57880-55ca-11eb-8f79-8b2ac8d2761d.png)

`Spring Cloud Netflix`의 `Archaius`는 설정 값들을 외부 환경으로 분리할 수 있고 하나 이상의 Configuration
 Source를 지원합니다(JDBC, REST, .properties 등). 또한 주기적으로 폴링 할 수 있어 따로 API를 호출하지 않고
 바뀐 설정값들을 읽을 수 있습니다.
 
`Spring Cloud Config`와 비교해보면 설정값이 바뀌어도 API를 따로 호출해도 되지 않아도 되며 `Spring Cloud Bus`
와 같이 시스템을 확장하지 않아도 `Archaius`는 자체적으로 해결하고 있습니다. 그렇다면 `Spring Cloud Netflix`를
사용하는게 좋을까?

##### 유지 관리 모드로 전환
`Spring Cloud Netflix`의 대부분의 모듈이 유지 관리 모드로 전환 되었는데 이 말은 더 이상 모듈에 
새로운 기능을 추가 하지 않음을 의미합니다. 다른 기술들은 더 많은 기능들을 제공하기 위하여 끊임 없이 발전하지만
유지 관리 모드로 전환된 기술들은 버그 및 보안 문제를 수정하기만 한다면 나중에 뒤쳐질 수 밖에 없습니다. 
이러한 이유로 `Spring Cloud Netflix`는 고려 대상에서 제외시켰습니다.

아래의 있는 모듈들이 유지 관리 모드로 전환되었습니다
1. Spring-Cloud-Netflix-Archaius
2. Spring-Cloud-Netflix-Hystrix-Contract
3. Spring-Cloud-Netflix-Hystrix-Dashboard
4. Spring-Cloud-Netflix-Hystrix-Stream
5. Spring-Cloud-Netflix-Hystrix
6. Spring-Cloud-Netflix-Ribbon
7. Spring-Cloud-Netflix-Turbine-Stream
8. Spring-Cloud-Netflix-Turbine
9. Spring-Cloud-Netflix-Zuul

##### 유지 관리 모드의 기능의 대치 기술
Spring Cloud Netflix는 기존에 있던 모듈들을 다른 기능들로 바꾸는 것을 권장하는데
다음은 아래와 같습니다.

|기존|권장|
|------|---|
|Hystrix|Resilience4j|
|Ribbon|Spring Cloud Loadbalancer|
|Zuul|Spring Cloud Gateway|
|Archaius|Spring Boot external config + Spring Cloud Config|

### Spring Cloud Consul

`Spring Cloud Consul`은 `Spring Boot`에 대한 `Consul`통합을 제공하기 때문에
몇가지 간단한 주석으로 빠르게 활성화하고 구성 할 수 있습니다. `Consul`은 오픈소스를
제공하고 있으며 엔터프라이즈 버전에 대한 비용을 지불하면 더 많은 기능들을 사용할 수 있습니다

![Spring Cloud Consul](https://user-images.githubusercontent.com/33123391/104689312-a48d0f80-5745-11eb-9f1a-43de240534b6.png)

위의 그림과 같이 `Consul`은 웹 UI를 이용하여 설정값들을 편리하게 설정할 수 있고 
`Spring Cloud Config`와 마찬 가지로 외부 환경에서 설정값들을 따로 관리할 수 있는 장점이 있습니다.
또한 `Service Discovery`, `Health Checking`, `Multi Datacenter`를 지원해줍니다

하지만 설정 파일을 실수로 삭제하게 되면 복원이 불가능하지만 Git2Consul을 이용해 영속화 이슈를 해결 할 수 있습니다.
Git2Consul은 Unix에서만 테스트 되었기 때문에 다른 운영체제 환경에서는 사용하기 꺼려집니다.

참고 문서
* https://medium.com/@yongkyu.jang/spring-cloud-config-server-f1e390f18cfc
* https://12factor.net/ko/config
* https://m.mkexdev.net/414
* https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now
* https://github.com/Netflix/Hystrix#hystrix-status
* https://netflixtechblog.com/announcing-archaius-dynamic-properties-in-the-cloud-bc8c51faf675
* https://github.com/Netflix/archaius/wiki/Overview
* https://github.com/breser/git2consul