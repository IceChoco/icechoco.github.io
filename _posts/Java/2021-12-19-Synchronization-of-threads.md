---
title: '[Java] 쓰레드의 동기화(Synchronization of threads)'
layout: post
categories: java
tags: java
comments: true
---

**이 글의 목적**
- 쓰레드 동기화(Synchronization of threads)에 대한 개념을 잡을 수 있다.

## 공유 객체를 사용할 때의 주의할 점
멀티 스레드 프로세스에서는 다른 스레드의 작업에 영향을 미칠 수 있다.
- 여러 쓰레드가 같은 자원을 공유하기 때문에 메모리도 공유한다. 그렇기 때문에 스레드 A가 작업하던 것을 마치지 못하고 스레드 B의 차례로 넘어갔을 때, 스레드 B가 스레드 A의 자원에 영향을 줄수가 있다.

### 예시
User1 스레드가 Calculator 객체의 memory 필드에 100을 먼저 저장하고 2초간 일시 정지 상태가 된다. 그 동안에 User2 스레드가 memory 필드 값을 50으로 변경한다. 2초가 지나 User1 스레드가 다시 실행상태가 되어 memory 필드의 값을 출력하면 User 2에서 저장한 50이 나온다.

- **MainThreadExample.java**: 메인 스레드가 실행하는 코드
```java
public class MainThreadExample {
    public static void main(String[] args) {
        Calculator calculator = new Calculator();

        User1 user1 = new User1();
        user1.setCalculator(calculator);
        user1.start();

        User2 user2 = new User2();
        user2.setCalculator(calculator);
        user2.start();
    }
}
```
- **Calculator.java**: 공유 객체
```java
public class Calculator {
    private int memory;

    public int getMemory() {
        return memory;
    }

    public void setMemory(int memory) { //계산기에 메모리 값을 저장하는 메소드
        this.memory = memory;           //매개값을 memory 필드에 저장
        try{
            Thread.sleep(2000);   //스레드를 2초간 일시정지시킴
        }catch(InterruptedException e){}
        System.out.println(Thread.currentThread().getName() + ":"+this.memory);//스레드의 이름: 메모리값
    }
}
```
- **User1.java**: User1 스레드
```java
public class User1 extends Thread {
    //private으로 해야 동기화가 의미가 있다. private이 아니게 되면 클래스 밖에서 값을 마음대로 바꿀 수 있기 때문이다.
    private Calculator calculator;

    public void setCalculator(Calculator calculator) {
        this.setName("CalculatorUser1"); //스레드 이름을 CalculatorUser1로 설정
        this.calculator = calculator;    //공유객체인 Calculator를 필드에 저장
    }
    
    public void run(){
        calculator.setMemory(100); //공유 객체인 Calculator의 메모리에 100을 저장
    }
}
```
- **User2.java**: User2 스레드
```java
public class User2 extends Thread {
    //private으로 해야 동기화가 의미가 있다. private이 아니게 되면 클래스 밖에서 값을 마음대로 바꿀 수 있기 때문이다.
    private Calculator calculator;

    public void setCalculator(Calculator calculator) {
        this.setName("CalculatorUser2"); //스레드 이름을 CalculatorUser2로 설정
        this.calculator = calculator;    //공유객체인 Calculator를 필드에 저장
    }

    public void run(){
        calculator.setMemory(50); //공유 객체인 Calculator의 메모리에 50을 저장
    }
}
```
- **쓰레드의 동기화**: 한 스레드가 진행중인 작업을 다른 쓰레드가 간섭하지 못하게 막는 것  
진행중인 작업이 다른 스레드에게 간섭받지 않게 하려면 `동기화`가 필요하다.

- **임계 영역(Critical section)**: 멀티 스레드 프로그램에서 단 하나의 스레드만 실행할 수 있는 코드 영역  
동기화를 하려면 간섭받지 않아야 하는 문장들을 `임계영역`으로 설정한다.  
임계영역은 락(lock, 자물쇠)을 얻은 단 하나의 쓰레드만 출입가능(객체 1개에 락 1개)

## Synchronized를 이용한 동기화
synchronized로 임계영역(lock이 걸리는 영역)을 설정하는 방법 2가지
```java
//1. 메서드 전체를 임계영역으로 지정
public synchronized void setMemory(int memory) { //계산기에 메모리 값을 저장하는 메소드
    this.memory = memory;           //매개값을 memory 필드에 저장
    try{
        Thread.sleep(2000);   //스레드를 2초간 일시정지시킴
    }catch(InterruptedException e){}
    System.out.println(Thread.currentThread().getName() + ":"+this.memory);//스레드의 이름: 메모리값
}

//2. 특정한 영역을 임계 영역으로 지정
public void setMemory(int memory) { //계산기에 메모리 값을 저장하는 메소드
    synchronized(this){//this: 공유 객체인 Calculator의 참조(잠금 대상)
        this.memory = memory;           //매개값을 memory 필드에 저장
        try{
            Thread.sleep(2000);   //스레드를 2초간 일시정지시킴
        }catch(InterruptedException e){}
        System.out.println(Thread.currentThread().getName() + ":"+this.memory);//스레드의 이름: 메모리값
    }
}
```

임계영역은 한 번에 한 쓰레드만 사용할 수 있기 때문에 **영역을 최소화**해야 한다. 임계영역이 많을 수록 성능이 떨어진다. 멀티 스레드의 장점이 동시에 여러개가 돌아가는 것이나, 임계영역에서는 1번에 1개의 스레드만 임계영역에 들어갈 수 있다. 그러므로 가능하면 임계영역의 갯수도 최소화하고, 영역도 좁아야한다.

### 참고
- [[자바의 정석 - 기초편] ch13-30~33 쓰레드의 동기화](https://www.youtube.com/watch?v=g4vP5wuAoPI&t=294s)