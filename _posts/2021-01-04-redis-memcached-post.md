---
title: 인스타그램과 같은 SNS 플랫폼은 어떻게 많은 트래픽을 처리할까? - 4편(In-Memory Database)
date: 2020-01-04 22:21:00 
---

전편에서 왜 In-Memory Database를 사용해야 하는지 살펴봤습니다. 그렇다면 In-Memory Database 중
어떤 기술을 사용해야 프로젝트에 적합할지 알아보겠습니다.

### 세션 특징
일단 In-Memory를 살펴보기 전에 세션의 특징을 알아보겠습니다. 세션은 `setAttribute(String name, Object value)`로 
생성 해주고 `getAttribute(String name)`로 name을 조회해 값을 반환해줍니다. 

