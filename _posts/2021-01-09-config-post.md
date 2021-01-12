---
title: Spring Cloud에서 설정 값들을 어떻게 관리하면 좋을까? - (Spring Cloud Config vs Spring Cloud Netflix vs Spring Cloud Consul)
date: 2021-01-09 14:03:00
---

### 배경

현재 진행하고 있는 프로젝트의 설정 값들을 분리하고자 `Spring Cloud Config`에 대해 찾아 보던 
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

그렇다면 `Spring Cloud Config`를 이용하면 어떠한 장점이 있을까?

수 많은 Application의 설정 파일을 한 곳에서 관리 할 수 있으며 만일 설정이 바뀔때 `{도메인}/actuator/refresh`로 API를
호출하면 컴파일을 따로 다시 할필요가 없어 재배포하지 않아도 됩니다. 또한 Spring Boot Application에서 `@EnableConfigServer`
선언하면 쉽게 임베드 할 수 있습니다.

`Spring Cloud Config` 단점은 무엇이 있을까? 

만일 설정값을 변경한다면 수동으로 `{도메인}/actuator/refresh` API를 호출해야 바뀐 설정값들을 인식할 수 있으며
Config 서버에 의존하는 Application이 많다면 모든 Application을 API 호출해줘야 하기 때문에
유지보수가 까다로워 집니다.

이러한 문제점을 해결하고자 Spring Cloud에서는 `Spring Cloud Bus`를 제공합니다.

### Spring Cloud Bus 
ing..

참고 문서
* https://medium.com/@yongkyu.jang/spring-cloud-config-server-f1e390f18cfc
* https://12factor.net/ko/config
* https://m.mkexdev.net/414