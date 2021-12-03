---
title: '[Effective Java] Day 7 - Item 3 :: Private 생성자나 열거 타입으로 싱글턴임을 보증하라(2)'
layout: post
categories: java
tags: java
comments: true
---

Day 6에서는 플라이웨이트 패턴(Flyweight pattern), Java의 String Pool, 예제 소스를 참조한 장점3,4,5 이해를 통해 내용을 보완하며 item1,2를 복습하였다 (게시글 수정으로 진행함).
Day 7 기록 시작!

## Item 3 :: Private 생성자나 열거 타입으로 싱글턴임을 보증하라 (2)
### 1. public static final 필드 방식의 싱글턴
```java
package item03;

public class Singleton1 {

    //instance는 static이니까 클래스가 로딩될 시점에 한번만 초기화됨.
    //초기화될 때 new를 사용하기 때문에 그때 처음 만들어진 instance가 계속해서 재사용하게됨
    public static final Singleton1 instance = new Singleton1();

    static int cnt;

    private Singleton1() {
        cnt++;
        if(cnt != 1){//리플렉션을 막기 위한 방법
            throw new IllegalStateException("this object should be singleton");
        }
    }

}
```
public static final 필드인 Singleton1.instance 초기화 할 때 private 생성자는 딱 한번만 호출되고 Singleton1은 싱글턴이 된다.  
예외로는 리플렉션 API를 사용해 private 생성자를 호출할 수 있지만 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지면 이러한 공격에 대한 방어가 가능하다.

```java
package item03;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

public class SingleTest {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        //private이므로 Singleton1 밖에서 생성 불가
        //Singleton1 singleton1 = new Singleton1();
        Singleton1 singleton2 = Singleton1.instance;

        System.out.println(singleton2); //item03.Singleton1@4eec7777 - 주소 잘 가져옴

        //아래와 같은 방식으로 리플렉션 API를 이용하여 private 생성자 호출 가능. But
        Constructor<Singleton1> constructor = Singleton1.class.getDeclaredConstructor();
        constructor.setAccessible(true);
        Singleton1 singleton3 = constructor.newInstance(); //Caused by: java.lang.IllegalStateException: this object should be singleton - 에러발생

        System.out.println(singleton3);
    }
}
```

public static final인 instance를 호출하는 singleton2는 정상적으로 객체를 잘 가져오지만,  
리플렉션 API를 활용하여 private 생성자를 직접 호출 하는 singleton3는 private 생성자에 추가된 리플렉션 방어 로직에 의하여 `this object should be singleton`에러가 발생한다.

#### 장점
1. static 팩토리 메소드를 사용하는 방식보다 간단하다.
2. 해당 클래스가 싱글턴임이 API에 명백히 드러난다.

### 2. static 팩토리 메소드 방식의 싱글턴
```java
package item03;

public class Singleton2 {

    private static final Singleton2 instance = new Singleton2();

    private Singleton2(){
    }

    public static Singleton2 getInstance(){
        return instance;
    }

}
```
#### 장점
1. public static factory 메소드를 호출하는 클라이언트 코드를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
    - 처음엔 싱글턴을 쓰다가 나중에 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있다.
2. static factory를 Generic 싱글턴 팩토리로 만들 수 있다.
3. static factory method 참조를 `Supplier<Singleton2>`와 같이 공급자로 사용할 수 있다.
    - `public static Singleton2 getInstance()` 자체가 Supplier 인터페이스의 구현체처럼 쓰일 수 있는 메소드다

##### interface Supplier<T>
JAVA 8부터 등장한 개념인 Supplier란 메서드를 'get()' 하나만 가지고 있고 아무타입이나 리턴하는(T) 인터페이스이다.

### 참고
- [[이팩티브 자바] #3 싱글톤을 만드는 여러가지 방법 그중에 최선은?](https://www.youtube.com/watch?v=xBVPChbtUhM&t=534s)