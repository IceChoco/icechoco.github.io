---
title: '[Java] Hash Code & Hash Table'
layout: post
categories: java
tags: java
comments: true
---

만약 Youtube에서 이미 업로드 되어있는 동영상을 다운 받아서 올릴 경우 에러가 발생한다.
전세계에 유투브 동영상 갯수도 정말 많고, 한 동영상 당 용량도 큰데 어떻게 바로 알고 이미 동영상이 있다는 메시지를 건네주는 걸까? 바로 해쉬테이블을 이용하기 때문이다.

## Hash Table이란?
**F(key) → HashCode → Index → Value**  
- F(key): 검색하고자 하는 키값을 입력받아서  ex) 문자열, 숫자, 파일데이터
  - 해시함수: 어떤 특정한 규칙을 이용하여 입력받은 key 값으로, 그 key 값이 얼마나 큰지에 상관없이 동일한 Hash코드를 만들어준다.
- HashCode: 해쉬 함수를 입력받은 해쉬코드를  
- Index: 배열의 인덱스로 환산하여  
- Value: 데이터를 접근하는 방식의 자료구조 이다.  

## HashCode의 활용 예
암호화폐의 핵심기술인 블록체인에서도 각 사용자들의 공공장부를 비교할 때 해시코드를 이용한다.  
- **블록체인**: 10분 간격으로 성사된 거래 기록을 블록체인 창고에 저장하고, 지금까지 일어난 모든 거래기록을 그 서비스를 이용하는 모든 사용자들이 전자지갑을 갖고있게 하는 방식

즉 어떤 회사가 창립한 뒤 모든 거래장부를 모든 사람들이 갖게 되는 것인데, 이때 원본을 가지고 비교하게 되면 시간이 너무 오래걸리게 된다. 
그래서 그 정보를 HashCode로 배포하고, 사람들은 거래장부를 HashCode로 비교하게 되는 것이다.

## 장점
HashCode의 장점은 검색속도가 빠르다는 것이다.


ㅣ


|구분      |Perm                                         |Metaspace|
|---------|---------------------------------------------|---------|
| 저장정보 | 클래스 meta / 메소드 meta / static 변수, 상수 | 클래스 meta / 메소드 meta|
| 관리 포인트| Heap 영역 튜닝 + Perm 영역 별도 | Native 영역 동적 조정
| GC| Full FC | Full FC
| 메모리 측면 | -XX: PermSize / -XX: MaxPermSize | -XX:MetaSpaceSize / -XX: MaxMetaSpaceSize

1:40

## 참고
- [[자료구조 알고리즘] 해쉬테이블(Hash Table)에 대해 알아보고 구현하기](https://www.youtube.com/watch?v=Vi0hauJemxA&t=13s)