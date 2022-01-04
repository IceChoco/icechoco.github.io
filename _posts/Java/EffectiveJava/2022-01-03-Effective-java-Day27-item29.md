---
title: '[Effective Java] Day 27 - Item 29 :: 이왕이면 제너릭 타입으로 만들라'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day27에서는 item 29에 대한 내용을 다룬다.

## Item 29 :: 이왕이면 제너릭 타입으로 만들라
기존에 구현되어 있는 Object 기반의 스택 클래스를 보자
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(){
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e){
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop(){
        if(size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;//다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty(){
        return size == 0;
    }

    private void ensureCapacity(){
        if(elements.length == size){
            elements = Arrays.copyOf(elements, 2*size+1);
        }
    }
}
```

이 클래스는 원래 제너릭 타입이어야 마땅하다. 제너릭으로 만들어보자!  
- 제너릭으로 바꾼다고 해도 현재 버전을 사용하는 클라이언트는 아무런 해가 없다(로타입을 통한 하위 호환성 지원)
- 클라이언트가 스택에서 꺼낸 객체를 형변환할 때 런타임 오류가 날 수 있다.

### Object 기반의 스택 클래스 → 제너릭으로 바꾸는 방법
- **클래스 선언에 타입 매개변수를 추가**하라
  - 원소의 타입이름은 보통 E를 사용한다.

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(){
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e){
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop(){
        if(size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;//다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty(){
        return size == 0;
    }

    private void ensureCapacity(){
        if(elements.length == size){
            elements = Arrays.copyOf(elements, 2*size+1);
        }
    }
}
```

타입 매개변수를 추가하는 경우 컴파일 오류가 발생한다.  
 → E와 같은 실체화 불가 타입으로는 배열을 만들 수 없기 때문
```java
Stack.java:12:20: generic array creation
    elements = new E[DEFAULT_INITIAL_CAPACITY];
                   ^
```

#### 적절한 해결책
##### 방법 1) object 배열을 생성하기
object 배열을 생성한 다음 제너릭 배열로 형변환하자
```java
public Stack(){
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```
컴파일 오류가 사라지고 비검사 형변환 경고를 내보낼 것이다.
![item29_unchecked_cast](/assets\img/item29_unchecked_cast.png)
  
- 스스로 타입 안정성을 해치지 않음을 확인해야한다.
  - elements 배열은 private 필드에 저장되어 외부로 전달되는 일이 전혀 없다.
  - push 메서드를 통해 배열에 저장되는 원소의 타입은 항상 E다.
  - 따라서 비검사 형변환은 확실히 안전하다.

비검사 형변환이 안전함을 직접 증명했다면 **@SuppressWarnings 애너테이션으로 해당 경고를 숨긴다.**
```java
// 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
// 따라서 타입 안정성을 보장하지만, 이 배열의 런타임 타입은 Eㅊ가 아닌 Obejct[]다.
@SuppressWarnings("unchecked")
public Stack(){
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```
##### 방법 2) elements 필드의 타입을 E[]에서 Object[]로 바꾸기
```java
private Object[] elements;

public E pop(){
    if(size == 0)
        throw new EmptyStackException();
    E result = elements[--size];
    elements[size] = null;//다 쓴 참조 해제
    return result;
}
```
이렇게하면 첫 번째랑은 다른 컴파일 오류가 나타난다.
![item29_incompatiable_types](/assets\img/item29_incompatiable_types.png)

배열이 반환한 원소를 E로 형변환 해보자.
```java
private Object[] elements;

public E pop(){
    if(size == 0)
        throw new EmptyStackException();
    E result = (E)elements[--size]; //배열이 반환한 원소를 E로 형변환
    elements[size] = null;
    return result;
}
```

그럼 컴파일 오류가 사라지고 비검사 형변환(unchecked cast) 경고가 나타난다.
![item29_unchecked_cast2](/assets\img/item29_unchecked_cast2.png)

이번 역시 아까처럼 직접 비검사 형변환이 안전한지 증명하고, @SuppressWarnings 애너테이션으로 경고를 숨기자.
```java
//비검사 경고를 적절히 숨긴다.
public E pop(){
    if(size == 0)
        throw new EmptyStackException();
    //push에서 E타입만 허용하므로 이 형변환은 안전하다.
    @SuppressWarnings("unchecked") E result = (E)elements[--size];
    elements[size] = null;//다 쓴 참조 해제
    return result;
}
```

### 배열을 사용한 코드를 제너릭으로 만드는 위 두가지 방법 비교
- **방법 1**
  - 가독성이 더 좋다
  - 배열의 타입을 E[]로 선언하여 오직 E타입만 받음을 확실히 어필한다.
  - 코드가 더 짧다.
  - 형변환: 배열 생성 시 단 한번만 해주면 된다.
- **방법 2**
  - 형변환: 원소를 읽을 때 마다 해줘야 한다.
  - E가 Object 타입이 아닌 한 런타임 타입과 컴파일 타임이 달라 **힙오염**을 일으킨다.

### 추가내용
아이템 28에서 배열보다는 리스트를 우선하라고 했는데... 약간 모순아닌가? 싶을 수도 있다.  
제너릭 타입 안에서 리스트를 사용하는게 항상 가능한 것도 아니고, 무조건 좋은 것만은 아니다.
1. 자바가 리스트를 기본 타입으로 제공하지 않았으므로, ArrayList 같은 제너릭 타입도 결국은 배열을 사용해 구현해야 한다.  
2. HashMap 같은 제너릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.
3. 타입 매개변수에 int, double과 같은 기본 타입은 올 수 없다.
 - 대신 박싱된 기본타입인 Integer, Double을 사용해 우회 가능하다.
4. `<E extends Delayed>`: Delayed 자기 자신과 그 하위타입만 받겠다라는 뜻이다.
 - 이를 **한정적 타입 매개변수(bounded type parameter)**라고 한다.

### 결론
- 클라이언트에서 직접 형변환해야 하는 타입보다는 제너릭이 더 안전하고 쓰기 편하다.
- 그러므로 **기존 타입 중 제너릭이었어야 하는 게 있다면 과감히 제너릭 타입으로 변경하자!**
- 기존 클라이언트에 영향을 줄까 걱정하지 않아도 된다. 아무런 영향을 주지않고, 오히려 새로운 사용자를 훨씬 편하게 해준다!