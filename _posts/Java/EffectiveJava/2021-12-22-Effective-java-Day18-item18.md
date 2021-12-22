---
title: '[Effective Java] Day 18 - Item 18 :: 상속보다는 컴포지션을 사용하라'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day18에서는 item 18에 대한 내용을 다룬다.

## Item 18 :: 상속보다는 컴포지션을 사용하라
일반적인 구체 클래스를 패키지 경계를 넘어, 즉 다른 패키지의 구체 클래스를 상속하는 일은 위험하다.  
이 책에서 말하는 상속이란 클래스가 다른 클래스를 확장하는 구현 상속을 말한다.

### 메소드 호출과 달리 상속은 캡슐화를 깨뜨린다.
- 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.
- 따라서 상위 클래스의 설계자가 확장을 충분히 고려하고 문서화도 제대로 해두지 않으면, 하위 클래스는 상위 클래스의 변화에 발맞춰 수정돼야만 한다.

#### 1. 메서드 재정의하는 경우
##### 1) 예시 1
HashSet의 성능을 높이기 위해 처음 생성된 이후 원소가 몇개 더해졌는지 알기 위해 아래와 같이 변형된 HashSet을 만들었다.  
그리고 여기에 추가된 원소의 수를 저장하는 변수와 접근자 메서드를 추가하고, 원소를 추가하는 메서드인 add와 addAll을 재정의했다.
```java
// 코드 18-1 잘못된 예 - 상속을 잘못 사용했다! (114쪽)
public class InstrumentedHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```
- 출력결과

```java
6
```

getAddCount 메소드를 수행하면 3을 출력하리라 생각하겠지만 실제로는 6이 나온다.
- 원인: hashSet의 addAll 메서드가 add 메서드를 사용하여 구현하기 때문
- InstrumentedHashSet의 addAll 동작 방식
  1. addCount 변수에 원소의 사이즈인 3을 더함 → HashSet의 addAll 호출(AbstractCollection의 addAll 이용)
  ```java
  @Override public boolean addAll(Collection<? extends E> c) {
      addCount += c.size();
      return super.addAll(c);
  }
  ```
  2. HashSet의 addAll: 각 원소를 add 메서드를 호출하여 추가.  
     이때 불리는 add는 InstrumentedHashSet에서 재정의한 메서드.
     따라서 addCount 값에 원소 하나 당 중복해서 더해져, 최종값이 6으로 늘어남.
  ```java
  public boolean addAll(Collection<? extends E> c) {
  boolean modified = false;
  for (E e : c)
      if (add(e))
          modified = true;
  return modified;
  }
  ```

addAll 메서드는 주어진 컬렉션을 순회하며 원소 하나당 all 메서드를 한 번만 호출하도록 수정할 수 있다.
```java
@Override public boolean addAll(Collection<? extends E> c) {
    for(E e : c) add(e);
    return true;
}
```
- 문제점
  - 상위 클래스의 메서드 동작을 다시 구현하기 때문에 어려움
  - 시간이 많이 듦
  - 자칫 오류를 내거나 성능을 떨어트릴 수 있음
  - 하위 클래스에서는 접근할 수 없는 상위 클래스의 private 필드를 써야하는 상황이라면 구현 자체 불가

##### 2) 예시 2
다음 릴리스에서 상위 클래스에 새로운 메서드를 추가한 경우, 보안 때문에 컬렉션에 추가된 모든 원소가 특정 조건을 만족해야 하는 클래스가 있다고 가정해보자.
- 컬렉션을 상속하여 원소를 추가하는 모든 메서드를 override해 필요한 조건을 먼저 검색하게 하면?
  - 상위 클래스에 또다른 원소 추가 메서드가 생기면 유효하지 않음
      - 즉, 하위 클래스에서 재정의하지 못한 상위 클래스의 새로운 메서드를 사용하여 조건을 검사하지 않은 원소가 추가될 수 있음
      - 실제로도 컬렉션 프레임워크 이전부터 존재하던 `Hashtable`과 `Vector`를 컬렉션 프레임워크에 포함시키자,
        이와 같은 보안 구멍들을 수정해야하는 사태가 발생했었음
  
#### 2. 클래스를 확장하고 새로운 메서드를 추가하는 경우
- 앞의 메서드를 재정의 하는 것보다 더 안전하긴 하지만 그래도 위험하다.  
- ex) 하위 클래스에서 재정의 하지 않고 새로운 메서드를 추가함   
  - 다음 릴리즈에서 상위 클래스 내 시그니처가 같고 반환타입이 다른 메소드 추가  
    → 기존에 새로운 메서드를 작성했던 하위 클래스는 컴파일 조차 되지 않음
  - 반환 타입마저 같다면?  
    → 상위 클래스의 메서드를 재정의 한 셈. 첫 번째 문제와 같은 상황.  
    → 하위 클래스에서 만든 메서드가 상위 클래스의 메서드가 요구하는 규악을 만족하지 못할 가능성↑

### Composition을 사용하자
- **Composition(컴포지션)**: 기존 클래스가 새로운 클래스의 구성요소로 쓰이는 설계 방식
- 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자
```java
// 1. 집합 클래스 자신
// 코드 18-2 래퍼 클래스 - 상속 대신 컴포지션을 사용했다. (117-118쪽)
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```
```java
// 2. 전달 메서드만으로 이뤄진 재사용할 수 있는 전달 클래스 (118쪽)
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
    { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
    { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
    { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
    { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
    { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```
- **전달 메서드(forwarding method)**
  - 새 클래스인 `ForwardingSet`의 인스턴스 메서드
    - `Set`을 private 필드로 참조. 그리고 그 `Set` 클래스에 대응하는 메서드를 호출한 결과를 반환한다.
    - 이와 같은 방식을 **전달(forwarding)**이라고하며, 새 클래스의 메서드들을 **전달 메서드(forward method)**라 함
    ```java
    public class ForwardingSet<E> implements Set<E> {
        private final Set<E> s;
        public ForwardingSet(Set<E> s) { this.s = s; }
    
        public boolean addAll(Collection<? extends E> c){
            return s.addAll(c);      
        }
    }
    ```
  - 새로운 클래스는 기존 클래스 내부의 구현 방식에 따른 영향에서 벗어남
  - 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않음

- InstrumentedSet을 사용하면 대상 Set 인스턴스를 특정 조건하에서만 임시로 계측 가능
```java
static void walk(Set<Dog> dogs){
    //이 메서드에서는 dogs 대신 iDogs를 사용함
    InstrumentedSet<Dog> iDogs = new InstrumentedSet<Dog>(dogs);
    //...
}
```

- **래퍼 클래스**: 다른 인스턴스를 감싸고(wrap) 있다는 뜻
  - ex) Set 인스턴스를 감싸고 있는 InstrumentedSet 클래스
  - 다른 Set에 계측 기능을 덧씌운다는 뜻에서 **데코레이터 패턴(Decorator Pattern)**이라고도 함
  - **단점**
    - 콜백(callBack) 프레임워크와는 어울리지 않는다.
       - 콜백 프레임워크 에서는 자기 자신의 참조를 다른 객체에 넘겨서, 다음 호출(콜백) 때 사용하도록 함
       - 내부 객체는 자시을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘김
       - 콜백 때는 래퍼가 아닌 내부 객체를 호출함 → 이를 **SELF 문제**라고 함
- **위임(delegation)**
  - 넓은 의미: Composition + 전달 클래스의 조합
  - 좁은 의미: 래퍼 객체가 내부 객체(여기서는 Set 클래스)에 자기 자신의 참조를 넘기는 경우

### SELF 문제


### 상속 vs Composition 차이
1. 상속
  - 구체 클래스 각각을 따로 확장해야 함
  - 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도 정의 필요
2. Composition
  - 한 번만 구현해두면 어떠한 Set 구현체라도 계측 가능
  - 기존 생성자들과도 함께 사용 가능
  ```java
  Set<Instance> times = new InstrumentedSet<Instance>(new TreeSet<Instance>(cmp));
  Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
  ```

### 참조
- [Java 개발자가 배우는 C++ - 03 - const와 final](https://github.com/HomoEfficio/dev-tips/blob/master/Java%20%EA%B0%9C%EB%B0%9C%EC%9E%90%EA%B0%80%20%EB%B0%B0%EC%9A%B0%EB%8A%94%20C%2B%2B%20-%2003%20-%20const%EC%99%80%20final.md)
- [[아이템 17] 가변 동반 클래스 동작(사용) 예시 #15](https://github.com/JunHyeok96/effective-java/issues/15)
- [Wrapper Classes are not suited for callback frameworks](https://stackoverflow.com/questions/28254116/wrapper-classes-are-not-suited-for-callback-frameworks)