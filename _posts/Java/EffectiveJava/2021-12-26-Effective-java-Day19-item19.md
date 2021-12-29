---
title: '[Effective Java] Day 19 - Item 19 :: 상속을 고려한 설계 주의 사항과 문서화 (1)'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day19에서는 item 19에 대한 내용을 다룬다.

## Item 19 :: 상속을 고려한 설계 주의 사항과 문서화
언제 상속을 사용해야 하는가에 관하여....
[오브젝트](https://book.naver.com/bookdb/book_detail.naver?bid=15007773) `Ch10. 상속과 코드 재사용` 중 발췌

> - **중복 코드를 결정하는 기준은 코드의 모양이 아니다.** 모양이 유사하다는 것은 단지 중복의 징후일 뿐이다.
    중복 여부를 결정하는 기준은 **코드가 변경에 반응하는 방식**이다.
>- 상속은 결합도를 높인다. 상속이 초래하는 부모 클래스와 자식 클래스 사이의 강한 결합이 코드를 수정하기 어렵게 만든다.
> - 상속은 코드의 재사용을 위해 캡슐화의 장점을 희석시키고 구현에 대한 결합도를 높임으로써 객체지향이 가진 강력함을 반감시킨다.

> 상속과 합성은 재사용의 대상이 다르다.
> 코드 재사용을 위해서는 합성이 상속보다 더 좋은 방법이다.
> **상속의 목적은 코드 재사용이 아니다.** 상속은 타입 계층을 구조화하기 위해 사용해야 한다.
> 상속은 상황에 따라 적절한 메소드를 선택할 수 있는 메커니즘, 객체가 메시지를 수신했을 때 메시지를 처리할 적절한 메소드를 상속 계층 안에서 탐색한다.
> 즉, **상속은 다형성을 기능하게 하는 타입 계층을 구축하기 위한 것이다.**

그러니 코드 재사용이 목적이라면 상속보다는 composition을 사용하고,
item17에서 다룬 방법(final class, private 생성자)으로 상속을 금지하는 것이 좋다.

그럼에도 불구하고 상속을 사용해야겠다면...
아래 사항들을 고려하라
### 상속 설계 주의 사항
#### 1. overridable methods 문서화
상속용 클래스는 overridable methods를 내부적으로 어떻게 이용하는지 문서로 남겨야 한다.
- 클래스 공개 API에서 클래스 자신의 또 다른 overridable 메소드를 호출하는지,
  어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지 등의 정보를 담아야 함
- method override가 공개 API에 영향을 줄 수 있기 때문

java API example
- [`Iterable#forEach`](https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html) javadoc
- 내부 구현에 대한 설명을 담고 있다
  ![image](https://user-images.githubusercontent.com/29528531/146139110-e1d17f0e-5be8-4322-80f3-cb9a9ca6757b.png)
  - 모든 요소가 처리되거나 작업이 예외를 throw할 때까지 Iterable의 각 요소에 대해 지정된 작업을 수행합니다. 구현 클래스에서 달리 지정하지 않는 한 작업은 반복 순서로 수행됩니다(반복 순서가 지정된 경우). 작업에 의해 throw된 예외는 호출자에게 전달됩니다.

실제 코드도 위 javadoc에서 설명하는 내용과 동일하게 구현되어 있다.
    
```java
/**
    *  ...
    * @implSpec
    * <p>The default implementation behaves as if:
    * <pre>{@code
    *     for (T t : this)
    *         action.accept(t);
    * }</pre>
    *
    * ...
    */
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```
- 내부 구현을 전부 노출하고 있어 정보 은닉이라든지 캡슐화가 깨지기 때문에 좋은 예는 아니다.
- `좋은 API 문서란 '어떻게'가 아닌 '무엇'을 하는지를 설명해야 한다`라는 격언과 대치된다고 책에서도 언급하고 있다.

#### 2. protected로 공개해야 할 hook method 선별
(성능 또는 다형성을 위해)클래스의 내부 동작 중간에 끼어들 수 있는 hook을 잘 선별해서 protected method 형태로 공개해야 할 수도 있다.

* * * * *
- **Hook**  
Head first, Design Patterns (오라일리) 책에는 후크에 대해서 이렇게 말하고 있다.
> 후크(Hook) 는 추상 클래스에 들어있는, 아무 일도 하지 않거나 기본 행동을 정의하는 메소드로, 서브 클래스에서 오버라이드 할 수 있습니다.

```java
public abstract class IntroduceTemplate {

    String name;
    int age;

    public IntroduceTemplate(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public final void introduceOneSelf(){
        sayName();
        sayAge();
        saySpeciality();
        sayMessage();
    }

    public void sayName(){
        System.out.println("안녕하십니까? 저는 "+name+"입니다.");
    }

    public void sayAge(){
        System.out.println("저의 나이는 "+age+"세 입니다.");
    }

    //hook method
    public void saySpeciality(){
    }

    public abstract void sayMessage();
}
```
위 소스는 템플릿 메소드 패턴을 사용한 면접 시물레이션 예제이다. 이 면접에서는 이름과 나이를 말하고, 자기의 장기를 이야기하고, 마지막으로 하고 싶은 말을 하는 과정으로 면접을 봐야한다고 가정하자. 그런데 여기서 이름과 나이를 말하는 방법은 정해져있고, 장기는 있는 경우에만 말하고 없는 경우는 말할 필요가 없다. 자기소개는 양식이 없이 한다.
  
따라서 final 선언을 해준 introduceOneSelf() 메소드는 템플릿을 제공하고 있다. final 선언을 한 이유는 하위 클래스에서 오버라이드 할 수 없도록 하기 위함이다. 말그대로 이름과 나이를 말하는 메소드는 이미 구현이 되어있고 후크 메소드로 사용이 될 saySpeciality() 메소드는 구현이 안되어있으며 아무 일도 안한다. sayMessage()는 추상 메소드로 되어있다. 상속받는 클래스는 무조건 이 메소드를 구현해야한다.
  
서브 클래스를 한번 살펴보자.
```java
public class Ara extends IntroduceTemplate {

    public Ara(String name, int age) {
        super(name, age);
    }

    @Override
    public void saySpeciality(){
        System.out.println("저는 블로그에 맛집 정보 기록하는 것을 잘합니다 :D");
    }

    @Override
    public void sayMessage() {
        System.out.println("열심히 하겠습니다!");
    }

    public static void main(String[] args) {
        IntroduceTemplate Ara = new Ara("조아라",26);
        Ara.introduceOneSelf();
    }
}
```

saySpeciality 메소드는 기존 슈퍼클래스에서 아무것도 하지 않는 메소드였으나 이 메소드를 다시 오버라이드하여 재구현했다. `오버라이드 하여 전혀 다른 메소드로 만든다`라는 측면에서는 추상메소드와 크게 다를바가 없어 보이지만 차이가 있다. 추상메소드는 `강제적`이고 후크메소드는 `선택적`이다. 더군다가 abstract라는 키워드도 필요 없다. 필요에 따라서 오버라이드를 해도 되고, 안해도 되는 메소드가 바로 후크 메소드이다.

##### 템플릿메소드 패턴(Template-method pattern) 
> 어떤 같은 형식을 지닌 특정 작업들의 세부형식을 다양화하고자 할 때 사용하는 패턴  

템플릿 메소드에서의 상속은 일정 형식이 있다. 부모 클래스에 전반 과정을 수행하는 **메인 메소드**가 있고 그 과정 가운데 **세부 메소드**가 있다.  
메인 메소드를 호출하면 실행 중에 세부 메소드들이 호출되는 형태를 가진다.
- **메인 메소드**: final로 선언되어 자식 클래스에서 오버라이딩 불가.

자식 과정에서는 그 세부 메소드들을 오버라이딩 한다. 자식 클래스들은 세부 메소드 하나하나 자기만의 스타일로 개발할 수 있고, 이와 같은 방식으로 다양한 하위 클래스들을 생성할 수 있다.  
즉 어떤 일을 수행하는 몇가지 방법이 있는데 그 전반적인 과정에 공통적인 절차가 있을 때 코드를 효율적으로 짜기 위해 만들어진 패턴이다.

* * * * *
- `AbstractList.java`의 공개 API `clear()`와 `removeRange(int,int)` example
    - `removeRange` javadoc 중

  > **Overriding this method** to take advantage of the internals of the list implementation **can substantially improve the performance of the clear operation on this list and its subLists.**

    - `리스트 구현의 내부 구조를 활용하도록 이 메소드를 재정의하면 이 리스트와 부분리스트의 clear 연산 성능을 크게 개선할 수 있다.`

```java
// AbstractList
public void clear() {
    removeRange(0, size());
}

protected void removeRange(int fromIndex, int toIndex) {
    ListIterator<E> it = listIterator(fromIndex);
    for (int i=0, n=toIndex-fromIndex; i<n; i++) {
        it.next();
        it.remove();
    }
}
```
```java
// ArrayList
protected void removeRange(int fromIndex, int toIndex) {
    modCount++;
    int numMoved = size - toIndex;
    System.arraycopy(elementData, toIndex, elementData, fromIndex,
                        numMoved);

    // clear to let GC do its work
    int newSize = size - (toIndex-fromIndex);
    for (int i = newSize; i < size; i++) {
        elementData[i] = null;
    }
    size = newSize;
}
```
- protected hook을 결정하는 방법?**하위 클래스 몇개를 구현해서 테스트 해보기**
    - protected method 하나하나가 내부 구현에 해당하므로 그 수는 가능한 한 적어야 한다.
    - 너무 적게 노출해서 상속으로 얻는 이점마저 없애지 않도록 주의해야 한다.

#### 3. 객체 초기화와 overridable method
- 생성자 또는 Clonable,Serializable interface method 에서 overridable method를 호출하지 않게 주의한다.
    - 자식 클래스의 인스턴스가 초기화되기 전에 부모 클래스 생성자, `clone()`, `readObject()` 메소드 등에서 overridable method를 호출해 초기화 되지 않은 자식 클래스 필드에 접근해 에러를 발생시킬 수 있다.
    - 하위 클래스에 final field가 있다면, final field 초기화 전에 field에 접근, final field의 상태가 두 가지가 되는 문제가 있다.

### 참조
- [후크 메소드 (Hook Method)](https://mrtint.tistory.com/358)
- [객체지향 디자인패턴 2](https://www.youtube.com/watch?v=q3_WXP9pPUQ&t=541s)