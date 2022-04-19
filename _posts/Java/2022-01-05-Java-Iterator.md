---
title: '[Java] Iterable과 Iterator'
layout: post
categories: java
tags: java
comments: true
---

이펙티브 자바 3판을 읽다가 `Iterable<E>`를 입력 매개변수 타입으로 받는 소스코드 발견했다. Iterator가 반복자인 것만 알고있었는데 Iterable은 뭐지? 이름만 봐서는 인터페이스 같은데... 궁금해서 찾아보게 되었다.

## 1. Iterable이란?
![iterable_hierarchy](/assets\img/iterable_hierarchy.PNG)
- Iterable은 Collection의 상위 인터페이스
  - Collection은 List, Set, Queue 인터페이스의 상위 인터페이스

![collection](/assets\img/collection.PNG)

내부 구현코드를 확인해보면 Collection 인터페이스의 상위 인터페이스는 Iterable임을 알 수 있다.

```java
/**
 * Implementing this interface allows an object to be the target of the enhanced
 * {@code for} statement (sometimes called the "for-each loop" statement).
 *
 * @param <T> the type of elements returned by the iterator
 *
 * @since 1.5
 * @jls 14.14.2 The enhanced {@code for} statement
 */
public interface Iterable<T> {
    /**
     * Returns an iterator over elements of type {@code T}.
     *
     * @return an Iterator.
     */
    Iterator<T> iterator();

    ...
}    
```

Iterable의 내부 구현코드를 보면 이 인터페이스 안에는 iterator 메소드가 추상메소드로 선언되어있다. 그러므로 Collection 인터페이스 계층구조에서 List, Set, Queue를 구현하는 클래스들은 다 iterator 메소드를 가지고 있다. <span style="color:red">따라서 Iterable의 역할은 iterator() 메소드를 하위 클래스에서 무조건 구현을 하게 만들기 위함이다.</span>

## 2. Iterator란?
![item31_java_collection_frameworks](/assets\img/item31_java_collection_frameworks.PNG)

Iterator 인터페이스는 Collection과는 별개로 존재하는 인터페이스이다.

```java
public interface Iterator<E> {
    /**
     * Returns {@code true} if the iteration has more elements.
     * (In other words, returns {@code true} if {@link #next} would
     * return an element rather than throwing an exception.)
     *
     * @return {@code true} if the iteration has more elements
     */
    boolean hasNext();

    /**
     * Returns the next element in the iteration.
     *
     * @return the next element in the iteration
     * @throws NoSuchElementException if the iteration has no more elements
     */
    E next();

    /**
     * Removes from the underlying collection the last element returned
     * by this iterator (optional operation).  This method can be called
     * only once per call to {@link #next}.
     * <p>
     * The behavior of an iterator is unspecified if the underlying collection
     * is modified while the iteration is in progress in any way other than by
     * calling this method, unless an overriding class has specified a
     * concurrent modification policy.
     * <p>
     * The behavior of an iterator is unspecified if this method is called
     * after a call to the {@link #forEachRemaining forEachRemaining} method.
     *
     * @implSpec
     * The default implementation throws an instance of
     * {@link UnsupportedOperationException} and performs no other action.
     *
     * @throws UnsupportedOperationException if the {@code remove}
     *         operation is not supported by this iterator
     *
     * @throws IllegalStateException if the {@code next} method has not
     *         yet been called, or the {@code remove} method has already
     *         been called after the last call to the {@code next}
     *         method
     */
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
    ...
}    
```

Iterator 인터페이스의 내부 구현은 위와 같이 되어있다. 따라서 <span style="color:red">hasNext(), next(), remove()</span> 등의 메소드 사용이 가능하다.
- 용도: 컬렉션 클래스의 데이터를 하나씩 읽어올 때 사용
  - 표준화가 되어 있지 않다면 컬렉션 클래스의 데이터를 읽어올 때 마다, 해당 클래스의 데이터를 꺼내오는 메서드들을 다 알고 있어야 함. 이름이 다를 수도 있는데 그걸 다 알고 있어야 하는건 비 효율적이다!
  - 위와 같은 이유로 인해 Iterator가 존재하는 것

위와같은 Iterator 인터페이스는 객체지향 프로그래밍의 중요한 목적 중의 하나임

### 객체지향 프로그래밍의 중요한 목적
- <span style="color:red">공통 인터페이스</span>를 정의하여 표준을 정의
- 위 인터페이스를 구현하여 표준을 따르도록 하며 <span style="color:red">코드의 일관성</span>을 유지
- 재사용성을 극대화

```java
public class Test {
    public static void main(String[] args) {
        LinkedList<String> list = new LinkedList<>();
        list.add("Cho");
        list.add("a");
        list.add("ra");
        Iterator it = list.iterator();

        while(it.hasNext()){
            System.out.println(it.next()+" ");
        }
    }
}
```
- 참조변수 list로 iterator 메소드를 호출
- iterator 메소드는 Linkedlist 형태의 Iterator를 반환
- 반환한 Iterator의 hasNext, next 메소드를 통해 내용을 출력함

## 참조
[[Java] Iterable 과 Iterator 이란?](https://devlog-wjdrbs96.tistory.com/84)


