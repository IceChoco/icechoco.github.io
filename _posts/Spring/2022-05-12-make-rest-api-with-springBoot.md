---
title: '[Spring] SpringBoot로 Rest API 만들기 - Swagger API 문서 자동화'
layout: post
categories: java, spring
tags: java
comments: true
---

평소에 내가 속한 부서에서는 프론트와 백엔드 개발자가 나뉘어 있지 않고 한 사람이 다 하다보니 API 문서 자동화를 제공해주는 Swagger를 사용하고 있지 않았다. 
외부 시스템과 연계해야 할 때는 API 문서를 엑셀 또는 파워포인트로 정리하여 공유가 되었다. 그런데, 최근에 외부 시스템과 원활한 IF를 위해 Swagger를 적용하는 곳이 생기고 있다.    
그런데 이번에 YAPP 백엔드 팀원 분이 Swagger를 사용해봤더니 프론트와 백엔드 간의 의사소통이 원활했다라는 말을 듣고 이번 사이드 프로젝트에도 적용하기로 하였다.

Swagger는 문서 자동화 툴이며 간단한 설정으로 테스트가 가능한 Web UI를 지원한다. API를 테스트하기 위해 부가적으로 Postman과 같은 프로그램을 깔지 않아도 된다!  
또한 최소한의 작업을 통해 자동으로 API Document를 만들어주기 때문에 프론트 개발자에게 문서 내용을 전달하기위해 워드나 파워포인트를 만들지 않아도 된다는 장점이 있다.

## build.grald에 swagger 라이브러리 추가하기
`build.grale` 파일에 아래와 같이 라이브러리를 추가한다.
```java
	implementation group: 'io.springfox', name: 'springfox-swagger2', version: '2.9.2'
	implementation group: 'io.springfox', name: 'springfox-swagger-ui', version: '2.9.2'
```

## build.grald에 swagger 라이브러리 추가하기
yapp.bestFriend.config 패키지에 SwaggerConfig 파일을 작성한다.
```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket docket(){
        return new Docket(DocumentationType.SWAGGER_2).select()
                .apis(RequestHandlerSelectors.basePackage("yapp.bestFriend.controller"))
                .paths(PathSelectors.any())
                .build()
                .apiInfo(apiInfo());
    }

    private ApiInfo apiInfo(){
        return new ApiInfoBuilder()
                .title("절친 API 문서")
                .description("절친 API 서버 문서입니다.")
                .version("1.0")
                .build();
    }

}
```
- **basePackage("yapp.bestFriend.controller")).paths(PathSelectors.any())**
    - yapp.bestFriend.controller 하단의 Controller 내용을 읽어 mapping된 resource들을 문서화 시킨다.
    - `PathSelectors.ant("/v1/**"")`와 같이 설정하여 v1으로 시작하는 resource들만 문서화 시키는 것도 가능하다.
- ApiInfo를 작성하면 문서에 대한 설명과 작성자 정보를 표시할 수 있다.

## UseController 수정하기
```java
@Api(tags = {"회원 API"})
public class UserController {

}
```
- **@Api(tags = {"회원 API"})**
  - UserController를 대표하는 최상단 타이틀 영역에 표시되는 값 세팅

### 참고
- [SpringBoot2로 Rest api 만들기(4) - Swagger API 문서 자동화](https://daddyprogrammer.org/post/313/swagger-api-doc/)