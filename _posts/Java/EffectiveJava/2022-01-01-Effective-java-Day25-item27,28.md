---
title: '[Effective Java] Day 25 - Item 27, 28 :: 비검사 경고를 제외하라, 배열보다는 리스트를 사용하라'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day25에서는 item 27,28에 대한 내용을 다룬다.

## Item 27 :: 비검사 경고를 제외하라
제너릭을 활용하면 아래와 같은 컴파일러의 비검사 경고를 볼 수 있다.  
- 비검사 형변환 경고(unchecked cast warning)
- 비검사 메서드 호출 경고(unchecked method invocation warning)
- 비검사 매개변수화 가변인수 타입 경고(unchecked parameterized vararg type warning)
- 비검사 변환경고(unchecked conversion warning) 등

**비검사 경고 예**  
![item27_unchecked_assignment](/assets\img/item27_unchecked_assignment.png)
javac 명령줄 인수에 `-Xlint:unchecked`를 추가하면 뭐가 잘못됐는지 위와 같이 설명해준다.  
  
인텔리제이에서 명령줄 인수를 추가하기 위해서는 Settings - Build, Execution, Deployment - Compiler - Java Compiler에서 설정할 수 있다.
![item27_intelliJ_additional_option](/assets\img/item27_intelliJ_additional_option.png)

위 소스는 아래와 같이 바꾸면 경고가 사라진다. 자바 7부터는 다이아몬드 연산자인 `<>`로 타입 매개변수인 `<String>`를 대체할 수 있다. 컴파일러는 타입 매개변수를 추론한다.
```java
Set<String> exaltation = new HashSet<String>();
```

### 할 수 있는 한 모든 비검사 경고를 제거하라.
경고를 제거할 수는 없지만 타입이 안전하다고 확신할 수 있다면 @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨기자.  

#### @SuppressWarnings
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
1. 개별 지역변수 ~ 클래스 전체까지 선언할 수 있다.
2. 가능한 한 좁은 범위에 적용하자.  
 - ex) 변수 선언, 아주 짧은 메서드, 생성자
 - 절대로 클래스 전체에 적용해서는 안된다.
3. 한 줄이 넘는 메서드나 생성자에 달린 @SuppressWarnings 애너테이션을 발견하면 지역변수 선언쪽으로 옮기자.
 - 이를 위해 지역변수를 새로 선언하는 수고를 해야 할 수도 있지만, 그만한 값어치가 있을 것이다!
 - ex) ArrayList의 toArray 메서드
```java
@SuppressWarnings("unchecked") //unchecked: 미확인 오퍼레이션과 관련된 경고 억제
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
4. @SuppressWarnings("unchecked") 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남기자.
 - 다른 사람이 그 코드를 이해하는데 도움이 된다.
 - 다른 사람이 그 코드를 잘못 수정하여 타입 안전성을 잃는 상황을 줄여준다.

### 결론
- 최선을 다해 모든 비검사 경고를 제거하라
  - 모두 제거하면 타입 안정성이 증명된다.
  - ClassCastException이 발생할 일이 없다.
- 경고를 제거할 수 없고 타입이 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchekced")`를 달아 경고를 숨겨라
  - 경고를 숨기기로 한 근거를 주석으로 남겨라

### 참고
- [Valid @SuppressWarnings Warning Names](https://www.baeldung.com/java-suppresswarnings-valid-names)

## Item 28 :: 배열보다는 리스트를 사용하라
### 배열과 리스트(제너릭 타입)의 차이
#### 1. 배열 (Array)
**1) 배열은 함께 변한다. 즉, 공변(convariant)이다.**
- Sub extends Super인 경우, Sub[] extends Super[]이다.  

```java
class Sub extends Super {
}

Super[] superman = new Sub[1];
```
**2) 배열은 실체화(reify)된다.**  
- 컴파일타임: 타입 불안전 / 런타임: 타입 안전
```java
Object[] objectArray = new Long[1];
objectArray[0] = "String을 넣어봅시다!";
```
Object 배열에 Long용 저장소를 할당했다.  
그러므로 objectArray에는 Long타입만 올 수 있다. String은 못온다.  
그런데 위 코드는 컴파일 에러가 나지 않는다.
```java
Exception in thread "main" java.lang.ArrayStoreException: java.lang.String
at item28.BetweenArrayAndList.main(BetweenArrayAndList.java:7)
```
런타임에서 에러가 난다.  
→ 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인하기 때문

#### 2. 리스트 (List)
**1) 리스트는 함께 변하지 않는다. 즉, 불공변(invariant)이다.**
- 상위-하위타입 관계가 아니다.
```java
List<Super> superList = new ArrayList<Sub>();//컴파일 에러!
```

**2) List는 실체화(reify)되지 않는다.**  
- 컴파일타임: 타입 안전 / 런타임: 타입 불안전
```java
//컴파일 에러
List<Object> objList = new ArrayList<Long>();
objList.add("String을 넣어봅시다!");
```
리스트는 컴파일할 때 바로 알 수 있다. 단, 런타임 시 타입정보가 소거된다.  
원소 타입을 컴파일 타임에만 검사, 런타임에는 알 수 조차 없다.  

→ 소거: 제너릭이 지원되기 전의 레거시 코드와 제너릭 타입을 함께 사용할 수 있게 해주는 메커니즘  
  자바5가 제너릭으로 순조롭게 전환될 수 있도록 해준 일등공신!

#### 실체화(reify)
- **실체화**: JVM이 type 정보를 완전히 알게 되거나, 알게 하는 것, 또는 알고 체크하는 것
- 실체화 가능 타입(reifiable type)
  - 매개변수화 타입 가운데 실체화 될 수 있는 타입은 비한정적 와일드카드 뿐이다.
  - ex) `List<?>`, `Map<?,?>`
- 실체화 불가 타입(non-reifiable type)
  - 실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가진다.
  - ex) E, `List<E>`, `List<String>`

### 제너릭 배열을 만들지 못하게 막은 이유는?
#### 1. 배열과 제너릭은 잘 어우러지지 못한다.
`new List<E>[]`, `new List<String>[]`, new E[] → 컴파일 시 오류 발생!
#### 2. 타입 안전하지 않다.
위 두가지 이유 때문인데 자세한건 아래 예시를 보고 이해해보자.  
**ex) 제너릭 배열 생성이 허용된다 가정**
```java
List<String>[] stringList = new List<String>[1]; // (1) 허용된다 가정
List<Integer> initList = List.of(42);            // (2) 
Object[] objects = stringList;                   // (3)
objects[0] = initList;                           // (4)
String s = stringList[0].get(0);                 // (5)
```
(1)처럼 제너릭 배열이 생성이 허용된다고 일단 가정해보자.  
(2)에서 원소가 42 하나인 `List<Integer>`를 생성한다.  
(3)은 (1) 에서 생성한 `List<String>` 배열을 Object 배열에 할당한다. 즉, Object 배열은 `List<String>` 배열을 참조하게 된다. 배열은 공변이므로 가능하다.  
![item28_If_there_is_generic_array](/assets\img/item28_If_there_is_generic_array.PNG)
(4)는 Objects 배열의 첫 원소에 원소가 42 하나인 initList를 저장한다.
   
- 문제점
   - `List<String>` 인스턴스만 받겠다고 선언한 stringList 배열에 `List<Integer>` 인스턴스가 들어가있다.
  
(5)는 이 배열의 첫 리스트에서 원소를 꺼내려 한다. 컴파일러는 꺼낸 원소 42를 자동으로 String으로 형변환한다.
```java
String s = (String) 42; // ClassCastException 발생
```

이런 일을 방지하기 위해서 제너릭배열이 생성되지 않도록 (1)에서 컴파일 오류를 내야하는 것이다.

### 배열대신 List처럼 원소를 관리하자
배열로 형변환할 때 제너릭 배열 생성오류 or 비검사 형변환 경고가 뜨는 경우  
→ 배열인 E[] 대신 컬렉션인 `List<E>`를 사용하자!
  
**ex) 생성자에서 컬렉션을 받는 Chooser 클래스**  
이 클래스는 컬렉션 안의 원소 중 하나를 무작위로 반환하는 choose 메서드를 제공한다.
#### 제너릭 미사용 버전
```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices){
        choiceArray = choices.toArray();
    }

    public Object choose(){
        Random rnd = ThreadLocalRandom.current();
        return  choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```
이 클래스를 사용하려면 choose 메서드 호출 시 마다 반환된 Object를 원하는 타입으로 형변환 해야한다.  
혹시나 원하는 타입과 다른 타입의 원소가 들어있는 경우 런타임 시 형변환 오류가 발생할 것이다.
#### 제너릭 사용 버전 1
```java
public class Chooser2<T> {//제너릭으로 변환
    private final T[] choiceArray;

  //@SuppressWarnings("unchecked") // 비검사 형변환 경고(unchecked cast경고)
    public Chooser2(Collection<T> choices){
        choiceArray = (T[]) choices.toArray(); //unchecked cast: 'java.lang.Object[]' to 'T[]'
    }

    public Object choose(){
        Random rnd = ThreadLocalRandom.current();
        return  choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```
`choiceArray = (T[]) choices.toArray();`에서 unchecked cast: 'java.lang.Object[]' to 'T[]' 경고가 발생한다.  
- T[]가 무슨 타입인지 알 수 없으니 컴파일러는 이 형변환이 런타임에도 **안전한지 보장할 수 없다**라는 뜻  
- 제너릭에는 원소의 타입 정보가 소거되어 런타임에는 무슨 타입인지 알 수 없다.  
- 컴파일러가 안전을 보장하지 못할 뿐 소스코드는 동작하며, List에 비해 **빠르다.**  
- 코드작성자가 안전하다고 확신한다면 주석을 남기고 애너테이션을 달아 경고를 숨겨도 된다.  
   - 단, 경고의 원인을 완전히 제거하는게 훨씬 좋은 방법이다(item 27)

#### 제너릭 사용 버전 2
```java
public class Chooser3<T> {//제너릭으로 변환
    private final List<T> choiceList;//리스트로 변경

    public Chooser3(Collection<T> choices){
        choiceList = new ArrayList<>(choices);
    }

    public Object choose(){
        Random rnd = ThreadLocalRandom.current();
        return  choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```
- 장점: 타입 안전성과 상호운용성이 좋아진다. 런타임이 아닌 컴파일시점에 에러를 잡아준다.
- 단점: 코드가 조금 복잡해진다. 성능이 살짝 느려질 수 있다.

### 결론
- 배열과 리스트를 섞어쓰기란 쉽지 않다.
  - **Array**: 공변. 실체화된다.
    - 컴파일타임: 타입 불안전 / 런타임: 타입 안전
  - **List**: 불공변. 타입정보가 소거된다.
    - 컴파일타임: 타입 안전 / 런타임: 타입 불안전
- 섞어쓰다가 컴파일 오류나 경고를 만나면, 배열을 List로 대체해라.

### 참고
 - [Java 배열과 리스트의 공변성과 반공변성, 무공변성](https://junroot.github.io/programming/Java-%EB%B0%B0%EC%97%B4%EA%B3%BC-%EB%A6%AC%EC%8A%A4%ED%8A%B8%EC%9D%98-%EA%B3%B5%EB%B3%80%EC%84%B1%EA%B3%BC-%EB%B0%98%EA%B3%B5%EB%B3%80%EC%84%B1,-%EB%AC%B4%EA%B3%B5%EB%B3%80%EC%84%B1/)