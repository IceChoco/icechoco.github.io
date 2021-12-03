---
title: '[Effective Java] Day 3 - Item 1 :: 생성자 대신 정적 팩터리 메서드를 고려하라'
layout: post
categories: java
tags: java
comments: true
---

어제인 Day 2에서는 클래스의 멤버(Member), 메서드 시그니처, 서브클래싱과 서브타이핑, 그리고 'Item 1 - 생성자 대신 정적 팩터리 메서드를 고려하라'의 앞부분에 대해서 공부했다. 확실히 평일에 늦게 퇴근하는 날은 시간적인 여유가 좀 적은 것 같다. 아무리 늦퇴하는 날이라고 하더라도 기록은 멈추지 말아야지. 그 어떤 사정도 결국은 핑계고 내 미래를 책임져주지 않으니T_T 그럼 Day 3 기록 시작!

### 정적 팩터리 메서드의 장점
#### 1. 이름을 가질 수 있다.
정적 팩터리 메서드는 생성자보다 읽기 편하다.


하나의 시그니처로는 생성자를 하나만 만들 수 있다.
```java
    public Student(String studentID, int grade);     //가 생성된 상태에서
    public Student(String studentID, int GPA);  //이렇게 생성이 불가능. 컴파일 에러 발생!
```

물론 입력 매개변수들의 순서를 다르게 한 생성자를 새로 추가하는 식으로 아래 소스 같이 제한을 피해볼 수도 있으나 아주 좋지 않은 발상이다.
```java
    public Student(String studentID, int grade);     //가 생성된 상태에서
    public Student(int GPA, String studentID);  //이렇게 생성이 가능하며 에러 발생하지 않긴 하지만... 매우 비추천
```

하지만 정적 팩터리 메서드는 이러한 제약이 없다. 한 클래스에 시그니처가 같은 생성자가 여러개 필요할 것 같으면, 생성자를 정적 팩터리 메서드로 바꾼 다음 각각의 차이를 잘 드러내는 이름을 지어주면 된다.
```java
class Student{
    Student(String studentID, int grade, int GPA) {}
   
    private String studentID;
    private int grade;
    private double GPA;

    public static Student StudentWithIDandGrade(String studentID, int grade){
        return new Student(studentID, grade, 0);
    }

    public static Student StudentWithIDAndGPA(String studentID, int GPA){
        return new Student(studentID, 0, GPA);
    }
}
```

#### 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
반드시 새로운 객체를 만들 필요가 없다. 플라이웨이트 패턴도 이와 비슷한 기법이다.

```java
package item1;

public class Foo {
    String name;
    String address;
   
    public Foo(){
    }
   
    //생성자
    public Foo(String name) {
        this.name = name;
    }
   
    //생성자 - 위의 생성자랑 시그니처가 같으므로 생성 불가
//  public Foo(String address) {
//      this.address = address;
//  }
   
    // static 팩토리 메서드
    public static Foo withName(String name){
        return new Foo(name);
    }
   
    public static Foo withAddress(String address) {
        Foo foo = new Foo(address);
        return foo;
    }
   
    private static final Foo GOOD_NIGHT = new Foo();
   
    public static Foo getFoo() {
        return GOOD_NIGHT;//객체를 아예 새로 생성하지 않고 미리 만들어놓은 것을 가져옴. 특히 생성비용이 큰 경우 성능을 상당히 끌어올려줌.
    }
   
    public static void main(String[] args){
        //장점 1: foo 코드보다는 foo2 의미가 더 읽기 편하다. Ara가 무슨 의미인지 알 수 있으므로.
        Foo foo = new Foo("Ara");
        Foo foo1 = Foo.withName("Ara");
       
        //장점 2: 반드시 새로운 객체를 만들 필요가 없다. 플라이웨이트 패턴도 이와 비슷한 기법임.
        Foo foo2 = Foo.getFoo();
    }
}
```
반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩터리 방식의 클래스는 언제 어느 인스턴스를 살아있게 할지를 철저히 통제할 수 있다. 이를 **인스턴트 통제 클래스**라 한다. 즉, 값이 있으면 새로 만들지 않고 기존 객체를 재사용하는 클래스를 말한다.  

**인스턴트 클래스를 통제하는 이유**  
1. **싱글턴(Singleton)**으로 만들 수 있다.
    - **싱글턴**: 객체의 인스턴스가 오직 1개만 생성되는 패턴을 의미  
    싱글턴이 사용되는 <u>예시</u>로는 어플의 다크모드가 있다. 사용자가 어플의 설정에서 다크모드를 설정해놓으면 다른 페이지로 이동하더라도 이 다크모드가 그대로 유지되어야한다. 즉, 어떤 페이지에 있던 이 세팅을 관리하는 객체는 반드시 같은 것을 사용해야 한다. 그럼 이 객체의 인스턴스가 하나만 생성되어 있어야 하는 것이 좋은데 이럴 때 싱글턴 패턴을 사용한다.  
    아래는 기본적인 싱글턴 코드이며 멀티 쓰레드 환경 등에서 오류가 발생할 소지가 있다. 그러므로 여기서는 싱글턴을 이해하는 용도로 소스코드를 참조하고, 실제 사용 시에는 싱글턴을 보다 안전하게 사용할 수 있는 방법들을 검색 후 사용하는 것을 권장한다.

        - **Settings.java**

        ```java
        package singleton;

        public class Settings {

            private Settings() {};//private: Settings 클래스에서만 사용 가능. 외부 클래스에서 사용하려고 하면 에러 발생.
            
            /* static으로 선언된 것들(정적공간) : 객체가 얼마나 생성되든 메모리의 지정된 공간에 딱 하나만 존재한다. 
            * 								컴파일 할 때 부터 이 요소가 차지할 메모리 용량을 이미 알 수 있도록 딱 정해져 있기 때문에
            * 								동적요소들과 대비되는 개념으로 static, 정적이라고 불림
            */
            private static Settings settings = null;//★★★ 자기 자신인 Settings 타입의 객체를 static으로 생성. null로 초기화 ★★★

            // 정적 팩토리 메소드
            public static Settings getSettings() {
                //1. 다른곳에서 getSettings를 실행하기 전이라면
                if(settings == null){
                    settings = new Settings();//Settings 객체를 선언해서 settings 변수에 넣어주고 아래에서 반환
                }
                //2. settings가 이미 만들어진 상태라면 settings를 그대로 반환
                return settings;
            }
            
            private static boolean darkMode = false;
            private static int fontSize = 13;
            
            public static boolean getDarkMode(){//public: 누구든지 이 클래스의 메소드를 호출 가능
                return darkMode;
            }

            public static int getFontSize() {
                return fontSize;
            }
            
            /* static이 아닌 것들(동적공간) : 객체가 생성될 때마다 메모리의 공간을 새로 차지*/
            public void setDarkMode(boolean _darkMode) {
                darkMode = _darkMode;
            }

            public void setFontsize(int _fontSize) {
                fontSize = _fontSize;
            }

        }
        ```
        - **FirstPage.java**

        ```java
        package singleton;

        public class FirstPage {
            //Settings객체의 static 메소드는 이미 메모리의 정적 공간에 자리를 차지하고 있는 상태이기 때문에
            //해당 객체를 new로 생성하지 않고도 클래스에서 바로 불러낼 수 있음
            private Settings settings = Settings.getSettings();
            
            public void setAndPrintSetting(){
                settings.setDarkMode(true); //정적 settings 객체에 값을 넣음
                settings.setFontsize(15);
                
                System.out.println(settings.getDarkMode()+" "+settings.getFontSize());//그 내용들을 출력함
            }
        }

        ```
        - **SecondPage.java**

        ```java
        package singleton;

        public class SecondPage {
            //Settings객체의 static 메소드는 이미 메모리의 정적 공간에 자리를 차지하고 있는 상태이기 때문에
            //해당 객체를 new로 생성하지 않고도 클래스에서 바로 불러낼 수 있음
            private Settings settings = Settings.getSettings();
            
            public void setAndPrintSetting(){
                settings.setDarkMode(true); //정적 settings 객체에 값을 넣음
                settings.setFontsize(15);
                
                System.out.println(settings.getDarkMode()+" "+settings.getFontSize());//그 내용들을 출력함
            }
        }

        ```
        - **MyProgram.java**

        ```java
        package singleton;

        public class MyProgram {

            public static void main(String[] args) {
                new FirstPage().setAndPrintSetting();
                new SecondPage().printSettings();
            }

        }
        ```
    
    근데... 그냥 정적변수를 쓰지 왜 싱글턴을 쓸까? 그건 바로 interface의 사용, lazy loading 등 싱글턴으로 할 수 있는 것들이 더 많기 때문이다!
2. **인스턴스화 불가**로 만들 수 있다.  
   ex) 위 소스코드의 FirstPage 클래스에서 `private Settings settings = new Settings();`로 인스턴스화 불가
3. 불변 값 클래스에서 **동치인 인스턴트가 하나임을 보장**할 수 있다. (`a==b`일 때, `a.equlals(b)`가 성립)  
   인스턴트 통제는 플라이웨이트 패턴의 근간이 되며, 열거타입은 인스턴스가 하나만 만들어짐을 보장한다.

##### 플라이웨이트 패턴(Flyweight pattern) 
어떤 클래스의 인스턴스 한 개만 가지고 여러 개의 "가상 인스턴스"를 제공하고 싶을 때 사용하는 패턴.  
즉, 인스턴스를 가능한 대로 공유시켜 쓸데없이 new 연산자를 통한 메모리 낭비를 줄이는 방식.  
디자인 패턴 GoF 중 `공유(Sharing)을 통하여 대량의 객체를 효과적으로 지원하는 방법`

대표적인 예 - Java의 String Pool
- 동일한 문자열에 대해서는 다시 사용될 때에 새로운 메모리를 할당하는 것이 아니라 String Pool에 있는지 검사해서 가져옴.
- String을 생성할 때 new를 통해 String을 생성하면 Heap영역에 존재하게 되고 리터럴을 이용할 경우 String constant pool이라는 영역에 존재하게 된다.

1.New 연산자를 이용한 방식

```java
public static void main(String[] args) {
    String a = new String("A");
    String b = new String("A");

    System.out.println(a == b);      //주소값 비교
    System.out.println(a.equals(b)); //주소 안에 들어 있는 값 비교
}

false
true
```
**<span style="color:red">new 연산자</span>**를 사용하여 새로운 String 객체를 생성하면 JVM에서 **<span style="color:red">Heap영역</span>**에 String 객체를 생성하게 된다.
그렇기 때문에 a를 Heap 메모리에 생성하고, b를 또 Heap 메모리에 생성하고 이 둘은 다른 객체가 된다.

2.리터럴을 이용한 방식

```java
public static void main(String[] args) {
    String a = "A";
    String b = "A";

    System.out.println(a == b);      //주소값 비교
    System.out.println(a.equals(b)); //주소 안에 들어 있는 값 비교

true
true
```
new 연산자가 아닌 **<span style="color:red">리터럴("")</span>로 String 객체를 생성하면 JVM은 우선 <span style="color:red">String Contstrant Pool 영역</span>을 방문하게 됩니다. 거기서 같은 값을 가진 String 객체를 찾으면 그 객체의 주소 값을 반환하여 참조하게 됩니다. 찾지 못하면 String Constrant Pool에 해당 값을 가진 String 객체를 생성하고 그 주소 값을 반환하게 됩니다.** String constrant Pool 영역은 Heap 영역 내부에서 String 객체를 위해 별도로 관리하는 저장소입니다.  

리터럴로 a를 생성할 때 String Constrant Pool에 "A"라는 값을 가진 String 객체가 생성되었고, b 또한 리터럴로 생성되었기 때문에 String Constrant Pool에 방문해보았더니 a를 생성할 때 만들어준 "A"라는 값을 가진 String 객체를 발견한 것 입니다.  
결과적으로 a와 b는 같은 객체를 참조하고 있는 것 입니다.

구글링 중 너무 이미지로 잘 정리해주신 [by hyeraan님의 글](https://hyeran-story.tistory.com/123)이 있어 공유합니다.

![constrantPool](/assets\img/constrantPool.PNG)

#### 3. 반환 타입의 하위 객체를 반환할 수 있는 능력이 있다.
반환할 객체의 클래스를 자유롭게 선택할 수 있게하는 유연성이 있다.

### 참고
- [[디자인패턴/Design Pattern] Flyweight Pattern/플라이웨이트 패턴](https://lee1535.tistory.com/106)
- [객체지향 디자인패턴 1](https://www.youtube.com/watch?v=lJES5TQTTWE&t=91s)
- [[Java] equals()과 == 차이점, String Contrant Pool(상수풀)](https://hyeran-story.tistory.com/123)