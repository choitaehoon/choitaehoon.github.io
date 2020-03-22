---
title: "#### 인덱스 설정 과 explain 실행 계획 후 성능 확인"
date: 2020-03-22 23:55:00 
---

![image6](https://user-images.githubusercontent.com/33123391/77251297-ed447b00-6c90-11ea-82d1-18adc0b38b81.png)

student의 테이블의 형태는 위와 같습니다.

전체 칼럼 수는 **약 110만건** 데이터 생성 했습니다. 



각 칼럼의 개수는 아래와 같습니다.

![images5](https://user-images.githubusercontent.com/33123391/77251536-35b06880-6c92-11ea-8dde-7d4f15dc075d.png)

인덱스를 생성 하지 않은 상태에서 이름 찾는 쿼리를 실행 시켜봤습니다. 예시는 아래의 그림과 같습니다

![image1](https://user-images.githubusercontent.com/33123391/77251746-7f4d8300-6c93-11ea-8d8e-bb35715d3fdf.png)

rows를 보면 거의 모든 행을 다 돌고 있습니다. 한 마디로 말하자면 full scan이 일어나고 있습니다.filter도 보면 10%밖에 효율을 못내고 있습니다. 한번 인덱스를 걸어 실행 계획을 보겠습니다. 



인덱스 생성 또한 아래와 같습니다.

![images4](https://user-images.githubusercontent.com/33123391/77251457-ccc8f080-6c91-11ea-9712-a68d57125a6e.png)



그리고 index를 살펴 보겠습니다.

![image3](https://user-images.githubusercontent.com/33123391/77251648-e4ed3f80-6c92-11ea-971f-f62da0e3401c.png)

위의 사진을 보면 인덱스 생성이 잘 된 것을 볼 수 있습니다. 인덱스는 기본적으로 BTREE 알고리즘을 사용 하고 있습니다.



인덱스를 설정 하고 explain 결과

![images2](https://user-images.githubusercontent.com/33123391/77252699-02250c80-6c99-11ea-9e60-857dfe1db8d8.png)



**약 110만건에서 2만5천건의 행만 접근했습니다.** 많은 칼럼들을 조회 하지 않고 부분만 조회하는 range scan 했습니다. 이렇게 함으로써 성능을 향상 시킬 수 있었습니다. 한 눈에 보기 쉽게 mysql이 Visual Explain을 지원 하고 있습니다. 그림은 아래와 같습니다.

![image3](https://user-images.githubusercontent.com/33123391/77251985-e9b2f300-6c94-11ea-9ed0-164f43b98dbc.png)



하지만 성능 개선하겠다고 무작정 인덱스를 설정 하면 안됩니다. 인덱스를 설정하면 안되는 경우를 살펴 보겠습니다.

* 중복 데이터가 적은 경우
  * **중복 데이터가 많다는건 그만큼 인덱스를 통해 조회** 해야 한다는 의미 입니다. 하지만 중복 데이터가 거의 없다면 인덱스를 통해 대부분 데이터가 필터링 됩니다.
* 인덱스의 키가 길 경우
  * 인덱스는 페이지 단위로 구성 되는데 페이지 크기는 16KB로 되어 있습니다. 인덱스의 키가 크면 클 수록 페이지의 담을 수 있는 수는 작아 집니다.
* Update, delete 작업이 잦을 때 
  * 위에서 설명 한 것 처럼 인덱스는 페이지 단위로 구성 되어 있습니다. 만약 update가 발생 되면 페이지를 계속 수정 되어야 할 것이고 delete가 발생 할 경우 페이지에서 삭제가 되지 않고 그대로 남아 있어 메모리를 차지하고 있습니다. 그래서 정리하면 select 작업이 많고 update, delete가 적을 때 사용 해야 합니다.