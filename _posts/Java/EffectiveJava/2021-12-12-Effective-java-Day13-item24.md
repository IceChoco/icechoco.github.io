---
title: '[Effective Java] Day 13 - Item 24 :: 멤버 클래스는 되도록 static으로 만들라'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

이 아이템에서는 각각의 중첩 클래스를 언제, 왜 사용해야하는지 얘기하고 있다.

## 중첩클래스란?
**중첩 클래스(nested class)**: 다른 클래스 안에 정의된 클래스
- 정적 멤버 클래스
- (비정적) 멤버 클래스
- 익명 클래스
- 지역 클래스

정적 멤버 클래스를 제외한 나머지는 내부 클래스(inner class)이다.

## 1. 정적 멤버 클래스
### 일반 클래스와의 차이
- 다른 클래스 안에 선언됨
- 바깥 클래스의 private 멤버에도 접근 가능

다른 정적 멤버와 **똑같은 접근 규칙**을 적용 받는다.  
- private으로 선언하면 바깥 클래스에서만 접근 가능
- 흔히 바깥 클래스와 함께 쓰이는 public 도우미 클래스로 쓰임
```java
public class Calculator {
    public enum Operation{// 열거타입도 정적 멤버 클래스이다
        PLUS, MINUS
    }
}
```
```java
public class Client {
    public static void main(String[] args) {
        Calculator.Operation test = Calculator.Operation.PLUS;
    }
}
```

### 정적 멤버 클래스와 접근제어자
private 정적 멤버 클래스는 바깥 클래스가 표현하는 객체의 한 부분(구성요소, *멤버변수를 의미하는 듯)을 나타날 때 쓴다.

public이나 protected 멤버라면 정적으로 만들지, 안 만들지에 대해서 신중해야 한다. 이미 릴리즈 된 비정적 클래스를 추후 버전에서 정적으로 바꾼다면 하위 호환성이 깨지게 된다.

## 2. 비정적 멤버 클래스
### 정적 멤버 클래스와의 차이
- 구문상 차이: static의 유무
- 의미상 차이: 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다.
- 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용하여 바깥의 인스턴스 메서드 호출 또는 바깥 인스턴스의 참조를 가져올 수 있다.
  - 정규화된 this: `클래스명.this` 형태로 바깥 클래스의 이름을 명시하는 용법

개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 **독립적으로 존재**할 수 있다면 정적 멤버 클래스로 만들어야 한다.  
비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없다.

```java
public class NonStaticExample {
    private final String name;

    public NonStaticExample(String name) {
        this.name = name;
    }

    public String getName() {
        NonStaticClass nonStaticClass = new NonStaticClass();
        return nonStaticClass.getNameWithOuter();
    }

    private class NonStaticClass {
        public String getNameWithOuter() {
            // 정규화된 this 를 이용해서 바깥 클래스의 인스턴스 메서드를 사용할 수 있다.
            return NonStaticExample.this.name;
        }
    }
}
```

비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버클래스가 인스턴스화 될 때 확인되며, 더 이상 변경할 수 없다.  

드물게 직접 `바깥 인스턴스의 클래스.new Member Class(args)`를 호출해 수동으로 만든다. 비정적 내부 클래스는 NonStaticExample 객체를 생성 후 NonStaticExample 객체를 이용하여 NonStaticClass 객체를 생성해야한다. 즉, 비정적 내부 클래스는 바깥 클래스에 대한 참조가 필요한 것이다.

```java
NonStaticExample nonStaticExample = new NonStaticExample("NonStatic");
NonStaticExample.NonStaticClass nonStaticClass = nonStaticExample.new NonStaticClass();
```

하지만 이 관계 정보는 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 차지하며, 생성시간도 더 걸린다.  
비정적 내부 클래스의 경우 바깥 인스턴스로의 숨은 외부 참조 정보를 가지고 있기 때문에 메모리 누수가 발생할 여지가 있다. 바깥 클래스는 더 이상 사용되지 않지만 **내부 클래스의 참조로 인해 GC가 수거하지 못해서 바깥 클래스의 메모리 해제를 하지 못하는 경우가 발생**할 수 있다.

![item24-nonStatic](/assets\img/item24-nonStatic.PNG)

정적 내부 클래스의 경우 바깥 클래스에 대한 참조 값을 가지고 있지 않기 때문에 메모리 누수가 발생하지 않는다. 메모리 누수가 발생할 수 있는 문제점이 있기 때문에 만약 **내부 클래스가 독립적으로 사용되어 바깥 인스턴스에 접근할 일이 없다면 무조건 static 클래스로 선언**하여 사용하는 것이 좋다! 바깥 클래스에 대한 참조를 가지지 않아 메모리 누수가 발생하지 않기 때문이다.

### 비정적 멤버 클래스의 쓰임새: 어댑터의 역할
- 비정적 클래스를 어댑터 패턴을 이용하여 바깥 클래스를 다른 클래스로 제공 할 때 사용하면 좋다.    
- Map 인터페이스의 구현체: 자신의 컬렉션 뷰를 구현할 때 비정적 멤버 클래스를 사용
  - ex) **HashMap의 `keySet()`**  
       `keySet()`을 사용하면 `Map`의 key에 해당하는 값들을 `Set`으로 반환해 주는 데 어댑터 패턴을 이용해서 `Map`을 `Set`으로 제공한다.
       ![keySet-of-HashMap](/assets\img/keySet-of-HashMap.png)
       
- Collection 인터페이스의 구현체: 자신의 반복자 구현

## 3. 익명 클래스(anonymous class)
- 이름이 없는 일회용 클래스.
- 바깥 클래스의 멤버도 아니다.
- **정의와 생성을 동시에:** 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
- 비정적인 문맥에서 사용될 때만 바깥 클래스의 멤버 접근이 가능하다.

```java
public class AnonymousExample2 {
    private double x;
    private double y;

    public double operate() {//비정적 문맥
        Operator2 operator = new Operator2() {//구현 인터페이스 이름
            @Override
            public double plus() {
                //바깥 인스턴스 x, y 참조
                System.out.printf("%f + %f = %f\n", x, y, x + y);
                return x + y;
            }
        };
        return operator.plus();
    }
}

interface Operator2 {
    double plus();
}
```
- 정적 문맥에서라도 상수 정적 변수(static final) 이외의 정적 멤버는 가질 수 없다. 즉, 상수 표현을 위해 초기화된 final 기본 타입과 문자열 필드만 가질 수 있다.
   - 자바 8에서 람다의 도입과 함께 조금 완화되어 final이 아니더라도 참조 가능하나 참조되는 값은 의미적으로는 여전히 상수여야만 함
   - 따라서 final로 선언하지 않더라도 값을 변경할 순 없음

```java
public class AnonymousExample3 {
    private double x;
    private double y;

    static double operate() {//정적 문맥
        Operator3 operator = new Operator3() {//구현 인터페이스 이름
            static final double a = 0;
            static double b = 0; //컴파일에러 발생. 자바 16부터는 에러 발생 X
            @Override
            public double plus() {
              //return x; //바깥 인스턴스 x, y 참조 불가
                return a;
            }
        };
        return operator.plus();
    }
}

interface Operator3 {
    double plus();
}
```
- 이전에는 즉석에서 작은 함수 객체나 처리 객체를 만드는데 익명 클래스를 사용했지만 이제 람다에게 물려주었다.

### 익명 클래스 예시 
1. Comparator 구현 - 람다가 나오고 나서는 자주 사용하지 않는 방식
```java
List<Integer> list = Arrays.asList(10, 5, 6, 7, 1, 3, 4);
Collections.sort(list, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return Integer.compare(o1, o2);
    }
});
System.out.println(list);
```
Java 8 람다가 나오고 나서는 주로 아래와 같이 사용한다.
```java
Collections.sort(list, (o1, o2) -> Integer.compare(o1, o2));
```
```java
Collections.sort(list, Comparator.comparingInt(o -> o));
```

2. 정적 팩터리 메서드 구현 시 자주 사용됨
```java
public interface IntListHelper {
      
    static List<Integer> intArrayAsList(int[] a) {
        return new AbstractList<Integer>() {
            @Override
            public Integer get(int index) {
                return a[index];
            }
  
            @Override
            public int size() {
                return a.length;
            }
        };
    }
}
```

### 익명 클래스의 제약사항
- 선언한 지점에서만 인스턴스 생성가능하다.
- instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.
- 여러 인터페이스 구현이 불가능하고, 구현과 동시에 다른 클래스를 상속할 수도 없다.
- 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.
- 표현식 중간에 등장해 10줄이 넘어가면 가독성이 좋지 않다.

## 4. 지역 클래스
- 가장 잘 안쓰인다.
- 지역변수를 선언할 수 있는 곳이면 어디서든 선언 가능하고, 유효 범위도 지역변수와 같다.
- 멤버클래스처럼 이름이 있고 반복적해서 사용 가능하다.
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.
- 정적 멤버는 가질 수 없고, 가독성을 위해 짧게 작성해야 한다.

```java
public class LocalExam {
    void x() {
        class LocalClass {
            void doPrint() {
                System.out.println("LocalClass");
            }
        }
        LocalClass local = new LocalClass();
        local.doPrint();
    }

    public static void main(String[] args) {
        LocalExam a = new LocalExam();
        a.x();
    }
}
```
### Q. 지역클래스 내 메소드에서 지역 클래스 밖 surrounding scope의 변수를 사용하는 구조의 소스가 있다. 이때 매번 지역클래스를 호출할 때 마다 객체를 새로 생성하는걸까?
아니다. 지역클래스를 선언하면 컴파일 타임에서 객체가 선언이 되고, 그 후 사용할 때 마다 선언된 객체를 호출하는 구조이다.
즉, 매번 사용할 때 마다 새로운 객체를 생성하는 것이 아니라 기생성된 객체를 인스턴스화 한다.

```java
public class A {
    private int i = 10, j = 20;
    void x(int a){
        class B {
            private int b;
            B(int b){ this.b = b;}
            void y() {
                System.out.println(a + b);
            }
        }
        B bb = new B(a/10);
        bb.y();
    }
    void y(){x(200);}
    void z(){x(300);}

    public static void main(String[] args) {
        A a = new A();
        a.x(100);
    }
}
```

위 소스코드에 대한 [컴파일 결과](https://godbolt.org/z/d6o9jYz8E)를 보면

```java
class A$1B {
  private int b;
  final int val$a; //System.out.println(a + b);에서 사용하면서 생김
  final A this$0;  //B는 A의 지역 클래스이기 때문에, B를 빈 깡통으로 선언해도 생김
  A$1B(int);
  void y();
}
```

지역 클래스인 class B, 즉 `A$1B`가 위와 같이 한 번만 생성된다. surrounding scope의 변수인 a, 바깥쪽 클래스 A의 객체 this가 마치 멤버변수처럼 B 지역 클래스 안에 생성된 것을 볼 수 있다. 즉, closure처럼 capture됐다.

## 결론
- 메서드 밖에서도 사용해야 함 or 메서드 안에 정의하기엔 너무 길다 → **멤버 클래스**
   - 멤버 클래스의 인스턴스가 바깥 인스턴스를 참조하면 → **비정적 멤버 클래스**
   - 멤버 클래스의 인스턴스가 바깥 인스턴스를 참조하지 않으면 → **정적 멤버 클래스**
- 중첩클래스가 한 메서드 안에 쓰임  
  and 그 인스턴스를 생성하는 지점이 단 한 곳  
  and 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있음 → **익명 클래스**
   - 그렇지 않다면? → **지역 클래스**

### 참고
- [정적, 비정적 내부 클래스 알고 사용하기](https://tecoble.techcourse.co.kr/post/2020-11-05-nested-class/)