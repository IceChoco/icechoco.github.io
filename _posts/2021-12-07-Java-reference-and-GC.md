---
title: '[Java] Java Reference(참조)와 GC(Garbage Collector)'
layout: post
categories: java
tags: java
comments: true
---

**이 글의 목적**
- Java Reference의 종류인 Soft, Weak, Phanthom, Strong Refrenece에 대해서 이해하고 설명할 수 있다.
- <span style="color:grey">*네이버 NBP 웹플랫폼개발랩의 박세훈님이 올려주신 [Java Reference와 GC](https://d2.naver.com/helloworld/329631)를 보고 정리한 글입니다.* </span>

## GC(Garbage Collector, 가비지 컬렉터)
Java의 GC는 동작 방식에 따라 매우 다양한 종류가 있으나 공통적으로 크게 다음 2가지 작업을 수행한다.
1. 힙(heap) 내의 객체 중에서 가비지(garbage)를 찾아낸다.
2. 찾아낸 가비지를 처리해서 힙의 메모리를 회수한다.

최초의 Java에서는 이들 GC 작업에 애플리케이션의 사용자 코드가 관여하지 않도록 구현되어 있었다. 그러나 위 2가지 작업에서 좀 더 다양한 방법으로 객체를 처리하려는 요구가 있었다. 이에 따라 **JDK 1.2부터는** java.lang.ref 패키지가 추가되어 제한적이나마 **사용자의 코드와 GC가 상호작용할 수 있게** 하고 있다.

Java의 java.lang.ref 패키지는 전형적인 객체 참조인 Strong reference 외에도 **Soft, weak, phantom 3가지의 새로운 참조 방식**을 각각의 Reference 클래스로 제공한다. 이 3가지 Reference 클래스를 어플리케이션에 사용하면 GC에 일정 부분 관여할 수 있고, LRU(Least Recently Used) 캐시 같이 특별한 작업을 하는 애플리케이션을 더 쉽게 작성할 수 있다. 이를 위해서는 GC에 대해서도 잘 이해해야하고, 이들 참조방식의 동작도 잘 이해해야 한다.
- **LRU 캐시**  
캐시는 빠르고 작은 단기기억 공간이며 Key-Value 관계를 가지고 있다. 캐시 오버플로우가 났을 때 데이터를 지워주는 규칙을 LRU라고한다. LRU는 Least Recently Used의 약어이며 가장 과거에 사용된 데이터를 삭제해주는 것 이다.

## GC와 Reachability
Java GC는 객체가 가비지인지 판별하기 위해서 **reachability**라는 개념을 사용한다. 어떤 객체에 유효한 참조가 있으면 `reachable`로 없으면 `unreachable`로 구별하고, `unreachable` 객체를 가비지로 간주해 GC를 수행한다.  

한 객체는 여러 다른 객체를 참조하고, 참조 된 다른 객체들도 마찬가지로 또 다른 객체들을 참조할 수 있으므로 객체들은 참조 사슬을 이루게 된다. 이런 상황에서 유효한 참조 여부를 파악하려면 항상 **유효한 최초의 참조**가 있어야 하는데 이를 객체 참조의 **root set**이라고 한다.  

### JVM에서 런타임 영역(runtime data area)의 구조
- 런타임 영역은 JVM이 Java Bytecode를 실행하기 위해 사용하는 메모리 영역
![runtime-data-area](/assets\img/runtime-data-area.PNG)
<center>그림 1 런타임 데이터 영역(Oracle HotSpot VM 기준)</center><br>

런타임 데이터 영역 크게 세 부분으로 나눌 수 있다. 위 그림에서 객체에 대한 참조는 화살표로 표시되어 있다.  
1. **스레드**가 차지하는 영역 - 각 스레드 마다 존재  
 - **Java Stack**    
   1) 스레드 마다 1개만 존재하고, 스택 프레임은 메서드가 호출될 때 마다 생성된다. 메서드 실행이 끝나면 스택 프레임은 pop되어 스택에서 제거된다.  
   2) Stack Frame은 Local Variables Array, Operand stack, Frame Data를 갖는다.  
   3) Frame Data는 Constant Pool에 대한 참조, 이전 스택프레임에 대한 정보, 현재 메서드가 속한 클래스/객체에 대한 참조 등의 정보를 갖는다.
 - **PC Register(Program Counter)**: 각 스레드는 어떤 메서드를 항상 실행하고 있고, 그 때 PC는 그 메서드 안에서 Byte Code 몇 번째 줄을 실행하고 있는지를 나타내는 역할  
 - **Native Stack**  
   1) 성능향상을 위해 자바의 ByteCode가 아닌 C/C++로 작성된 코드를 컴파일해서 사용하는 경우가 있다. 그때 사용되는 메서드를 Natvie (method) Stack이라고 한다.  
   2) **JNI(Java Native Interface)**  
      자바는 특정 운영체제에 종속되지 않도록, JVM이라는 가상 머신 위에서 실행되게 끔 만들어진 언어이다.  
      운영체제에 맞는 JVM이 각기 존재하기 때문에, Java 개발자는 하나의 Java 파일만 만들면 운영체제와 상관없이 원하는 결과물을 쉽게 얻을 수 있다.  

      하지만 단점도 있다. 그 중 하나의 단점은 운영체제의 모든 기능을 JVM이 담지 못한다는 것이다. 따라서 구현하고 싶은 몇몇 기능들은 Java 언어 자체로도 해결 안되는 경우가 존재한다.  
      JNI는 **Java 언어 자체로 해결이 안되는 경우 대처할 수 있는 방법 중 하나**이다. 운영체제의 고유기능(Native)을 Java로 해결하는 것이 아닌 운영체제가 구현된 언어(보통 C, C++)로 운영체제의 고유 기능을 만든다.
2. **힙**: 프로그램을 실행하면서 생성한 모든 객체의 인스턴스 저장
3. 클래스 정보가 차지하는 영역인 **메서드 영역**
 - 클래스 로더가 클래스 파일을 읽어오면, 클래스 정보를 파싱해서 메서드 영역에 저장.
 - ex) 변수, 메서드, 정적 변수, 바이트 코드는 어떤게 있는가 등을 저장  

### 힙에 있는 객체들에 대한 참조
1. 힙 내의 다른 객체에 의한 참조
2. Java 스택, 즉 Java 메서드 실행 시에 사용하는 지역 변수와 파라미터들에 의한 참조
3. 네이티브 스택, 즉 JNI(Java Native Interface)에 의해 생성된 객체에 대한 참조
4. 메서드 영역의 정적 변수에 의한 참조

이들 중 힙 내의 다른 객체에 의한 참조(1)를 제외한 나머지 3개는 유효한 최초의 참조(root set)이다. 그리고 이들은 객체에 유효한 참조가 있는지(reachability)를 판가름하는 기준이 된다.
reachability를 더 자세히 보기위해 root set과 힙 내의 객체를 중심으로 다시보면 다음과 같다.

![rootSet-Objects-and-unreachable-objects](/assets\img/rootSet-Objects-and-unreachable-objects.PNG)
<center>그림 2 Reachable 객체와 Unreachable 객체</center><br>

위 그림에서 보듯, root set으로 부터 시작한 참조 사슬에 속한 객체들은 **reachable 객체**이고, 이 참조 사슬과 무관한 객체들이 **unreachable 객체**로 GC 대상이다. 오른쪽 아래처럼 reachable 객체를 참조하더라고, 다른 reachable 객체가 이 객체를 참조하지 않는다면 이 객체는 unreachable 객체이다.  

이 그림에서 참조는 모두 **java.lang.ref 패키지를 사용하지 않은 일반적인 참조**이며, 이를 흔히 strong reference라 부른다.

## Soft, Weak, Phantom Reference
![java-lang-references](/assets\img/java-lang-references.PNG)

java.lang.ref는 soft reference, weak reference, phantom reference를 클래스 형태로 제공한다. 예를 들면, java.lang.WeakReference 클래스는 참조 대상인 객체를 캡슐화한 WeakReference객체를 생성한다. 이렇게 생성된 WeakReference객체는 다른 객체와 달리 Java GV가 특별하게 취급한다. 캡슐화된 내부 객체는 weak reference에 의해 참조된다.

### 참고
- [Java Reference와 GC](https://d2.naver.com/helloworld/329631)
- [코딩 테스트, 중급, LRU cache](https://www.youtube.com/watch?v=HpuIrGiHwTo)
- [[10분 테코톡] 🎅무민의 JVM Stack & Heap](https://www.youtube.com/watch?v=UzaGOXKVhwU)
- [JAVA - JNI 사용하기](https://mommoo.tistory.com/71)