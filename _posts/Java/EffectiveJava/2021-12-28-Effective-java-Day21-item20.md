---
title: '[Effective Java] Day 21 - Item 20 :: 추상 클래스보다는 인터페이스를 우선하라'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day21에서는 item 20에 대한 내용을 다룬다.

## Item 20 :: 추상 클래스보다는 인터페이스를 우선하라

자바는 단일 상속만 지원하니, 추상 클래스 방식은 새로운 타입을 정의하는 데 커다란 제약을 안고 있다.  
반면 인터페이스는 여러개를 구현(implement) 할 수 있으며 어떤 클래스를 상속했든 같은 타입으로 취급된다.  
따라서 추상클래스 보다는 인터페이스를 우선으로 하여 구현하는것을 권장한다.

### 인터페이스의 장점
#### 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.
한가지만 상속할 수 있는 추상클래스와 달리 인터페이스는 제한이 없기 때문에 언제든지 기존에 작성된 클래스에 새로운 인터페이스를 추가할 수 있다.    
- **추상 클래스는 기존 클래스 위에 새로 끼워 넣기가 어렵다.**  
A, B 클래스가 같은 추상 클래스인 Abstract1을 확장하는 경우, Abstract1 클래스의 기존 자손이던 B, C, D, E까지 같이 묶여버린다.
C와 A의 공통조상이 Abstract1이 되는 것이 의미상 적절하지 않다면 클래스 계층구조에 큰 혼선을 유발한다.

#### 믹스인(mixin)정의에 안성맞춤이다.
믹스인이란, **상속관계를 만들지 않고 원하는 기능을 하위 클래스로 만드는 것**을 의미한다.  
즉, 주된 타입 외에도 특정 기능을 제공하는것을 믹스인이라 한다.  
java 에서는 `Comparable`을 예로 들 수 있는데, 인터페이스를 구현한 인터페이스는 인스턴스끼리 순서를 정할 수 있는 기능을 갖게 된다.  
예를들어 우리가 잘 아는 `Integer` 클래스를 보면 `Number` 인 동시에 다른 인스턴스들과 순서를 비교할 수 있는 `Comparable` 을 구현한 클래스이다.
```java
public final class Integer extends Number
        implements Comparable<Integer>, Constable, ConstantDesc {
    ...
}            
```

#### 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.
타입을 계층적으로 표현하면 구조적으로 잘 표현할 수 있지만 현실에서는 계층을 엄격히 구분하기가 어려울때도 있다. 가수겸 작곡가인 singer-songwriter를 예를 들어보자. singer-songwriter를 계층적으로 표현하면 다음과 같아진다.
```java
public abstract class Singer {
    abstract AudioClip sing(Song s);
}
public abstract class SongWriter {
    abstract Song compose(int chartPosition);
}

public abstract class SingerSongWriter {
    abstract AudioClip sing(Song s);
    abstract Song compose(int chartPosition);
}
```
다중 상속이 불가능하기 때문에 Singer 와 Songwriter를 조합한 클래스를 별도로 만들어야 한다.

예제에서는 조합이 두개뿐이지만 만약 조합이 여러개라면 모든 조합별로 클래스를 지원해야 하므로 많은 클래스가 만들어지게 된다. 이를 **조합 폭발(combinatorial explosion)** 이라 한다. 
##### 조합폭팔
Logger를 만들때는 FileLogger, ConsoleLogger, DBLogger 와 같은 종류가 있을 수 있다. 그리고 이 로거들을 이용해 Plain, JSON, CSV 등의 포맷으로 로그를 저장할 수도 있다. 만약 추상 클래스로 이 모든 조합을 추상클래스로 구현하여 상속받게 한다면 총 9가지의 클래스가 만들어진다.

```
1. FileLoggerPlain
2. FileLoggerJSON
3. FileLoggerCSV
4. ConsoleLoggerPlain
5. ConsoleLoggerJSON
6. ConsoleLoggerCSV
7. DBLoggerPlain
8. DBLoggerJSON
9. DBLoggerCSV
```

여기서 지원해야하는 logger 종류나 포맷이 늘어난다면 지원해야할 조합의 수는 기하급수적으로 늘어난다. 흔히 조합 폭발(combinatorial explosion) 이라 부르는 현상이다.
인터페이스로 정의하면 `Singer`와 `Songwriter` 를 모두 구현해도 문제가 되지 않는다.
```java
public interface Singer {
    AudioClip sing(Song s);
}
public interface Songwriter {
    Song compose(int chartPosition);
}
public interface SingerSongwriter extends Singer, Songwriter {
}
```

#### default method를 제공해 일감을 덜어줄 수 있다.
인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면, 그 구현을 default method로 제공해줄 수 있다. 

단, 몇가지 규칙을 지켜야 한다.

1. 상속하려는 사람을 위한 `@implSpec` 자바독 태그를 붙여 문서화해야 한다.
2. `equals` 와 `hashCode` 같은 Object의 메서드를 default method로 제공해서는 안된다.

### 인터페이스의 단점
java 8 부터 default method 가 제공되어 interface도 구현된 method를 가질 수 있게 되었지만 여전히 제약사항이 존재한다.
#### 인스턴스 필드를 가질 수 없다.
인터페이스에 정의되는 모든 필드는 public 이고 static 이며 final이다.
```java
public interface Parent {
   Integer field = 1; // 암묵적으로 `public static final`이 붙는다.
}
```
#### public 이 아닌 정적 멤버는 가질 수 없다.
단, private 정적 메서드는 예외이다.
```java
public interface Parent {
   private static Integer field = 1; // error: 제어자 'private'은(는) 허용되지 않습니다

   private static List<Integer> intArrayAsList(int[] a) {
   return new AbstractList<Integer>() {
       @Override
       public Integer get(int index) {
           return a[index];
       }

       @Override
       public int size() {
           return a.length;
       }
   };
}
```
#### 직접 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.
클래스와 달리 인터페이스는 메소드가 추가된다면, 기존에 그 인터페이스를 구현하고 있던 다른 클래스들이 영향을 받을 것이기 때문이다.

### 단점 보완
#### 추상 골격 구현(skeletal implementation) 클래스를 함께 제공
인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법도 있다.
인터페이스에서 제공하지 못한 기능을 추상 클래스에서 구현하여 제공할 수 있다.

**추상 골격 구현(skeletal implementation) 클래스 만들기**
- 인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드 몇 개도 함께 제공한다. 
- 인터페이스로 구현하지 못한 클래스들은 골격 구현 클래스에서 구현한다. 
- 클래스의 이름은 관례상 `Abstract~` 로 짓는다.
    
    ex) AbstractSet, AbstractMap, AbstractList ...

**장점**
- 추상 클래스처럼 구현을 도와주는 동시에, 추상클래스로 타입을 정의할 때 따라오는 제약에서 자유롭다.
  여러 인터페이스를 구현하여 추상 골격 구현 클래스를 만들 수 있으므로 계층구조가 없는 타입을 만들 수 있으며 기존의 추상 골격 구현 클래스에 새로운 인터페이스를 쉽게 추가할수도 있다.     

**Q. 왜 또 추상클래스지? 똑같이 단일상속에 제약을 받지 않을까?**
1. 추상 골격 구현(skeletal implementation)을 구현한 클래스는 type으로서 interface 를 사용하게 된다.
    `AbstractMap`과 `HashMap` 구현  
    - 추상 골격 구현 클래스인 AbstractMap은 Map interface 타입을 사용함.  
    ```java
    public abstract class AbstractMap<K,V> implements Map<K,V> {} //추상 골격 구현 클래스
    public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V> {} //구체 클래스
    ```  
   `HashMap`의 사용
    ```java
    AbstractMap map = new HashMap<String, String>(); // X
    Map map = new HashMap<String, String>(); // O
    ```
2. `HashMap` 이 `AbstractMap`을 상속하지 않고 다른 클래스를 사용하게 되더라도 사용하는 클래스에서는 문제가 없다.
   
    왜냐하면 `HashMap`의 Type은 `AbstractMap` 이 아니라 `Map`이고, `AbstractMap`은 그저 구현에 도움을 주는것 뿐이기 때문이다.
   `AbstractMap`을 상속하였다고 해서 `HashMap`과 `AbstractMap`이 `is-a` 관계를 가지지는 않는다.
    그러니, 사용하는 측에서 `~Abstract~` 가 붙은 클래스가 부모라면 `is-a`관계가 아니라는 생각을 가져야겠다.
    반대로 `is-a` 관계인 추상 클래스에 `~Abstract~` 를 붙여 이름을 지으면 안되겠다.

3. `AbstractMap`을 상속하지 못한다면 nested class로도 활용할 수 있다. (시뮬레이트한 다중상속, simulated multiple inheritance)
    ```java
    public class HashMap<K,V> implements Map<K,V> {
        private final DefaultMap defaultMap;
        public HashMap() {
            this.defaultMap = new DefaultMap();
        }
        @Override
        public int size() {
            return this.defaultMap.size();
        }
        ...
        private class DefaultMap extends AbstractMap<K, V> {
            ...
        }
    }
    ```
    인터페이스를 구현한 HashMap 클래스에서  
    AbstractMap 골격 구현을 확장한 private 내부 클래스를 정의하고,  
    각 메서드 호출을 내부 클래스의 인스턴스인 this.defaultMap에 전달하는 것이다.

**주의사항**
- 인터페이스에서 `equals`와 `hashCode` 같은 Object의 메서드는 디폴드로 제공하면 안된다.
  - 해당 메서드들은 모두 골격 구현 클래스에 구현하자
- 인터페이스의 메서드 모두가 기반 메서드와 default method가 된다면 골격 구현 클래스를 별도로 만들 이유는 없다.
- 상속해서 사용하는것을 가정하므로 아이템 19 에서 이야기한 설계 및 문서화 지침을 모두 따라야 한다.

### 결론
- 상속을 하는 것보다 인터페이스를 사용하는 것이 더 안전하다.
- 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다.
- 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현(skeletal implementation) 클래스을 함께 제공하는 방법을 꼭 고려해보자.
- 골격 구현은 '가능한 한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다. '가능한 한'이라고 한 이유는, 인터페이스에 걸려 있는 구현상의 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하기 때문이다.

## 참조
- [combinatorial explosion](https://stackoverflow.com/a/59241553)
- [skeletal implementation](https://stackoverflow.com/a/13437007)
- [mixin](https://ko.wikipedia.org/wiki/%EB%AF%B9%EC%8A%A4%EC%9D%B8)