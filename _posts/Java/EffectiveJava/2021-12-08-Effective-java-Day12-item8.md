---
title: '[Effective Java] Day 12 - Item 8 :: finalizer와 cleaner 사용을 피하라'
layout: post
categories: java
tags: java
comments: true
---

Day 12 기록 시작!
* * *
**finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.** 오동작, 낮은 성능, 이식성 문제의 원인이 되기도 한다. Finalizer를 유용하게 쓸 수 있는 경우는 극히 드물다.  

딱 두가지의 경우
* 자원의 소유자가 Close 메서드를 호출하지 않는 것에 대한 안전망 역할로 자원을 반납하고자 하는 경우
* 네이티브 리소스를 정리해야 하는 경우

자바 9에서는 Finalizer를 사용자제(deprecated) API로 정하고 `Cleaner`를 그 대안으로 소개했다. 별도의 스레드를 사용하므로 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.  

```java
package item08;

import java.lang.ref.Cleaner;

public class Room implements AutoCloseable { //Room 클래스는 AutoCloseable을 구현한다.
    // finalizer와 달리 cleaner는 클래스에 public api로 노출되지 않는다.
    private static final Cleaner cleaner = Cleaner.create(); // parameter로 threadfactory 선택 가능.

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안된다!
    private static class State implements Runnable {
        int numJunkPiles; // 방(Room) 안의 쓰레기 수

        public State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // room의 close 메서드 호출 시, 또는 gc가 room을 회수할 때. 호출 된다.
        @Override
        public void run() {
            System.out.println("방 청소(junk) : " + numJunkPiles);
            numJunkPiles = 0;
        }
    }

    //방의 상태. Cleanable과 공유한다.
    private final State state;

    //Cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);//Room → State 호출
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() {
        cleanable.clean();
    }
}
```

C++에서의 파괴자(destructor)와는 다른 개념이다. C++의 destructor은 특정 객체와 관련된 자원을 회수할 때도 사용하지만, 자바에서는 `try-with-resources`와 `try-finally`가 그 역할을 한다.

## 단점 1. 언제 실행될지 알 수 없다.
finalizer와 cleaner는 즉시 수행된다는 보장이 없고 언제 실행될지 알 수 없다. 어떤 객체가 더이상 필요 없어진 시점에 그 즉시 finalizer 또는 cleaner가 제때 실행되지 않을 수 있다. 실행되기까지 시간이 얼마나 걸릴지는 아무도 모른다. 따라서 **제때 실행되어야 하는 중요한 작업을 절대로 finalizer와 cleaner에서 하면 안된다.** 예를 들어, 파일 리소스를 반납하는 작업을 finalizer 안에서 처리한다면, 실제로 그 파일 리소스 반납이 언제 될지 알 수 없고, 자원 반납이 되지 않아 더이상 새로운 파일을 열지 못하는 상황이 발생할 수도 있다.  

아래 소스를 보자.  
```java
public class FinalizerExample {

    @Override
    protected void finalize() throws Throwable {//가비지 컬렉션이 될때 호출됨
        System.out.println("Clean up");
    }

    public void hello(){
        System.out.println("hi");
    }
}
```
```java
public class SampleRunner {

    public static void main(String[] args) throws InterruptedException {
        SampleRunner runner = new SampleRunner();
        runner.run();
        //run 메서드가 끝남과 동시에 finalizerExample의 유효성이 끝나면서 GC의 대상이 됨
        Thread.sleep(1000l);//1초
        //종료
    }

    private void run(){
        FinalizerExample finalizerExample = new FinalizerExample();
        finalizerExample.hello();
    }
}
```

SampleRunner.run이라는 메서드에서 FinalizerExample이라는 객체의 인스턴스를 생성하고, hello라는 메서드를 호출했다. 이 finalizerExample는 `finalize`가 선언되어있다. run 메서드가 끝남과 동시에 finalizerExample의 유효성이 끝나면서 GC의 대상이 된다.  
- 출력
```java
hi
```
run 메서드 호출 완료 후 1초가 지난 후에도 finalizerExample가 GC의 대상이 됐다고 해서, 바로 GC가 되지 않았다. 이렇게 finalize라는 메서드가 호출이 안될 수도 있고, 언제 호출이 될지 아무도 모른다.

## 단점 2. finalizer는 인스턴스의 자원 회수를 지연시킬 수 있다.
Finalizer 쓰레드는 우선순위가 낮아서 언제 실행될지 모른다. 따라서, Finalizer 안에 어떤 작업이 있고, 그 작업을 쓰레드가 처리하지 못해서 대기하고 있는 경우, 해당 인스턴스는 GC가 되지 않고 계속 쌓이다가 OutOfMemory가 발생할 수도 있다.  

Cleaner는 별도의 스레드로 동작하며 자신을 수행할 스레드를 제어할 수 있다. 그래서 자원 회수 지연이라는 부분에 있어 해당 스레드의 우선순위를 높게 줌으로써 조금 나을 수도 있다. 하지만 여전히 쓰레드는 백그라운드에서 수행되며 언제 처리될 지 알 수 없다.

## 단점 3. finalizer나 Cleaner를 아예 실행하지 않을 수도 있다.
자바 언어 명세는 finalizer나 cleaner의 수행 시점 뿐만 아니라 수행여부조차 보장하지 않는다. 따라서 **Finalizer나 Cleaner로 저장소 상태를 변경하는 일을 하지 말라.** DB같은 공유 자원의 영구 락 해제를 finalizer나 cleaner로 반환하는 작업을 한다면 분산 시스템 전체가 멈춰 버릴 수 있다.

`System.gc`나 `System.runFinalization`에 속지말라. 그걸 실행해도 finalizer나 cleaner를 실행한다고 보장할 수 없다. 그걸 보장해주겠다고 만든 
- `System.runFinalizersOnExit`와 
- 그 쌍둥이인 `Runtime.runFinalizersOnExit`  

는 심각한 결함때문에 망했고 수십년간 사용자제(deprecated) 상태다. 자바 10까지만 있고 11부터는 사라졌다.

## 단점 4. 심각한 성능문제를 동반한다.
`AutoCloseable` 객체를 만들고 `try-with-resourecs`로 자원 반납을 하는데 걸리는 시간은 12ns 인데 반해, Finalizer를 사용한 경우에 550ns로 약 50배가 걸렸다. Cleaner를 사용한 경우에는 66ns로 약 5배가 걸렸다.

## 단점 5. finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.
Finalizers는 객체를 생성할 때 취약점이 존재한다. finalizer의 개념은 java 메서드가 os로 리턴해야하는 자원을 해제 할 수 있게 하는 것인데 finalizer에서 자바 코드가 실행될 수 있다. finalizer 공격 원리를 알아보자.
```java
package item08.FinalizerAttack;

public class Vulnerable {
    Integer value = 0;

    Vulnerable(int value){
        if(value <= 0){
            throw new IllegalArgumentException("FinalizerExample value must be positive");
        }
        this.value = value;
    }

    @Override
    public String toString(){
        return (value.toString());
    }
}
```
```java
public class AttackVulnerable extends Vulnerable{
    static Vulnerable vulnerable;

    public AttackVulnerable(int value){
        super(value);
    }

    public void finalize(){//실행이 될지 안될지 알 수 없음
        vulnerable = this;
    }

    public static void main(String[] args) {
        try{
            new AttackVulnerable(-1);
        }catch (Exception e){
            System.out.println(e);
        }
        System.gc();             //테스트를 위한 gc 실행 및
        System.runFinalization();//finalizer 실행 권장
        if(vulnerable != null){
            System.out.println("Vulnerable object " + vulnerable + " created!");
        }
    }
}
```

어떤 **Vulnerable**라는 클래스와, 그 Vulnerable라는 클래스를 상속받은 **AttackVulnerable**라는 클래스가 있다.  
이 AttackVulnerable 클래스는 finalize가 호출됐을 때 vulnerable static 변수에 this가 저장된다. 객체는 다시 접근할 수 있게 됐고 gc되지 않는다.  

AttackVulnerable 클래스의 메인 메서드에서 새로운 AttackVulnerable 객체 생성을 시도한다. vlaue가 -1이기 때문에 Exception이 발생하고 catch 블록으로 온다. 그럼 이 객체가 아예 만들어지지 않고 죽어야지 정상인데 **죽으면서 finalize가 실행**이 된다.  

`System.gc()`와 `System.runFinalization()`는 gc를 실행하고 모든 finalizer를 실행하도록 권장한다. 테스트를 위해 강제로 실행될 수 있도록 추가해주었다(앞에서 본 것 처럼 이 2개가 finalizer가 실행되는 것을 보장해주진 않는다)  

이 finalize라는 메서드 안에서 이 **인스턴스는 static 필드에 접근**할 수 있다. 그래서 vulnerable가 GC가 되지 못하게 할 수 있다. 결국에는 원래 vulnerable는 죽어야 하는 인스턴스인데 finalize 때문에 좀비처럼 살아있게 되는 것이다. 그리고 노출이 되지 않아야 하고 사용을 못해야 하는 인스턴스의 메서드도 접근이 가능하게 된다.  

그 다음 출력을 통해 값이 잘못된 vulnerable 객체가 만들어졌음을 알 수 있다.
```java
java.lang.IllegalArgumentException: FinalizerExample value must be positive
Vulnerable object 0 created!
```

실행하면 위와 같은 결과가 나온다. 왜 Vulnerable value가 -1이 아니라 0일까? Vulnerable 생성자에서 인자 검사 전까지는 value 할당(`this.value = value;`)을 하지 않았기 때문이다. 그래서 value는 초기값 0이다.  

이런 경우를 막기위해서는 A라는 클래스를 상속 자체를 막을 수 있는 **final 클래스**로 만들거나,
```java
public final class Vulnerable {
    //...
}
```
**finalize라는 메서드 자체를 final로** 만들어주면 finalize를 더이상 상속하지 못하게 된다. 그러니 이 클래스 자체가 조금 더 보안에 안전하게 된다.
```java
public class Vulnerable {
    //...
    public final void finalize(){//finalier attack 공격 방지
    }
    //...
}
```

## 자원 반납 하는 방법
자원반납이 필요한 클래스가 `AutoCloseable` 인터페이스를 구현하고, 그 클래스의 객체를 생성하는 쪽은 `try-with-resource`를 사용하거나 `close()` 메소드를 명시적으로 호출하면 된다. `try-with-resource`를 사용하면 명시적으로 close()를 호출하지 않아도 try 블럭이 끝날 때 AutoCloseable 인터페이스에 있는 close를 호출하여 closing 해준다.  

호출된 `close`메서드는 이 객체가 더이상 유효하지 않음을 필드에 기록하고, 다른 메서드는 이 필드를 검사하여 객체가 닫힌 후에 불렀다면 `IllgegalStateException`을 던져야 한다.
```java
public class SampleResource implements AutoCloseable {

    @Override
    public void close() throws RuntimeException{
        System.out.println("close");
    }

    public void hello(){
        System.out.println("hello");
    }
}
```
```java
public class SampleRunner {

    public static void main(String[] args) throws InterruptedException {
        try(SampleResource sampleResource = new SampleResource()){
            sampleResource.hello();
        }
    }
}
```

## 왜 finalizer와 cleaner가 필요할까?
### 1.자원의 소유자가 Close 메서드를 호출하지 않는 것에 대한 안전망(Safety-net) 역할
cleaner나 finalizer가 즉시 또는 끝나기 전까지 호출되리라는 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것이 아예 안하는 것보다는 나으니 말이다.  
실제로 자바에서 제공하는 `FileInputStream`,`FileOutputStream`,`ThreadPoolExecutor`가 대표적이다.

```java
public class SampleResource implements AutoCloseable {
    boolean close;

    @Override
    public void close() throws RuntimeException{
        if(this.close){
            throw new IllegalStateException();
        }

        close = true;
        System.out.println("close");
    }

    public void hello(){
        System.out.println("hello");
    }

    protected void finalize() throws Throwable{
        if(!this.close) close(); //finalizer 안에서 자기 자신의 자원을 반납하도록!
    }
}
```

### 2. 네이티브 피어와 연결된 객체 회수 역할
- **네이티브 피어**: 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체
- gc가 회수할 수 있는 대상이 아닌 네이티브 객체를 회수하기 위해서는, cleaner나 finalizer에서 회수할 수 있도록 처리
- 단, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때만 해당

### 참고
- [[이팩티브 자바] #6 불필요한 객체를 만들지 말자](https://www.youtube.com/watch?v=0yUxPUXS1pM&t=115s)
- [Finalizer attack](https://yangbongsoo.tistory.com/8?category=919799)