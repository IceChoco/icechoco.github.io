---
title: '[Effective Java] Day 22 - Item 21, 22, 23 :: 인터페이스는 구현하는 쪽을 생각해 설계하라, 인터페이스는 타입을 정의하는 용도로만 사용하라, 태그 달린 클래스보다는 클래스 계층구조를 활용하라'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day22에서는 item 21, 22, 23에 대한 내용을 다룬다.

## Item 21 :: 인터페이스는 구현하는 쪽을 생각해 설계하라
인터페이스에서 새로운 메소드를 추가하면 기존의 구현체들에 영향이 가게 된다.
이러한 문제를 해결하여 새로운 메소드를 추가할 수 있도록 한 것이 자바 8 부터 등장한 **default 메소드**이다.
### default 메소드
```java
public interface JavaStudyable { // ~able 네이밍 규칙을 가진다.
    public abstract void study(); //일반적인 메서드 선언

    public default void reset(){ //default 메서드 선언 후 구현
        System.out.println("쉽시다아");
    }
}
```

default 메서드를 사용함으로써 구현체에 영향을 주지 않고 처음 설계를 자유롭게 변경하면 좋겠지만, 책에서는 default 메서드를 남발하지 말라고 되어있다.  
default 메서드들이 어떻게 사용되는지 알아보고 default 메서드를 지양하는 이유를 알아보자.

### default 메소드 사용 예시
```java
public interface JavaStudyable { // ~able 네이밍 규칙을 가진다.
    public abstract void study(); //일반적인 메서드 선언

    public default void reset(){ //default 메서드 선언 후 구현
        System.out.println("쉽시다아");
    }
}

class RealTest implements JavaStudyable{
    @Override
    public void study(){
        System.out.println("공부합시당");
    }

    public static void main(String[] args) {
        RealTest rt = new RealTest();
        rt.study();
        rt.reset();
    }
}
```
```
공부합시당
쉽시다아
```
default 메소드인 reset을 따로 RealTest class에서 구현하지 않았지만 오류없이 해당 메소드가 실행된다.  
인터페이스에 새로운 default 메서드가 추가되어도 새로운 default 메서드를 구현하지 않은 기존 구현체에 영향을 주지 않는 다는 것을 알 수 있다.

```java
public interface JavaStudyable { // ~able 네이밍 규칙을 가진다.
    public abstract void study(); //일반적인 메서드 선언

    public default void reset(){ //default 메서드 선언 후 구현
        System.out.println("쉽시다아");
    }
}

class RealTest implements JavaStudyable{
    @Override
    public void study(){
        System.out.println("공부합시당");
    }

    public static void main(String[] args) {
        RealTest rt = new RealTest();
        rt.study();
        rt.reset();
    }
}
```
```
공부합시당
신나는 15분의 휴식시간!
```
default 메서드를 Override하여 재정의하는 것도 가능하다.

### default 메소드가 추가된 인터페이스를 사용 시 문제점
**1. 구현체는 default 메서드가 추가 되었는지 모른다.**  
해당 인터페이스를 가져다 쓰는 개발자는 Override 하지 않아도 실행되는 default 메서드에 의해 의도하지 않은 결과를 얻을 수 있다.  
**2. 문제점이 런타임 시에 발견될 수 있다.**  
default가 아닌 메소드는 컴파일 시점에 잘못 구현됨을 알 수 있지만 default 메소드는 런타임시에 알아차릴 수 있다.  
**3. 모든 상황에서 불변식을 해치지 않는다고 보장할 수 없다.**  
자바 7까지는 "현재의 인터페이스에 새로운 메소드는 추가될 일은 영원히 없다"라는 가정하에 개발되어 왔다.  
ex) 자바 8 Collection 인터페이스에 추가된 default removeIf 메소드  
```java
public interface Collection<E> extends Iterable<E> {
    ...
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
    ...
}    
```
org.apache.commons.collections4.collection.SynchronizedCollection는 클라이언트가 제공한 객체에 `lock`을 거는 능력을 추가하여 스레드간 동기화를 보장하는 클래스다.  
하지만 4.3 버전 이하에서는 removeIf 메소드가 Override 되질 않고 default 형태의 removeIf 메소드가 그대로 사용되어 동기화가 이루어질 않는다. 멀티스레드 환경에서 해당 메소드를 사용하게 되면 ConcurrentModificationException 이 발생하거나 예기치 못한 결과를 얻을 수 있다.  
이를 해결하기 위해서는 아래와 같이 default removeIf 호출전 동기화 작업을 해주어야 한다(자바 플랫폼 라이브러리가 이렇게 조치를 취함)
```java
public class Collections {
    static class SynchronizedCollection<E> implements Collection<E>, Serializable {
        @Override
        public boolean removeIf(Predicate<? super E> filter) {
            synchronized (mutex) {return c.removeIf(filter);}
        }
    }    
}    
```

## Item 22 :: 인터페이스는 타입을 정의하는 용도로만 사용하라
인터페이스는 타입을 정의하는 용도로만 사용해야한다. 상수 공개용 수단으로 사용하지 말자.  

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.  
클래스가 어떤 인터페이스를 구현한다는건, 인스턴스로 무엇을 할 수 있는지를 클라이언트를 알려주는 행위  

### 안티패턴 - 사용 금지
메서드 없이, 상수를 뜻하는 static final 필드로만 가득찬 인터페이스
```java
public interface PhysicalConstants {
    // 아보가드로 수
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    // 볼츠만 상수
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    // 전자 질량
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

#### 문제점
- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아닌 내부 구현이다.
    - 내부 구현을 클래스의 API로 노출하는 행위가 됨.
```java
// 인터페이스에서 protected도 불가!! : 제어자 'protected'은(는) 허용되지 않습니다
protected static final double AVOGADROS_NUMBER = 6.002_140_857e23;
```
- 사용자에게 혼란을 줌.
    - 사용자에게는 아무런 의미가 없다.
- 다음 릴리즈에 사용하지 않더라도 바이너리 호환성을 위해 여전히 상수 인터페이스를 구현하고 있어야 한다.
    - 다음 릴리즈에 call에서 AVOGADROS_NUMBER를 사용하고 있지 않더라도 삭제하면 main의 myResult 할당부에서 에러 발생

```java
public class Calculator implements PhysicalConstants {
    public double call(int input){
        return input * AVOGADROS_NUMBER;
    }
    public double call2(int input){
        return input * BOLTZMANN_CONSTANT;
    }
    public double call3(int input){
        return input * ELECTRON_MASS;
    }

    public static void main(String[] args) {
        Calculator calculator = new Calculator();
        double myResult = 100 * Calculator.AVOGADROS_NUMBER;
        System.out.println(myResult);
    }
}
```
- final이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위 클래스의 이름공간이 그 인터페이스가 정의한 상수들로 오염되어 버린다.

### 대안
상수를 공개할 목적이라면 몇 가지 대안이 있다.  
**1. 클래스나 인터페이스 자체에 추가**  
ex) Integer와 Double에 선언된 MIN_VALUE와 MAX_VALUE 상수
```java
public final class Integer extends Number implements Comparable<Integer>, Constable, ConstantDesc {
    @Native public static final int MIN_VALUE = 0x80000000;
    @Native public static final int MAX_VALUE = 0x7fffffff;
}
```
**2. 상수 유틸리티 클래스 제공**
```java
public class PhysicalConstants {
    //인스턴스화 방지
    private PhysicalConstants(){}

    // 아보가드로 수
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    // 볼츠만 상수
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    // 전자 질량
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}

public class Calculator{
    public double call(int input){
        return input * PhysicalConstants.AVOGADROS_NUMBER;
    }
    public double call2(int input){
        return input * PhysicalConstants.BOLTZMANN_CONSTANT;
    }
    public double call3(int input){
        return input * PhysicalConstants.ELECTRON_MASS;
    }

    public static void main(String[] args) {
        Calculator calculator = new Calculator();
        double myResult = 100 * PhysicalConstants.AVOGADROS_NUMBER;
        System.out.println(myResult);
    }
}
```
**3. enum 활용**
```java
public enum PhysicalConstants {
    AVOGADROS_NUMBER(6.022_140_857e23),
    BOLTZMANN_CONSTANT(1.380_648_52e-23),
    ELECTRON_MASS(9.109_383_56e-31);

    private double value;

    PhysicalConstants(double value) {
        this.value = value;
    }

    public double getValue(){
        return value;
    }
}

public class Calculator{
    public double call(int input){
        return input * PhysicalConstants.AVOGADROS_NUMBER.getValue();
    }
    public double call2(int input){
        return input * PhysicalConstants.BOLTZMANN_CONSTANT.getValue();
    }
    public double call3(int input){
        return input * PhysicalConstants.ELECTRON_MASS.getValue();
    }

    public static void main(String[] args) {
        Calculator calculator = new Calculator();
        double myResult = 100 * PhysicalConstants.AVOGADROS_NUMBER.getValue();
        System.out.println(myResult);
    }
}
```

## Item 23 :: 태그 달린 클래스보다는 클래스 계층구조를 활용하라
### 태그 달린 클래스
> **태그 클래스란**  
> 두가지 이상의 의미를 표현할 때 그 중 현재 표현하는 의미를 태그값으로 알려주는 클래스

```java
class Figure {

    enum Shape {
        RECTANGLE, CIRCLE
    }

    // 태그 필드
    final Shape shape;

    // RECTANGLE 용 필드
    double length;
    double width;

    // CIRCLE 용 필드
    double radius;

    // RECTANGLE
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    // CIRCLE
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

#### 태그 클래스의 단점
1. Enum 타입 선언, 태그 field인 shape, switch문 등 여러 구현이 혼합 되어 있다.
2. 이로 인해 가독성이 떨어진다.
3. 사용하지 않는 필드로 인해 메모리도 추가적으로 사용한다.
4. 또 다른 타입을 추가하게 되면 switch 문의 수정이 일어난다.
5. 인스턴스 타입만으로 객체의 의미를 알 수 없다.

태그가 달린 클래스는 코드가 길어지고 확장성에 취약하다.  
또한, 태그가 추가 될 때마다 사용하지 않는 필드들이 추가 될 수 있다.

#### 계층 구조의 클래스를 만드는 방법
1. 계층구조의 최상위(root)가 될 추상클래스를 정의한다.
2. 태그값에 따라 달라지는 동작(메서드)들을 최상위 클래스의 추상 메서드로 선언한다.
3. 태그값에 상관없이 동작이 일정한 메서드는 최상위 클래스에 일반 메서드로 정의한다.
4. 모든 하위 클래스에 공통으로 사용하는 상태값(필드)들은 루크 클래스에 정의한다.

#### 계층구조로 변환
```java
abstract class Figure {
    abstract double area();
}

class Rectangle extends Figure {

    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }

    @Override
    double area() {
        return length * width;
    }
}


class Circle extends Figure {

    final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}
```

### 클래스 계층구조의 장점
1. 독립된 의미를 가지는 상태값(필드)들이 제거 되어 각 구현 클래스는 간결해진다
2. 살아남은 field는 모두 final이므로 정의할 수 있다
3. 실수로 빼먹은 case 문으로 인해 runTime 에러를 방지할 수 있다
4. 최상위 클래스를 수정하지 않고 타입을 확장 할 수 있다
5. 타입사이의 자연스러운 계층 관계를 반영할 수 있다

### 결론
* 태그 달리 클래스를 써야 하는 상황은 거의 없다
* 새로운 클래스가 태그가 필요하다면 계층구조를 고려해보자
* 기존 클래스가 태그를 사용하고 있다면 계층 구조로 리팩토링을 고민해보자