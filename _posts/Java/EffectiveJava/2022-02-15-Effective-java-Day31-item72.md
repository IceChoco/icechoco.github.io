---
title: '[Effective Java] Day 31 - Item 72 :: 표준 예외를 사용하라'
layout: post
categories: java
tags: java
comments: true
---

Item 72 :: 표준 예외를 사용하라

## Exception의 재사용
- 자바 라이브러리는 대부분 API에서 쓰기 충분한 Exception들을 제공하므로, 재사용하는 것이 좋음
- 장점
  1. 내가 작성한 API가 다른 사람이 익히고 사용하기 쉬워짐
  2. API를 사용한 프로그램도 낯선 예외를 사용하지 않게 되어 읽기 쉬움
  3. 예외 클래스의 수가 적을수록 메모리 사용량 ↓, 클래스 적재시간 ↓

## 자주 쓰이는 표준 예외

|예외      |주요 쓰임               |                         
|----------|---------------------------------------------|
| IllegalArgumentException | 허용하지 않는 값이 인수로 건너 졌을 때(null은 따로 NPE 처리) |
| IllegalStateException | 객체가 요청한 작업을 처리하기에 적절하지 않은 상태일 때 |
| NullPointerException | null을 허용하지 않는 메서드에 null을 건냈을 때 |
| IndexOutOfBoundsException | 인덱스가 허용 범위를 넘었을 떄 |
| CouncurrentModificationException | 허용하지 않는 동시 수정이 발견됐을 때 |
| UnsupportedOperationException | 요청한 작업을 지원하지 않을 때 |

### IllegalArgumentException
- 호출자가 인수로 부적절한 값을 넘길 때  
- ex) AES256 방식으로 암호화하는 클래스 - 매개변수의 null 여부 및 자리수 체크
  * AES(Advanced Encryption Standard) : 고급 암호화 표준이며, 대칭키 를 쓰는 블럭 암호. 높은 안전성과 속도로 인해 인기를 얻어 전 세계적으로 사용되고 있음.
```java
public class AES256Cipher extends AESCommon implements ICipher {
    public AES256Cipher(byte[] key, byte[] iv) {
        super(key, iv);
       
        if(key == null || key.length != 32) {
            throw new IllegalArgumentException("Invalid 'key' value.");
        }
        if(iv == null || iv.length != 16) {
            throw new IllegalArgumentException("Invalid 'iv' value.");
        }
    }
}
```

### IllegalStateException
- 대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때
- 제대로 초기화되지 않은 객체를 사용하려 할 때
- ex) 인스턴스화 불가한 유틸리티 클래스의 기본 생성자를 호출할 때
```java
public class UtilityClass{  
    private UtilityClass()
    {
        //클래스 안에서 실수로라도 생성자를 호출하는 것을 방지
        throw new IllegalStateException("UtilityClass");
    }
}
```
> **IllegalArgumentException VS IllegalStateException**
> - 카드 덱을 표현하는 객체가 있고, 인수로 건넨 수 만큼의 카드를 뽑아 나눠주는 메서드가 있을 때,
> 덱에 남아 있는 카드 수보다 큰 값을 건네면 어떤 예외를 던져야할까?
>   - 인수의 값이 무엇이었든 어차피 실패했을 것이라면 → `IllegalStateException`  
>     인수의 값에 따라 성공할 수 있었을 것이라면 → `IllegalArgumentException`을 던지는게 일반적인 규칙

위 2개 처럼 잘못된 인수나 상태라고 예외를 던질수도 있지만 아래와 같은 특수한 케이스는 따로 구분해서 사용

### NullPointerException
Null 값을 허용하지 않는 메서드에 null을 건네는 경우 IllegalARgumentException 대신 NPE를 사용
```java
public class UtilityClass{
    //...
    public static void exp(String ptr) {
        if (ptr == null)
            throw new NullPointerException();
    }

    public static void main(String[ ] args) {
        String input = null;
        UtilityClass.exp(input); // static method
    }
}
```

### IndexOutOfBoundsException
어떤 시퀀스의 허용 범위를 넘는 값을 건넬 때 IllegalArgumentException 대신 IndexOutOfBoundsException 사용
- ex) Google Guava 라이브러리의 Preconditions Class 내 checkIndex 메소드
  - 주어진 index가 0보다 작거나, length보다 크거나 같은지(허용범위를 넘는지) check
  - 유효성 검사를 보다 효과적으로 수행 가능

```java
public class Preconditions {
    @IntrinsicCandidate
    public static <X extends RuntimeException>
    int checkIndex(int index, int length, BiFunction<String, List<Number>, X> oobef) {
        if (index < 0 || index >= length)
            throw outOfBoundsCheckIndex(oobef, index, length);
        return index;
    }

    private static RuntimeException outOfBoundsCheckIndex(
            BiFunction<String, List<Number>, ? extends RuntimeException> oobe,
            int index, int length) {
        return outOfBounds(oobe, "checkIndex", index, length);
    }    

    private static RuntimeException outOfBounds(
            BiFunction<String, List<Number>, ? extends RuntimeException> oobef,
            String checkKind,
            Number... args) {
        List<Number> largs = List.of(args);
        RuntimeException e = oobef == null
                                ? null : oobef.apply(checkKind, largs);
        return e == null
                ? new IndexOutOfBoundsException(outOfBoundsMessage(checkKind, largs)) : e;
    }
}    
```

### ConcurrentModificationException  
- 싱글 스레드에서 사용하려고 한 객체나 외부 동기화 방식으로 사용하려고 한 객체를 여러 스레드가 동시에 수정하려 할 때
- ex) 싱글 스레드 환경에서 Collection을 fail-fast iterator하고 있을 때 Collection을 수정하는 경우(add(), remove())
  - fail-fast: 순차적 접근이 모두 끝나기 전에 콜렉션 객체에 변경이 일어날 경우 순차적 접근이 실패되면서 예외를 return하는 방식

```java
public static void main(String[] args) {
    List<String> listOfPhones = new ArrayList<String>(Arrays.asList( "iPhone 6S", "iPhone 6", "iPhone 5", "Samsung Galaxy 4", "Lumia Nokia"));
        // This is wrong way, will throw ConcurrentModificationException
    for(String phone : listOfPhones){
        if(phone.startsWith("iPhone")){
            listOfPhones.remove(phone); // will throw exception
         }
     }
}

//ArrayList의 내부 구현
private class Itr implements Iterator<E> {
    int expectedModCount = modCount;
    //...
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size) throw new NoSuchElementException();
           
        Object[] elementData = ArrayList.this.elementData;
       
        if (i >= elementData.length) throw new ConcurrentModificationException();
       
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    // 이 메소드에서 expectedModCount와 modCount를 비교
    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }

    // Structural modifications을 발생시키는 메소드
    public E remove(int index) {
        Objects.checkIndex(index, size);
        final Object[] es = elementData;

        @SuppressWarnings("unchecked") E oldValue = (E) es[index];
        fastRemove(es, index);

        return oldValue;
    }

    // Structural modifications이 일어날 때 modCount가 증가하는 걸 볼 수 있음
    private void fastRemove(Object[] es, int i) {
        modCount++;
        final int newSize;
        if ((newSize = size - 1) > i)
            System.arraycopy(es, i + 1, es, i, newSize - i);
        es[size = newSize] = null;
    }
}
```

### UnsupportedOperationException  
- 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때
- 보통은 구현하려는 인터페이스의 메서드 일부를 구현할 수 없을 때 사용
- ex) add만 가능한 List의 구현체

```java
public class UoeTest implements List {
    //...
    @Override
    public boolean add(Object o) {
        return ...;
    }

    @Override
    public Object remove(int index) {
        throw new UnsupportedOperationException("UoeTest doesn't support this method.");
    }
    //...
}

public class Client {
    public static void main(String[] args) {
        List<String> list = new UoeTest();
        list.add("TEST를 하자");
        list.remove(0);
    }
}

```
```java
//실행결과
Exception in thread "main" java.lang.UnsupportedOperationException: UoeTest doesn't support this method.
    at item72.UoeTest.remove(UoeTest.java:16)
    at item72.Client.main(Client.java:9)
```

Exeption, RuntimeException, Throwable, Error는 직접 재사용하지 말자.
- 추상클래스라고 생각하자
- 다른 예외들의 상위 클래스이므로 여러 성격의 예외들을 포괄하기 때문에 안정적으로 테스트 할 수 없다.

상황에 부합한다면 항상 표준 예외를 재사용하자.
- API 문서를 참고해 그 예외가 어떤 상황에서 던져지는지 꼭 확인 필요
- 예외의 이름뿐만 아니라 예외가 던져지는 맥락도 부합할 때만 재사용해야함

더 많은 정보를 제공하고싶다면 표준 예외를 확장해서 재사용해도 좋다.
- 단, 예외는 직렬화 할 수 있고(12장) 직렬화에는 많은 부담이 따르니 나만의 예외를 따로 만들지 않는 것이 좋음

### 참고
- [자바 ConcurrentModificationException](https://velog.io/@youngerjesus/Java-avoid-ConcurrentModificationException)