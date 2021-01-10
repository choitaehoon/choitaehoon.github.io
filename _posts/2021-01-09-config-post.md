---
title: 인스타그램과 같은 SNS 플랫폼은 환경 설정들을 어떻게 관리해야할까? - 
(Spring Cloud Config vs Spring Cloud Netflix vs Spring Cloud Consul)
date: 2021-01-09 14:03:00
---

### 배경

현재 진행하고 있는 프로젝트의 환경 설정들을 수정하고자 구글에 검색해본 결과 `Spring Cloud Config`를 통해
`Twelve-Factor App`을 접했습니다. 이 방법론 중 Config 부분을 살펴보면 
`모든 설정 정보는 코드로부터 분리된 공간에 저장되어야 하고
런타임에서 코드에 의해 읽히며 SaaS는 동일한 코드를 여러 환경(운영/개발)에 배포한다.`로 설명되어 있습니다.

현재 프로젝트에서는 애플리케이션 안에 설정 정보가 있으며 배포된 상황에서 설정이 변경된다면 재배포해야 하는 구조입니다
이러한 문제점을 해결하고자 검색해서 알아본 결과 `Spring Cloud Config`,
`Spring Cloud Netflix`, `Spring Cloud Consul`있다는 것을 파악했습니다.

그렇다면 이러한 기술 중 어떤 것이 저희 프로젝트에 알맞을지 파악해보겠습니다.

참고 문서
* https://medium.com/@yongkyu.jang/spring-cloud-config-server-f1e390f18cfc
* https://12factor.net/ko/config