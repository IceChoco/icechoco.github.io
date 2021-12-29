---
title: '[Effective Java] Day 17 - Item 17 :: 변경가능성을 최소화 하라(2)'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day17에서는 item 17에 대한 내용을 다룬다.

## Item 17 :: 클래스의 mutability(상태 변경 가능성)을 최소화하라(Minimize mutability)
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

### 함수형 방식(functional approach)
- 사칙연산 메소드(plus, minus, times, divideBy)들을 보면 <u>절차적</u> 혹은 <u>명령형 방식</u>을 이용하여 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 생성하여 반환함
- **절차적(procedural) 방식**
    - 함수 내부에서 새로운 객체를 만들어 반환하는 방법 ex) return new Complex(re + c.re, im + c.im);
    - 변하지 않는 값과 함수로 이루어져 있음(변수 선언 금지)
    - 입력되는 파라미터로만 동작
    - 함수 외부의 값 사용 금지. 함수 내부에서 **임의의** 타 객체 생성 금지 
- **명령형(imperative) 방식**
    - 명령형 프로그래밍은 무엇을 어떻게 할 것인가에 가까운 방식. 선언형(Declarative)과 대조됨
- 메서드 이름도 add와 같은 동사가 아닌 **plus와 같은 전치사**를 사용
- 객체의 값을 변경하지 않는다는 사실을 강조하려는 의도
- BigInteger와 BigDecimal 클래스는 이러한 명명규칙을 따르지 않아 잘못된 사용을 유발함

```java
public Complex plus(Complex c) {
    return new Complex(re + c.re, im + c.im);
}
```

### 불변(immutable) 객체의 장점
1. 쓰기 쉽고 단순하다.
 - 생성자에서 불변식(invariants)을 성립시켜 놓으면 그 불변식들이 계속 보장된다.
 - 만들어질 때의 상태(state) 그대로, 오직 한 상태만을 가진다.
 - 가변객체라면 복잡한 상태공간(state space)를 가질 수 있다.
        - 각 setter 메소드가 유발하는 상태 전이(state transition)가 잘 문서화되지 않으면, 사용하기 어려울 수 있다.
2. 스레드 안전(Thread safe)하다.
 - 동기화(Synchronized)가 불필요하다.
3. 자유롭게 공유될 수 있다.
 - 다른 스레드에 영향 받을 일이 없으므로
4. 따라서 불변 클래스라면 사용자가 이미 존재하는 인스턴스를 재사용하도록 권장해라
 - 자주 쓰이는 값들을 상수(public static final)로 제공하라
 ```java
 public static final Complex ZERO = new Complex(0, 0);
 public static final Complex ONE = new Complex(1, 0);
 public static final Complex I = new Complex(0, 1);
 ```
 - 정적 팩터리(Static factory, item 1)에서 자주 사용되는 인스턴스를 캐싱하라
        - 박싱된 기본 타입(Boxed primitive) 클래스 전부와 BigInteger가 이렇게 한다.
        - 메모리 사용량과 가비지 컬렉션 비용이 줄어든다.
        - public 생성자 대신 static factory를 제공하면, 나중에 클라이언트 측 변경 없이 캐시 기능을 나중에 덧붙일 수 있다.
5. 방어적 복사(defensive copy)가 불필요하다(item 50)
 - 원본과 똑같으니 복사 자체가 의미가 없다.
 - <u>clone 메서드</u>나 <u>복사 생성자(copy constructor)</u>도 필요없다.
 - String의 copy constructor는 되도록 사용하지 말아라(item 6)
   ```java
   String s = new String("Ara"); //따라하지 마세요!
   ```
6. 내부 자료구조를 공유할 수 있다.   
 - ex) BigInteger는 내부적으로 부호를 int 변수 하나에, 값을 int 배열에 저장
 - negate 메서드는 크기가 같고 부호만 반대인 새로운 BigInteger 객체를 생성한다. 이 객체는 값이 저장된 int 배열을 복사하지 않고 원본 인스턴스와 공유해도 괜찮다.
    ```java
    BigInteger howMuch = new BigInteger("5500");
    System.out.println(howMuch.negate()); //result: -5500
    ```
7. 다른 객체를 만드는 좋은 빌딩 블록이 된다.
 - 복잡한 객체를 만들 때, 몇몇 구성요소들의 값이 바뀌지 않는다면 불변식을 유지하기 좋다.
 - ex) Map의 key들, set의 element들
        - 안에 담긴 값이 바뀌면 불변식이 허물어지는데, 불변 객체를 사용하면 그런 걱정을 하지 않아도 됨.
8. 실패 원자성(Failure atomicity)이 보장된다(item 76)
 - 실패 원자성: method가 Exception을 던지거나 실패한다 해도, 그 객체는 호출 전 상태를 유지하고 있어야 한다.

### 불변(immutable) 객체의 단점
#### 다른 값이 필요할 때마다 항상 별도의 객체를 만든다.
- 값의 가짓수가 많다면 이들을 모두 만드는데 비용이 클 수 있다.
- ex) 백만 비트짜리 BigInteger에서 비트 하나를 바꾸는 경우  
불변 객체이기 때문에 flipBit 메서드를 통해 원본과 단지 한 비트만 다른 백만 비트짜리 새로운 BigInteger 인스턴스를 생성한다.
```java
BigInteger moby = ...; //수백만 비트짜리
moby = moby.flipBit(0);
```
반대로 BigSet은 **가변**객체이므로 flip 메서드를 통해 원하는 비트 하나만 상수 시간안에 바꿔준다. BigInteger와 달리 매우 가벼운 연산이다.
```java
BigSet moby = ...;
moby.flip(0);
```
- 여러 단계의 오퍼레이션에서 매 단계마다 새로운 객체가 생성된다면 문제가 더 심각해진다.
  - ex
    - 여러 단계를 거치는 수학 계산
    - 문자열에 여러 개의 문자열을 덧붙이기
  - **해결책**
    1. 자주 있을법한 연산을 기본 기능(primitive)로 제공
     - 불변 객체는 내부적으로 스마트하게 만들 수 있다.
     - ex) BigInteger는 package-private한 가변 동반 클래스(companion class)를 내부적으로 사용하여 모듈러 지수 같은 복잡한 다단계 계산을 수행한다.
    2. 어떤 복잡한 다단계 연산이 필요할 지 미리 알 수 없다면, public 가변 동반 클래스를 제공한다.
     - ex) String class는 가변 동반 클래스로 StringBuilder를 제공한다.

#### 가변 동반 클래스 동작(사용) 방법 예시
`BigInteger`는 package-private한 **가변 동반 클래스(companion class)**를 내부적으로 사용하여 모듈러 지수 같은 복잡한 다단계 계산을 수행한다고 하였는데 어떻게 동작하는 것일까?
  
우선 간단한 예시로, 우리가 `StringBuilder` 클래스를 사용할 때 순서를 살펴보면 아래와 같다.
```java
// StringBuilder.java
@Override
public StringBuilder append(Object obj) {
    return append(String.valueOf(obj));
}

@Override
@HotSpotIntrinsicCandidate
public StringBuilder append(String str) {
    super.append(str);
    return this;
}

@Override
@HotSpotIntrinsicCandidate
public String toString() {
    // Create a copy, don't share the array
    return isLatin1() ? StringLatin1.newString(value, 0, count)
                      : StringUTF16.newString(value, 0, count);
}
```

1. `StringBuilder` 객체 인스턴스를 생성한다
```java
StringBuilder sb = new StringBuilder();
```
2. `String`을 하나씩 `append` 메서드를 통해 추가한다.
```java
sb.append("DdangGeulEe ");
sb.append("GomGomEe");
```
3. `toString` 메서드를 통해 `String` 결과를 얻는다.
```java
System.out.println(sb.toString());
```

위 StringBuilder의 예처럼 `BigInteger` 클래스 또한 `BigInteger` 클래스의 가변 동반 클래스인  
`BitSieve, SignedMutableBigInteger, MutableBigInteger`를 이용하여 클라이언트 코드에서 결과를 직접 만들고  
그 결과를 `BigInteger`로 캐스팅하는 방식을 이용할거라고 생각했다면 큰 오산이다.
  
`BitSieve, SignedMutableBigInteger, MutableBigInteger`와 같은 BigeInteger의 가변 동반 클래스들은 모두
`package-private` 접근제한을 이용하고 있다.  
이 때문에 클라이언트가 직접적으로 저 클래스들을 조작할 수 있는 방법이 없다.  
그럼 도대체 어떻게 사용할 수 있는 걸까? 방법은 간단하다.  
클라이언트 코드에서는 이 가변 동반 클래스들을 직접 사용할 수 없다. 대신 `BigInteger` 클래스의 메서드 안에서 이 가변 동반 클래스들을 사용한다.

```java
// 1. BigInteger 클래스의 gcd 메서드
public BigInteger gcd(BigInteger val) {
    if (val.signum == 0)
        return this.abs();
    else if (this.signum == 0)
        return val.abs();

    MutableBigInteger a = new MutableBigInteger(this);
    MutableBigInteger b = new MutableBigInteger(val);

    MutableBigInteger result = a.hybridGCD(b);

    return result.toBigInteger(1);
}

/*
 * 2. MutableBigInteger 클래스의 toBigInteger 메서드
 * Convert this MutableBigInteger to a BigInteger object.
 */
BigInteger toBigInteger(int sign) {    // <-- 접근제한자 package-private 에 주목
    if (intLen == 0 || sign == 0)
        return BigInteger.ZERO;
    return new BigInteger(getMagnitudeArray(), sign);
    // 이마저도 불변을 지키기 위해서 방어적 복사를 진행함
}
```

클라이언트 코드에서 `BigInteger`의 메서드를 사용하면, 그 내부에서 가변 동반 클래스들을 사용하고 있다.  
그리고 그 결과를 다시 `BigIntger`로 캐스팅한 후 반환하는 방식으로 구현되어있다.

### 불변 클래스를 만드는 몇 가지 설계 방법들
가장 쉽게 자신을 상속하지 못하게 하는 방법은 final 클래스로 선언하는 것이지만 더 유연한 방법이 있다.  
생성자를 `private`또는 `package-private`으로 만들고 public static factory를 제공하라(item 1)
```java
public class Complex {
    private final double re; // private final 변수 re, im
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    // 코드 17-2 정적 팩터리(private 생성자와 함께 사용해야 한다.) (110-111쪽)
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    //... 
}    
```
- 생성자가 protected나 public이 아니므로 사용자는 상속을 할 수 없다. 이는 클래스를 final로 하는 것과 같은 효과를 낸다.
- 하지만 `package-private`으로 생성자를 만드는 경우 패키지 내에서는 여러가지 구현을 사용할 수 있다.
- 또한 static factory method 내에서 object caghing 등이 가능하다.
- `BigInteger`나 `BigDecimal`을 설계할 당시에는 불변 클래스를 final로 만들거나 또는 그와 같은 효력이 있도록 만들어야 한다는 사실을 몰랐다.
- 하위호환성(backward compatibility)이 발목을 잡아 이제와서 고칠수도 없다.
- `BigInteger`나 `BigDecimal`의 인스턴스들이 불변이어야 클래스의 보안을 지킬 수 있다면, 이들을 방어적 복사(defensive copy)하는 것을 고려해라.
```java
public static BigInteger safaInstance(BigInteger val){
    return val.getClass() == BigInteger.class ?
            val : new BigInteger(val.toByteArray());
}
```

"어떤 메서드도 객체의 상태(state) 중 외부에 비치는 값을 변경할 수 없다"와 같이 규칙을 조금 완화할 수 있다.
- 예를 들어 계산 비용이 큰 값을 **non-final 필드에 캐시**해 놓을 수 있다. 이 방법은 불변 클래스이므로 가능하다.
- ex) **lazy initialization**: 지연 초기화
  - PhoneNumber의 hashCode(Item 11 p.71), String의 hashCode
  ```java
  private int hashCode; //자동으로 0으로 초기화된다.
  public int hashCode() {
        int result = hashCode;
        if(result == 0){
            result = Integer.hashCode(firstNumber);
            result = 31 * result + Integer.hashCode(secondNumber);
            result = 31 * result + Integer.hashCode(thirdNumber);
            hashCode = result;
        }
        return result;
  }
  ```
- 단 직렬화(Serializable)할려면 readObject나 readResolve를 따로 만들어주거나, ObjectOutputStream의 writeUnshared 및 readUnshared를 사용해야한다(Item 88). 기본 직렬화 방법(Default serialized form)이면 충분하더라도 꼭 그렇게 해야한다.

### 결론
- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
- 단순한 값 객체는 항상 불변으로 만들자
  - ex) PhoneNumber, Complex
  - java.util.Date나 java.awt.Point도 불변이어야 했지만 그렇지 않게 만들어졌다.
- 무거운 값 객체도 불변으로 만들 수 있는지 고심하자
  - ex) String, BigInteger
- 성능때문에 꼭 필요하다면 public 가변 동반 클래스(mutable companion class)를 제공하자(item 67)
  - ex) StringBuilder
- 불변으로 만들 수 없고 가변으로 만들어야 할 때에도, 변경할 수 있는 부분(state 변화)을 최소화해라.
  - 다른 합당한 이유가 없다면 모든 필드는 `private final`이어야 한다.(item15 + item 17)
- 생성자 안에서 객체의 초기화를 완벽히 끝내고 모든 불변식을 성립시켜야 한다.
  - 생성자나 static factory 이외의 public 초기화 메소드가 있어선 안된다.
  - 객체를 재사용할 수 있게 초기화하는 메소드도 제공하면 안된다. 성능향상은 미미하나 복잡하다.
  - ex) **CountDownLatch**
    - 가변 클래스지만 가질 수 있는 상태의 크기(state space)를 매우 작게 만들었음
    - 카운트가 0에 도달하면 더는 재사용할 수 없음
    ```java
    public class CountDownLatch {
        Sync(int count) {
            setState(count);
        }

        /**
        * Constructs a {@code CountDownLatch} initialized with the given count.
        *
        * @param count the number of times {@link #countDown} must be invoked
        *        before threads can pass through {@link #await}
        * @throws IllegalArgumentException if {@code count} is negative
        */
        public CountDownLatch(int count) {
            if (count < 0) throw new IllegalArgumentException("count < 0");
            this.sync = new Sync(count);
        }
    }    
    ```
- 책의 예제인 Complex 클래스는 그대로 사용하지 말라.
  - 실무에서 쓸만한 수준이 아니다.
  - 곱셉과 나눗셈에서 반올림이 되지 않고, 복소수 NaN과 무한대(infinity)도 올바르게 지원하지 않는다.

### 참조
- [Java 개발자가 배우는 C++ - 03 - const와 final](https://github.com/HomoEfficio/dev-tips/blob/master/Java%20%EA%B0%9C%EB%B0%9C%EC%9E%90%EA%B0%80%20%EB%B0%B0%EC%9A%B0%EB%8A%94%20C%2B%2B%20-%2003%20-%20const%EC%99%80%20final.md)
- [[아이템 17] 가변 동반 클래스 동작(사용) 예시 #15](https://github.com/JunHyeok96/effective-java/issues/15)