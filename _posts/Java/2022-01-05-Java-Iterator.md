---
title: '[Java] IterableRHK Iterator'
layout: post
categories: java
tags: java
comments: true
---

이펙티브 자바 3판을 읽다가 `Iterable<E>`를 입력 매개변수 타입으로 받는 소스코드 발견했다. Iterator가 반복자인 것만 알고있었는데 Iterable은 뭐지? 이름만 봐서는 인터페이스 같은데... 궁금해서 찾아보게 되었다.

## 1. Iterable이란?
![iterable_hierarchy](/assets\img/iterable_hierarchy.PNG)
- Iterable은 Collction의 상위 인터페이스
  - Collection은 List, Set, Queue 인터페이스의 상위 인터페이스

![collection](/assets\img/collection.PNG)

흠


## 참조
[[Java] Iterable 과 Iterator 이란?](https://devlog-wjdrbs96.tistory.com/84)


