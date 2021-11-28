---
title: '회원가입기능 만들기'
layout: post
categories: frontEnd
tags: Web react
comments: true
---

client에서 이름, e-maile, PW 등을 입력 후 서버에 보내주면 서버에서 그 데이터를 받아야 합니다. 서버에서 클라이언트로 부터 데이터를 받을 때 bodyParser라는 dependencies가 필요합니다. 이 dependencies를 이용하여 Client에서 보내주는 자료들을 이름, e-maile, PW를 받을 수 있습니다.

## bodyParser 다운받기
아래 명렁어를 터미널에 입력하여 다운받습니다.
```
npm install body-parser --save
```

지금은 로그인을 하거나, 회원가입할 때 만들어준 클라이언트가 없습니다. 그러므로 데이터를 클라이언트에 보내줄 수가 없으므로 대처하기 위해 postman을 다운받을 수 있습니다.

## CRA 살펴보기 - 디렉토리 구조
```
client  
├─ node_modules //NPM또는 Yarn을 통해 설치된 여러가지 라이브러리  
├─ public  
├─── src  
│ ├── _actions   //_actions, _reducer: Redux를 위한 폴더 
│ ├── _reducer  
│ ├── components  
│ │ ├─── views     //페이지를 넣을 폴더 
│ │   ├─── LandingPage //처음 홈페이지 접속 시 최초 보여지는 페이지
│ │   ├─── LoginPage   //로그인 페이지
│ │   ├─── NavBar      //사이드바랑 상단바
│ │   ├─── RegisterPage
│ ├── App.js //Routing 관련 일을 처리(로딩페이지에 가고 싶다면 로딩페이지에 가도록 분류해주고, 랜딩페이지에 가고 싶다면 랜딩페이지에 갈 수 있는 분류를 해줌)
│ └── Config.js //환경변수를 정하는 곳
├─ hoc //Higher Order Component의 약자  
├─ utils //여러군데에서 쓰일 수 있는 것들을 이곳에 넣어둬서 어디서든 쓸수 있게 해줌
```
- **utils**: Landing 페이지에서 사용하는 소스를 loginPage, RegisterPage에서 쓴다고 하면 loginPage, RegisterPage에 똑같은 소스를 복사하는게 아니라 utils에 넣어두고 Landing, loginPage, RegisterPage 세 군데에서 사용할 수 있게 해줍니다.
### hoc
hoc는 다른 컴포넌트를 갖는 펑션입니다. 예를들어 여러가지 컴포넌트를 넣어 놓고 로직에 의해 해당 컴포넌트에 들어갈 자격이 있는지 판단을 한 후 다음 액션을 취할 수 있도록 해주는 Auth라는 HOC가 있다고 해봅시다. Auth 안에 로그인이 된 사람만 들어갈 수 있는 페이지가 있는 경우, Auth는 해당 유저가 해당 페이지에 들어갈 자격이 되는지를 알아낸 후에 자격이 된다면 Logged IN COMPONENT에 가게 해주고 아니라면 다른 페이지로 보내버립니다. 예시로 든 자격이 아니더라도 다른 기능을 hoc에 넣어놓은 후 많은 컴포넌트들이 이용할 수 있게 된다고 생각하면 됩니다.

## CRA 살펴보기 - src/App.js
React에서는 페이지 간의 이동을 할 떄 React Router Dom이라는 것을 사용합니다. 우선 react-router-dom을 설치합니다. 설치 완료 후 App.js 안에서 페이지 이동을 할 수 있도록 라우팅을 구현해줍니다.
```
npm install react-router-dom --save
```
지금까지는 Client를 구현하지 않았기 때문에 postman을 이용해서 backend가 잘 구축되어 있는지 여러 기능들을 테스트해봤습니다. 지금부터는 React.js를 사용해서 Server에 req.를 보내보도록 하겠습니다. 이때 사용할 것이 AXOIS 입니다. AXOIS는 jQuery 사용할 때 AJAX와 같은 개념입니다.
```
npm install axios --save
```

## Proxy Server가 뭐에요?
클라이언트와 서버가 둘이서 요청을 주고 응답을 받는데 이 안에 프록시 서버가 들어갈 수 있습니다. 만약에 유저가 있고 인터넷이 있다고 생각했을 때, 유저가 무언가를 보낸 경우 프록시 서버에서 많은 일들을 할 수 있습니다.
> 1. 요청받은 IP를 Proxy Server에서 임의 변경 가능  
- 만약에 유저의 아이피가 '111.111.111.111'이라고 가정해보죠. 그럼 이때 프록시 서버에서 유저의 아이피를 임의로 변경 할 수 있습니다. 그럼 인터넷에서는 접근하는 사람의 IP를 모르게 되는거죠.
2. 요청받은 데이터 임의 변경 가능  
- IP 뿐만이 아니라 유저에서 보내는 데이터도 중간에서 임의로 바꿀 수 있습니다.
3. 방화벽 기능
4. 웹필터 기능
5. 캐쉬 데이터, 공유 데이터 제공 기능
 - 어떤 서버에 들어갔을 때 static한 이미지들을 프록시 서버에 저장 가능합니다. 유저가 무언가를 보고 싶을 떄 인터넷까지 가지않아도 프록시 서버에서 가져와 빠르게 데이터를 볼 수 있습니다.

이처럼 프록시 서버는 다양한 기능들을 수행하고 있습니다. Proxy Serve를 쓰는 이유를 정리해보자면
> 1. 회사에서 직원들이나 집안에서 아이들 인터넷 사이트 접근 제어
2. 캐쉬를 이용해 더 빠른 인터넷 제공
3. 더 나은 보안 제공
4. 이용 제한된 사이트 접근 가능  

정도가 있습니다.

<!--author-->