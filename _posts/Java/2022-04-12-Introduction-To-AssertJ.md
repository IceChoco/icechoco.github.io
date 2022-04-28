---
title: '[AssertJ] AssertJ란?'
layout: post
categories: java
tags: java AssertJ
comments: true
---

오늘의 목표: 공식문서를 보고 Junit과 AssertJ를 직접 사용하며 사용법 익히기!  
  
우아한 테크캠프 프리코스 과정에서는 AssertJ와 Junit을 사용하여 단위테스트를 구현하라는 미션이 주어졌다. 평소에 나는 업무를 진행할 때 배치 프로그램을 로컬에서 수행하기 위해 Junit을 사용하고, 단위 테스트를 구현해본적이 없었다. 따라서 AssertJ와 Junit을 공식 문서를 통해 배우고, 프리코스 과제 구현을 통해 다양하게 사용하며 다양한 기능을 익히고자 이 포스팅을 작성하게 되었다.

참고로 아래 글은 자바에서 AssertJ를 이용해 테스트를 작성하기 위한 여러 사용법에 대해 간단하게 소개하는 내용이다. 따라서 assertJ가 뭔지 잘 모르거나 단위 테스트를 막 시작하시는 분들이 가볍게 봐주시면 좋을 것 같다.

## AssertJ란?
- Java 테스트에서 능숙하고 풍부한 Assertions을 작성하는 데 사용되는 오픈 소스 커뮤니티 기반 라이브러리
  - Junit과 함께 편리하게 사용하게 해주는 라이브러리로, 테스트코드 작성을 훨씬 편리하게 도와준다.
- 이 글은 **AssertJ-core**라는 기본 AssertJ 모듈에서 사용할 수 있는 툴에 초점을 맞추고 있다.
- Assertions
  - 사전적 의미: 사실임을 주장하는 것
  - 표명, 가정 설정문, 어서션이라고도 함
  - 프로그램 안에 추가하는 **참 거짓을 미리 가정하는 문**

그런데 Junit에서 기본 제공되는 assertion이 이미 있는데 왜 assertJ를 사용해야할까?

### 1. Gradle Dependencies
assetJ를 사용하기 위해서는 build.gradle 파일에 있는 `dependency` 부분에 아래와 같은 코드를 추가해줘야 한다.
```
dependencies {
    testImplementation "org.assertj:assertj-core:3.20.2"
}
```
위 디펜던시는 오직 기본 Java assertions만 제공한다. 따라서 만약 advanced assertions를 사용하고 싶다면, 추가로 별개의 모듈을 디펜던시에 추가해줘야한다.  
Java 7 이전 버전의 경우 Assertj-core version 2.x.x를 사용해야 한다. 최신 버전은 [여기](https://search.maven.org/search?q=a:assertj-core)에서 찾을 수 있다.

### 2. 소개
AssertJ는 여러 클래스들의 집합과 풍부하고 아름다운 assertions을 쉽게 작성할 수 있는 유틸리티 메소드를 제공한다.
- 사용 가능한 언어 
  - Standard Java
  - Java 8
  - Guava
  - Joda Time
  - Neo4J and
  - Swing components

사용 가능한 모든 언어들은 프로젝트 [웹사이트](https://joel-costigliola.github.io/assertj/)에서 확인 가능하다.  
몇 개의 예시를 갖고 시작해보자. AssertJ의 문서에서 바로 다음 내용을 확인할 수 있다.
```
assertThat(frodo)
  .isNotEqualTo(sauron)
  .isIn(fellowshipOfTheRing);

assertThat(frodo.getName())
  .startsWith("Fro")
  .endsWith("do")
  .isEqualToIgnoringCase("frodo");

assertThat(fellowshipOfTheRing)
  .hasSize(9)
  .contains(frodo, sam)
  .doesNotContain(sauron);
```
위의 예들은 빙산의 일각에 불과하지만, 이 라이브러리로 Assertions를 쓰는 것이 어떤식으로 사용되는지에 대한 예시를 보여준다.

### 3. AssertJ 실행하기
이 섹션에서는 AssertJ를 설정하고 이를 통해 무엇을 할 수 있는지에 대해 알아본다.
#### 3.1 시작하기
클래스패스에 있는 라이브러리의 jar을 사용하면, 테스트 클래스에 하나의 static import를 추가하여 쉽게 활성화가 가능하다.
```
import static org.assertj.core.api.Assertions.*;
```
#### 3.2 Assertions 작성하기
asssertion을 작성하기 위해서는, 항상 객체를 `Assertions.assertThat()` 메서드에 전달하는 것부터 시작해야한다. 그런 다음 실제로 assertions를 수행한다.  
일부 다른 라이브러리와는 달리 아래 코드는 실제로 아무것도 참이라고 주장하지 않으며, **절대로** 테스트에 실패하지 않는다.  
```
assertThat(anyRefenceOrValue);
```
IDE의 코드완성기능을 사용하면 AssertJ Assertions를 쉽게 작성 가능하다. 인텔리제이 IDEA 2021.3 버전에서는 아래처럼 나온다.
![assertJ_IntelliJ](/assets\img/assertJ_IntelliJ.png)

위처럼 상황별로 선택할 수 있는 다양한 메소드가 제공되며, 이 메소드들은 String 타입에만 사용이 가능하다.  
이 API에 대해 자세히 살펴보고 특정 Assertion에 대해 알아보자.

#### 3.3 Object Assertions
두 객체가 동일한지를 동일성을 비교하거나, 객체의 내부값이 같은지 동등성을 비교하기 위해 다양한 방법을 사용할 수 있다.  
2개의 Object가 동일한지에 대해 비교할 수 있는 두가지 방법을 보자.  
다음은 Dog 타입인 fido, fidosClone를 포함한 총 7가지의 객체가 있다.
```java
public class Dog {
    private String name;
    private Float weight;

    //Getter, Setter, Constructor
}

public class DogTest {
  Dog fido;
  Dog fidosClone;
  HungryDog hyngryDog;
  HungryDog hyngryDogClone;
  HungryDogs hyngryDogs;
  HungryDogs hyngryDogsClone;
  List<DogName> dogNameList;

  @BeforeEach
  void setting(){
    fido       = new Dog("Fido", (float) 5.25);
    fidosClone = new Dog("Fido", (float) 5.25);

    hyngryDog = new HungryDog(1L, new DogName("bowWow"), BigDecimal.valueOf(1000L));
    hyngryDogClone = new HungryDog(1L, new DogName("bowWow"), BigDecimal.valueOf(1000L));
    hyngryDogs = new HungryDogs(1L, dogNameList, BigDecimal.valueOf(1000L));
    hyngryDogsClone = new HungryDogs(1L, dogNameList, BigDecimal.valueOf(1000L));


    dogNameList = Arrays.asList(new DogName("bowWow"),new DogName("bowWow"),new DogName("bowWow"),new DogName("bowWow"));
  }
}
```
##### 동일성 검증
아래와 같은 assertion을 통해 쉽게 두 객체를 비교할 수 있다.
```java
@Test
void isEqualTo(){
    assertThat(fido).isEqualTo(fidosClone);
}
```

위를 실행하면 `isEqualTo` 메소드는 두 객체의 참조를 비교하므로 fail을 반환한다.  
그 대신 필드값을 비교하려면 다음과 같이 ~~`isEqualToComparingFieldByFieldRecursively()`(AsserJ 3.12.0 버전 이전 사용 방식)~~ `usingRecursiveComparison()` 메소드를 사용하면 된다.  
클래스 멤버도 내부 필드값으로 비교하며, 리스트도 내부 컬렉션 값의 필드를 비교한다.
이 메서드를 활용한 다양한 사용법은 [여기](https://javadoc.io/doc/org.assertj/assertj-core/3.13.0/org/assertj/core/api/RecursiveComparisonAssert.html)에서 볼 수 있다.

```java
@Test
void isEqualToComparingFieldByFieldRecursively(){
    assertThat(fido).usingRecursiveComparison()
                    .isEqualTo(fidosClone);

    //AssertJ 3.12.0 버전 이전 사용 방식
    //.isEqualToComparingFieldByFieldRecursively(fidosClone);
}
```

Fido랑 FidosClone 객체는 필드와 필드를 재귀적으로 비교할때 똑같다. 왜냐하면 한 객체의 필드를 다른 객체의 필드와 비교하기 때문이다.  
이 밖에도 객체의 필드들에 대해 비교하고, 축소하고, 조사하는 다양한 assertion 메소드가 있다. 모든 메소드를 확인하려면 공식 [AbstractObjectAssert docs](https://joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractObjectAssert.html)를 참조하자.

`using`이라는 키워드로 접근하면 아래와 같이 사용할 수 있는 Comparator가 조회된다.
![assertj_using](/assets\img/assertj_using.PNG)

아래는 알아두면 좋은 몇가지 키워드들이다.

- usingComparator(): 자신이 직접 정의한 Custom Comparator를 통해 비교하고 싶은 경우
- ignoring: 특정 값이나, 타입 등을 제외하고 비교하고 싶은 경우
  - ignoringFields(): 특정 필드 값을 제외하고 비교하고 싶은 경우
  ```java
  assertThat(fido).usingRecursiveComparison()
    .ignoringFields("weight")
    .isEqualTo(fidosClone);
  ```
  - ignoringFields(): 특정 필드 값을 제외하고 비교하고 싶은 경우
  ```java
  assertThat(fido).usingRecursiveComparison()
                .ignoringActualNullFields()
                .isEqualTo(fidosClone);
  ```

```java
        //그냥 객체 vs 객체
        assertThat(fido)
                .usingRecursiveComparison()
                .isEqualTo(fidosClone);

        //클래스 필드가 있는 객체
        assertThat(hyngryDog)
                .usingRecursiveComparison()
                .isEqualTo(hyngryDogClone);

        //리스트 필드가 있는 객체
        assertThat(hyngryDogs)
                .usingRecursiveComparison()
                .isEqualTo(hyngryDogsClone);
```

##### 중첩된 값을 꺼내서 검증하기
전체를 검증하는 것이 아니라 특정 객체에 속해있는 것만 비교하고 싶다면 `extracting()`을 쓰면 된다.  
매개변수는 Lamda를 받으며 특정 값을 꺼내서 사용할 수 있도록 한다.  
- 만약 비교하고자하는 필드가 리스트라면?
  - usingRecursiveFieldByFieldElementComparatorOnFields()
  - usingComparatorForElementFieldsWithType()

```java
  @Test
  void extracting(){
          //hungryDog의 이름을 가져와서 hungryDogClone의 이름과 필드 비교
          assertThat(hungryDog)
          .extracting(HungryDog::getName)
          .usingRecursiveComparison()
          .isEqualTo(hungryDogClone.getName());

          List<HungryDog> dogNames = Arrays.asList(hungryDog,hungryDog,hungryDog,hungryDog);

        assertThat(dogNames)
        .extracting(HungryDog::getName)       //개 4마리의 이름을
        .usingRecursiveFieldByFieldElementComparatorOnFields() //setting에서 이름만 담아놨던 리스트의 원소랑 하나씩 필드를 비교한다.
        .isEqualTo(dogNameList);

        //AssertJ 3.12.0 버전 이전 사용 방식
        //.usingFieldByFieldElementComparator() //setting에서 이름만 담아놨던 리스트의 원소랑 하나씩 필드를 비교한다.
  }
```

##### Filtering 하고 검증하기
`FilterOn`이라는 키워드를 사용하여 특정 Collection에 필터링을 먼저 한 뒤에 값을 검증할 수 있다.  
메서드 체이닝을 통해 `Stream`을 사용하는 것처럼 `fileredOn`도 다양한 필터링 조건을 추가할 수 있다. 매개변수는 Lamda를 받는다.
```java
@Test
void filterOn(){
    HungryDog americanDog = new HungryDog(1L, new DogName("bowWow"), BigDecimal.valueOf(1000L));
    HungryDog koreanDog = new HungryDog(1L, new DogName("meongmeong"), BigDecimal.valueOf(1000L));
    HungryDog gaeNyangI = new HungryDog(1L, new DogName("nyaong"), BigDecimal.valueOf(1000L));
    HungryDog multinationalDog = new HungryDog(1L, new DogName("bowWowMeong"), BigDecimal.valueOf(1000L));

    List<DogName> dogNameList = Arrays.asList(new DogName("bowWow"),new DogName("meongmeong"),new DogName("nyaong"),new DogName("bowWowMeong"));

    assertThat(dogNameList)
            .filteredOn((dogName) -> dogName.getName().contains("bowWow"))
            .usingRecursiveComparison()
            .isEqualTo(Arrays.asList(new DogName("bowWow"),new DogName("bowWowMeong")));
}
```

#### 3.4 Boolean Assertions
참거짓 테스트를 위한 간단한 메소드들이 있다.
- isTrue()
- isFalse()
예시 소스코드를 보자
```java
@Test
void isTrue(){
    assertThat("".isEmpty()).isTrue();
}
```

#### 3.5 Iterable/Array Assertions
Iterable 또는 Array의 경우 내용이 존재하는지에 대해 asserting 할 수 있는 다양한 방법이 있다. 가장 일반적인 assertions 중 하나는 `Iterable` 또는 `Array`에 주어진 element가 포함되어 있는지 확인 하는 것이다.
```java
@Test
void arrayContain() {
    List<String> list = Arrays.asList("1","2","3");
    assertThat(list).contains("1");
}
```
List가 비어있지 않은지 검사하기
```java
@Test
void arrayIsNotEmpty() {
    List<String> list = Arrays.asList("1","2","3");
    assertThat(list).isNotEmpty();
}
```

주어진 문자로 시작하는지 검사하기 ex) "1"
```java
@Test
void arrayStartsWith() {
    List<String> list = Arrays.asList("1","2","3");
    assertThat(list).startsWith("1");
}
```

동일한 개체에 대해 둘 이상의 Assertion을 작성하려는 경우 체이닝을 통해 쉽게 결합할 수 있다.  
제공된 list가 비어 있지 않은지, "1" 원소를 포함하는지, null을 포함하지 않는지, 원소 "2", "3"을 순서대로 포함하는지를 확인하는 Assertion의 예는 다음과 같다.
```java
@Test
void arrayTotalCheck() {
    List<String> list = Arrays.asList("1","2","3");
    assertThat(list).isNotEmpty().contains("1").doesNotContainNull().containsSequence("1","2");
    //containsSequence("1","2"): 원소 "2", "3"을 순서대로 포함하는지
}
```

#### 3.6 Character Assertions
character에 대한 Asssertions는 비교 대상 문자가 유니코드 테이블에 있는지 확인하는 작업을 포함한다.  
- 문자가 'a'가 아니고 유니코드 테이블에 있는지, 'b'보가 같거나 크고, 소문자인지 확인하는 assertion
```java
@Test
void characterTest(){
    char someCharacter = 'b';
    assertThat(someCharacter)
            .isNotEqualTo('a')
            .inUnicode()
            .isGreaterThanOrEqualTo('b')
            .isLowerCase();
}
```

#### 3.6 Class Assertions
해당 필드, 클래스 유형, annotations의 존재 등을 체크한다.
- Runnable 클래스가 인터페이스인지 확인
```java
@Test
void chkIsInterface(){
    assertThat(Runnable.class).isInterface();
}
```
- A 클래스에 B 클래스를 할당할 수 있는지 확인
```java
@Test
void IsAssignableFrom(){
    //assertThat(A.class).isAssignableFrom(B.class);
    assertThat(Exception.class).isAssignableFrom(NoSuchElementException.class);
}
```

#### 3.7 File Assertions
지정된 File 인스턴스가 존재하는지, 디렉토리 또는 파일인지, 특정 콘텐츠가 있는지, 읽을 수 있는지, 확장명이 있는지 등을 확인할 수 있다.
- 파일이 존재하고, 디렉토리가 아닌 파일이며, Read&Write가 가능한지 확인
```java
@Test
void chkFile(){
    File file = new File("C:\\github\\java-baseball-precourse\\src\\main\\java\\study\\Dog.java");
    assertThat(file)
            .exists()
            .isFile()
            .canRead()
            .canWrite();
}
```

#### 3.8 Double / Float / Integer Assertions
Numeric assertions는 주어진 오프셋을 포함하고 있는지 또는 포함하고 있지 않은지 모든 숫자값에 대해 비교하는 다양한 연산자들을 제공한다.
- **오프셋(offset)**
  - 컴퓨터 과학에서 배열이나 자료 구조 오브젝트 내의 오프셋(offset)은 일반적으로 동일 오브젝트 안에서 오브젝트 처음부터 주어진 요소나 지점까지의 변위차를 나타내는 정수형
  - 이를테면, 문자 A의 배열이 abcdef를 포함한다면 'c' 문자는 A 시작점에서 **2의 오프셋**을 지님.
  - 어셈블리어와 같은 저급 프로그래밍 언어에서 오프셋은 상대 주소(relative address)로 부름.

예를들어, 만약 주어진 Precision에 따라 두 값이 동일한지 체크를 하고 싶다면 아래처럼 하면 된다.
- Precision: 몇 자리 갭을 허용할거냐로 해석할건지 정확도를 지정하는 것. 0.123 이랑 0.124 같게 보려면 precision에 0.001d를 줘야함.

```java
@Test
void chkNumeric(){
    assertThat(5.123).isEqualTo(5.124, withPrecision(0.001d));//같음
    //assertThat(5.123).isEqualTo(5.124, withPrecision(0.0001d));//다름
    //assertThat(6.1).isEqualTo(5, withPrecision(1d));//다름
}
```

- isCloseTo: 그 숫자가 주어진 오프셋 값 내에서 인자로 주어진 값에 가까운지 확인한다.
```java
@Test
void percentage(){
    assertThat(BigDecimal.valueOf(10000L))
            .isCloseTo(BigDecimal.valueOf(9999L), Percentage.withPercentage(90));

    assertThat(BigDecimal.valueOf(10000L))
            .isNotCloseTo(BigDecimal.valueOf(5200L), Percentage.withPercentage(90));
}
```

이 밖에도 isBeetween(), isNegative() 등이 있다.

#### 3.9 InputStream Assertions
InputStream Assertion은 1가지만 있다.
- hasSameContentAs(InputStream expected)
```java
@Test
void testHasSomeContentAs(){
    byte[] bytes = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11};
    InputStream given = new ByteArrayInputStream(bytes);

    byte[] oneBytes = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11};
    InputStream result = new ByteArrayInputStream(oneBytes);

    assertThat(given).hasSameContentAs(result);
}
```

#### 3.10 Map Assertions
맵에 특정 항목, entry, key/value 값이 포함되어 있는지 확인한다.
- 맵이 비어있지 않고, Key 2 포함, Key 10 미포함, (2,"a")를 포함하는지 확인
```java
@Test
void chkMap(){
    HashMap<Integer, String> map = new HashMap<Integer, String>();
    map.put(2,"a");
//  map.put(10,"b");

    assertThat(map).isNotEmpty()
            .containsKey(2)
            .doesNotContainKey(10)
            .contains(entry(2,"a"));
}
```

#### 3.11 Throwable Assertions
Exception 메시지, stacktraces, 익셉션이 throw 되었는지 원인 확인 또는 검증할 때 사용
- 주어진 예외가 발생했는지 확인하고 "c"로 끝나는 메시지가 있는지 확인
```java
assertThat(ex).hasNoCause().hasMessageEndingWith("c");
```

`assertThatThrownBy`와 `assertThatExceptionOfType` 두 가지 방법으로 에외를 발생시켰는지 확인이 가능하다.
- assertThatThrownBy(): Java 8 Exception assertion 표준 스타일
- assertThatExeptionOfType(): 예외 클래스 넣기

------

## 학습 테스트
### String 클래스에 대한 학습 테스트
#### 요구사항 1
- "1,2"을 `,`로 split 했을 때 1과 2로 잘 분리되는지 확인하는 학습 테스트를 구현한다.
- "1"을 `,`로 split 했을 때 1만을 포함하는 배열이 반환되는지에 대한 학습 테스트를 구현한다.

##### 힌트
- 배열로 반환하는 값의 경우 assertj의 contatins()를 활용해 반환 값이 맞는지 검증한다.
- 배열로 반환하는 값의 경우 assertj의 contains Exactly()를 활용해 반환값이 맞는지 검증한다.

##### 테스트
```java
@Test
void split(){
    String str1 = "1,2";
    String str2 = "1";

    String[] strArray1 = str1.split(",");
    String[] strArray2 = str2.split(",");

    //"1,2"를 ,로 split 했을 때 1과 2로 잘 분리되는지 확인
    assertThat(strArray1).contains("1","2");

    //"1"을 ,로 split 했을 때 "1만을" 포함하는 배열이 반환되는지 확인
    assertThat(strArray1).containsExactly("1");
}
```
- **assertJ의 contains와 containsExactly의 차이**
  - contains: 순서와 상관 없이 실제 그룹이 주어진 값들을 포함하고 있는지를 테스트
  - containsExactly: 순서까지 고려해서 실제 그룹이 주어진 값들을 포함하고 있는지를 테스트

#### 요구사항 2
- "1,2" 값이 주어졌을 때 String의 substring() 메소드를 활용해 ()을 제거하고 "1,2"를 반환하도록 구현한다.

##### 테스트
```java
@Test
void subString(){
    String str1 = "(1,2)";
    String result = str1.substring(1,4);
    System.out.println(result);

    assertThat(result).isEqualTo("1,2");
}
```

#### 요구사항 3
- "abc" 값이 주어졌을 때 String의 charAt() 메소드를 활용해 특정 위치의 문자를 가져오는 학습 테스트를 구현한다.
- String의 charAt() 메소드를 활용해 특정 위치의 문자를 가져올 때 위치 값을 벗어나면 `StringIndexOutOfBoundException`이 발생하는 부분에 대한 학습 테스트를 구현한다.
- JUnit의 `@DisplayName`을 활용해 테스트 메소드의 의도를 드러낸다.

##### 힌트
- [AssertJ Exception Assertions](https://joel-costigliola.github.io/assertj/assertj-core-features-highlight.html#exception-assertion) 문서 참고

##### 테스트
```java
@Test
@DisplayName("특정 위치의 문자를 가져온다.")
void charAt(){
    String str = "abc";
    int idx = 2;
    str.charAt(idx);
}

@Test
@DisplayName("charAt - assertThatThrownBy()를 사용하여 Exception 처리")
void chatAtException(){
    String str = "abc";
    int idx = 3;

    assertThatThrownBy(()->{
       char c = str.charAt(idx);
    }).isInstanceOf(StringIndexOutOfBoundsException.class)
            .hasMessageContaining("%d", idx);
}

@Test
@DisplayName("charAt - assertThatExceptionOfType()를 사용하여 Exception 처리")
void chatAtException2(){
    String str = "abc";
    int idx = 3;

    assertThatExceptionOfType(StringIndexOutOfBoundsException.class)
            .isThrownBy(() -> {
                char c = str.charAt(idx);
            }).withMessageContaining("%d", idx);
}
```

### 결론
단위 테스트 코드 작성시 다양한 Assertions를 활용하여 작성하자. 실제로 과제 테스트 코드를 구현하면서 이거 있을 것 같은데? 하고 구글에 쳐보니까 적절한 메서드가 있는 경우가 굉장히 많았다.  
많이 사용해보면서 단위 테스트 코드 작성에 익숙해지자!

## 참조
- [Introduction to AssertJ](https://www.baeldung.com/introduction-to-assertj)
- [Assertion의 기본 지식](https://blog.naver.com/kunjalan/222622885642)
- [AssertJ의 다양한 메소드 활용해보기](https://tecoble.techcourse.co.kr/post/2020-11-03-assertJ_methods/)

