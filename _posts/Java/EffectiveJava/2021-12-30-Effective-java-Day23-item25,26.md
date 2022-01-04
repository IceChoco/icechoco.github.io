---
title: '[Effective Java] Day 23 - Item 25, Item 26 :: 톱레벨 클래스는 한 파일에 하나만 담으라, 로 타입은 사용하지 말라'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day23에서는 item 25, 26에 대한 내용을 다룬다.

## Item 25 :: 톱레벨 클래스는 한 파일에 하나만 담으라
>A top level class is a class that is not a nested class.
>
>A nested class is any class whose declaration occurs within the body of another class or interface.
>
>출처 : [Oracle docs Chapter 8.Classes ](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html)

톱레벨 클래스란 **중첩 클래스가 아닌 클래스**이다. 보통 우리가 사용하는 파일 내에 하나만 존재하는 클래스를 톱레벨 클래스라고 한다.  
중첩클래스는 **다른 클래스 안에 정의된 클래스**이다.

### 왜 톱레벨 클래스는 한 파일에 하나만 담아야 할까?
- 자바 컴파일러는 소스 파일 하나에 톱레벨 클래스를 여러개 선언하는 것을 막지 않는다.(하지만 심각한 위험 감수 필요)
- 부득이하게 꼭 그래야 하는 상황이 있다면, static member class를 사용하는 것을 고려해야한다.

```java
// Song.java
class Song {
    static final String NAME = "Dance"
}
class Music {
    static final String NAME = "Rap";
}

// Music.java
class Song {
    static final String NAME = "Rock"
}
class Music {
    static final String NAME = "Hip-hop";
}
```
```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Song.NAME + Music.NAME);
    }
}
```

이 코드를 실행하면 IDE(여기서는 intellij) 자체적으로 에러를 발생한다.

![item25_intellij_error](/assets\img/item25_intellij_error.PNG)

IDE에서 잡아준 에러를 무시하고 강제로 Main class를 실행하면 아래와 같은 결과가 발생한다.

![item25_build_result](/assets\img/item25_build_result.PNG)

위 처럼 순서를 정해주지 않고 실행하면 에러가 발생하지만, javac 명령어를 통해서 컴파일 순서를 정해주면 동작이 될 수 있다. ex) javac Main.java Song.java

하지만 이는 컴파일러에 어느 파일을 순서에 따라서 동작이 달라질 수 있기 때문에 절대로 이렇게 개발해서는 안된다.

위의 예제를 **static member class**로 바꾸면 아래와 같다.

```java
public class Test {
    private static class Song{
        static final String NAME = "Dance";
    }

    private static class Music{
        static final String NAME = "Rap";
    }

    public static void main(String[] args) {
        System.out.println(Song.NAME + Music.NAME);
    }
}
```

### 결론
- 하나의 소스 파일에는 하나의 톱레벨 클래스(혹은 톱레벨 인터페이스)만 두자
- static member class를 사용하자


## Item 26 :: 로 타입은 사용하지 말라
### 1. 제너릭이란? (Generic type)
클래스 내부에서 사용할 데이터 타입을 나중에 인스턴스를 생성할 때 확정하는 것.  
클래스와 인터페이스 선언에 타입 매개변수가 쓰이면 이를 **제너릭 클래스** 혹은 **제너릭 인터페이스**라 한다.
  
ex) List 인터페이스
```java
public interface List<E> extends Collection<E> {
    ...
}    
```
원소의 타입을 나타내는 타입 매개변수 E를 받는다. 따라서 이 인터페이스의 완전한 이름은 `List<E>`지만 짧게 그냥 List라고 자주 쓴다.
제너릭 클래스 + 제너릭 인터페이스를 통틀어 **제너릭 타입**이라 한다.  

#### 제너릭 타입은 **매개변수화 타입**을 정의한다.  
`클래스 이름(혹은 인터페이스)<실제 타입 매개변수들 나열>`
- List<String>
  - 원소의 타입이 String인 List를 뜻하는 매개변수화 타입
  - String: 정규 타입 매개변수 E에 해당하는 실제 타입 매개변수

#### 제너릭 타입을 하나 정의하면 그에 딸린 **로 타입(raw type)**도 함께 정의된다.
**로타입**: 제너릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때  
ex) `List<E>`의 로타입은 List

- 로타입은 타입선언에서 제너릭 타입 정보가 전부 지워진 것 처럼 동작한다.
- 제너릭이 도래하기 전 코드와 호환되도록 하기 위해 생겼다.

**제너릭을 지원하기 전 raw type의 사용**
```java
//Stamp 인스턴스만 취급하는 Collection 로타입 선언
private final Collection stamps = ...;

//실수로 동전을 넣은 경우 "Unchecked call" 경고만 내뱉고 오류없이 컴파일된다.
stamps.add(new Coin(...));

//이렇게 동전을 꺼내기 전에는 오류를 알아채지 못한다.
for(Iterator i = stamps.iterator(); i.hasNext();){
    Stamp stamp = (Stamp)i.next(); //ClassCastException을 던진다.
    stamp.cancel();
}
```
- **단점**
  - 오류가 런타임시에 발견된다. 좋은 코드는 오류가 발생하기 전 컴파일 즉시 발견할 수 있는 코드다.
    - 런타임에 문제를 겪는 코드와 원인을 제공한 코드가 물리적으로 떨어져있을 가능성이 커진다.
  - 로타입을 쓰면 제너릭이 안겨주는 안전성(typesafe)과 표현력을 모두 잃는다. 절대로 써서는 안된다.

**잘못된 예**  
- 모르는 타입의 원소도 받는 로타입 사용  
```java
//2개의 집합(Set)을 받아 공통 원소의 수를 반환하는 메서드
static int numElementsInCommon(Set s1,Set s2){
    int result = 0;
    for (Object o1 : s1)
        if(s2.contains(o1))
            result++;
    return result;
}
```
위 소스는 정상 작동하나 로 타입을 사용해 안전하지 않다.  
→ **비한정적 와일드카드 타입(unbounded wildcard type)**을 사용하자!

#### 제너릭을 활용하면 매개변수화 타입을 주석이 아닌 타입 선언 자체에 담을 수 있다.
> 매개변수화 타입: stamps에는 Stamp만 들어간다는 정보  

```java
//매개변수화된 컬렉션 타입 - 타입 안전성 확보!
private final Collection<Stamp> stamps = ...;

//컴파일 오류 발생!
stampsC.add(new Coin(...));

for(Iterator i = stamps.iterator(); i.hasNext();){
    Stamp stamp = (Stamp)i.next(); //여기서 에러나지 않음을 컴파일러가 보장한다.
    stamp.cancel();
}
```

List 같은 로타입은 사용하면 안되지만 `List<Object>`사용은 된다.  
- 차이점
  - List
    - 제너릭 타입에서 완전히 발을 뺀 것
  - `List<Object>`
    - 모든 타입의 객체를 허용한다는 의사를 컴파일러에 전달

#### 제너릭의 하위 타입 규칙
매개변수 List를 받는 메서드에 `List<String>`을 넘길 수 있다.  
그러나 `List<Object>`를 받는 메서드에 `List<String>`를 넘길 수 없다.   
→ `List<String>`: **로타입 List의 하위타입, `List<Object>`의 하위타입은 아님**
```java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();

    addList(strings, Integer.valueOf(42)); //매개변수 List를 받는 메서드에 `List<String>`을 넘길 수 있다.
    safeAddList(strings, Integer.valueOf(42)); //컴파일 에러! List<String>은 List<Object>의 하위타입이 아니다.

    String s = strings.get(0);//class java.lang.Integer cannot be cast to class java.lang.String
}

private static void addList(List list, Object o){
    list.add(o);//컴파일러 경고 발생: 매개변수화된 클래스 List의 raw type 사용
}

private static void safeAddList(List<Object> list, Object o){
    list.add(o);
}
```

### 2. 비한정적 와일드카드 타입(unbounded wildcard type)
제너릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않을 때 물음표(?)를 사용하자.