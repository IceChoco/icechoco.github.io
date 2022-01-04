---
title: '[Effective Java] Day 28 - Item 30 :: 이왕이면 제너릭 메서드로 만들라'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day28에서는 item 30에 대한 내용을 다룬다.

## Item 30 :: 이왕이면 제너릭 메서드로 만들라
### 제너릭 메서드
메서드의 선언부에 제너릭한 타입이 선언된 메서드
```java
public static <E> Set<E> Union(Set<E> s1, Set<E> s2){
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```
- 타입 매개변수 목록: `<E>`
  - 메서드 내에 지역적으로 사용될 제너릭 타입임을 알려주는 용도
- 반환 타입: `Set<E>`
- 입력 매개변수 타입: `Set<E>`

- **특징**
  - 타입 안전: 엉뚱한 타입을 넣으려는 시도를 컴파일 과정에서 차단
  - 쓰기 쉬움: 타입 체크 및 형변환을 하지 않아도 되서 코드가 간결함

### 제너릭 싱글턴 팩토리
요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 메서드.  
제너릭 싱글턴 팩토리로 불변 객체를 여러 타입으로 활용할 수 있다.
#### UnaryOperator
- 자바의 대표적인 함수형 인터페이스
- 항등함수를 담은 인터페이스(항등함수: 어떤 값을 넣어도 같은 값이 나오는 함수)  
    - 인수 1개를 받아 인수와 동일한 타입의 값을 리턴하는 함수를 표현
- `Function<T,T>`를 상속했으므로, 부모의 T applty(T t)가 인터페이스의 함수형 메서드
- 함수형 인터페이스는 람다 표현식으로 인스턴스 생성 가능
- 예시
  - UnaryOperator, `Function<T, R>` 인터페이스
    ```java
    @FunctionalInterface
    public interface UnaryOperator<T> extends Function<T, T> {
        static <T> UnaryOperator<T> identity() {
            return t -> t;
        }
    }

    @FunctionalInterface
    public interface Function<T, R> {
        R apply(T t);
        ...
    }
    ```
  - UnaryOperator 활용한 제너릭 싱글턴 팩토리 패턴
    ```java
    // 제네릭 싱글턴 팩터리 패턴 (178쪽)
    public class GenericSingletonFactory {
        //f(x) = x 항등함수 객체
        private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

        //입력된 타입 매개변수에 따라 형변환 후 반환하는 제너릭 메서드
        static <T> UnaryOperator<T> identityFunction() {
            return (UnaryOperator<T>) IDENTITY_FN;
        }
    }

    // 코드 30-5 제네릭 싱글턴을 사용하는 예 (178쪽)
    public static void main(String[] args) {
        String[] strings = {"물","스탠드","살찜"};
        UnaryOperator<String> sameString = identityFunction();
        for(String s : strings)
            System.out.println(sameString.apply(s));
    }
    ```

### 재귀적 타입 한정(recursive type bound)
자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용범위를 한정시키는 것
```java
//코드 30-6 재귀적 타입 한정을 이용해 상호 비교할 수 있음을 표현했다.
public static <E extends Comparable<E>> E max(Collection<E> c);
```
- 타입 매개변수 목록: `<E extends Comparable<E>>`
  - 이 메소드에서 사용되는 타입 E는 Comparable 인터페이스를 구현한 타입으로 제한함
  - E는 자신과 비교할 수 있다
    - E타입 원소들은 상호비교 가능하다는 뜻을 표현
- 반환 타입: `E`
- 입력 매개변수 타입: `Collection<E>`

#### Comparable
타입 매개변수 T는 `comparable<T>`를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다.
```java
public interface Comparable<T> {
    public int compareTo(T o);
}

//코드 30-7 컬렉션에서 최댓값을 반환한다. - 재귀적 타입 한정 사용
public static <E extends Comparable<E>> E max(Collection<E> c){
    if(c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어있습니다!");

    E result = null;
    for(E e: c)
        if(result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return result;
}
```
### 결론
입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드는 제너릭하게 만들자!  
더 안전하고 사용하기도 쉽다 :D