---
title: '[Spring] AOP를 SpringConfig에 @Bean으로 등록 했을 때 순환 참조 발생'
layout: post
categories: spring
tags: spring
comments: true
---

AOP를 사용하는 방법은 크게 2가지가 있다.
 1. `@Component`로 선언하는 방법
 2. `SpringConfig.java` 파일에 `@Bean`으로 등록하는 방법

김영한님의 Spring 강의 로드맵대로 강의를 들으며 2번 방법대로 AOP를 구현하던 중 아래와 같이 빈 순환 참조 에러가 발생했다.
```
The dependencies of some of the beans in the application context form a cycle:

   meberController defined in file [C:\Ara\공부\김영한님 스프링 완전 정복 로드맵\hello-spring\out\production\classes\hello\hellospring\controller\MeberController.class]
      ↓
   memberService defined in class path resource [hello/hellospring/SpringConfig.class]
┌─────┐
|  timeTraceAop defined in class path resource [hello/hellospring/SpringConfig.class]
└─────┘
```
```java
package hello.hellospring;

@Configuration
public class SpringConfig {
    @Bean
    public TimeTraceAop timeTraceAop() {
        return new TimeTraceAop();
    }
}
```
```java
@Aspect //AOP 쓸려면 무조건 이 Annotation을 달아야함
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        //...
    }
}
```

TimeTraceAop의 AOP 대상을 지정하는 @Around 코드를 보면 그 범위를 hello.hellospring 패키지와 동일하거나, 그 하위에 있는 소스들로 지정하고 있다.  
SpringConfig는 hello.hellospring에 위치하고있으므로 timeTraceAop() 메서드도 AOP로 처리하게 된다. 그런데 이건 자기 자신인 TimeTraceAop를 생성하는 코드가 되면서 순환참조 문제가 발생한다.
반면에 SpringConfig 파일이 아닌 @Component 방식을 사용하면 AOP의 대상이 되는 코드 자체가 없기 때문에 문제가 발생하지 않고 정상적으로 작동한다.

```java
@Aspect //AOP 쓸려면 무조건 이 Annotation을 달아야함
@Component
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        //...
    }
}
```

그렇다면 Bean 방식으로는 해결할 수 없는걸까? 아니다. 아래와 같이 @Around를 수정하여 대상에서 SpringConfig 파일을 제외해주면 된다.
```java
@Aspect //AOP 쓸려면 무조건 이 Annotation을 달아야함
//@Component
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..)) && !target(hello.hellospring.SpringConfig)")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        //...
    }
}
```
위처럼 수정한 다음 소스를 돌려보면 정상적으로 수행되는 것을 볼 수 있다.