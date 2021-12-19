---
title: '[Effective Java] Day 8 - Item 14 :: Comparable을 구현할지 고려하라'
layout: post
categories: java
tags: java
comments: true
---

CompareTo는 Comparable 인터페이스의 유일무이한 메서드이다. 두 가지 성격만 빼면 Object의 equals와 같다.

### compareTo의 성격
1. 단순히 같은지 비교하는 것에 더해 **순서**까지 비교할 수 있다.
2. 제너릭하다

```java
public interface Comparable<T> {
    /**
     * 기준 값.compareTo( 비교대상 )
     *
     * @param   o – 비교대상 오브젝트
     * @return 기준 값 < 비교대상  : a negative integer
     *         기준 값 == 비교대상 : 0
     *         기준 값 > 비교대상  : positive integer
     * @throws NullPointerException
     * : 기준 값.compareTo(null)은 NullPointer를 발생시킴.
     *   기준 값.equals(null)이 false를 번환하는 것과 차이가 있음.
     * @throws ClassCastException
     * : 이 객체와 비교할 수 없는 타입의 객체가 주어지면 발생
     */
    public int compareTo(T o);
}
```

Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연스러운 순서가 있음을 뜻한다. 그렇기 때문에 Comparable을 구현한 객체들의 배열은 `Arrays.sort(a)`로 손쉽게 정렬할 수 있다.

```java
public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```

위는 입력값에 대해 중복 제거 후 알파벳순으로 출력하는 프로그램이다. String이 Comparable을 구현하였기 때문에 가능하다.

- Comparable이 구현된 String
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
        ...
        public int compareTo(String anotherString) {
            int len1 = value.length;
            int len2 = anotherString.value.length;
            int lim = Math.min(len1, len2);
            char v1[] = value;
            char v2[] = anotherString.value;

            int k = 0;
            while (k < lim) {
                char c1 = v1[k];
                char c2 = v2[k];
                if (c1 != c2) {
                    return c1 - c2;
                }
                k++;
            }
            return len1 - len2;
        }
}
```

사실상 자바 플랫폼 라이브러리의 모든 값 클래스와 열거타입이 Comparable을 구현했다.  
알파벳, 숫자, 연대 같이 순서가 명확한 클래스를 작성하는 경우 반드시 Comparable을 구현하자.
 
CompareTo는 Object로 인수를 받을 경우에 한해 타입이 다른 객체 간의 비교도 허용한다. 그러나 대부분 비교할 객체들이 구현한 공통 인터페이스를 매개로 비교가 이루어지며, 타입이 다른 객체가 주어지면 간단히 ClassCastException을 던진다.
 
### compareTo 메서드의 일반 규약
compareTo 규약을 지키지못하면 비교를 활용하는 클래스와 어울리지 못한다.  
비교를 활용하는 클래스 ex)  
- TreeSet, TreeMap : 정렬된 컬렉션
- Collecions, Arrays: 검색과 정렬알고리즘을 활용하는 유틸리티 클래스

**참고**: a.compareTo(b)에서 a > b 는 positive, a == b는 0, a < b는 negative  
#### 1. 대칭성 - 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.
  1) a가 b보다 작으면 b는 a보다 커야한다. → a.compareTo(b)가 -1 이면 b.compareTo(a)는 1이어야 한다.  
  2) Aa b와 같으면 b는 a와 같아야한다. → a.compareTo(b)가 0이면 b.compareTo(a)는 0이어야 한다.  
  3) a가 b보다 크면 b는 a보다 작아야한다 → a.compareTo(b)가 1이면 b.compareTo(a)는 -1이어야 한다.  

#### 2. 추이성 - 첫 번째가 두 번째보다 크고, 두 번째가 세번째보다 크면 첫 번째는 세번째보다 커야한다
a.compareTo(b)가 1 이고 b.compareTo(c)가 1 이면 a.compareTo(c)는 1이어야 한다.

#### 3. 반사성 - 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.
a.compareTo(b)가 0이고 a.compareTo(c)가 0이면 a.compareTo(c)도 0이다.

이상의 세 규약은 `equals` 규약과 똑같이 **<span style="color:red">반사성, 대칭성, 추이성</span>**을 충족해야 함을 뜻한다.  
그래서 주의사항도 똑같다.

#### (권장) 4. compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다.
이를 잘 지키면 compareTo로 줄지은 순서와 equals의 결과가 같게 된다. 같지 않아도 동작은 하지만 이 클래스의 객체를 정렬된 컬랙션에 넣으면 Collection, Set, Map 등 그 컬렉션이 구현한 인터페이스의 정의된 동작이 잘 이루어지지 않는다.

#### compareTo 와 equals 결과 일관성을 지키지 않은 사례
```java
    public static void main(String[] argc){
        Set bigDecimalHashSet = new HashSet<>();
        bigDecimalHashSet.add(new BigDecimal("1.0"));
        bigDecimalHashSet.add(new BigDecimal("1.00"));

        Set bigDecimalTreeSet = new TreeSet();
        bigDecimalTreeSet.add(new BigDecimal("1.0"));
        bigDecimalTreeSet.add(new BigDecimal("1.00"));

        System.out.println(bigDecimalHashSet.size());   // 원소 2개 - equals 메소드로 비교하기 때문
        System.out.println(bigDecimalTreeSet.size());   // 원소 1개 - compareTo 메소드로 비교하기 때문


    }
```
동일한 객체를 삽입하였지만 HashSet과 TreeSet의 사이즈는 왜 다를까?  
**HashSet에서는 equals를 통한 동치비교**를 진행하지만, **TreeSet에서는 compareTo를 통한 동치비교**를 진행하기 때문이다.

```java
        System.out.println(new BigDecimal("1.0").equals(new BigDecimal("1.00")));    // false
        System.out.println(new BigDecimal("1.0").compareTo(new BigDecimal("1.00"))); // 0 (true)
```

BidDecimal 클래스에서는 equals에서는 둘을 서로 다른 객체로 인식하고, compareTo는 둘은 같은 객체(정수 1)로 인식한다는 것을 알 수 있다.

### compareTo 주의해야할 점

상속을 포기할 것이 아니라면, 기존 클래스를 확장한 구체 클래스에서 새로운 값 필드를 추가하여 CompareTo 규약을 만족시킬 수 있는 방법은 없다.
단 포기한다면 compareTo 제약을 만족하면서 새로운 값 필드를 추가할 수 있는 방법이 있다.

```java
package item14.composition;

// 단순한 불변 2차원 정수 점(point) 클래스 (56쪽)
public class Point implements Comparable<Point> {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public int compareTo(Point point) {
        int result = Integer.compare(x, point.x);
        if (result == 0) {
            return Integer.compare(y, point.y);
        }
        return result;
    }

    public static void main(String[] args) {
        ColorPoint cp1 = new ColorPoint(new Point(1,2),3);
        ColorPoint cp2 = new ColorPoint(new Point(1,2),4);

        System.out.println(cp1.compareTo(cp2)); //출력 -1. 즉, cp1 < cp2
    }
}

//View 메서드(asPoint)를 제공
class ColorPoint implements Comparable<ColorPoint>{
    private Point point;
    private int color;

    public ColorPoint(Point point, int color) {
        this.point = point;
        this.color = color;
    }

    public Point asPoint(){
        return point;
    }

    @Override
    public int compareTo(ColorPoint colorPoint) {
        int result = point.compareTo(colorPoint.point);
        if (result == 0) {
            return Integer.compare(color, colorPoint.color);
        }
        return result;
    }
}
```

따라서 이와같이 Point와 독립된 클래스를 만들고, ColorPoint 클래스에 Point 클래스의 인스턴스를 가리키는 필드를 둔 다음 내부 인스턴스를 반환하는 **뷰 메서드**를 제공하면 된다.  
상속을 포기하는 대신 compareTo의 일반 규약을 지킬 수 있다 (자세한 내용 item10 참고)
 
### CompareTo 작성요령
1. comparable은 타입을 인수로 받는 제너릭 인터페이스 이다.
 - compareTo 메서드의 인수타입은 컴파일타임에 정해진다.
 - 입력 인수의 타입을 확인하거나 형변환 할 필요가 없다.
 - 인수의 타입이 잘못됐다면 컴파일 자체가 되지 않는다.
2. null을 인수로 넣어 호출하면 NullPointerException을 던져야 한다.
 - 실제로도 null이 멤버에 접근하려는 순간 이 예외가 던져진다.
3. 객체 참조 필드를 비교할 경우, compareTo를 재귀적으로 호출한다.
```java
class ColorPoint implements Comparable<ColorPoint>{
    private Point point; //객체참조필드 : 클래스 내 변수로 선언된 객체
    private int color;
    ...
}
```

아래 두 가지 경우는 Comparator 인터페이스를 사용한다.  
  1) 클래스 내 변수 중 comparable을 구현하지 않은 필드가 있는 경우  
  2) 표준이 아닌 순서로 비교해야하는 객체가 섞여 있는 경우

Comparator 인터페이스 구현은 직접 해도되고 또는 자바가 제공하는 것 중 하나를 골라서 사용하면 된다. 아래는 자바에서 제공하는 Comparator 메서드를 사용한 예시이다.
```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    // 자바가 제공하는 비교자를 사용해 클래스를 비교한다.
    @Override public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
    ...
}
```
CaseInsensitiveString 클래스가 `Comparable<CaseInsensitiveString>`를 구현했다.  
CaseInsensitiveString의 참조는 CaseInsensitiveString 참조와만 비교할 수 있다는 뜻이며, Comparable을 구현할 때 일반적으로 쓰는 패턴이다.

### compareTo 메서드에서 `<`와 `>`를 사용하는 이전 방식은 쓰지 마라
  1) 자바 7 이전
```java
    if (a < b) {
        return -1;
    } else if (a > b) {
        return 1;
    } else {
        return 0;
    }
    Float.compare(a,b);
    Double.compare(a,b);
```
  2) 자바 7 이후 (권고사항)
```java
   Integer.compare(a,b);
   Float.compare(a,b);
   Double.compare(a,b);
```

관계연산자를 이용한 방식은 휴먼에러를 발생 시킬 수 있고, null 관련 오류를 일으킬 수 있다.
```java
public class MyInteger {

    // 1. 자바 7 이전 : 관계연산자 <, > 사용
    public static class RelationalMyInteger implements Comparable<RelationalMyInteger> {
        private Integer integer;

        @Override
        public int compareTo(RelationalMyInteger relationalMyInteger) {
            if (integer == relationalMyInteger.integer) {
                return 0;
            }
            if (integer > relationalMyInteger.integer) {
                return 1;
            } else {
                return -1;
            }
        }
    }

    // 2. 자바 7 이후 : compare 이용 비교
    public static class CompareMyInteger implements Comparable<CompareMyInteger> {
        private Integer integer;

        @Override
        public int compareTo(CompareMyInteger compareMyInteger) {
            return Integer.compare(integer, compareMyInteger.integer);
        }
    }

    public static void main(String[] args){

        RelationalMyInteger relationalMyInteger1 = new RelationalMyInteger();
        RelationalMyInteger relationalMyInteger2 = new RelationalMyInteger();

        // 1. 자바 7 이전 : 관계연산자
        // 출력: 0
        System.out.println(relationalMyInteger1.compareTo(relationalMyInteger2));

        CompareMyInteger compareMyInteger1 = new CompareMyInteger();
        CompareMyInteger compareMyInteger2 = new CompareMyInteger();

        // 2. 자바 7 이후 : compare 이용 비교
        // Exception in thread "main" java.lang.NullPointerException
        System.out.println(compareMyInteger1.compareTo(compareMyInteger2));
    }
}
```

정적 메서드 compare에 null 값이 사용되는 경우 정상적으로 NullPointerException이 던져지지만,  
관계연산자 비교에서는 정상적으로 컴파일이 진행되고 서로 같다는 결과를 반환한다.

### 핵심필드가 여러개라면 가장 핵심적인 것부터 비교하자
가장 핵심적인 것부터 비교하다가 비교 결과가 0이 아니라면 그 즉시 결과를 곧장 반환하자.
```java
public int compareTo (PhoneNumber pn){
    int result = Short.compare(areaCode, pn.areaCode);  //가장 중요한 필드
    if (result == 0){   // 여기서 비교가 끝날 수도 있다.
        result = Short.compare(prefix, pn.prefix);      //두 번째로 중요한 필드
        if(result == 0)
            result = Short.compare(lineNum, pn.lineNum);//세 번째로 중요한 필드
    }
    return result;
}
```

### 비교자 생성 메서드를 활용한 연쇄 비교자 생성
Comparator 생성 메서드 패턴을 이용한다면 보다 읽기 쉽고 간결한 코드가 될 수 있으나 약간의 성능저하가 뒤따르니 유의하자.
```java
    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);

    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
```
#### comparingInt(int 타입의 키 추출 함수)
객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받아서, 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적메서드
```java
    default Comparator<T> thenComparingInt(ToIntFunction<? super T> keyExtractor) {
        return thenComparing(comparingInt(keyExtractor));
    }
```
앞의 예에서 comparingInt는 람다를 인수로 받아 areaCode의 순서를 정하는 `Comparator<PhoneNumber>`를 반환한다.  
이 때 Java 컴파일러의 타입 추론을 돕기 위해 입력 인수의 타입을 명시해줘야 한다.

#### thenComparingInt(int 타입의 추출자 함수)
Comparator의 인스턴스 메서드로, int 키 추출자 함수를 입력받아 다시 비교자를 반환한다.  
이때 첫 번쨰 비교자를 적용한 다음 새로 추출한 키로 추가 비교를 수행한다.
  
원하는 만큼 연달아 호출이 가능하며, 입력인수의 타입을 명시하지 않는다.
  
    
이 외에도 Comparator는 보조 생성 메서드가 많고 **자바의 숫자용 기본 타입을 모두 커버**한다.
```java
.comparingInt(ToIntFunction keyExtractor);                       // int, short
.comparingLong(ToLongFunction keyExtractor);                     // long
.comparingDouble(ToDoubleFunction keyExtractor);                 // float, double
// 객체 참조용 비교자 생성 메서드
.comparing(Function keyExtractor);                               // 키의 자연적 순서를 이용한 비교
.comparing(Function keyExtractor, Comparator keyComparator);     // 비교자 추가
// 객체 참조용 보조 비교자 생성 메서드
.thenComparing(Comparator keyComparator);                        // 원본 키에 보조 비교자 추가
.thenComparing(Function keyExtractor);                           // 키의 자연적 순서를 비용한 보조 비교
.thenComparing(Function keyExtractor, Comparator keyComparator); // 키와 비교자 모두 추가
```

### '값의 차'를 기준으로 반환하는 compareTo / compare 메소드
첫 번째 값이 두번째 값보다 작으면 음수를, 두 값이 같으면 0을, 첫 번째 값이 크면 양수를 반환하는 compareTo / compare 메소드를 마주할 수 있다.
- 해시코드 값의 차를 기준으로 하는 비교자
```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
}
```

위 방식은 첫 번째가 두 번째보다 크고, 두 번째가 세번째보다 크면 첫 번째는 세번째보다 커야하는 추이성을 위배한다.  
값의 차를 이용한 방식을 이용하여 추이성을 위배하는 사례는 아래와 같다.
```java
int a = Integer.MAX_VALUE;
int b = 0;
int c = Integer.MIN_VALUE;

// expected a > b > c
System.out.println(a-b);
System.out.println(b-c);
System.out.println(a-c);

//출력
2147483647
-2147483648
-1
```

값의 차를 이용하는 방식을 사용하면 안되는 이유는 크게 2가지 인데, 이 중 가장 핵심적인 이유는 `Integer overflow 가능성` 때문이다.
1. Integer overflow 가능성
2. IEEE 754 부동소수점 계산 방식에 따른 오류 가능성  

아래 두 방식 중 하나를 사용하자
```java
// 정적 compare 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
}

// 비교자 생성 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder =
        Comparator.comparingInt(o -> o.hashCode());
```

### 결론
1. **순서를 고려해야하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하자**
 - 이를 통해 인스턴스들을 쉽게 정렬, 검색, 비교 기능을 제공하는 컬렉션과 어우러지게 사용 가능하다.
2. **compareTo 메서드에서 필드 값 비교 시 `<`와 `>`연산자는 사용하지 말자**
 - 그 대안으로 아래 2가지 중 하나를 사용하자.
        - 박싱된 기본 타입 클래스가 제공하는 compare 메서드
        - Comparator 인터페이스가 제공하는 비교자 생성 메서드