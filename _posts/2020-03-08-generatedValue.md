---
title: "JPA GeneratedValue 기본키 생성 전략"
date: 2020-03-08 16:15:00 
categories: JPA GeneratedValue AUTO
---

mysql에 hibernate_sequence 테이블을 따로 관리 하여 자동키를 번호를 생성 했다. 이점이 궁금하여 내부와 어떤 실행 전략을 시키는지 알아보려고 한다.

#### JPA @GeneratedValue 내부 구조

![jpa](/Users/choitaehoon/Documents/jpa.jpg)

GenerationType에 TABLE, SEQUENCE, IDENTITY, AUTO의 타입으로 이루어져 있는데 이 종류 중에 default로 AUTO로 하고 있다. 그렇다면 AUTO 를 사용하는 경우 기본키 생성 전략을 어떻게 결정 할까?

![auto](/Users/choitaehoon/Desktop/auto.png)

**이 그림은 popit.kr에서 발취한 자료입니다.**

실행 과정을 살펴보자

```
1. 데이터 타입이 UUID인지 확인 한다.

2. 맞으면 UUID Generator가 된다. 틀리면 Numberal 확인

3. Hibernate.id_new_generator_mappings가 True인지 확인

4. True 이면 다음 단계 확인. 틀리면 native Generator는 다시 하이버네이트에 설정된 Dialect로 결정

5. 사용하고 있는 데이터 베이스가 시퀀스를 지원하는지 확인한다. 

6. 지원 하면 그 시퀀스 전략을 따르고 아니라면 Table Generator를 따른다

   

   (Mysql은 시퀀스를 지원하지 않는다. 지원 하는 데이터베이스는 Oracle, H2, PostgreSQL)
```



#### #### Hibernate.id_new_generator_mappings

![generator](/Users/choitaehoon/Desktop/generator.png)

spring-boot 공식 문서에 기본적으로 true를 지원 한다. 일시적으로 복원해야 하는 경우 false로 설정하라는 문구가 존재한다

