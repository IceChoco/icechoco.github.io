---
title: '[Effective Java] Day 2 - 클래스의 멤버(Member), 메서드 시그니처, 서브클래싱과 서브타이핑, Item 1'
layout: post
categories: java
tags: java
comments: true
---

어제인 Day 1에서는 프로그래밍의 기본 원칙과 자바가 지원하는 타입에 대해서 공부했다. '1장 들어가기'에서는 책에서 사용하는 기술 용어를 명확히 하고 주로 넘어가는데, 그 용어에 대해서 기본적인 개념을 미리 한 번 정리하고 들어가면 좋겠다 싶어 정리하는 시간을 가졌다. 이번 포스팅도 책을 읽으면서 관련된 개념을 정리하는 식으로 기록하려한다. 공부하면서 지금까지 너무 기본기에대해서 간과한 것 같아 좀 앞으로가 두렵기도 하고, 잘 할 수 있을까 걱정되기도 하지만 좋은 스트레스를 받고 있는 것 같아 설레기도 하다. 오늘도 책상에 앉은 나 칭찬해! 그럼 Day 2 기록 시작!

### Ⅰ.들어가기
#### 클래스의 멤버(Member)
멤버는 한국어로 구성원이라는 뜻이다. 비유하면 클래스의 멤버라는 뜻은 클래스라는 설계도의 구성원이라고 생각하면 된다. 간단하게 멤버 == 구성요소로 생각하면 쉽다.
```java
    class Calculator{
        int left, right;//필드(속성, 변수)
        
        public void setOperands(int left, int right){//메서드
            this.left = left; //setOperands에 입력값으로 전달한 left를 인스턴스의 변수로 지정
            this.right = right;
        }
        
        public void sum(){
            System.out.println(this.left + this.right);
        }
        
        public void avg(){
            System.out.println((this.left + this.right)/2);
        }
    }

    public class memberExam {

        public static void main(String[] args) {
            Calculator c1 = new Calculator(); //c1이라는 인스턴스를 만듦
            c1.setOperands(10, 20); //메서드를 호출하여 10과 20을 인자로 전달
            c1.sum();
            c1.avg();
            
            Calculator c2 = new Calculator();//c2라는 두번째 인스턴스
            c2.setOperands(20, 40);
            c2.sum();
            c2.avg();
            
            //인스턴스"의" 변수로 지정 → 인스턴스 마다 서로다른 값의 변수를 소유할 수 있다.
            //같은 left여도 c1은 값이 10이고, c2는 값이 20이니까!
        }

    }
```
1. **필드(field)**: 클래스 내 전역에서 접근할 수 있는 변수
2. **메서드(method)**: 어떤 특정 작업을 수행하기위한 명령문의 집합
3. **멤버 클래스**: 클래스 내 선언된 클래스
4. **멤버 인터페이스**: 클래스 내 선언된 인터페이스

#### 메서드 시그니처
메서드 시그니처란 **메서드의 이름과 입력 매개변수(parameter)의 타입들**로 이루어진 조합을 말한다. 메서드 <u>반환값의 타입은 시그니처에 포함되지 않는다</u>. 아래 코드에서 메서드 시그니처는 **<span style="color:green">setOperands(int left, int right)</span>** 이다.
```java
public void setOperands(int left, int right){//메서드
    this.left = left; //setOperands에 입력값으로 전달한 left를 인스턴스의 변수로 지정
    this.right = right;
}
```
아래 두 메서드는 다른 시그니처를 가진다.
```java
setOperands(int left);
setOperands(int left[]);
```
아래 메서드들은 모두 같은 시그니처를 가진다.
```java
int setOperands(int left);
String setOperands(int right);
int setOperands(int top) throws java.lang.Exception;
```

#### 이 책에서 자바언어 명세와 다르게 사용하는 기술용어
**1.** 상속(inheritance)을 서브클래싱(subclassing)과 동의어로 쓴다.
  - **상속**  
  다른 클래스가 가지고 있는 정보를 자신이 포함하겠다는 의미. 즉, 다른 클래스에 대한 정보를 상속받아 자신이 그대로 사용할 수 있도록 한다. 상속을 적절히 활용할 때 불필요한 코드의 수를 줄일 수 있어서 상당히 효율적인 개발이 이루어질 수 있다.  
  아래는 person을 상속한 Studnet 클래스의 예시이다.

  ```java
    public class Student extends Person {
        
        private String sudentID;
        private int grade;
        private double GPA;
        
        public String getSudentID() {
            return sudentID;
        }
        public void setSudentID(String sudentID) {
            this.sudentID = sudentID;
        }
        public int getGrade() {
            return grade;
        }
        public void setGrade(int grade) {
            this.grade = grade;
        }
        public double getGPA() {
            return GPA;
        }
        public void setGPA(double gPA) {
            GPA = gPA;
        }
        
        public Student(String name, int age, int height, int weight, String sudentID, int grade, double gPA) {
            super(name, age, height, weight);//자신의 부모가 가지고 있는 생성자를 실행하겠다.
            this.sudentID = sudentID;
            this.grade = grade;
            GPA = gPA;
        }

        public void show() {
            System.out.println("------------------------------------------------------");
            System.out.println("학생 이름:"+ getName());
            System.out.println("학생 나이:"+ getAge());
            System.out.println("학생 키:"+ getHeight());
            System.out.println("학생 몸무게:"+ getWeight());
            System.out.println("학번:"+ getSudentID());
            System.out.println("학년:"+ getGrade());
            System.out.println("학점:"+ getGPA());
        }
        
    }
  ```
  - **서브클래싱과 서브타이핑**  
  상속을 사용하는 목적에 의해서 서브클래싱과 서브타이핑으로 나뉜다.
    - **서브클래싱(subclassing) == 구현상속(implementation inheritance), 클래스 상속(class inheritance)**  
    다른 클래스 코드 재사용을 목적으로 함. 행동이 호환되지 않으므로 자신이 부모의 인스턴스를 대체할 수 없다
    - **서브타이핑(subtyping) == 인터페이스 상속(interface inheritance)**  
    타입 계층을 구성하기 위한 목적. 행동이 호환되기 때문에 자식 클래스의 인스턴스가 부모클래스 인스턴스를 대체할 수 있다  

**2.** 인터페이스 상속 대신 "클래스가 인터페이스를 **구현한다**" or "인터페이스가 다른 인터페이스를 **확장한다**"라고 표현한다.  
  - **인터페이스**  
    - 인터페이스에서는 반드시 **사전에 정의된 추상 메소드와 상수만을 가질 수 있다**. 추상클래스는 추상 메소드 외에 멤버 변수나 일반 메소드를 가질 수 있지만 인터페이스는 가질 수 없다.  
    - 인터페이스는 하나의 클래스에서 **2개 이상의 인터페이스를 상속**받을 수 있으나, 추상클래스는 다중 상속이 불가능하다.
    - 조금 더 추상화의 정도가 높아서 완전히 설계의 목적으로서 사용이 되는 하나의 자바 특징이다.  
    ex) Dog, Cat이라는 인터페이스를 다중 상속받은 Main 클래스의 예시

    ```java
        package interface_exam;

        public class Main implements Dog, Cat{//implements: 인터페이스를 상속 받는다.

            public static void main(String[] args){
                Main main = new Main();
                main.crying();
                main.one();
                main.two();
            }

            @Override
            public void crying() {
                // TODO Auto-generated method stub
                System.out.println("월! 월!");
            }

            @Override
            public void one() {
                System.out.println("One!");
            }

            @Override 
            public void two() {
                // TODO Auto-generated method stub
                System.out.println("two!");
                
            }
        }

    ```
**3.** 아무것도 명시하지 않은 접근 수준을 얘기할 때는 '패키지 접근'이라는 용어 대신 전통적인 **패키지-프라이빗**을 쓴다.
 - **접근수준(access level)**  
 자바의 access level은 public, protected, (package-private), private 로 4개가 있다.
 그중에 (package-private)을 이 책에서 사용하겠다는건데 아래 예제를 보자.

 ```java
package test;

public class AccessModifier {

	// public
	public void publicMethod() {
		System.out.println("publicMethod() is called");
	}
    
	// protected
	protected void protectedMethod() {
		System.out.println("protectedMethod() is called");
	}
	
        // package-private
	void packagePrivateMethod() {
		System.out.println("packagePrivateMethod() is called");
	}
	
        // private
	private void privateMethod() {
		System.out.println("privateMethod() is called");
	}
    
}
 ```
package-private 를 제외한 나머지 access level은 함수의 반환타입 앞에 명시적으로 적어주었는데
package-private는 access level을 적어주지않는것을 말한다.
적어주지않는다는 말은 access level이 없다는 말은 아니며, **해당클래스와 같은 패키지 안에서 접근이 가능**한 제어자 이다.

#### 공개 API(exported API) / API(application programming interface)
- 프로그래머가 클래스, 인터페이스, 패키지를 통해 접근할 수 있는 모든 클래스, 인터페이스, 생성자, 멤버, 직렬화된 형태
- 언어 구성요소 중 하나인 인터페이스와 헷갈리지 않게 하기 위해 흔히 쓰는 인터페이스 대신 API를 쓴다.
- API를 사용하는 프로그램 작성자(사람)를 그 API의 **사용자(user)**라 하고, API를 사용하는 클래스(코드)는 그 API의 **클라이언트(Client)**라 한다.
- 공개 API는 그 API를 정의한 패키지 밖에서 접근할 수 있는 API 요소로 이뤄진다. 즉, 모든 클라이언트가 접근할 수 있고, API 작성자가 지원하기로 약속한 API요소들이다.
    - **API 요소**:  클래스, 인터페이스, 생성자, 멤버, <u>직렬화된 형태</u>를 총칭함.
        - 직렬화된 형태 (Serialized form)
        ```
        Serialization is a mechanism of converting the state of an object into a byte stream.
        ```
        이를 해석해 보면 직렬화는 **byte stream으로 객체의 상태를 변환하는 메커니즘**이라고 할수있는데  
        이때 byte stream이란 데이터를 Byte단위로 주고받는 것을 말한다. 대표적으로 자바에서의 Byte Stream은 **InputStream**과 **OutputStream**이 있다. 이 클래스를 통과하는 단위는 당연히 Byte로 이루어져 있다.  
        그림, 텍스트, zip, jar 등과 같은 모든 데이터는 원래 Byte로 되어있다. 이 Byte들을 적절하게 변환하면 데이터가 되는 것이며 Byte Stream을 사용하는것은 원시 Byte를 그대로 주고 받겠다는 의미가 담겨져 있다.
- javadoc 유틸리티를 기본모드로 실행하면 이 API 요소들만 담긴 문서가 만들어진다.
- 패키지의 공개 API는 그 패키지의 모든 public 클래스와 인터페이스의 public 혹은 protected 멤버와 생성자로 구성된다.
- 자바 9에서 등장한 모듈 시스템 개념을 적용하면 공개할 패키지를 선택할 수 있다.

### Ⅱ. 생성자 대신 정적 팩터리 메서드를 고려하라(Item 1)
클래스는 생성자와 별도로 정적 팩터리 메서드(static factory method)를 제공할 수 있다. 정적 팩터리 메서드란 클래스의 인스턴스를 반환하는 단순한 정적 메서드를 말한다.  
다음 코드는 boolean 기본 타입의 박싱 클래스인 Boolean에서 발췌한 간단한 예이다. 
```java
public class item1 {
	
	public static final Boolean TRUE = new Boolean(true);

	public static final Boolean FALSE = new Boolean(false);

	public static boolean valueOf(boolean b){
		return b? TRUE: FALSE;
	}

	public static void main(String[] args) {
		System.out.println(Boolean.valueOf(true));
		System.out.println(Boolean.valueOf(false));
	}
}
```
- **박싱(boxing)**:  기본 자료형의 데이터를 대응되는 래퍼 클래스의 객체로 만드는 동작을 의미.

### 참고
- [자바 기초 프로그래밍 강좌 16강 - 상속 ① (Java Programming Tutorial 2017 #16)](https://www.youtube.com/watch?v=iYW83DF6MHk)
- [[Effective Java] 들어가기](https://godtaehee.tistory.com/m/2?category=970294)