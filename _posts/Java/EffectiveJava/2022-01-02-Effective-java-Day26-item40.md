---
title: '[Effective Java] Day 26 - Item 40 :: @Override 애너테이션을 일관되게 사용하라'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day26에서는 item 40에 대한 내용을 다룬다.

## Item 40 :: @Override 애너테이션을 일관되게 사용하라
### 1. 재정의하려는 모든 메서드에 @Override 애너테이션을 사용하자
@Override
- 메서드 선언에만 달 수 있다.
- 상위 클래스나 상위 인터페이스의 메서드를 재정의 했음을 뜻한다.
- 디폴트 메서드를 지원하기 시작하면서, 인터페이스 메서드를 구현한 메서드에도 @Override를 다는 습관을 들이면 시그니처가 올바른지 재차 확신할 수 있다.

메서드를 재정의할 때 습관적으로 @Override 애너테이션을 사용하자!  
컴파일러가 매개변수 타입 오류 또는 오타로 인한 실수를 잡아주면서 버그들을 예방해준다.

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram b){
        return b.first == first && b.second == second;
    }

    public int hashCode(){
        return 31*first+second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for(int i=0;i<10;i++){
            for(char ch = 'a';ch<='z';ch++){
                s.add(new Bigram(ch, ch));
            }
        }
        System.out.println(s.size());
    }
}
```
소문자 a부터 z까지 26번 X 10번 반복해 HashSet에 추가 → 그 집합의 크기를 출력
```java
260
```
- HashSet인데 260이 출력되는 이유?
  - equals 메서드를 **overriding이 아닌 overloading**했기 때문
  - Object의 equals를 재정의하기 위해서는 매개변수 타입을 Object로 해야하나  
    타입을 Bigram으로 만들어 Object의 equals와 별개인 equals를 새로 작성한 꼴이 됨
  - overriding이 되지 않아 Object의 equals로 객체 식별성을 확인
    - 같은 소문자를 가진 바이그램 10개를 다 다르게 인식

@Override 애너테이션을 추가해보자
```java
@Override
public boolean equals(Bigram b){
    return b.first == first && b.second == second;
}
```
![item40-override-warning](/assets\img/item40-override-warning.PNG)

컴파일러가 잘못된 부분을 알려주므로 실수를 예방할 수 있다.

```java
@Override
public boolean equals(Object o){
    if(!(o instanceof Bigram))
        return false;
    Bigram b = (Bigram) o;
    return b.first == first && b.second == second;
}
```

### 2. 예외) 구체클래스에서 상위 클래스의 추상메서드를 재정의 한 경우
구체 클래스에서 @override 애너테이션을 추가하지 않아도 추상메서드를 구현하지 않았다고 컴파일러가 바로 알려준다.
그러므로 @override 애너테이션을 달지 않아도 된다. 물론 달아도 상관없다.
![item40_abstract_warning](/assets\img/item40_abstract_warning.PNG)

**IDE 활용하기**
- 대부분 IDE는 재정의할 메서드 선택하면 @Override를 자동으로 붙여준다.
![item40-IntelliJ-select-methods](/assets\img/item40-IntelliJ-select-methods.PNG)
- 아래 설정을 활성화해놓으면 @Override 달려있지 않은 메서드가 실제로 재정의 할 경우 경고를 준다.
![item40-override-inspection](/assets\img/item40-override-inspection.PNG)

## 결론
- **재정의하려는 모든 메서드에 @Override 애너테이션을 사용하자**
  - 그럼 실수를 컴파일러가 바로 알려줄 것!
- **예외) 구체클래스에서 상위 클래스의 추상메서드를 재정의 한 경우**
  - 이 경우는 애너테이션을 달지 않아도 된다. 하지만 헷갈리지 않기 위해서 이 경우에도 @override 애네터이션을 추가해주는 것이 좋다고 생각한다.