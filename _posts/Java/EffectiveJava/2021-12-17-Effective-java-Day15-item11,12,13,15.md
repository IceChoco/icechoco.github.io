---
title: '[Effective Java] Day 15 - Item 11, 12, 13, 15 :: equals를 재정의하려거든 hashcode도 재정의하라, toString을 항상 재정의하라, clone 재정의는 주의해서 진행하라, 클래스와 멤버의 접근 권한을 최소화하라(1)'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava hashCode hashTable
comments: true
---

Day15에서는 item 11, 12, 13, 15에 대한 내용을 다룬다.

## Item 11 :: equals를 재정의하려거든 hashcode도 재정의하라.

그렇지 않으면 hash를 사용하는 HashMap, HashSet과 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.  

### hashcode의 규약
- equals 비교에 사용되는 정보가 변경되지 않았다면, 객체의 hashcode 메소드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. (단, Application을 다시 실행한다면 값이 달라져도 상관없다.)
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체는 똑같은 hashCode를 반환한다.
- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

hashCode 재정의를 잘못했을 때 문제가 되는 부분은 두 번째 조항이다. 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야한다.

```java
public class PhoneNumber {
    private int firstNumber;
    private int secondNumber;
    private int thirdNumber;

    public PhoneNumber(int firstNumber, int secondNumber, int thirdNumber) {
        this.firstNumber = firstNumber;
        this.secondNumber = secondNumber;
        this.thirdNumber = thirdNumber;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof PhoneNumber)) {
            return false;
        }

        PhoneNumber p = (PhoneNumber) o;
        return this.firstNumber == p.firstNumber &&
                this.secondNumber == p.secondNumber &&
                this.thirdNumber == p.thirdNumber;
    }

    public static void main(String[] args) {
        System.out.println("Instance 1 hashcode : " + new PhoneNumber(707, 867, 5307).hashCode());
        System.out.println("Instance 2 hashcode : " + new PhoneNumber(707, 867, 5307).hashCode());

        // Instance 1 hashcode : 1028214719
        // Instance 2 hashcode : 1706234378

        HashMap<PhoneNumber, String> map = new HashMap<>();
        map.put(new PhoneNumber(707, 867, 5307), "제니");
        System.out.println(map.get(new PhoneNumber(707, 867, 5307))); // null 반환
    }
}
```

put할 때의 인스턴스와 get할 때의 두 가지 인스턴스를 같은 버킷에 담았다 하더라도 get 메서드는 여전히 null을 반환한다. 그 이유는 HashMap의 경우 해시코드가 다른 엔트리끼리는 동치성 비교를 시도조차 하지 않도록 최적화되어 있기 때문이다.  

위와 같은 문제는 적절한 hashCode 메소드를 작성하는 것만으로 해결이 가능하다.  

## 최악의 hashCode 구현
```java
@Override
public int hashCode() {
    return 42;
}
```
동치인 모든 객체에서 똑같은 해시코드를 반환하니 적법하다.  
하지만 끔찍하게도 모든 객체에게 똑같은 값을 반환한다. 따라서 모든 객체가 같은 해시테이블 버킷에 담겨 연결리스트처럼 동작한다.  
평균 수행시간이 O(1)인 해시테이블이 O(n)으로 느려져서 성능이 낮아진다.

hashCode가 같다면 HashMap에서는 어떻게 동작할까?
```java
HashMap<PhoneNumber, String> map = new HashMap<>();
map.put(new PhoneNumber(707, 867, 5307), "제니");
System.out.println(map.get(new PhoneNumber(707, 867, 5307))); // 제니
System.out.println(map.get(new PhoneNumber(707, 867, 5301))); // null
```

hashcode가 같더라도 동치성비교(equals)를 실행하여 객체에 대한 값(value)를 조회한다.

![item11_HashMap_getNode](/assets\img/item11_HashMap_getNode.PNG)

## 좋은 해시 함수 만들기
(세 번째 규약) 서로 다른 인스턴스에 대해 다른 해시코드를 반환한다.  
이상적인 해시 함수는 주어진 인스턴스들을 32비트 정수 범위에 균일하게 분배하는 것이다.  

```java
@Override
public int hashCode() {
    // 1. int 변수 result 선언 후 값을 핵심 필드에 대한 hashcode로 초기화한다.

    // 2. 기본 타입 필드라면 Type.hashCode()를 사용한다.
    // Type은 기본타입의 Boxing 클래스이다.
    int result = Integer.hashCode(firstNumber);

    // 3. 참조 타입이라면 참조타입에 대한 hashCode 함수를 호출한다.
    // 4. 값이 null이면 0으로 사용한다.
    result = 31 * result + Address.hashCode() == null ? 0 : Address.hashCode();

    // 5. 필드가 배열이라면 핵심 원소를 각각 필드처럼 다룬다.
    for (String arg : args) {
        result = 31 * result + arg == null ? 0 : arg.hashCode();
    }

    // 6. 배열의 모든 원소가 핵심 필드라면 Arrays.hashCode를 이용한다.
    result = 31 * result + Arrays.hashCode(args);

    return result;
}
```
파생 필드는 hashCode 계산에서 제외해도 된다.  
equals 비교에 사용되지 않는 필드는 반드시 제외해야한다.  
31 * result를 곱하는 순서에 따라 result 값이 달라진다.  
 - String의 hashCode를 31을 곱하지 않고 구현한다면 모든 아나그램(anagram, 구성하는 철자가 같고 그 순서만 다른 문자열)의 해시코드가 같아지기 때문  

곱할 순서를 31로 지정하는 이유는 31이 홀수이면서 소수(prime) 이기 때문이다. 2를 곱하는 것은 shift 연산과 같은 결과라서 추천하지 않는다.  
31 * i는  (i << 5) - i와 같다.
 - 31 * 1은 31 == 100000(2) - 1은 32 - 1 = 31
 - 31 * 2는 62 == 1000000(2) - 2은 64 - 2 = 62

```java
@Override
public int hashCode() {
    int result = Integer.hashCode(firstNumber);
    result = 31 * result + Integer.hashCode(secondNumber);
    return 31 * result + Integer.hashCode(thirdNumber);
}
```

## hashCode를 만들어주는 모듈
- Objects.hash
    - 내부적으로 AutoBoxing이 일어나 성능이 떨어짐.
- Lombok @EqulasAndHashCode
- Google @AutoValue

## hashCode 재정의 할 때 주의 할점
- 불변 객체에 대해서는 hashCode 생성 비용이 많이 든다면, hashCode를 캐싱하는 것도 고려하자. 
    - 스레드 안전성까지 고려해야한다.
- 성능을 높인답시고 hashCode를 계산할 때 핵심필드를 생략해서는 안된다.
    - 속도는 빨라지지만 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수 있다.
- hashCode 생성규칙을 사용자에게 자세히 공표하지 말자.
    - 클라이언트가 이 값에 의지하지 않게 되고 추후에 계산 방식을 바꿀 수도 있다.
    - 자세한 규칙을 공표하지 않는다면 해시 기능에서 결함을 발견했거나 더 나은 해시 방식을 알아낸 경우 다음 릴리즈에 수정이 가능하다.
    
## Item 12 :: toString을 항상 재정의하라
###  toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.
Object의 기본 toString 메서드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다.   
실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는게 좋다.

``` java
class PhoneNumber {

	int areaCode, prefix, lineNum;
	
	public PhoneNumber(int areaCode, int prefix, int lineNum) {
		this.areaCode = areaCode;
		this.prefix = prefix;
		this.lineNum = lineNum;
	}

	/* @Override
	public String toString() {
		return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
	} */
}
```   

> 주석 처리 후 실행 시 결과

``` java
    public static void main(String[] args) {
		PhoneNumber pn = new PhoneNumber(123, 456, 7890);
		System.out.println(pn + "에서 오류가 발생했습니다.");
	}
```
toString을 재정의하지 않는다면 `item12.PhoneNumber@7cd84586에서 오류가 발생했습니다.`  
(클래스_이름@16진수로_표시한_해시코드)

하지만, 주석을 해제하면 `123-456-7890에서 오류가 발생했습니다.`

### toString을 구현할 때면 반환값의 포맷을 문서화할지 정해야 한다.
단점: 포맷을 명시하면 좋으나 포맷을 명시하면 평생 포맷에 얽매이게 된다.   
포맷 명시와는 관계없이 의도는 명확히 밝혀야 하고, **toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API 를 제공하자.**  
toStirng 값에 의존하지 않도록 필요한 정보들은 액세스 메서드인 getter를 따로 구현해두는 것이 좋다.  
그렇지 않으면 정보가 필요한 경우 toString 반환값을 파싱할 수 밖에 없는데, 이렇게 쓰는 것은 매우 좋지 않다.

#### 포맷 명시 예제 코드
```java
/**
  * 이 전화번호의 문자열 표현을 반환한다.
  * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다. 
  * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
  * ...
  * (생략)
  * ...
  */
  @Override
  public String toString() {
      return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
  }
```

### 포맷 명시하지 않는 예제 코드
```java
/**
  * 상세 형식은 정해지지 않았으며, 향후 변경될 수 있다.
  * "[약물 #9: 유형=사랑, 겉모습=먹물]"
  * ...
  * (생략)
  * ...
  */
  @Override
  public String toString() { ... }

```
### 결론
**모든 구체 클래스에서 Object의 toString을 재정의하자.**   
> 상위 클래스에서 이미 알맞게 재정의한 경우는 예외다.   
> toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다.   
> toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.
> 
> (+ 의견)  
> toString을 리턴하는 건 로그를 프린트 하거나, 디버깅할 때 쓰는 것이 좋다고 생각한다.

## Item 13 :: clone 재정의는 주의해서 진행하라
객체를 복사할 수 있는 방법으로 Object클래스는 clone 메소드를 제공하고 있다. 이 메소드를 사용하기 위해서 주의해야할 부분들에 대해서 살펴보려고 한다. 또한, 객체를 복사할때 clone보다 더 좋은 방법에 대해서도 알아보도록 하자.

### Cloneable 인터페이스와 clone 메소드의 관계
- Cloneable은 복제할 수 있는 클래스임을 명시하는 **믹스인 인터페이스** 이다. 하지만 복제를 위한 어떤 메소드도 정의하고 있지 않다. (* 믹스인 인터페이스는 기존 클래스의 주된 기능에 새로운 선택적 기능을 부여)
- 복제를 위한 clone 메소드는 **Object에 protected로 선언**되어 있다. 이 clone 메소드는 Coneable의 구현 유무에 따라서 동작이 바뀐다. Cloneable을 구현하면 복사한 객체를 반환하며 그렇지 않으면 CloneNotSupportedException을 던진다.  
![item13-object-clone](/assets\img/item13-object-clone.png)
- 외부에서 clone 메소드를 접근하기 위해서는 public으로 오버라이딩해야 한다.

```java
// Cloneable 인터페이스를 구현
public class PhoneNumber implements Cloneable{
   // Object의 Clone메소드를 public으로 오버라이딩
   @Override
   public PhoneNumber clone() {
   		try {
      	    return (PhoneNumber)super.clone();
     	} catch(CloneNotSupportedException e) {
     	    throw new AssertionError();  // 일어날 수 없는 일이다.
     	}
   }
}
```

Cloneable을 구현한 클래스에서 clone 메소드를 사용하기 위해서는 이런 이례적이며 일반적이지 않은 방법을 준수해야 한다.

### clone의 명세는 강제력이 없다
#### clone 명세
> x.clone() != x // 서로 다른 객체
>
> x.clone().getClass() == x.getClass() // 동일한 클래스를 참조 
>
> x.clone().equals(x) // 동치 관계

* 일반적으로 위 식들에 대해서 결과는 참(True)이다. 하지만 반드시 만족해야하는 것은 아니다.
* 관례상 clone으로 반환되는 객체는 super.clone을 호출해서 얻음. x클래스와 x클래스의 모든 상위 클래스(Object제외)에서 이 관례를 따른다면 x.clone.getClass() == x.getClone()은 참.
* 관례상 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하기 위해서 super.clone으로 얻은 객체의 필드 중 하나를 수정해야할 수도 있다. ex) 가변 필드

#### 상속시 발생할 수 있는 문제 상황
```java
class A {
    @Override
    public A clone() {
        return new A(); // 컴파일러는 경고를 하지 않는다!!
    }
}

class B extends A {
    @Override
    public B clone() {
        return (B)super.clone(); // A가 생성되었다.
    }
}
```

Class B가 클래스 A를 상속할 때, 하위 클래스인 B의 clone은 B타입 객체를 반환해야한다. 그런데 A의 clone 메소드가 super.clone이 아닌 생성자를 통한 객체를 반환하면 하위 클래스에서 문제가 될 수 있다 (다운캐스팅으로 인한 런타임 에러) 달리 말해 super.clone을 연쇄적으로 호출하도록 구현해놓으면 clone이 처음 호출된 상위 클래스의 객체가 만들어진다.

### 제대로 동작하는 clone 메소드

- 모든 필드가 기본 타입 or 불변 객체를 참조하면 super.clone을 통해서 객체의 복제본을 얻을 수 있다.

  >쓸데없는 복사를 지양하기 위해 불변 클래스는 clone 메소드를 제공하지 않는게 좋다.

- Super.clone에서 얻은 객체를 반환하기 전에 자신의 클래스로 형변환한다. 이렇게 함으로 클라이언트가 형변환하지 않아도 된다.

  > 자바는 공변 반환 타이핑을 지원한다(since java 1.5). 메소드 오버라이딩시 리턴타입을 원본 메소드 리턴타입의 Subclass로 오버라이딩 가능

- 검사 예외인 CloneNotSupportedException에 대해서 처리한다. 자세한 내용은 Item 71에서 ... (사실은 비검사 예외였어야 했다.)

```java
@Override
public PhoneNumber clone() {
    try {
        return (PhoneNumber)super.clone();
    } catch(CloneNotSupportedException e) {
        throw new AssertionError();  // 일어날 수 없는 일이다.
    }
}
```
> 가변 상태를 참조하지 않는 클래스용 clone 메소드

### 가변 필드를 가지고 있다면...
#### 클래스의 필드에 배열이 있을 경우
```java
public class Stack implements Cloneable {
   private Object[] elements; // 가변 필드
   private int size = 0;
   private static final int DEFAULT_INITIAL_CAPACITY = 16;

   public Stack() {
      this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
   }

   public void push(Object e) {
      ensureCapacity();
      elements[size++] = e;
   }

   public Object pop() {
      if (size == 0)
         throw new EmptyStackException();
      Object result = elements[--size];
      elements[size] = null; // Eliminate obsolete reference
      return result;
   }

   public boolean isEmpty() {
      return size == 0;
   }

   @Override
   public Stack clone() {
      try {
         Stack result = (Stack)super.clone();
         result.elements = elements.clone(); // 새로운 배열 생성
         return result;
      } catch (CloneNotSupportedException e) {
         throw new AssertionError();
      }
   }

   private void ensureCapacity() {
      if (elements.length == size)
         elements = Arrays.copyOf(elements, 2 * size + 1);
   }
}
```

- elements 배열의 clone 메소드를 재귀적으로 호출해서 각각 다른 배열을 가질 수 있다. 이를 통해 elements 공유로 불변식을 해칠 수 있는 문제가 해결한다.
  > 배열은 clone 기능을 제대로 사용한 유일한 예

- super.clone을 통해 복제본이 생성된 후 elemnts 배열의 clone 호출로 새로운 값을 할당하기 위해서 final 한정자를 제거해야 한다.

#### 재귀적으로 clone을 호출해도 해결하지 못하는 경우
링크드리스트의 집합인 buckets는 clone을 통해서 각 링크드리스트들의 복사본을 만들 수 없다. 직접 deepCopy가 가능한 메소드를 만들어줘야하는데 재귀를 이용할 경우 리스트의 길이에 따라서 스택 오버플로가 발생할 수 있다.
```java
public class HashTable implements Cloneable {    private Entry[] buckets; // 복잡한 가변 필드

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;     //지금 키
            this.value = value; // 지금 값
            this.next = next;   // 다음 원소
        }

        //재귀 - 비권장
        Entry deepCopy() {
            return new Entry(key, value, next == null ? null : next.deepCopy());
        }
    }
    @Override
    public HashTable clone() throws CloneNotSupportedException {
        try {

            HashTable result = (HashTable)super.clone();
            // 2. buckets 필드를 새로운 buckets 배열로 초기화
            result.buckets = new Entry[buckets.length];
            // 3.원본 테이블에 담긴 모든 버킷을 순회
            for (int i = 0; i < buckets.length; i++) {
                //버킷이 비어있지 않으면, 그 인덱스의 버킷을 deepCopy함
                if(buckets[i] != null) {
                    result.buckets[i] = buckets[i].deepCopy();
                }
            }
            return result;
        } catch ( CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

이를 방지하기 위해서는 반복문을 통해서 링크드리스트 전체 노드를 복사해주는 것이 좋다.
```java
//반복문 - 권장
Entry deepCopy() {
    // 첫번째 인덱스의 deepCopy가 호출되면,
    // Entry 값이 똑같은 result라는 Entry 생성
    Entry result = new Entry(key, value, next);
    // p는 result와 같은 주소값을 참조. 즉, p.next로 값을 수정하면 result.next도 바뀜
    for(Entry p = result; p.next != null; p = p.next) {
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    }
    return result;
}
```

#### 고수준 api 사용해서 복사
새로운 bukets 배열을 생성하고 원본 bukets의 모든 데이터를 put(key, value) 메소드를 통해서 동일하게 만들 수 있다.

하지만 필드 단위 객체 복사를 우회하기 때문에 Cloneable 아키텍처와 어울리지 않고 저수준에서 처리할 때보다 느릴 수 있다.

### 기타 고려사항
#### 재정의될 수 있는 메서드를 호출하면 안됨
- clone 메소드에서 하위 클래스에서 재정의 할 수 있는 메소드를 호출하면 안된다. 하위 클래스의 복제 과정에서 자신의 상태를 교정할 기회를 읽고 원본과 복제본의 상태가 달리질 수 있다. (의도되지 않은 상태, 오작동)

#### CloneNotSupprotedException 처리
- Clone을 재정의한 메서드에서는 사용하는 쪽의 편의를 위해서 CloneNotSupprotedException를 던지지 않아야된다. 

#### 상속해서 쓰기 위한 클래스에서 clone
- 상속용 클래스는 Cloneable을 구현해서는 안된다.
- 2가지 구체적 구현 방법
  - Object와 같이 제대로 동작하는 clone 메소드를 구현하고 protected로 두고 CloneNotSupportedException도 던질 수 있도록 구현. 하위 클래스에서 선택할 수 있도록 한다.
  ```java
  @Override
  protected final Object clone() throws CloneNotSuppotedException {
  	return super.clone();
  }
  ```

  - clone 메소드를 동작하지 않게 구현하고 하위 클래스에서 재정의하지 못하도록 한다.
  ```java
  @Override
  protected final Object clone() throws CloneNotSuppotedException {
  	throw new CloneNotSuppotedException();
  }
  ```

#### 쓰레드 동기화
- Object의 clone 메소드는 동기화를 신경 쓰지 않았다. clone 메소드를 오버라이딩 할 때 동기화에 대해서도 고려해야 한다.

### 더 좋은 방법
이미 클론 사용중인게 아니라면 복사생성자와 복사팩토리를 사용하자
- Cloneable을 이미 구현한 클래스를 확장하는게 아니라면 복사 생성자와 복사 팩토리가 더 나은 객체 복사 방식을 제공한다.

#### 복사 생성자
자신과 같은 클래스의 인스턴스를 인수로 받는 생성자
```java
public Yum(Yum yum){...};
```

#### 복사 팩터리
복사 생성자를 모방한 정적 팩터리
```java
public static Yum newInstance(Yum yum){...};
```

위 2가지 방식은 생성자를 통해서 객체를 생성하며 클래스의 필드에 final을 사용하는데 제약이 없고 불필요한 CloneNotSupportedException 예외를 처리하지 않아도 된다.

인터페이스 타입의 인스턴스를 인수로 받아서 객체를 생성할 수 있다.
```java
HashSet<T> s = ...;
TreeSet<T> t = new TreeSet<>(s);
```

## Item 15 :: 클래스와 멤버의 접근 권한을 최소화하라(1)
**내부 구현을 외부 컴포넌트로부터 잘 숨겼다면** 잘 설계된 컴포넌트라고 할 수 있다. 오직 **API를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는 것이다** → `정보 은닉` 혹은 `캡슐화`라는 개념으로 부른다.

정보 은닉은 다음과 같은 장점을 가진다.

- API를 통해서 다른 컴포넌트와 소통하기 때문에 API를 먼저 설계하고 나면 여러 컴포넌트를 병렬로 개발할 수 있어 **시스템 개발 속도를 높인다.**
- 각 컴포넌트를 더 빨리 파악하여 디버깅할 수 있고, 다른 컴포넌트로 교체하는 부담도 적어 **시스템 관리 비용을 낮춘다.**
- 정보 은닉 자체가 성능을 높여주지는 않지만, 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화 할 수 있기 때문에 **성능 최적화에 도움을 준다.**
- 외부에 거의 의존하지 않는 컴포넌트라면 **소프트웨어 재사용성을 높인다.**
- 시스템 전체가 아직 완성되지 않은 상태에서도 개별 컴포넌트의 동작을 검증할 수 있기 때문에 **큰 시스템을 제작하는 난이도를 낮춰준다.**