---
title: '[Effective Java] Day 30 - Item 63 :: 문자열 연결은 느리니 주의하라'
layout: post
categories: java
tags: java 이펙티브자바 effectiveJava
comments: true
---

Day30에서는 item 63에 대한 내용을 다룬다.

## Item 63 :: 문자열 연결은 느리니 주의하라
### 1. 문자열 연결 연산자(+)
```java
public String statement(){
    long start = System.currentTimeMillis();

    String result="";
    for(int i=0;i<numItems();i++)
        result += lineForItem(i); //문자열 연결

    System.out.println(System.currentTimeMillis()-start);
    return result;
}
```
- 여러 문자열을 하나로 합쳐주는 편리한 수단
  - 한 줄 짜리 출력값
  - 작고 크기가 고정된 객체의 문자열을 만들 때
- **문자열 n개를 잇는 시간: n^2에 비례**
  - 문자열은 불변이라 두 문자열을 연결할려면 양쪽의 내용을 모두 복사해야 함
  - 따라서 성능 저하가 발생하기 쉬움

### 2. StringBuilder
```java
public static final int LINE_WIDTH = 80;

public String statement2(){
    StringBuilder sb = new StringBuilder(numItems() * LINE_WIDTH); //전체 결과를 담기에 충분한 크기로 초기화
    for(int i=0;i<numItems();i++)
        sb.append(lineForItem(i)); //문자열 연결

    return sb.toString();
}
```
#### 문자열 n개를 잇는 시간: 선형으로 늘어남
품목이 100개, lineForItem이 길이 80인 문자열을 반환할 떄 `StringBuilder`를 활용한 방식이 문자열 연결 연산자(+)보다 **6.5배**나 빨랐음
  - 품목의 수가 늘어나면 성능차이는 더 난다.

10만개의 길이 80인 문자열을 연결해보았다.  
![item63_Concatenating_Strings](/assets\img/item63_Concatenating_Strings.PNG)  
문자열 연결 연산자(+)는 111.686초, `StringBuilder`는 0.017초로 무려 **6,569배** 차이가 난다.

#### 전체 결과를 담기에 충분한 크기로 초기화하면, 기본 값으로 사용하는 것 보다 더 빠름  
용량을 100,000*80으로 설정하고, 실제로는 500만개의 길이가 80인 문자열을 붙였다.  

```java
public static final int LINE_WIDTH = 80;

public String statement2(){
    StringBuilder sb = new StringBuilder(100000 * LINE_WIDTH);
    for(int i=0;i<numItems();i++)//numItems: 5,000,000
        sb.append(lineForItem(i)); //문자열 연결

    return sb.toString();
}
```
![item63_diffrent_capacity](/assets\img/item63_diffrent_capacity.PNG)

용량도 500만개로 맞춰서 붙여봤다.  
```java
public static final int LINE_WIDTH = 80;

public String statement2(){
    StringBuilder sb = new StringBuilder(5000000 * LINE_WIDTH);
    for(int i=0;i<numItems();i++)//numItems: 5,000,000
        sb.append(lineForItem(i)); //문자열 연결

    return sb.toString();
}
```
![item63_suitable_capacity](/assets\img/item63_suitable_capacity.PNG)

아래와 같이 StringBuilder에서 append할 때 **용량이 부족하면 입력받은 String의 길이 만큼 용량을 추가하는 로직을 수행**한다.
용량이 부족할 경우 이 작업이 추가로 이루어지기 때문에 초기 용량 값 설정이 속도에 영향을 준다.
![item63_stringBuilder](/assets\img/item63_stringBuilder.PNG)

### 3. Concat
10만개의 길이 80개인 문자열을 붙였다.  
```java
//3. Concat
public String statement3(){
    String result="";
    for(int i=0;i<numItems();i++)//numItems: 100,000
        result.concat(lineForItem(i)); //문자열 연결

    return result;
}
```
![item63_concat](/assets\img/item63_concat.PNG)
- concat과 StringBuilder의 차이점
  - **concat**: 매번 새로운 String 객체를 만듦
  - **StringBuilder**: append로 charArray를 만들었다가, toString메서드 호출 시 String 객체를 만듦

### 4. StringBuffer
10만개의 길이 80인 문자열을 붙였다. 
```java
public static final int LINE_WIDTH = 80;

//4. StringBuffer
public String statement4(){
    StringBuffer sb = new StringBuffer(numItems() * LINE_WIDTH);
    for(int i=0;i<numItems();i++)//numItems: 100,000
        sb.append(lineForItem(i)); //문자열 연결

    return sb.toString();
}
```
![item63_stringBuffer](/assets\img/item63_stringBuffer.PNG)

#### StringBuffer VS stringBuilder
StringBuffer의 append도 StringBuilder랑 똑같은 AbstractStringBuilder의 append 메서드를 사용한다.  
둘의 차이는 synchronized에 있다. StringBuffer는 thread safe하고, 빌더는 그렇지 않다.  

![item63_StringBuilder-And-StringBuffer](/assets\img/item63_StringBuilder-And-StringBuffer.PNG)

아래의 예시 소스를 통해 간단하게 비교해볼 수 있다.  
StringBuilderTest 클래스와 StringBufferTest 클래스는 각각 스레드가 실행되면  
append 메서드를 사용하여 A를 100번 연결한다.

```java
public class StringBuilderTest extends Thread {
    private static StringBuilder sb;

    public StringBuilderTest(StringBuilder sb) {
        this.sb = sb;
    }

    public void run(){
        for(int i=0;i<100;i++)
            sb.append("A");
    }

    public static final StringBuilder getSb(){
        return sb;
    }
}

public class StringBufferTest extends Thread {
    private static StringBuffer sb = new StringBuffer();

    public StringBufferTest(StringBuffer sb) {
        this.sb = sb;
    }

    public void run(){
        for(int i=0;i<100;i++)
            sb.append("A");
    }

    public static final StringBuffer getSb(){
        return sb;
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        StringBuilder sb = new StringBuilder();
        StringBuilderTest builderThreads = new StringBuilderTest(sb);
        Thread thread1 = new Thread(builderThreads);
        Thread thread2 = new Thread(builderThreads);

        thread1.start();
        thread2.start();

        try{
            thread1.join();
            thread2.join();
        }catch (Exception e){
            e.printStackTrace();
        }

        System.out.println("StringBuilderTest: " + builderThreads.getSb().length());

        //**************************************************************

        StringBuffer sb2 = new StringBuffer();
        StringBufferTest bufferThreads = new StringBufferTest(sb2);
        Thread bufferThread1 = new Thread(bufferThreads);
        Thread bufferThread2 = new Thread(bufferThreads);

        bufferThread1.start();
        bufferThread2.start();

        try{
            bufferThread1.join();
            bufferThread2.join();
        }catch (Exception e){
            e.printStackTrace();
        }

        System.out.println("StringBufferTest: " + bufferThreads.getSb().length());
    }
}
```
![item63-StringBuilderVsStringBuffer](/assets\img/item63-StringBuilderVsStringBuffer.PNG)

synchronized를 사용하면 block과 unblock을 처리하는 과정이 추가되어 StringBuilder보다 성능이 저하된다.  
그러므로 단일 스레드 환경에서는 StringBuilder, 멀티 스레드 환경에서는 StringBuffer를 사용하는 것이 좋다.

### 비교결과
#### 속도
10만개의 길이 80인 문자열을 붙였을 때 아래의 순서대로 속도가 빠르게 나왔다(단위: 초)  
**StringBuilder**(0.017) > **StringBuffer**(0.025) > **문자열 연결 연산자(+)**(111.686) > **Concat**(113.945)  

#### null 연산
null과 "a"라는 문자열을 더하면 concat만 NPE를 발생시키며,  
빌더, 버퍼, + 방식은 모두 null을 문자열로 더하여 anull을 출력하니 조심하자.  
그래서 append 하기 전에 null 체크를 하는게 최선이라고 생각한다. 더 좋은 아이디어가 있을까?

```java
public static void main(String[] args) {
    String a = "a";
    String b = null;

//#1
    String c = "";
    c += a;
    c += b;
    System.out.println(c);//anull
//#2
    String c2 = "";
    c2.concat(a);
    c2.concat(b);
    System.out.println(c2);//NPE
//#3
    StringBuilder sb = new StringBuilder();
    sb.append(a).append(b);
    System.out.println(sb);//anull
//#4
    StringBuffer sb2 = new StringBuffer();
    sb2.append(a).append(b);
    System.out.println(sb2);//anull
}
```

## 결론
- 성능에 신경을 써서 **많은 문자열을 연결해야한다면 문자열 연결 연산자(+)를 피해라**
- 대신 `StringBuilder`의 append 메서드를 사용해라