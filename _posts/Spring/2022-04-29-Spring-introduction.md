---
title: '[Spring] Spring이란?'
layout: post
categories: spring
tags: spring
comments: true
---

오늘의 목표: 김영한님의 스프링 완전 정복 로드맵을 처음부터 끝까지 보고 실무 개발에 꼭 필요한 스프링 학습하기. 코드로 스프링 전반에 대한 이해도를 높이는 것을 목표로 한다.

아래와 같이 Spring 프로젝트로 전반적인 흐름을 흘려보며 진행하려 한다.
- 스프링 프로젝트 생성
- 스프링 부트로 웹 서버 실행
- 회원 도메인 개발 등...

**프로젝트 사용 기술**
- Spring Boot: 스프링을 쉽게 쓰기 위해 제공해주는 툴
  - 스프링: 자바 엔터프라이즈 어플리케이션 개발을 편리하게 해주기 위한 프레임 웤
  - JEE: Java Enterprise Application
  - 스프링 부트가 다루는 영역: 대표적으로 빌드, 빌딩, 배포 및 관리가 있음. 어플리케이션 개발 관련 거의 모든 분야에서 도움을 줌.
- Gradle
- Thymeleaf
- JPA
- Tomcat
- HIBERNATE

학습 방법: 처음부터 끝까지 직접 코딩하기

**스프링** 완전 정복 로드맵
- 스프링 입문
- 스프링 핵심 원리
- 스프링 웹 MVC
- 스프링 DB 데이터 접근기술
- 실전! 스프링 부트

## 프로젝트 환경 설정
- 프로젝트 생성
- 라이브러리 살펴보기
- View 환경설정
- 빌드하고 실행하기

### 프로젝트 생성
**사전 준비물**
- Java 11 설치
- IDE: IntelliJ 또는 Eclipse 설치

**스프링 부트 스타터 사이트로 이동해서 스프링 프로젝트 생성**
- https://start.spring.io
  - Spring Boot 기반으로 스프링 관련 프로젝트를 만들어주는 사이트
  - Project: Maven Project, Gradle Project
    - 필요한 라이브러리를 불러오고 빌드해주는 대표적인 두가지 툴
    - 과거에는 Maven을 많이 썼지만 요즘에는 Gradle을 많이 사용 하는 추세
    - 스프링 라이브러리 자체도 예전에는 Maven을 사용하다가 요즘에는 Gradle로 넘어옴
  - Spring Boot
    - 스프링 부트 버전
      * 스냅삿: 개발 중인 버전. 가급적 사용 지양 권장
      * M버전: 배포는 했지만 바뀔 수 있는 버전
      * RC버전(Release Condidate)
      * GA버전(General. AVailable):
        아무것도 뒤에 안달려있는 버전. 최종적인 정식
      
- 프로젝트 메타데이터
  - 내가 만드는 프로젝트의 중요한 정보들
  - 여기서 중요한건 Group, Artifact
  - 종류
    - Artifact
      - 프로젝트명과 동일하게 설정한다.
      - 빌드되어 나올 때의 결과물
      - 제품의 이름으로, 버전 정보를 생략한 jar 파일의 이름이다.
      - 소문자로만 작성하며 특수문자는 사용하지 않는다.
      
- Dependencies : 아래 2개 선택
  - spring-boot-start-web
    - spring-boot-starter-tomcat: 톰캣(웹서버)
    - spring-webmvc: 스프링 웹 MVC
  - spring-boot-starter-thymeleaf: 타임리프 템플릿 엔진(View)

위와 같이 선택이 완료되었으면 `Generate` 버튼을 클릭한다. 해당 버튼을 클릭하면 압축 형태의 스프링 부트 프로젝트 파일이 생성된다.  
이 파일을 원하는 경로에 이동시킨 후 압축을 해제하고, 자신이 사용하는 IDEA를 통해 열어보자. 나의 경우는 IntelliJ를 사용하고 있다.

#### 스프링 부트 프로젝트 살펴보기 - 디렉토리 구조
```
hello-spring  
├─ gradle
│ ├── wrapper // Gralde을 쓰는 폴더
├─── src  
│ ├── main
│ │ ├─── java      // 실제 패키지와 소스 파일들이 위치한 곳
│ │ ├─── resources // 실제 자바코드 파일을 제외한 어떤 xml이나 properties, HTML 등 설정파일    
│ ├── test         // TestCode와 관련된 소스들이 위치한 곳
│ │ ├─── java      
│ │   ├─── hello.hellospring
│ ├── build.gradle // 버전 설정 및 라이브러리 가져오는 역할
```

- build.gradle

```java
plugins {
	id 'org.springframework.boot' version '2.5.13'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'hello'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11' //자바 11버전

repositories {
	mavenCentral() // 메이븐 센트럴에서 라이브러리들 다운로드 받게 간단한 설정해주기.
	               // 필요하면 특정사이트 URL을 넣어줄 수 있음.
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
	useJUnitPlatform()
}
```

- plugins
  - `io.spring.dependency-management` 플러그인은 스프링 부트의 의존성들을 관리해주는 플러그인이기 때문에 꼭 추가해주어야 한다.
- repositories
  - 각종 의존성(라이브버리)들을 어떤 원격 저장소에서 받을지 결정
  - MavenCentral
    - 기본적으로 MavenCentral을 많이 사용하지만, 최근에는 **라이브러리 업로드 난이도**때문에 jcenter도 많이 사용함
    - 본인이 만든 라이브러리를 업로드하기 위해서는 **정말 많은 과정과 설정**이 필요
  - jcenter
    - 위 문제점이 개선되어 **라이브러리 업로드가 간단**
    - jcenter에 라이브러리를 올리면 mavenCentral에도 업로드 될 수 있도록 자동화 가능
- dependencies
  - 프로젝트 개발에 필요한 의존성들을 선언하는 곳

스프링 부트는 tomcat과 웹서버를 내장하고 있다. 그래서 WAS를 자체적으로 띄우면서 Spring Boot가 같이 올라오는 구조이다.  
최초 프로젝트를 생성하게 되면 `HelloSpringApplication` 클래스에 main 메서드가 위치하게 된다. 실행시켰을 때 정상적으로 서버가 띄워진다면 정상적으로 설치가 잘 된 것이다.

### 라이브러리 살펴보기
Gradle과 Maven과 같은 빌드툴들은 의존 관계를 관리해준다. 예를들어 dependencies에 `org.springframework.boot:spring-boot-starter-web`를 떙겨오는 경우 이 라이브러리가 의존성을 갖고 있는 라이브러리들을 자동으로 다운로드 받아 준다.

**스프링 부트 라이브러리**
- spring-boot-starter(공통): 스프링 부트 + 스프링 코어 + 로깅
  - spring-boot
    - spring-core
  - spring-boot-starter-logging
    - logback, slf4j

**테스트 라이브러리**
  - junit: 테스트 프레임워크. 요즘은 junit5를 많이 사용함
  - mockito: 목 라이브러리
  - assertj: 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
  - spring-test: 스프링 통합 테스트 지원

### View 환경설정
#### Welcome Page 만들기
```html
<!DOCTYPE HTML>
<html>
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
Hello
<a href="/hello">hello</a>
</body>
</html>
```
> Spring Boot supports both static and templated welcome pages. It first looks for an index.html file in the configured static content locations. If one is not found, it then looks for an index template. If either is found, it is automatically used as the welcome page of the application.

- 스프링 부트가 제공하는 Welcome Page기능
  - 스프링부트는 `resources/static/index.html`에 파일을 만들면, 이 파일을 웰컴 페이지로 만들어준다.
  - 스프링부트는 static content가 위치한 곳에서 먼저 `index.html` 파일을 찾고, 만약에 못찾으면 `index` template을 찾는 식으로 동작한다.
  - [Spring boot docs - 2.1.4. Welcome Page](https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#web.reactive.webflux.static-content)

#### thymeleaf 템플릿 엔진
Spring Boot는 다음 4가지 템플릿 엔진에 대한 자동 구성을 해준다.
- FreeMarker
- Groovy
- Thymeleaf
- Mustache

템플릿 엔진을 쓰면 loop 등을 사용하여 내가 원하는 대로 모양을 바꿀 수 있다.
  - [Thymeleaf 공식 사이트](https://www.thymeleaf.org/)
  - [스프링 공식 튜토리얼](https://spring.io/guides/gs/serving-web-content/)
  - [스프링부트 메뉴얼](https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/
    html/spring-boot-features.html#boot-features-spring-mvc-template-engines)

```java
@Controller
public class HelloController {

    @GetMapping("hello")
    public String hello(Model model){

    }
}
```
웹 어플리케이션에서 `GetMapping` 어노테이션을 달면 `서버 URL/hello` 수행 시 이 메소드를 수행하겠다는 뜻이다.
![Spring-operating-environment](/assets\img/Spring-operating-environment.png)
- 컨트롤러에서 리컨 값으로 문자를 반환하면 뷰 리졸버(`viewResolver`)가 화면을 찾아서 처리한다.
  - 스프링 부트 템플릿엔진 기본 viewName 매핑
  - `resources:templates/`+{ViewName}+`.html`

> 참고로 `spring-boot-devtools` 라이브러리를 추가하면, `html` 파일을 컴파일만 해주면 서버 재시작 없이 View 파일 변경이 가능하다.  
> 인텔리J 컴파일 방법: 메뉴 build → Recompile
 
### 빌드하고 실행하기
1. 프로젝트가 있는 경로에서 cmd 창을 실행하여 `gradlew.bat build`를 수행한다.
2. `cd build/libs`
3. `java -jar hello-spring-0.0.1-SNAPSHOT.jar`를 수행하면 스프링이 실행된다.
  - 즉 서버를 배포할 떄는 위 jar 파일만 복사해서 서버에 넣은다음, `java -jar` 명령어를 통해 실행시키면 된다. 그렇게 하면 서버에서도 스프링이 동작하게 된다.
4. 실행확인

## spring 웹 개발 기초
- 정적 컨텐츠
  - Welcome 페이지 개발했던 것처럼 서버에서 하는 것 없이 파일을 그대로 Web 브라우저에 그대로 내려주는 것이다.
- MVC와 템플릿 엔진
  - 가장 많이 하는 방식이다. 과거에 JSP, PHP가 흔히 말하는 템플릿 엔진이다. HTML을 그대로 내려주는 것이 아니라 서버에서 동적으로 수행한 뒤에 내려준다.
  - 이것들을 하기 위해서 Controller, Model, View를 합해 MVC라고 하는데 요즘 이 패턴으로 개발을 많이 한다. (지금 회사에서도 MVC 패턴으로 개발 중이다.)
- API
  - 만약 안드로이드나 아이폰 Client와 통신을 해야하는 경우 Json 데이터 포맷으로 내려주는 것을 API 방식이라 한다.
  - 리액트로 프론트엔드가 개발된 경우에 많이 씀
  - 서버끼리 통신할 때

### 정적 컨텐츠
스프링 부트는 정적 컨텐츠 기능을 `/static` 폴더에서 찾아서 자동으로 제공한다.
- [Spring boot docs - 2.1.4. Welcome Page](https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#web.reactive.webflux.static-content)

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>static content</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
정적 컨텐츠 입니다.
</body>
</html>
```
![spring-static-content-feature](/assets\img/spring-static-content-feature.png)

### MVC와 템플릿 엔진
과거에는 Controller와 View가 따로 나뉘어 있지않고 View에 모든 것을 다 구현했었는데 이를 **Model 1 방식**
이라 한다. 하지만 요즘은 MVC 스타일로 많이 한다.
- MVC
  - Model
  - Controller: 비즈니스 로직과 관련이 있거나 내부적으로 처리해야 하는 것을 구현해야함
  - View: 화면과 관련된 것만 구현해야 함
  
![spring-MVC-template-engine](/assets\img/spring-MVC-template-engine.png)

### API
정적 컨텐츠를 제외하면 HTML로 내리느냐, 아니면 API 방식으로 데이터를 내리느냐를 결정하면 된다.
```java
@GetMapping("hello-string")
@ResponseBody
public String helloString(@RequestParam("name") String name){
    return "hello"+name;
}
```
`@ResponseBody`란 http에서 헤더부와 body부가 있는데, 그 body부에 이 데이터를 내가 직접 넣어주겠다라는 뜻이다.
템플릿 엔진과의 차이점은 View가 없이 이문자가 그대로 전달된다는 차이점이 있다.