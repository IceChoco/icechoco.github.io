---
title: 'Mongo DB란?'
layout: post
categories: database
tags: database React
comments: true
---

회사에서는 주로 oracle을 많이 사용하고, 다른 SQL을 사용할 기회가 오면 PostgreSQL를 잠깐 사용하고 있습니다. 그런데 RDBMS의 경우 저장할 데이터가 정형화되어있어 많은 양, 다양한 종류의 데이터를 저장하기에는 적합하지 않습니다. 그러다보니 네이버와 카카오, NHN 등 국내 140여 기업이 몽고DB 데이터베이스관리시스템(DBMS)을 사용하고 있습니다.

## Mongo DB의 인기
Mongo DB는 가장 유명한 No SQL DB입니다. 
![mongodb-survery](/assets\img/mongodb-survery.png)
스택오버플로우의 설문조사에 의하면, MongoDB는 DB 기술 중 5위로 인기가 좋습니다. 또한, 4위까지는 RDBMS인 점을 감안하면 MongoDB는 No SQL 중 가장 인기가 좋습니다.  

![mongodb-survery-most](/assets\img/mongodb-survery-most.png)
가장 원하는 DB 설문조사에서는 Mongo DB가 1위를 차지하였습니다. 앞으로 Mongo DB를 배우고자 하는 사람이 많은 것을 알 수 있습니다.

## Mongoose
Mongo DB를 편하게 사용할 수 있게 해주는 툴입니다. 설치방법은 터미널에 아래 명령어를 입력하면 됩니다.
```
npm install mongoose --save
```
![mongoose-package-json](/assets\img/mongoose-package-json.png)  
위 명령어를 입력함으로써 package.json 파일의 dependencies 안에 mongoose 버전이 추가된 것을 확인할 수 있습니다. 

## MongoDB Model이란?
Model은 **스키마를 감싸주는 역할**을 합니다. 여기서 스키마는 하나하나의 정보들을 지정해주는 역할을 합니다.  
예를 들어 상품 판매 사이트의 경우 글의 작성자, 제목, 상세 설명이 필요할 것입니다.  
그럼 제목과 상세설명의 타입은 String이다, 제목의 최대값은 50이다와 같이 구조를 지정해줄 수 있는데 이것이 바로 스키마입니다.  
→ 업무에서 사용하는 데이터셋인 DVO(Data Value Object)와 비슷해보인다.

<!--author-->