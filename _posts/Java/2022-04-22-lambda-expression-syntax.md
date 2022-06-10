---
title: '[Java] 람다식 문법'
layout: post
categories: java
tags: java
comments: true
---

예제로 람다식 문법에 대해 배워보자. 아래 예제는 0부터 9까지의 숫자를 출력하는 2가지 방법에 대해 서술한다.

## 1. 전통적인 방법
```java
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}
```
위는 기초 프로그래밍에서 흔히 볼 수 있는 for문을 이욯한 아주 기초적인 코드다.  
이는 각각의 요소들을 하나하나 일일이 검증하며 순차적으로 값을 확인하여 조건절이 끝날떄까지 진행되고 있으며, 이러한 코드는 특별한 경우라면 최적화되지 않고 들어오는 순서대로 진행된다.

## 2. 더 나은 방법
그러나 빨리 끝나는 일을 먼저 하거나 한번에 여러가지 일을 하는 것이 당연히 효율적이며 짧은 시간내에 작업을 끝낼 수 있다.  
또한 **1부터 10까지 1씩 증가하면서 이 코드를 순차적으로 실행해라**라고 **명령**하는 것 보다는  
**여기 있는거 다 해**라고 **설명**하는 것이 더욱 직관적이며 간결하다.

이러한 방식을 `Tell, Don't Ask`원칙이라 하며, 우리말로 한다면 `묻지 않고 시키기`로 볼 수 있다.
아래는 `Tell, Don't Ask`원칙에 따라 람다식으로 재작성한 코드로, 이렇게 할 경우 기초적인 수준에서 생기는 장점은 for문과는 달리 개념적으로 설명이 단순해 이해가 빠르다는 점이며,  
여기서는 잘 드러나지 않지만 복잡한 프로그래밍을 할 때 코드가 간결해지는 장점이 있다.
게다가 순수 함수로만 코드를 작성하면 for문과 i등의 변수를 사용하는 방식과 다르게 매번 같은 동작이 보장되서 병렬처리가 보다 수월해진다.


- Java 8부터 지원되는 람다식을 사용한 코드
```java
IntStream.range(0,10).forEach((int value) -> System.out.println(value));
```

- 컴파일러의 추론을 통해 파라미터의 자료형을 생략한 코드
```java
IntStream.range(0,10).forEach((value) -> System.out.println(value));
```

- 메서드 참조를 사용한 코드
```java
IntStream.range(0,10).forEach(System.out::println);
```



### 참고
- [소트웍스 앤솔러지 - 규칙 8: 일급 콜렉션 사용](https://namu.wiki/w/%EB%9E%8C%EB%8B%A4%EC%8B%9D)