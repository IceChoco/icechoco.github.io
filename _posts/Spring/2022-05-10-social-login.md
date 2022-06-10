---
title: '[Spring] 카카오, 구글 소셜 로그인 구현하기'
layout: post
categories: java, spring
tags: java
comments: true
---

Spring Boot 구조로 구성된 사이드 프로젝트에 적용 예정인 OAuth2 소셜로그인에 대해 정리하기위해 이 글을 쓰게 되었다.

## 사전 지식
### OAuth2
OAuth란 **인터넷 사용자들이 비밀번호를 제공하지 않고 다른 웹사이트 상의 정보들에 대해 접근 권한을 부여할 수 있는 수단을 제공하는 것**이다.
예를 들어 구글에 저장되어 있는 사진을 가지고 어떤 서비스를 만들고자 할 때 사용된다.

**OpenID connect**
- 권한관리를 위한 OAuth 2.0 위에 인증 관리를 올린 것

**Spring Security(스프링 세큐리티)**
- 자바 어플리케이션에 권한(Authorization)과 인증(Authentication)을 구현하는데 필요한 기능을 제공하는 프레임 워크

### JSON Web Token
JSON Web Token(JWT)은 참여자 간에 안전하게 정보를 주고 받기 위해 JSON 객체에 정보를 내장하는 방법을 정의하는 개방형 표준(RFC7519)이다.

토큰의 종류
- access_token: 리소스 서버에 실제로 접근하기 위한 권한을 갖고 있음
  - 중요한 정보이기 때문에 토큰의 보호기능으로서 만료시간이 있으나 짧음(대개 몇 분 정도로 유지)
- Refresh Token: 새로운 access_token 토큰을 발급 받기 위해 요청
- ID Token: 굳이 서버에 가지 않아도 사용자의 정보를 바로 받아서 쓸 수 있음

#### 토큰의 구조
- Header: 토큰의 Type, **서명에 대한 알고리즘**, BASE64로 인코딩됨
- ★ Payload: 데이터, valims(key-value 형태의 데이터), BASE64로 인코딩됨
- signature: 헤더와 payload의 내용이 임의로 조작되었는지를 검증해줌

##### 1. Header
토큰의 서명을 생성하기 위해 어떤 알고리즘을 사용하였는지를 나타냄
```
{
    "alg": "ES256",
    "type": "JWT",
    "kid": "Key ID"
}
```
- alg
- type
- kid: Key ID Header Parameter, JWS(Json web Signature)를 암호화하기 위해 사용된 키를 가리킴

### 2. Payload(Data)
Payload는 실제 JWT의 컨텐츠이다.

### 3. signature
보통 RS256 방법을 많이 쓴다.
- **HS256**
  - Secret key가 있으며 로그인과 validation이 둘다 이루어짐
- **RS256**
  - private key로만 사이닝을 하고, 다른 서비스들은 public key를 가지고 Validation하는 것

그럼 keycloak는 이 public key를 언제 다운받을까? public key를 안갖고 있는 최초 1회에 다운로드 받는다.

### Statefule & Stateless

|           | Stateful(Session)                                 | Stateless(Token)       |
|-----------|---------------------------------------------------|------------------------|
| 세션 정보 저장  | In-Memory, Database와 같은 원격 저장소를 사용함               | X                      |
| 세션 회수(취소) | 가능                                                | X (토큰 만료 시간으로 보완)      |
| 정보 노출     | X(중요한 정보들을 client에 노출없이 서버에 저장해 쓸 수 있음, 세션 ID만 노출) | JWT는 위조는 불가하지만 내용은 공개됨 |
| 확장성       | 또 다른 메모리가 있으면 replication을 해야하기 때문에 확장성이 좋지 않음 | 아주 높음 |
| 예시        | JSession | JWT |

## 전체 시퀀스 다이어그램

![total_sequence_diagram](/assets\img/total_sequence_diagram.jpg)

> 시퀀스 상단: OAuth 2.0 표준을 따르는 소셜 로그인 시퀀스 다이어그램  
> 시퀀스 하단: JWT 토큰의 유효기간이 끝났을 때의 시퀀스 다이어그램

### 시퀀스 설명
위 다이어그램과 같이 보길 권장한다.
1. 유저 - 소셜 로그인을 요청
2. 프론트엔드 - 백엔드로 GET "/oauth2/authorization/{provider-id}?  
   redirect_uri=http://localhost:3000/oauth/redirect"으로 OAuth 인가 요청
3. 백엔드 - Provider 별로 Authorization Code 인증을 할 수 있도록 리다이렉트 (Redirect: GET "https://oauth.provider.com/oauth2.0/authorize?...")
4. 유저 - 리다이렉트 화면에서 provider 서비스에 로그인
5. 로그인이 완료된 후, Authorization server → 백엔드로 Authorization Code 응답
6. 백엔드 - Authorization Code를 이용해 Authorization Server에 Access Token 요청
7. Authorization Server → 백엔드로 Access Token 응답
8. 백엔드 - Access Token을 활용하여 Resource Server에 User data 요청
9. 백엔드 - 획득한 User Data를 DB에 저장 후, JWT 액세스 토큰과 Refresh Token을 생성
10. Refresh Token은 수정이 불가능한 쿠키에 저장하고, 액세스 토큰은 프론트엔드 리다이렉트 URI에 쿼리스트링에 토큰을 담아 리다이렉트 (Redirect: GET http://localhost:3000/oauth/redirect?token={jwt-token})
11. 프론트엔드에서 토큰을 저장 후, API 요청 시 헤더에 Authorization: Bearer {token}을 추가하여 요청
12. 백엔드에서는 토큰을 확인하여 권한 확인
13. 토큰이 만료된 경우, 쿠키에 저장된 리프레스 토큰을 이용하여 액세스 토큰과 리프레시 토큰을 발급


### 참고
- [JWT를 소개합니다](https://meetup.toast.com/posts/239)
- [스프링부트 소셜 로그인](https://deeplify.dev/back-end/spring/oauth2-social-login#%EC%B9%B4%EC%B9%B4%EC%98%A4-oauth-%EC%84%9C%EB%B9%84%EC%8A%A4-%EB%93%B1%EB%A1%9D)
- [Demystifying OAuth 2.0 - A Tutorial & Primer](https://devansvd.com/oauth/)