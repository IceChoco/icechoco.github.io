---
title: '[Effective Java] Day 10 - Item 5 :: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라 (2)'
layout: post
categories: java
tags: java
comments: true
---

Day 10 기록 시작!

## Item 5 :: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라 (2)
많은 클래스는 하나 이상의 자원에 의존한다. 이 책에서는 맞춤법 검사기인 `SpellChecker`와 `dictionary`를 예로 들고있다. 즉, `SpellChecker`가 `dictionary`를 사용하고, 이를 의존 하는 리소스 또는 의존성이라고 부른다.  
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

**의존객체 주입**은 유연성과 테스트 용이성을 높여준다. 또한 private을 통해 불변을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있다.  
또 생성자, 정적 팩터리(아이템 1), 빌더(아이템 2) 모두에 똑같이 응용할 수 있다.  

**팩터리 메서드 패턴**은 의존객체 주입의 변형으로, 생성자의 자원 팩터리를 넘겨주는 방식이다.  
- **팩터리**: 호출할 떄 마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체
- 대표적인 예: Suppleir<T> 인터페이스  

Supplier<T>를 입력으로 받는 메서드는 보통 한정적 와일드 카드 타입(bounded wildcard type)으로 입력을 제한해야한다.
이 방식을 사용해 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다.

```java
public class SpellChecker {

    private Lexicon dictionary;

    //생성자에 의존객체인 사전을 주입
    public SpellChecker(Supplier<Lexicon> dictionary){
        this.dictionary = Objects.requireNonNull(dictionary.get());
    };

    public static boolean isValid(String word){
        throw new UnsupportedOperationException();
    }

    public static List<String> suggestions(String typo){
        throw new UnsupportedOperationException();
    }

    public static void main(String[] args) {
        Lexicon lexicon = new KoreanDictionary();

//        SpellChecker spellChecker = new SpellChecker(new Supplier<Lexicon>() {
//            @Override
//            public Lexicon get() {
//                return lexicon;
//            }
//        });
        SpellChecker spellChecker = new SpellChecker(() -> lexicon); //위 소스를 이와 같은 람다식으로 수정
        spellChecker.isValid("hello");
    }
}

interface Lexicon{}

class KoreanDictionary implements Lexicon{}

class testDictionary implements Lexicon{}
```
의존 객체 주입이 유연성, 테스트 용이성을 개선해주긴하나 의존성이 수천 개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들 수 있다. 그 점은 대거, 주스, 스프링 같은 의존 객체 주입 프레임워크를 사용해서 해결할 수 있다.


## Item 6 :: 불필요한 객체 생성을 피하라
똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다. 재사용은 빠르고 세련됐다. 특히 불변 객체(아이템 17)는 언제든 재사용할 수 있다.
### 문자열 객체 생성
```java
String s = new String("Ara"); //따라하지 마세요!
String d = new String("Ara"); //따라하지 마세요!

System.out.println(s == d);//false
```
자바의 문자열인 String을 new로 생성하면 항상 새로운 객체를 만들게 된다. 다음과 같이 String 객체를 생성하는 것이 좋다.
```java
String name = "Ara";
String name2 = "Ara";

System.out.println(name == name2);//true
```
문자열 리터럴을 재사용하기 때문에 같은 자바 가상 머신 안에서 이와 똑같은 문자열 리터럴이 존재한다면 같은 객체를 재사용함이 보장된다.


### 결론
의존하는 리소스에 따라 클래스 동작을 달리 하는 클래스를 만들 때는 싱글턴과 정적 유틸리티 클래스를 사용하지말자. 그런 경우에는 리소스를 생성자나 정적 팩토리로 전달하는 의존성 객체 주입을 사용하여 유연함, 재사용성, 테스트 용이성을 향상시키자.

### 참고
- [[이팩티브 자바] #5 의존성 주입](https://www.youtube.com/watch?v=24scqT2_m4U&t=907s)