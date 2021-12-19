---
title: '[Java] 제너릭(Generic)의 사용 및 이해'
layout: post
categories: java
tags: java
comments: true
---

**이 글의 목적**
- 제너릭(Generic)이 무엇인지 이해하고 왜 사용하는지 알 수 있다.

## 제너릭이란?
클래스 내부에서 사용할 데이터 타입을 나중에 인스턴스를 생성할 때 확정하는 것을 말한다.

```java
public class generic {
    class Person<T>{
        public T info;
    }

    Person<String> p1 = new Person<String>();
    Person<StringBuilder> p2 = new Person<StringBuilder>();
    }
}
```

String 타입의 info 필드를 갖는 `Person<String>` 클래스를 생성하고, `Person<String>` 클래스 타입의 p1을 만든다.

## 그럼 제너릭을 왜쓰지?
```java
package item03;

class StudentInfo{
    public int grade;
    StudentInfo(int grade){ this.grade = grade; }
}

class StudentPerson{
    public StudentInfo info;
    StudentPerson(StudentInfo info){ this.info = info; }
}

class EmployeeInfo{
    public int rank;
    EmployeeInfo(int rank){ this.rank = rank;}
}

class EmployeePerson{
    public EmployeeInfo info;
    EmployeePerson(EmployeeInfo info){this.info = info; }
}

public class noGeneric {
    public static void main(String[] args) {
        StudentInfo si = new StudentInfo(2);
        StudentPerson sp = new StudentPerson(si);
        System.out.println(sp.info.grade); //2
        EmployeeInfo ei = new EmployeeInfo(1);
        EmployeePerson ep = new EmployeePerson(ei);
        System.out.println(ep.info.rank); //1
    }
}
```
위 소스를 보면 EmployeePerson과 StudentPerson의 구조가 같은 것을 알 수 있다. 이런 소스코드는 타입이 안전하다는 장점을 갖고 있지만 소스가 중복된다는 단점이 있다.

```java
package item03;

class StudentInfoY{
    public int grade;
    StudentInfoY(int grade){ this.grade = grade; }
}

class EmployeeInfoY{
    public int rank;
    EmployeeInfoY(int rank){ this.rank = rank;}
}

class Person{
    public Object info;
    Person(Object info){this.info = info; }
}

public class yesGeneric {
    public static void main(String[] args) {
        //타입이 안전하지 않은 2가지 경우
        //1. Object로 설정해버려서 의도한 바와 다르게 String 타입이 들어감
        Person p1 = new Person("부장");

        //2. 컴파일 단계에서 에러가 발생하지 않음.
        EmployeeInfoY eiOut = (EmployeeInfoY)p1.info;
        System.out.println(eiOut.rank);
    }
}
```

중복되는 2가지 클래스를 제거하고 Person이라는 클래스를 활용하면 간결하고 유지보수하기 쉬운 소스코드를 만들 수 있다. 단, 아래와 같이 타입이 안전하지 않은 문제가 발생한다.
1. 두가지 클래스를 모두 매개변수로 받기 위해 데이터 타입을 Object로 설정해버리면서 의도한 바와 다르게 String 타입이 들어간다.
2. 매개변수로 넣었던 부장을 person타입의 p1에서 다시 꺼낸 뒤, 형변환을 통해 EmployeeInfoY에 담으려하면 컴파일 에러가 발생하지 않고 런타임 단계에서 발생한다.  
 
**타입이 안전하면서도 소스코드의 중복성을 제거하기 위해** 제너릭이 등장하였다.

## 제너릭의 특성
### 복수의 제너릭
클래스 내에서 여러개의 데이터 타입이 미지의 상태로 필요한 경우 사용 가능하다.
```java
package item03;

class StudentInfoD{
    public int grade;
    StudentInfoD(int grade){ this.grade = grade; }
}

class EmployeeInfoD{
    public int rank;
    EmployeeInfoD(int rank){ this.rank = rank;}
}

class PersonD<T, S>{
    public T info;
    public S id;
    PersonD(T info, S id){
        this.info = info;
        this.id = id;
    }
}

public class GenericDemo {
    public static void main(String[] args) {
        Integer id = 1;
        PersonD<EmployeeInfoD, Integer> p1 = new PersonD<EmployeeInfoD, Integer>(new EmployeeInfoD(1), id);
        System.out.println(p1.id.intValue());
    }
}
```
 
### 제너릭의 생략
제너릭은 클래스 뿐만 아니라 메서드에서도 사용이 가능하다. 그리고 변수의 데이터 타입을 java가 알 수 있기 때문에 객체 생성 시 <>로 데이터 타입 표기하는 것을 생략해도 된다.
```java
package item03;

class StudentInfoD{
    public int grade;
    StudentInfoD(int grade){ this.grade = grade; }
}

class EmployeeInfoD{
    public int rank;
    EmployeeInfoD(int rank){ this.rank = rank;}
}

class PersonD<T, S>{
    public T info;
    public S id;
    PersonD(T info, S id){
        this.info = info;
        this.id = id;
    }
    public <U> void printInfo(U info){
        System.out.println(info);
    }
}

public class GenericDemo {
    public static void main(String[] args) {
        EmployeeInfoD e = new EmployeeInfoD(1);
        Integer id = 1;
        PersonD p1 = new PersonD(e, id);
        p1.printInfo(e);
    }
}
```

### 제너릭의 제한
제너릭이란 클래스 내부적으로 데이터 타입이 미정인 경우, 인스턴스화 시킬 때 데이터 타입을 지정하는 것을 말한다.  
하지만 제너릭으로 하다보면 모든 객체가 모두 들어올 수 있다는 문제가 있다. 이럴떈 상속을 이용하여 제너릭에 제한 조건을 줄 수 있다.
```java
package item03;

interface Info{
//abstract class Info{
    public abstract int getLevel();
}

class EmployeeInfo implements Info{
//class EmployeeInfo extends Info{
    public int rank;
    EmployeeInfo(int rank){
        this.rank = rank;
    }
    public int getLevel(){
        return this.rank;
    }
}

//제너릭 안의 extends는 부모가 누구냐는 뜻. 그러므로 인터페이스여도 implements를 사용하지 않아도 된다.
class Person<T extends Info>{
    public T info;
    Person(T info){
        this.info = info;
        info.getLevel();
    }
}

public class GenericDemo {
    public static void main(String[] args){
        Person<EmployeeInfo> p1 = new Person<EmployeeInfo>(new EmployeeInfo(1));
        //String은 Info의 자식이 아니기 때문에 아래는 에러 발생
        //Person<String> p2 = new Person<String>("부장");
    }
}
```

이상 제너릭의 기본적인 사용법과 기본적인 개념을 알아봤다.

### 참고
- [Java - 제네릭 (4/5) : 제네릭의 특징 2](https://www.youtube.com/watch?v=MhUb5itcJvk)