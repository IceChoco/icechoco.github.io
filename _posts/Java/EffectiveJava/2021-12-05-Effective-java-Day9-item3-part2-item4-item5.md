---
title: '[Effective Java] Day 9 - Item 3, 4, 5 :: Private 생성자나 열거 타입으로 싱글턴임을 보증하라(3), 인스턴스화를 막으려거든 private 생성자를 사용하라, 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라'
layout: post
categories: java
tags: java
comments: true
---

Day 8에서는 Comparable 구현에 대해서 공부했다. 이번 Day 9에서는 item 3를 알아본 Day 7에 이어서 뒷부분인 직렬화에 대해 알아보겠다. Day 9 기록 시작!

## Item 3 :: Private 생성자나 열거 타입으로 싱글턴임을 보증하라 (3)
### 직렬화 (Serialization)
[Day 7]([https://icechoco.github.io/java/2021-12-03-Effective-java-Day7-item3/)에서 살펴본 public static final 필드 방식, 정적 팩터리 방식의 싱글턴을 모두 직렬화하려면 역 직렬화할 때 마다 새로운 인스턴스가 만들어진다. 이 문제를 해결하려면 모든 인스턴스 필드에 `transient`를 추가(직렬화 하지 않겠다)하고 `readResolve` 메소드를 다음과 같이 구현하면 된다.

```java
//싱글턴임을 보장해주는 readResolve 메서드
private Object readResolve(){
    //'진짜' Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
    return INSTANCE;
}
```

### 3. Enum 방식의 싱글턴
직렬화/역직렬화 할 때 코딩으로 문제를 해결할 필요도 없고, 리플렉션으로 호출되는 문제도 고민할 필요도 없고, 코드도 간결한 방법이 있다.
```java
public enum Singleton3 {
    INSTANCE;

    public String getNmae(){
        return "Ara";
    }
}
```
위 처럼 선언한 뒤 객체를 가져올 때 아래와 같이 가져다 쓰면 된다.
```java
String name = Singleton3.INSTANCE.getNmae();
```
코드는 좀 불편하게 느껴지지만 싱글턴을 구현하는 최선의 방법이다. 하지만 이 방법은 Enum말고 다른 상위 클래스를 상속해야한다면 사용할 수 없다. 그러나 인터페이스는 구현할 수 있다.

## Item 4 :: 인스턴스화를 막으려거든 private 생성자를 사용하라
### 유틸리티 클래스
- 인스턴스 메서드와 인스턴스 변수를 일절 제공하지 않고, **정적 메서드와 변수만을 제공하는 클래스**를 뜻한다.
- 클래스 본래의 목적인 '데이터와 데이터 처리를 위한 로직의 캡슐화'를 실행하는 것이 아닌, '비슷한 기능의 메서드와 상수를 모아서 캡슐화'한 것이 유틸리티 클래스이다.
#### 배열을 입력 받아 최솟값과 최댓값을 구하는 유틸리티 클래스 작성
```java
public class MinMax {

    public static int min(int[] a) {
        int min = 0;
        for (int i=0; i<a.length; i++) {
            min = a[0];
            if (min>a[i]) min=a[i];
        }  //for
        return min;    }  //min

    public static int max(int[] a) {
        int max = 0;
        for (int i=0; i<a.length; i++) {
            max = a[0];
            if (max<a[i]) max=a[i];
        }  //for
        return max;    }  //max
}  //MinMa
```
#### 실행클래스
```java
public class MinMaxTester {

    public static void main(String[] args) {
        int[] x = {100, 70, 1, 30, 64};
        System.out.println("배열 x의 최댓값은: "+ MinMax.max(x));
        System.out.println("배열 x의 최솟값은: "+ MinMax.min(x));
    }
}
```
#### 결과
```
배열 x의 최댓값은: 100
배열 x의 최솟값은: 64
```

정적 메서드와 정적 필드만을 담은 유틸리티 클래스를 남용하는 경우가 많지만 그래도 유용하게 쓰이는 경우가 있다.
- 예시
  - java.lang.Math, java.util.Arrays처럼 기본 타입 값이나 배열 관련 메서드들을 모아놓은 것
  - java.util.Collections처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아 놓은 것

정적 메서드와 정적 필드만을 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 게 아니다. 추상 클래스를 만드는 것만으로는 인스턴스화를 막을 수 없다.
```java
public abstract class UtilClass {

    public static String getName(){
        return "IceChoco";
    }

    static class AnotherClass extends UtilClass{

    }

    public static void main(String[] args) {
        //UtilClass에 abstract를 붙이면 1차적으로 아래처럼 인스턴스화 할 수 없긴함
        //UtilClass utilClass = new UtilClass();//가 아닌
        UtilClass.getName();//으로 사용되길 원함

        //AnotherClass의 메서드는 사용할 수 없지만 인스턴스화는 할 수 있음. 정말 쓸모없는 인스턴스...
        //의미 없는 인스턴스를 아래처럼 만들 수 있는 가능성을 배제하기 위해 private생성자를 만들라고 권고한다.
        AnotherClass anotherClass = new AnotherClass();

    }

}
```
위 소스코드처럼 UtilClass 클래스를 상속받아서 하위 클래스 AnotherClass를 만든 뒤 인스턴스화 할 수 있기 때문이다.  
그리고 생성자를 명시하지 않으면 컴파일러가 자동으로 아무 인자가 없는 public 생성자를 만들어주기 때문에 그런 경우에도 인스턴스를 만들 수 있다.  

**private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.**
- 인스턴스를 막을 수 없는 유틸리티 클래스
```java
//인스턴스화 불가 class
public class UtilClass {
    //유틸 클래스라 인스턴스를 만들지 못하게 막았습니다.
    private UtilClass() {
        throw new AssertionError();
    }
    //...
}
```
  
명시적 생성자가 private이니 UtilClass 바깥에서는 접근할 수 없다.  

꼭 AssertionError 반환이 필요하진 않지만, 혹시라도 클래스 안에서 실수로 생성자를 호출하지 못하도록 해준다.
  
생성자를 제공하지만 호출할 수 없어 코드가 직관적이지 않다. 그러므로 적절한 주석을 추가해주자.  

추가로 이방식은 상속을 불가능하게 하는 효과도 있다. 상속을 하게 되면 묵시적이든 명시적이든 상위 클래스의 생성자를 호출하게 되는데, 이 클래스의 생성자가 private이라 호출이 막혀 상속을 할 수 없다.

## Item 5 :: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
많은 클래스는 하나 이상의 자원에 의존한다. 이 책에서는 맞춤법 검사기인 `SpellChecker`와 `dictionary`를 예로 들고있다. 즉, `SpellChecker`가 `dictionary`를 사용하고, 이를 의존 하는 리소스 또는 의존성이라고 부른다. 이때 `SpellChecker`를 다음과 같이 구현하는 경우가 있다.

### 부적절한 구현
#### static 유틸 클래스
```java
public class SpellChecker {

    //한국 사전으로 고정되어있음
    private static final Lexicon dictionary = new KoreanDictionary();

    private SpellChecker(){
        // 객체생성방지
    }

    public static boolean isValid(String word){
        throw new UnsupportedOperationException();
    }

    public static List<String> suggestions(String typo){
        throw new UnsupportedOperationException();
    }

    public static void main(String[] args) {
        SpellChecker.isValid("hello");
    }
}

interface Lexicon{}

class KoreanDictionary implements Lexicon{}
```
위소스 내 모든 유틸리티 메소드는 public한 static으로 만들어져 있다. 이 메소드들에서 dictionary을 참조해야 하므로 생성자도 static 타입으로 생성해야 한다. 생성자는 `new koreanDictionary()`을 통해 만들어 final로 지정했기 때문에 고정이 되어 변경하기가 힘들다.  
  
SpellChecker 클래스를 테스트 할 때도 dictionary까지 테스트하게된다. 이렇게 되면 유연하지 않고 테스트하기 어렵다는 단점이 있다.

#### 싱글턴 클래스(정적 팩터리 방식)
```java
public class SpellChecker {

    //한국 사전으로 고정되어있음
    private final Lexicon dictionary = new KoreanDictionary();

    private SpellChecker(){
        // 객체생성방지
    }

    public static final SpellChecker INSTANCE = new SpellChecker(){
    };

    public static boolean isValid(String word){
        throw new UnsupportedOperationException();
    }

    public static List<String> suggestions(String typo){
        throw new UnsupportedOperationException();
    }

    public static void main(String[] args) {
        SpellChecker.INSTANCE.isValid("hello");
    }
}

interface Lexicon{}

class KoreanDictionary implements Lexicon{}
```
사전을 하나만 사용할거라면 위와 같은 구현도 만족스러울 수 있겠지만, 실제로는 각 언어의 맞춤법 검사기는 사용하는 사전이 각기 다르다. 또한 테스트 코드에서는 테스트용 사전을 사용하고 싶을 수도 있다.  

위 소스코드가 부적절하고 유연하지 않은 이유는 언어가 바뀌면 위 소스코드에서 `new KoreanDictionary()`이 부분이 바뀌어야 하기 때문이다.

**어떤 클래스가 사용하는 자원에 따라 동작이 달라지는 클래스에는 스태틱 유틸리티 클래스와 싱글톤을 사용하는 것은 부적절하다.**

이런 경우 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식이 낫다. 이는 의존 객체 주입의 한 형태로, 맞춤법 검사기를 생성할 때 의존객체인 사전을 주입해주면 된다.

#### 적절한 구현
```java
public class SpellChecker {

    private Lexicon dictionary;

    //생성자에 의존객체인 사전을 주입
    public SpellChecker(Lexicon dictionary){
        this.dictionary = Objects.requireNonNull(dictionary);
    };

    public static boolean isValid(String word){
        throw new UnsupportedOperationException();
    }

    public static List<String> suggestions(String typo){
        throw new UnsupportedOperationException();
    }

    public static void main(String[] args) {

    }
}

interface Lexicon{}

class KoreanDictionary implements Lexicon{}
```

### 참고
- [[이팩티브 자바] #3 싱글톤을 만드는 여러가지 방법 그중에 최선은?](https://www.youtube.com/watch?v=xBVPChbtUhM&t=534s)
- [[이팩티브 자바] #4 인스턴스를 못만들게 하고 싶다면?](https://www.youtube.com/watch?v=A-t1T3_m15M)
- [Java15 클래스 변수, 클래스 메서드와 유틸리티 클래스](https://morningcoding.tistory.com/entry/Java15-%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%B3%80%EC%88%98-%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%A9%94%EC%84%9C%EB%93%9C%EC%99%80-%EC%9C%A0%ED%8B%B8%EB%A6%AC%ED%8B%B0-%ED%81%B4%EB%9E%98%EC%8A%A4)