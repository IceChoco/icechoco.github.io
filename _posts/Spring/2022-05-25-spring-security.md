---
title: '[Spring] Spring Security란?'
layout: post
categories: java, spring
tags: java
comments: true
---

**이 글을 쓰는 이유**  

>Spring Security는 복잡하고 방대한 내용을 알아야 한다. OAuth2를 활용한 로그인을 구현하다 보니 Spring Security에 대한 기본기를 다지고 가야겠다 생각되어 이 글을 쓰게 되었다.  
>
>참고로, 구글링 중 너무 글로 잘 정리해주신 [딩규님의 글](https://dingue.tistory.com/5)을 찾아서 이 글을 기반으로 추가로 공부한 내용을 더해 작성하였다. 좋은 글을 정리해주신 딩규님에게 감사 말씀을 드린다.

* * *  

우선 Spring Secuiry가 무엇인가?  
- **강력하면서 높은 수준의 커스터마이징 가능한 인증을 제공하고, 필터를 통한 접근 제어를 허용하는 프레임 워크**
- Spring 기반 애플리케이션의 보안에서는 표준

현재 가장 최신 버전은 5.7.1이나 이 글을 5.1.4 버전을 기반으로 작성해보았다.


## Filter
![](/assets\img/spring/spring_security_filter.png)
- spring security에서는 애플리케이션에 대한 모든 요청을 필터로 감싼뒤 처리
- Filter들은 chain 형식이며, 각각의 순서를 갖고 있음
- 필터 체인의 제일 마지막에 위치한 FilterSecurityInterceptor는 앞에 지나온 모든 필터들의 정보를 토대로 최종 결정
- 자동 설정 시 10개의 필터가 적용됨

## Authentication(인증), Authorization(권한)
- Authentication: 애플리케이션에서 사용자가 이 애플리케이션을 사용할 자격이 있는지 확인하는 과정
  - 인증을 성공적으로 마치면 사용자에게 설정된 권한을 할당
  - ex) 로그인
- Authorization: 접근하는 사용자가 해당 로직 및 url에 접근할 권한을 가지고 있는지 확인하는 과정(접근제어 및 권한 관리)
  - 웹 요청, url, 도메인 인스턴스에 대한 접근을 관리
  - 만약 접근한 사용자가 권한이 없거나 낮으면(ex 관리자 요청인데 일반 유저가 요청한 경우) 요청을 거부함

즉, Spring Security는 **Filter로 이루어진 시스템**이고, Filter를 이용한 Authentication, Authorization으로 유저 관리 및 접근 제어를 수행하는 보안 프레임워크이다.

## Authentication(인증)
### 1. 인증 관련 핵심 용어
- Principal: 서비스에 접근하는 유저를 가리킴(일반적으로 username, userId로 생각해도 무방함)
- Authentication 객체
  - Spring Security에서 한 유저의 인증 정보를 가지고 있는 객체
  - 사용자가 인증 과정을 성공적으로 마치면, Spring security는 사용자의 정보 및 인증 성공여부를 가지고 Authentication 객체를 생성한 후 보관함
- SecurityContextHolder: Authentication 객체를 보관하는 곳. 애플리케이션 어디에서든지 접근할 수 있음
  - ex) Obeject principal = SecurityContextHolder.getContext().getAuthentication().getPrinciPal();
- UserDetails: 일반 서비스의 사용자 객체를 Spring security에서 사용하는 사용자 객체와 호환해주는 어댑터
- UserDetailsService: spring security에서 로그인할 때 전달된 정보를 기반으로 DB에서 유저를 가져오는 책임을 가지는 인터페이스
- GrantedAuthority: 사용자에게 주어진 애플리케이션 사용 권한 객체
- PasswordEncoder
  - DB에 사용자의 정보 저장 시 비밀번호를 암호화
  - 인증 시 입력된 비밀번호와 저장된 비밀번호를 matching 해주는 객체

### 2. 인증 과정
아이디 비밀번호를 입력하는 폼 기반 로그인이라고 가정했을 떄 아래와 같은 과정을 거친다.
1) 유저가 입력한 아이디, 비밀번호를 Spring Security가 사용하는 유저의 객체로 변환시킴(UserDetails 이용)
2) 유저가 입력한 정보(예: id)를 기반으로 DB에서 사용자 정보를 가져옴(UserDtailsService 인터페이스가 하는 역할)
3) 유저가 입력한 정보와 DB에서 가져온 사용자 정보를 기반으로 인증 진행 (아래에서 설명할 AuthenticationManager 인터페이스의 authenticate() 메소드)
4) 만약에 인증에 성공했다면 유저에게 설정된 권한을 부여

### 3. 인증 관련 세부 내용
#### AuthenticationManager
```java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```
- `authenticate` 메소드를 하나 가지고 있는 인터페이스
- 이 메소드는 Spring security에서 인증을 담당

#### ProviderManager
```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {
    //....
}
```
- Spring Security에서 AuthenticationManager의 기본 구현체
- 이 클래스는 자신이 가지고 있는 여러 AuthenticationProvider 들에게 인증 처리를 위임함

#### AuthenticationProvider
```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {
  //...
    public ProviderManager(List<AuthenticationProvider> providers, AuthenticationManager parent) {
      this.eventPublisher = new ProviderManager.NullEventPublisher();
      this.providers = Collections.emptyList();
      this.messages = SpringSecurityMessageSource.getAccessor();
      this.eraseCredentialsAfterAuthentication = true;
      Assert.notNull(providers, "providers list cannot be null");
      this.providers = providers;
      this.parent = parent;
      this.checkState();
    }
//...
}
```
- 실질적으로 인증을 처리하는 클래스 (authentication 메소드를 구현한 클래스) 
- UserDetails와 UserDetailsService 그리고 PasswordEncoder를 이용해서 인증을 처리
- ProviderManager는 여러개의 AuthenticationProvider를 가질 수 있음
- 여러개의 AuthenticationProvider를 가지고 있는 경우 각 순서대로 인증을 진행하거나, skip함
- 만약 모든 AuthenticationProvider가 null을 반환(인증을 skip 하는 것)
- Spring Security에서 구현한 여러 AuthenticationProvider 구현체들이 있음
  - DaoAuthenticationProvider, LdapAuthenticationProvider: from 기반 로그인이나, HTTP Bagic authentication 인증을 구현한 구현체

#### DaoAuthenticationProvider
- 유저가 입력한 username, password를 DB에서 가져온 유저정보와 비교하여 인증을 처리하는 구현체
- password를 비교하기 위해서 PasswordEncoder를 설정해주어야 함
- DB에서 유저 정보를 가져오기 위해서 UserDetailsService를 설정해 주어야 함

#### UserDetailsService
```java
public interface UserDetailsService {
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```
- 사용자가 입력한 username을 기반으로 저장된 유저정보를 가져와서 UserDetails 객체로 변환하여 돌려주는 메소드
- `loadUserByUsername` 메소드를 하나 가지고 있는 인터페이스
- Spring Security에서 관련 구현체로 In-Memory Authentication, JdbcDaoImpl이 있음

##### In-Memory Authentication
Example. <user-service> {noop} XML Configuration
```xml
<user-service>
	<user name="user"
		password="{noop}password"
		authorities="ROLE_USER" />
	<user name="admin"
		password="{noop}password"
		authorities="ROLE_USER,ROLE_ADMIN" />
</user-service>
```
- 애플리케이션 메모리 안에 유저 정보를 저장해두고, 인증 요청이 왔을 때 이 정보들 중에서 입력된 정보와 일치하는 유저를 반환해주는 구현체
- 프로토타입용으로만 사용하는 것이 좋음(절대 실무에서는 사용하면 안됨)

##### JdbcDaoImpl
```java
public class JdbcDaoImpl extends JdbcDaoSupport implements UserDetailsService, MessageSourceAware {
    //...
}
```
- ORM을 사용하고 있지 않다면, 이 구현체를 사용해서 DB에서 쉽게 입력된 사용자 정보를 가져올 수 있음
- 하지만 ORM을 사용하고 있다면 굳이 이런 구현체를 사용하는 것보다, 직접 UserDetailsService 인터페이스를 구현하여 사용하느 ㄴ것이 좋음

#### UserDetails
- Spring Security에서는 사용자 객체가 필요하지만, 모든 서비스마다 사용자 정의가 다르기 때문에 서비스와 Spring security의 사용자 객체를 호환해주는 어댑터가 필요
- 그 어댑터의 역할을 UserDetails가 수행
- Spring security를 사용하는 애플리케이션에서는 이 인터페이스를 구현해야함

#### PasswordEncoder
패스워드 암호화의 발전 과정
1. 초기에 password는 암호화 되지 않음
  - DB 인증을 통해서 데이터를 접근하는데 다시 password를 암호화 할 필요성을 느끼지 못함
2. SQL Injection 같은 공격을 통해 password가 털림. password 암호화의 필요성을 느낌
3. 개발자들은 SHA-256과 같은 한방향 hash 암호화를 이용하여 password 암호화를 시작
4. 공격자들은 rainbowTable이라는 것을 생성하여 한방향 hash 암호화를 무력화시킴
5. 이에 개발자들은 랜덤 바이트를 비밀번호에 섞어서 ranbowTable을 무력화 시킴
6. 현대 HW가 좋아지고, 해쉬함수가 빠르다는 것을 공격자들이 이용(1초에 10억건씩 비밀번호를 체크)
7. 이에 개발자들은 해시 암호화 알고리즘이 내부 리소스들을 엄청나게 사용하도록 변경(암호화가 1초 이상 걸리도록 설정)
  - 이게 바로 로그인이 오래걸리는 이유

Spring Security에서는 work factor 조정을 통해 암호화 알고리즘의 속도를 설정할 수 있다.  
이런 work factor를 사용하는 암호화 알고리즘은 bcrypt, PBKDF2, scrypt, 그리고 Argon2가 있다.

#### DelegatingPasswordEncoder
```java
 String idForEncode = "bcrypt";
 Map<String,PasswordEncoder> encoders = new HashMap<>();
 encoders.put(idForEncode, new BCryptPasswordEncoder());
 encoders.put("noop", NoOpPasswordEncoder.getInstance());
 encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
 encoders.put("scrypt", new SCryptPasswordEncoder());
 encoders.put("sha256", new StandardPasswordEncoder());

 PasswordEncoder passwordEncoder = new DelegatingPasswordEncoder(idForEncode, encoders);
```
Spring Security 5.0 이전에는 암호화를 하지 않고 default PasswordEncoder가 평문 패스워드였다.  
Spring Security에서는 하위 호환성을 위하여 그동안 이를 업그레이드하지 않다가 `DelegatingPasswordEncoder`라는 구현체를 개발하였다.
- 이전의 포맷이나, 현재 포맷을 같이 사용할 수 있다.
- 미래에 쉽게 passwordEncoding 방식을 업그레이드 할 수 있다.

#### Spring security 저장 포맷
일반적으로 저장 포맷은 아래와 같다
```
{id}encodePassword
```
여기서 id는 어떤 passwordEncoder가 사용되었는지 볼 수 있는 식별자이고, encodedPassword는 선택된 PasswordEncoder가 원래 passwrod를 인코딩한 것이다.  
id는 password 시작에 있어야 하며 {로 시작해서 }로 끝난다. id를 찾을 수 없으면 id는 null이 된다.  
아래는 `password`라는 String을 passwordEncoder로 인코딩한 예이다.
```
 {bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG
 {noop}password
 {pbkdf2}5d923b44a6d129f3ddf3e3c8d29412723dcbde72445e8ef6bf3b508fbf17fa4ed4d6b99ca763d8dc
 {scrypt}$e0801$8bWJaSu2IKSn9Z9kM+TPXfOc/9bdYSrN1oD9qfVThWEwdRTnO7re7Ei+fUZRJ68k9lTyuTeUp4of4g24hHnazw==$OAOec05+bXxvuu/1qZ6NUR+xQYvYv7BeL1QxwRpY5Pc=
 {sha256}97cde38028ad898ebc02e690819fa220e88c62e0699403e94fff291cfffaf8410849f27605abcbc0
```

이런 식으로 변했기 때문에 기존에 password에 id가 달려있지 않았다면 아래와 같이 업그레이드 시켜주어야 한다.
아래는 bycrypt 암호화 알고리즘을 사용한 예시이다
- 기존
```
$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG
```
- 변경
```
{bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG
```

### 4. 인증 아키텍처
![](/assets\img/spring/spring_security_autehntication.png)

## Spring Security 자바 설정
### 1. SpringSecurityFilterChain 생성
```java
@Configuration
@EnableWebSecurity //Spring Security 설정 활성화
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {

  @Autowired
  private UserDetailsService jwtUserDetailsService;

  @Autowired
  private PasswordEncoder passwordEncoder;

  @Autowired
  private JwtRequestFilter jwtRequestFilter;

  //...
}    
```

위 소스를 보면 클래스 위에 `@EnableWebSecurity`라는 어노테이션이 달려있는 것을 볼 수 있다. 이 어노테이션을 사용하면 SpringSecurityFilterChain이 자동으로 생성 된다.  

#### 1-1. SpringSecurityFilterChain의 효과
- 애플리케이션의 모든 URL에 인증을 적용할 수 있다.
- 기본 로그인 폼을 제공한다.
- 폼 기반 인증을 통해 username과 password 인증을 할 수 있게 해준다.
- 유저에게 로그아웃 기능을 제공한다.
- CSRF 공격을 방어해준다.
- 세션 고정 공격을 방어해준다.
- 통합 Security Header를 제공한다.
  - 보안 요청을 위한 HTTP Strict Transport Security 제공
  - 통합 X-Context-Type-Options 제공
  - 통합 X-XSS-Protection 제공
  - Clickjacking 방어를 도와주는 통합 X-Frame-Options 제공

### 2. Spring MVC에서 AbstractSecurityWebApplicationInitializer를 통한 Spring Security 등록
> jar의 경우 스프링부트에서 자동 구성을 해주기 때문에 위 설정이 따로 필요 없다. 그 이유가 궁금하다면 아래 2-4를 참조하길 바란다.  
> war로 생성하려는 경우에는 아래와 같이 직접 구현을 해주어야 한다.

#### 2-1. AbstractSecurityWebApplicationInitializer without Spring
스프링이나 스프링 MVC를 사용하고 있지 않다면, `WebSecurityConfig`를 사용할 수 있도록 이 클래스를 상위로 넘겨야 한다. 예시는 아래에 있다:
```java
import org.springframework.security.web.context.*;

public class SecurityWebApplicationInitializer
    extends AbstractSecurityWebApplicationInitializer {

    public SecurityWebApplicationInitializer() {
        super(WebSecurityConfig.class);
    }
}
```

`AbstractSecurityWebApplicationInitializer`는 다음과 같은 일을 한다.
- 어플리케이션의 모든 URL에 자동으로 springSecurityFilterChain 필터를 등록한다.
- `WebSecurityConfig`를 로드하는 ContexLoaderListner를 추가한다.

#### 2-2. AbstractSecurityWebApplicationInitializer with Spring MVC
```java
import org.springframework.security.web.context.*;

public class SecurityWebApplicationInitializer
        extends AbstractSecurityWebApplicationInitializer {

}
```

이 이후에 WebSecurityConfig가 ApplicationInitializer로 로드되도록 설정해야 한다.

```java
public class MvcWebApplicationInitializer extends
        AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[] { WebSecurityConfig.class };
    }

    // ... other overrides ...
}
```
이렇게 하면 기존 ApplicationInitializer에 `WebSecurityConfig`를 로드한다.
예를 들어 스프링 MVC를 사용하고 있다면 `getRootConfigClasses()`안에 추가된다.

* * * 

그런데 내가 개발중인 프로젝트는 AbstractSecurityWebApplicationInitializer를 직접 구현한 적이 없는데 스프링 세큐리티를 적용하여 사용중이다.  
엥 그럼 필수가 아니란 말인가? AbstractSecurityWebApplicationInitializer를 구현하지 않았을 때는 어떻게 동작할까?  
그 궁금증을 해결하기 위해 아래 2가지의 스택오버플로우 글을 읽어 보았다.

#### 2-3 AbstractSecurityWebApplicationInitializer vs. AbstractAnnotationConfigDispatcherServletInitializer
<details>
<summary>[질문 상세 내용]</summary>
<div markdown="1">

3.2.8 버전의 순수 자바 어플리케이션에 세큐리티를 적용하려한다. 나는 [스프링 세큐리티 docs](http://docs.spring.io/spring-security/site/docs/3.2.2.RELEASE/reference/htmlsingle/#jc)를 참조하고 있다.  

3.1 섹션을 완료했다. 그리고 문서에서는 모든 url에 authentication(인증)이 필요하지만, 세큐리티 적용안하면 모든 URL을 로드할 수 있다.  
문서에서는 서블릿 필터 등을 생성한다고 한다.

WebSecurityConfigurerAdapter 하위 클래스만으로는 충분하지 않다. 그래서 **WAR 사용 시** springSecurityFilterChain을 등록에 관한 3.1.1 섹션을 보았다.  
이 섹션에서는 Servlet 3+ 환경에서 어떻게 사용해야하는지 설명한다. 난 `AbstractSecurityWebApplication`를 구현한 하위 클래스가 필요하다.  
하지만 난 이미 `AbstractAnnotationConfigDispatcherServletInitializer`에 대한 하위 클래스를 구현했다.  
나는 각각 하나씩 하위클래스를 갖고 있어야 할까? AbstractSecurityWebApplicationInitializer JavaDoc에 순서 지정에 대한 몇가지 논의가 있다.  
여기서는 내가 한 개 이상의 initializer 클래스를 갖고 있어야 함을 의미한다.
</div>
</details>

**[정리]**  

AbstractAnnotationConfigDispatcherServletInitializer에 대한 하위클래스를 이미 구현해놓은 상태인데 스프링 세큐리티 적용하려고 보니까 AbstractSecurityWebApplicationInitializer 얘도 하위클래스 확장하라고 하는데 나는 각각 둘다 확장해야되냐?는 질문이며,
답변은 맞다 AbstractSecurityWebApplicationInitializer 이것도 확장해야한다 라는 내용이다.

내가 궁금했던 케이스랑은 다르긴 한데 위 글의 질문자가 올린 코멘트를 통해서 docs는 **War로 생성한다는 가정하에** AbstractSecurityWebApplicationInitializer를 직접 구현하라고 되어있는 것을 알게 되었다.
자 그럼 다음 글을 읽어보자.

#### 2-4. WebSecurityConfigurerAdapter and AbstractSecurityWebApplicationInitializer absent
<details>
<summary>[질문 상세 내용]</summary>
<div markdown="1">

내가 본 대부분의 예시는  
1) Configuration 설정을 위한 WebSecurityConfigurerAdapter  
2) 세큐리티 필터 초기화를 위한 AbstractSecurityWebApplicationInitializer  

직장 내 프로젝트에서 스프링 세큐리티를 쓰고있는데 위에서 언급된 클래스 중 어떤 것도 사용할 수 없었다.  

> 그럼 어떻게 내 프로젝트에서 스프링세큐리티를 초기화하고 구성할 수 있을까?

이 프로젝트는 스프링부트를 사용하며 아래 클래스/인터페이스들을 상속/구현하고있다.  
a) AuthenticationProvider  
b) Authentication  
c) Filter  

</div>
</details>

**[정리]**  
jar를 이용한 스프링부트의 자동 구성에 의해 정의가 된다. 전체 workflow는:
1. `WebSecurityConfigurerAdapter` 빈을 등록하지 않았다면, `SpringBootWebSecurityConfiguration`이 자동으로 생성해준다.
2. `WebSecurityEnablerConfiguration`은 @EnableWebSecurity가 항상 존재하는지 확인한다.
3. @EnableWebSecurity는 스프링 세큐리티 디폴트 구성정보인 WebSecurityConfiguration을 임포트한다. 이 구성정보는 springSecurityFilterChain을 bean을 정의하고 있다.
4. SecurityFilterAutoConfiguration은 내장 서버에 springSecurityFilterChain을 등록해준다(FilterRegistrationBean 정의를 통해).  
   이것은 AbstractSecurityWebApplicationInitializer가 동작하는 방식과 동일하지만 스프링부트 내장형 서버의 방식이다.

단, war 이용시 위와 같은 자동 구성이 되지 않으니 `AbstractSecurityWebApplicationInitializer`를 직접 구현해야 한다.

### 3. HTTPSecurity
WebSecurityConfigurerAdapter에서 configure(HttpSecurity http) 메소드를 제공한다.  

이 메소드에서 URL 및 로그인 설정을 함으로써 Spring Security가 어떤 URL에 인증이 필요하고, 어떤 로그인 인증 과정을 사용하는지 알 수 있게 된다.

이 메소드를 통해서 자신만의 인증 메커니즘을 만들 수 있다.

```java
@Configuration
@EnableWebSecurity //Spring Security 설정 활성화
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {
    //...

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class)
            .formLogin().disable()
            .csrf().disable() //csrf 공격으로부터 안전하고 매번 api 요청으로부터 csrf 토큰을 받지 않아도 되므로 disable 처리함
            .cors().configurationSource(source()).and()
            .authorizeHttpRequests() //인가에 대한 설정
            .antMatchers("/api/oauth/**").permitAll()
            .antMatchers("/api/token/**").permitAll()
            .antMatchers("/h2-console/**").permitAll()
            .antMatchers("/swagger-resources/**").permitAll() // swagger 관련 리소스 시큐리티 필터 제거
            .anyRequest().authenticated()
    ;
  }
  
  //...
}
```

antMatchers를 이용하여 URL을 매치하고, access 및 hasRole 메소드를 이용해 URL에 권한을 부여할 수 있다.

- HTTPSecurity 메소드 일부

| 메소드                                 | 설명                                         |
|-------------------------------------|--------------------------------------------|
 | annonymous()                        | 인증 없이 접근 가능한 URL을 정의한다.                    | 
| authenticated()                     | 인증된 사용자만 접근 가능한 URL을 정의한다.                 | 
| fullyAuthenticated()                | 완전히 인증된 사용자만 접근할 수 있다.                     | 
| hasRole() or hasAnyRole()           | 인증된 사용자가 주어진 역할을 가지고 있으면 접근 가능한 URL을 정의한다. | 
| hasAuthority() or hasAnyAuthority() | 특정 권한을 가지는 사용자만 접근할 수 있다.                  | 
| hasIpAddress()                      | 특정 아이피 주소를 가지는 사용자만 접근할 수 있다.              | 
| access()                            | SpEL 표현식에 의한 결과에 따라 접근할 수 있다.              | 
| not()                               | 접근 제한 기능을 해제한다.                            | 
| permitAll() or denyAll()            | 접근을 전부 허용하거나 제한한다.                         | 

Role은 역할이고, Authority는 권한이이지만 표현의 차이일 뿐이다.  
Role은 "ADMIN"으로 표현하고 Authority는 "ROLE_ADMIN"으로 표현하는 것에 차이가 있다.

### 4. AuthenticationManagerBuilder

WebSecurityConfigureAdapter에서 아래 예제의 메소드를 통해서 인증 객체를 설정할 수 있다.
- Authentication을 사용한 예

```java
@Configuration
@EnableWebSecurity //Spring Security 설정 활성화
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {
  //...
  @Autowired
  public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(jwtUserDetailsService).passwordEncoder(passwordEncoder);
  }
  //...
}

@Service
public class UserDetailsService implements org.springframework.security.core.userdetails.UserDetailsService {

  @Autowired
  private UserRepository userRepository;

  @Override
  public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    Optional<User> user = userRepository.findById(Long.parseLong(username));

    if (user == null) {
      throw new UsernameNotFoundException("404 - User Not Found");
    }

    return new yapp.bestFriend.service.user.UserDetails(user.get());
  }

}
```
* * *
### 참고
- [Spring Security 개요 및 인증 과정](https://dingue.tistory.com/5)
- [In-Memory Authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/in-memory.html)
- [AbstractSecurityWebApplicationInitializer vs. AbstractAnnotationConfigDispatcherServletInitializer](https://stackoverflow.com/questions/22447603/abstractsecuritywebapplicationinitializer-vs-abstractannotationconfigdispatcher)
- [WebSecurityConfigurerAdapter and AbstractSecurityWebApplicationInitializer absent](https://stackoverflow.com/questions/56064735/websecurityconfigureradapter-and-abstractsecuritywebapplicationinitializer-absen)
- [토리맘의 한글라이즈 프로젝트 - Spring Security Java Configuration](https://godekdls.github.io/Spring%20Security/javaconfiguration/#1613-abstractsecuritywebapplicationinitializer-with-spring-mvc)
