---
title: '[Effective Java] Day 24 - Item 26 :: 로 타입은 사용하지 말라(2)'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day24에서는 item 26에 대한 내용을 다룬다.

## Item 26 :: 로 타입은 사용하지 말라(2)
### 2. 비한정적 와일드카드 타입(unbounded wildcard type)
제너릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않을 때 로타입이 아닌 물음표(?)를 사용하자.
?는 어떤 타입이라도 담을 수 있는 가장 범용적인 매개변수화 Set타입이다.
```java
//좋은 예: 비한정적 와일드카드 타입(unbounded wildcard type) 사용
static int numElementsInCommon(Set<?> s1,Set<?> s2){
    ...
}
```

**Set VS `Set<?>`**
  - 로 타입 `Set`
    - 불안전함 - 아무 원소나 넣을 수 있어 타입 불변식을 훼손하기 쉽다.
  - 비한정적 와일드카드 타입(Unbounded wildcard type) `Set<?>`
    - `Set<E>`의 비한정적 와일드카드 타입은 `Set<?>`이다.
    - 안전함 - Collection<?>에는 null제외 어떤 원소도 넣을 수 없다.
      - 다른 원소를 add하려 하면 컴파일타임에 오류메시지를 볼 수 있다.
      - 컬렉션의 타입 불변식을 훼손하지 못하게 막고, 컬렉션에서 꺼낼 수 있는 객체의 타입도 알 수 없게 한다.

### 3. 로타입 규칙 예외
#### 1) class 리터럴에는 로타입을 사용한다.
자바 명세가 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다.
- 허용: List.class, String[].class, int.class
- 불가: `List<String>.class`, `List<?>.class`
  
* * *
- **클래스 리터럴(class literal)**
  - 클래스 리터럴(Class Literal)은 `String.class`, `Integer.class` 등을 말하며, `String.class`의 타입은 `Class<String>`, `Integer.class`의 타입은 `Class<Integer>`다.
  - 타입 토큰(Type Token)은 쉽게 말해 타입을 나타내는 토큰이며, 클래스 리터럴이 타입 토큰으로서 사용된다.
  - `myMethod(Class<?> clazz)`와 같은 메서드는 타입 토큰을 인자로 받는 메서드이며, `method(String.class)`로 호출하면, `String.class`라는 클래스 리터럴을 타입 토큰 파라미터로 myMethod에 전달한다.  

* * *
#### 2) instance of 연산자는 로타입을 사용한다.
런타임에는 제너릭 타입 정보가 지워진다.  
instance of 연산자는 비한정적 와일드카드 타입(`Set<?>`) 이외의 매개변수화 타입(`Set<String>`)에는 적용할 수 없다.  
로타입과 비한정적 와일드 카드 타입의 instance of는 완전히 똑같이 동작한다. <?>는 코드만 지저분해지므로 차라리 로타입을 쓰자.
```java
if(o instance of Set){    //로 타입
    Set<?> s = (Set<?>)o; //와일드카드 타입
}
```
o의 타입이 Set임을 확인한 다음 와일드 카드인 `Set<?>`로 형변환 해야 한다.  
이는 검사형변환(checked cast)이므로 컴파일러 경고가 뜨지 않는다.

### 결론
- 로 타입을 사용하면 런타임에 예외가 일어날 수 있으므로 사용하지 마라
   - 로 타입은 제너릭이 도입되기 이전 코드와의 호환성을 위해 제공된 것이다.
- `Set<Obejct>`: 어떤 타입의 객체도 저장할 수 있는 매개변수화 타입. 안전함.
- `Set<?>`: 어떤 종류의 타입 객체만 저장할 수 있는 와일드카드 타입. 안전함.
- Set: 로타입. 제너릭 타입 시스템에 속하지 않음. 안전하지 않음.

### 제너릭 관련 용어정리
```java
public interface List<E> extends Collection<E> {
     private E member; 
}
```

|한글용어(영문용어)    | 설명                                    | 예 |
|-----------|---------------------------------------------|---------|
| 매개변수화 타입(parameterized type) | 각각의 제네릭 타입은 parameterizedType을 선언함 | `List<String>` |
| 실제 타입 매개변수(actual type parameter) |  | `String` |
| 제네릭 타입(Generic Type) | 제네릭 클래스와 제네릭 인터페이스를 통틀어 이르는 말 | `List<E>` |
| 정규 타입 매개변수(formal Type parameter) | 제네릭 선언에 사용된 매개변수 | E |
| 비한정적 와일드카드 타입(unbounded wildcard type) | 어떤 종류의 타입 객체만 저장할 수 있는 와일드카드 타입 | `List<?>` |
| 로타입(raw type) | 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않았을 때 | List |
| 한정적 타입 매개변수(bounded Type parameter) |  | `<E extends Number>` |
| 재귀적 타입 한정(recursive Type parameter) |  | `<T extends Comparable<T>>` |
| 한정적 와일드카드 타입(bounded wildcard type) |  | `List<? extends Number>` |
| 제너릭 메서드(generic method) |  | `static <E> List<E> asList(E[] a)` |
| 타입 토큰(type token) |  | String.class |

## 참고
- [What is a class literal in Java?](https://stackoverflow.com/questions/2160788/what-is-a-class-literal-in-java)
- [클래스 리터럴, 타입 토큰, 수퍼 타입 토큰](https://homoefficio.github.io/2016/11/30/%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%A6%AC%ED%84%B0%EB%9F%B4-%ED%83%80%EC%9E%85-%ED%86%A0%ED%81%B0-%EC%88%98%ED%8D%BC-%ED%83%80%EC%9E%85-%ED%86%A0%ED%81%B0/)