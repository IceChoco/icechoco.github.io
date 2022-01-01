---
title: '[Effective Java] Day 25 - Item 27 :: 비검사 경고를 제외하라'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day25에서는 item 27에 대한 내용을 다룬다.

## Item 27 :: 비검사 경고를 제외하라
제너릭을 활용하면 아래와 같은 컴파일러의 비검사 경고를 볼 수 있다.  
- 비검사 형변환 경고(unchecked method warning)
- 비검사 메서드 호출 경고(unchecked method invocation warning)
- 비검사 매개변수화 가변인수 타입 경고(unchecked parameterized vararg type warning)
- 비검사 변환경고(unchecked conversion warning) 등

**비검사 경고 예**


### ex) ArrayList의 toArray 메서드
```java
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

실제로는 메서드에 @suppressWarnings가 달려있지만 더 좋은 방법이 있다.  
반환값을 담을 지역변수를 하나 선언하고 그 변수에 애너테이션을 달아주는 방법이다.

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // 생성한 배열인 result와 매개변수로 받은 배열의 타입도 T[]로 형변환 하여 같으므로
        // 올바른 형변환이다.
        @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elementData, size, a.getClass());
        return result;
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```
- 장점
  - 컴파일이 깔끔하게 된다.
  - 비검사 경고를 숨기는 범위를 최소로 좁힐 수 있다.

- @suppressWarnings
  - unchecked: 미확인 오퍼레이션과 관련된 경고 억제

```java
/*
* The string {@code "unchecked"} is used to suppress
* unchecked warnings. Compiler vendors should document the
* additional warning names they support in conjunction with this
* annotation type. They are encouraged to cooperate to ensure
* that the same names work across multiple compilers.
* @return the set of warnings to be suppressed
*/
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE, MODULE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```

### 결론
- 최선을 다해 모든 비검사 경고를 제거하라
  - 모두 제거하면 타입 안정성이 증명된다.
  - ClassCastException이 발생할 일이 없다.
- 경고를 제거할 수 없고 타입이 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchekced")`를 달아 경고를 숨겨라
  - 경고를 숨기기로 한 근거를 주석으로 남겨라

### 참고
- [Valid @SuppressWarnings Warning Names](https://www.baeldung.com/java-suppresswarnings-valid-names)