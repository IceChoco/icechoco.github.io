---
title: '[Java] 와일드카드'
layout: post
categories: java
tags: java
comments: true
---

**이 글의 목적**
- 와일드카드 타입과 로 타입을 어떨 때 주로 사용하는지, 그 차이를 명확히 알 수 있다.

## 와일드카드 `<?>`
- 하나의 참조 변수로 대입된 타입이 다른 객체를 참조할 수 있다.
```java
ArrayList<? extends Product> list = new ArrayList<TV>(); //OK
ArrayList<? extends Product> list = new ArrayList<Audio>(); //OK
ArrayList<Product> list = new ArrayList<TV>(); //에러. 대입된 타입 불일치
```
일반적으로 Generic 클래스는 참조변수와 생성자에 대입된 타입이 일치해야한다.  
그런데 위 세번째 라인의 경우 참조변수는 Product, 생성자에 대입된 타입은 TV로 일치하지 않아 에러가 발생한다.

그래서 이 답답함을 없애기 위해 등장한 것이 바로 와일드카드이다.  
와일드카드를 사용하면 참조변수와 생성자에 대입된 타입이 일치하지 않아도 된다.  
총 3가지 용법이 있으며 주로 `<? extends T>`를 많이 쓴다.
- `<? extends T>`: Upper Bounded Wildcards. 상위 클래스 제한. T와 그 자손들만 가능
- `<? super T>`: Lower Bounded Wildcards. 하위 클래스 제한. T와 그 조상들만 가능
- `<?>`: Unbounded Wildcards, 비한정적 와일드카드 타입. 제한 없음. 모든 타입이 가능. `<? extends Object>`와 동일

메서드의 **매개변수**에도 와일드 카드를 사용한다.
```java
static Juice makeJuice(FruitBox<? extends Fruit> box){
    String tmp = "";
    for(Fruit f : box.getList()) tmp += f + " ";
    return new Juice(tmp);
}
```
```java
System.out.println(Juicer.makeJuice(new FruitBox<Fruit>()));
System.out.println(Juicer.makeJuice(new FruitBox<Apple>()));
```
Fruit의 자손인 Apple과 자기 자신이 메서드의 매개변수로 사용될 수 있다.

11:21

### 참고
- [[자바의 정석 - 기초편] ch12-12~14 와일드카드, 지네릭 메서드](https://www.youtube.com/watch?v=LL3PWmGFuQA)