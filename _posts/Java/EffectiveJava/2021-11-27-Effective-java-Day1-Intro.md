---
title: '[Effective Java] Day 1 - 프로그래밍의 기본 원칙과 자바가 지원하는 타입'
layout: post
categories: java
tags: java
comments: true
---

다음주 수요일부터 Joshua Bloch의 고전 명저인 Effective Java 3판의 내용을 같이 공부하고 토의하는 자바 스터디를 시작하게 되었다. 스터디 참여가 확정되고 난 후 바로 교보문고를 통해 책을 주문하였다. 오늘부터 약 두 달간 이 책과 함께 열심히 나를 불태워버리겠다. 책을 다 읽었을 때 쯤 JAVA 기본기가 지금보다 한층 더 레벨업 되어있을 나를 기대하며... Day 1 기록 시작!
### Ⅰ. 공부 시 참고하면 좋을 사이트
1. [이펙티브 자바 3판 번역 용어 해설](https://docs.google.com/document/u/0/d/1Nw-_FJKre9x7Uy6DZ0NuAFyYUCjBPCpINxqrP0JFuXk/mobilebasic#h.vim3vsbh8avu)
2. [백기선님 동영상 강의](https://m.youtube.com/watch?v=X7RXP6EI-5E&list=PLfI752FpVCS8e5ACdi5dpwLdlVkn0QgJJ) 
3. [깃헙 예제소스](https://github.com/WegraLee/effective-java-3e-source-code)

### Ⅱ.들어가기
자신이 공부한 외국어를 실전에서 활용해 보았다면 세 가지를 통달해야 한다는 사실을 알고 있을 것이다. 
1. 언어의 구조(문법)
2. 말할 대상의 이름(어휘)
3. 일상의 이야기를 풀어내는 관례적이고 효과적인 방법(용법)

**프로그래밍 언어를 공부할 때**도 이와 똑같다.
1. 언어의 핵심(절차적이냐 함수형이냐 객체 지향이냐)을 이해하고
2. 어휘(자료구조, 연산자, 표준 라이브러리의 기능)를 알고
3. 코드를 구성하는 관례적이고 효과적인 방법에 숙달해야 한다.

입으로 내뱉어 버린 말, 종이에 인쇄되어 출간된 책, 잡지와는 달리 프로그램은 계속 수정할 수 있다. 그래서 코드는 **구조가 수정하기 쉬워야한다.** 이때 이왕이면 다음 버전에서 기능 T가 어떻게 개선될지까지 고려해, 처음부터 개선된 T와 비슷한(즉, T'로 수정하기 편한) 구현 방식을 선택하는 것이 좋다.  
  
#### 프로그래밍의 아주 핵심적인 기본 원칙
1. **명료성(clarity)**
- 컴포넌트는 사용자를 놀라게 하는 동작을 해서는 절대 안된다.
- 정해진 동작이나 예측할 수 있는 동작만 수행해야 한다.  
 
2. **단순성(simplicity)**
- 컴포넌트는 가능한 한 작되, 그렇다고 너무 작아서도 안된다 (이 책에서 컴포넌트란 개별 메서드부터 여러 패키지로 이뤄진 복잡한 프레임워크까지 재사용 가능한 모든 SW 요소를 뜻함)
- 코드는 복사되는 게 아니라 재사용되어야 한다.
- 컴포넌트의 의존성은 최소로 유지해야 한다.
- 오류는 만들어지자마자 가능한 한 빨리(되도록 컴파일타임에) 잡아야 한다.

#### 주요 기술용어
자바가 지원하는 타입은 총 4가지 이다.  
1. **인터페이스(interface)**  
    - 에너테이션(Annotaion): 주석에 포함되면서 프로그래밍 언어에 영향을 미치지 않으면서 설정 정보와 같은 유용한 정보를 특정 프로그램에게 제공  
    ```java
    @test //이 메서드가 테스트 대상임을 테스트 프로그램에게 알린다.
    public void method(){
        ...
    }
    ```
2. **클래스(class)**  
    - 열거타입(enum)  
        - 관련된 상수들을 같이 묶어 놓은 것. Java는 비교 시 값과 타입을 모두 체크함.   
        - 모든 열거형은 java.lang.Enum이라는 클래스의 자손. 즉, 열거형의 조상은 enum. 

    ```java
        //           0      1        2       3
        enum Kind {CLOVER, HEART, DIAMOND, SPADE} //열거형 Kind를 정의
        enum Value {TWO, THREE, FOUR} //열거형 Value를 정의
            
        public class enumExam {
            
            static int x, y;	
            
            public static void main(String[] args) {
                Kind kind = Kind.CLOVER;
                Kind kind2 = Kind.valueOf("HEART");
                Kind kind3 = Enum.valueOf(Kind.class, "CLOVER");
                
            //	  //컴파일 에러 발생. 타입이 달라 비교 불가(Incompatible operand types Kind and Value)
            //      if(kind.CLOVER == Value.TWO){
            //          System.out.println("같아용");
            //      }
                
                System.out.println(kind==kind2); //false
                System.out.println(kind==kind3); //true
                System.out.println(kind.equals(kind3)); //true
                //System.out.println(kind > kind3);//에러. 객체라서 비교 연산자 사용 불가
                System.out.println(kind.compareTo(kind3));//0
                System.out.println(kind.compareTo(kind2));//-1
                
                //열거형의 모든 상수를 배열로 반환
                Kind[] kindArr = Kind.values();
                for(Kind d : kindArr)
                    System.out.println(d.name()+"="+d.ordinal());//이름과 순서
                
            }	

            //primitive type
            static final int CLOVER = 0;
            static final int HEART = 1;
            static final int DIAMOND = 2;
            static final int SPADE = 3;

            static final int TWO = 0;
            static final int THREE = 1;
            static final int FOUR = 2;
        }
    ```
3. **배열(array)**  
4. **기본타입(primitive)**  

이 4가지의 타입 중 인터페이스, 클래스, 배열은 **참조타입(reference type)**이라 한다.
즉, 클래스의 인스턴스와 배열은 객체인 반면, 기본타입은 그렇지 않다.