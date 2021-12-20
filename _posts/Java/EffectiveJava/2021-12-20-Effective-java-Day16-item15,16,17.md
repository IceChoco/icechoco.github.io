---
title: '[Effective Java] Day 16 - Item 15, 16, 17 :: 클래스와 멤버의 접근 권한을 최소화하라(2), Public 클래스에서는 Public 필드가 아닌 접근자 메서드를사용하라, 변경가능성을 최소화 하라'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day16에서는 item 15, 16, 17에 대한 내용을 다룬다.

## Item 15 :: 클래스와 멤버의 접근 권한을 최소화하라(2)
자바는 정보 은닉을 위한 다양한 도구를 제공한다. 그 중 접근 제한자(private, protected, public)를 이용해 클래스, 인터페이스, 멤버에 대한 접근성(접근 허용 범위)을 명시한다.  

정보 은닉을 높은 수준으로 유지하기 위해서는 **모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.** 

### 접근제어자
멤버(필드, 메서드, 중첩 클래스, 중첩 인터페이스)에 부여할 수 있는 접근 수준은 네 가지다.  
범위가 좁은 것부터 순서대로 살펴보자

|접근제한    | 적용대상                                    | 접근할 수 없는 클래스|
|-----------|---------------------------------------------|---------|
| private                  | 필드, 생성자, 메소드         | 모든 외부 클래스 |
| default(package-private) | 클래스, 필스, 생성자, 메소드  | 다른 패키지에 소속된 클래스 |
| protected                | 필드, 생성자, 메소드         | 자식 클래스가 아닌 다른 패키지에 소속된 클래스 |
| public                   | 클래스, 필드, 생성자, 메서드  | 없음 |

### 각 케이스에 따른 접근 제한자가 갖는 의미

1. (가장 바깥이라는 의미의) 톱레벨 클래스와 인터페이스에는 package-private과 public 두 가지 접근 수준을 가질 수 있다.
  - public으로 선언하면 그 패키지의 공개 API가 되므로 하위 호환을 위해 영원히 관리해줘야만 한다.
  - package-private으로 선언하면 해당 패키지 안에서만 이용할 수 있으므로 내부구현이 되어 언제든 수정할 수 있다. 그러므로 가능하면 package-private을 사용하자.
2. package-private 톱레벨 클래스나 인터페이스가 한 클래스에서만 사용된다면, 이를 사용하는 클래스 안에 private static으로 중첩시켜보자 (Item 24)
  - 톱레벨로 두면 같은 패키지의 모든 클래스가 접근할 수 있지만, private static으로 중첩시키면 바깥 클래스 하나에서만 접근할 수 있다.
3. 클래스의 공개 API를 설계한 후, **공개 API를 제외한 모든 멤버는 private으로** 만들자. 그런 다음 오직 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한해서만 package-private으로 풀어주자. <u>권한을 풀어주는 일을 자주 하게 된다면 시스템에서 컴포넌트를 더 분해해야 하는 것은 아닌지 다시 고민해보자.</u>
4. 코드를 테스트하려는 목적으로 클래스, 인터페이스, 멤버의 접근 범위를 넓히려 할 때가 있다. **public 클래스의 private 멤버를 package-private 까지  풀어주는 것은 허용할 수 있지만, 그 이상은 피하자.** (테스트 코드를 테스트 대상과 같은 패키지에 두면 pakcage-private 요소에 접근할 수 있다.)
5. public 클래스의 상수용 public static final 필드 외의 인스턴스 필드는 되도록 public이 아니어야 한다. (Item 16) final이 아닌 인스턴스 필드를 public으로 선언하면 불변식을 보장할 수 없게 된다. 또한 필드를 제거하는 것과 같은 리팩터링을 쉽게 할 수 없게 된다.
  - public static final 필드가 참조하는 객체도 불변인지 확인해야 한다.
6. **길이가 0이 아닌 배열은 모두 변경 가능하니 주의**하자. 클라이언트에서 배열의 내용을 수정할 수 있기 때문에 클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다.
```java
// 보안 허점이 숨어 있다.
public static final Thing[] VALUES = { ... };
```
```java
// 방법1. public 배열을 private으로 만들고 public 불변 리스트를 추가해서 사용할 수 있다.
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES = 
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```
```java
// 방법2. 배열을 private으로 만들고 그 복사본을 반환하는 public 메서드를 추가하는 방법이다. (방어적 복사)
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}
```

자바 9에서는 모듈 시스템이라는 개념이 도입되면서 두 가지 암묵적 접근 수준이 추가되었다. 패키지가 클래스의 묶음이듯, **모듈은 패키지들의 묶음이다.** 모듈은 자신이 속하는 패키지 중 공개(export)할 것들을 module-info.java 파일에 선언한다. **protected 혹은 public 멤버라도 해당 패키지를 공개하지 않았다면 모듈 외부에서는 접근할 수 없다.**

모듈 시스템을 활용하면 클래스를 외부에 공개하지 않으면서도 같은 모듈을 이루는 패키지 사이에서는 자유롭게 공유할 수 있다. 위의 암묵적인 접근 수준은 이처럼 public 클래스의 public 혹은 protected 멤버의 접근 범위가 모듈 내부로 한정되는 것을 말한다.

<u>이런 접근 수준을 적극 적으로 활용하고 있는 대표적인 예</u>가 바로 **JDK 자체**다. 그러나 모듈의 장점을 제대로 누리면서 사용하기 위해서는 해야 할 일들이 많다. 패키지들을 모듈 단위로 묶고, 모듈 선언에 패지키들의 모든 의존성을 명시해야 하는 등... 여러 작업이 수반된다. 아직까지는 JDK 외에도 모듈 개념이 널리 받아들여져 사용될지에 대하서는 예측하기 이른감이 있으므로 꼭 필요한 경우가 아니라면 당분간은 사용하지 않는 것을 권장한다.

## Item 16 :: Public 클래스에서는 Public 필드가 아닌 접근자 메서드를사용하라
### 접근 제한자를 고려해서 개발해라.
코드 16-1 이처럼 퇴보한 클래스는 public이어서는 안 된다!
```java
// 아무런 목적이 없는 퇴보한 클래스이다.
class Point {
    public double x;
    public double y;
}
```

* 왜 퇴보한 클래스인가?
  * 데이터 필드에 직접 접근할 수 있기 때문에 캡슐화의 이점을 제공하지 못한다.
    * 캡슐화란? 
      * 간단하게 말하면 관련있는 변수랑 함수를 클래스로 묶고, 실제 구현 내용을 외부에 감추는 것
      * 여기서 말하는 이점은 은닉성을 말하는듯 하다(중요한 데이터나 기능을 외부에서 접근하지 못하게 하는것)
  * API를 수정하지 않고는 내부 표현을 바꿀 수 없다
  * 불변식을 보장할 수 없다.
  * 외부에서 필드에 접근할 때 부수 작업을 수행할 수 없다.

코드 16-2 접근자와 변경자(mutator) 메서드를 활용해 데이터를 캡슐화한다.
```java
// 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써
// 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }

    public void setX(double x) {
        this.x = x;
    }

    public void setY(double y) {
        this.y = y;
    }
}
```

* 예외 (이 경우 데이터 필드가 노출되어도 무관하다.)
  * package-private 클래스
  * private 중첩 클래스

#### 타산지석으로 삼아야하는 java.awt.package 패키지의 Dimension 클래스
class 내부 필드 들의 접근제한자가 public이다.  
![item16-NotGoodExam-for-public-class](/assets\img/item16-NotGoodExam-for-public-class.PNG)


#### 불변 필드를 노출한 public  클래스
public 클래스의 필드가 불변이라면, 필드를 public으로 직접 노출하는 것으로 인한 단점이 조금은 줄어들지만 그래도 좋은 생각이 아니다.  
* 문제점
  * api를 변경하지 않고는 내부 표현 변경 불가
  * 외부에서 필드 접근 시 부수 작업 수행 불가
```java
// class의 내부 필드들이 모두 불변이다(final 사용)
public final class Time {

    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOURS = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("시간: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOURS)
            throw new IllegalArgumentException(": " + minute);
        this.hour = hour;
        this.minute = minute;
    }
    ... // 나머지 코드 생략
}
```

### 결론
- public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
- 불변 필드라면 노출해도 위험성은 덜 하지만 완전히 안심할 수 없다.
- 예외적으로, package-private 클래스나 private 중첩 클래스에서는 불변/가변 필드를 노출하는 편이 나은 경우가 있다.

## Item 17 :: 클래스의 mutability(상태 변경 가능성)을 최소화하라(Minimize mutability)
### Immutable(불변) 클래스란?
 - 각 instance에 저장된 정보가, 객체의 일생동안 바뀌지 않는 클래스
 - ex)
   - String
   - 기본 타입의 박싱된 클래스들(Boxed primitive classes) - Boolean, Int, Float, ...
   - BigInteger, BigDecimal
   - 상수같은 것들 

### Immutable(불변) 클래스를 만드는 5가지 규칙
#### 1. 객체의 상태(state)를 변경하는 메소드(mutator, setter)를 제공하지 말라.
- C++엔 Const member function이 있다. 하지만 **Java는 C++처럼 객체의 불변화 관련 역할을 담당하는 기능이 없다.**
  - `const T tmp`와 같이 선언되면 tmp 객체는 재할당이 불가하고, public 필드의 변경이나 메서드를 통한 private 필드의 변경 등 const로 선언된 객체의 상태를 바꾸는 일도 불가능하다. const object의 상태를 변경하지 않는 const method만 호출할 수 있다.
  - C++의 const는 Java의 final과 달리 클래스의 상속과 관계가 없다.
- 그럼 Java는 어떻게 하지? **어떤 method도 필드 값을 변경하지 못하게** 하면 된다.

#### 2. 상속되지 못하게 하라.
- 상속한 뒤에 하위 클래스에서 실수로 또는 악의적으로 객체의 상태(state)를 변하게 만드는 사태를 막아버리자.
- 보통 클래스를 final로 선언하면 된다.
- 다른 방법도 뒤에 나온다.

#### 3. 모든 필드를 final로 선언하라.
- 설계자의 의도를 명확히 드러내는 방법
- 새로 생성된 객체의 참조를 동기화(synchronization) 없이 다른 스레드로 전달하기 위해서는 반드시 final을 써야한다([JLS 17.5 메모리 모델 부분](https://docs.oracle.com/javase/specs/jls/se17/html/jls-17.html#jls-17.5); Goetz06, 16)
  - 생성자에서 final 필드를 초기화하라
  - 생성자가 끝나기 전에는 다른 스레드가 접근 가능한 곳에 이 객체로의 참조를 두지 말아라
  ```java
  public class FinalFieldExample {
      final int x;
      int y;
      static FinalFieldExample f;

      public FinalFieldExample() {
          x = 3; //생성자에서 final 필드를 초기화하라
          y = 4;
          // 생성자가 끝나기 전에는 다른 스레드가 접근 가능한 곳에 이 객체로의 참조를 두지 말아라
      }

      static void writer(){
          //이 객체로의 참조는 생성자 밖 별도의 메소드에 따로 둬라
          f = new FinalFieldExample();
      }

      static void reader(){
          if(f!= null){
              int i = f.x; // 3임을 보장함
              int j = f.y; // 0일수도 있음!
          }
      }
  }
  ```

#### 4. 모든 필드를 private으로 선언하라.
필드를 `public final`로만 선언해도 불변 객체가 되긴 하지만, API가 되버리기 때문에 나중에 바꿀 수 없다.

#### 5. 가변 필드를 밖에서 접근하지 못하게 하라 (나의 가변 필드에 대한 참조를 아무도 가지지 못하게 하라)
- 어떤 필드가 가변 객체 참조를 하고 있다면
  - **클라이언트가 제공하는 참조로** 그 필드를 초기화(initialize)하면 **안된다.**
  - 그리고 접근자(accessor, getter) 메소드가 **참조를 리턴하면 안된다.**
- 그럼 어떻게 할까...?
  - 생성자, getter, readObject에서 방어적 복사(defensive copy)를 하라(item 50, item 88)
    - 생성자에서는 파라미터로 받은 걸 복사해서 넘기고
    - getter는 그 복사본을 넘겨야 한다.
- 필드가 만약 불변 객체를 참조하고 있다면 위는 신경쓰지 않아도 된다.

### EX: 불변 복소수 클래스(Immutable complex number class)
```java
// 코드 17-1 불변 복소수 클래스 (106-107쪽)
public final class Complex { // final 클래스
    private final double re; // private final 변수 re, im
    private final double im;

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    // 코드 17-2 정적 팩터리(private 생성자와 함께 사용해야 한다.) (110-111쪽)
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

### 참조
[Java 개발자가 배우는 C++ - 03 - const와 final](https://github.com/HomoEfficio/dev-tips/blob/master/Java%20%EA%B0%9C%EB%B0%9C%EC%9E%90%EA%B0%80%20%EB%B0%B0%EC%9A%B0%EB%8A%94%20C%2B%2B%20-%2003%20-%20const%EC%99%80%20final.md)