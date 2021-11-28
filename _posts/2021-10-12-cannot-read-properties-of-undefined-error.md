---
title: '[React] TypeError: Cannot read properties of undefined (reading getState)'
layout: post
categories: frontEnd
tags: Web react
comments: true
---

리액트 프론트엔드 서버의 index.js 파일에서 Redux 확장 프로그램인 DevTools extension을 연결하고, 리액트 앱에 store를 연동해주기위해 provider 컴포넌트를 사용하여 설정해줬다. 그랬더니 `TypeError: Cannot read properties of undefined (reading 'getState')`에러가 발생했다.
![typeerror](/assets\img/typeerror.PNG)

원인이 뭐지? 하고 구글 디버깅을 해봤더니 index.js 파일 내에 오타가 있는 경우 발생하는 에러라는 글을 찾았다. 아래는 변경 전 index.js 파일 소스이다.  
- **변경 전 프론트엔드 서버의 index.js**  
![typeerror-source](/assets\img/typeerror-source.PNG)  
위 ReactDOM.render의 provider 컴포넌트의 태그 부분을 자세히 보니 provider 시작 태그를 닫기 전에 store를 적어줘야하는데, 태그를 닫은 후 store를 적으면서 발생한 에러였다. store 위치를 procvider 시작 태그 닫기 전에 넣자 에러 없이 수행되었다.  
- **변경 후**  
![ok-typeerror-source](/assets\img/ok-typeerror-source.PNG)  

오늘도 구글신이 날 도왔다!

### 참고
- [[react에러]Cannot read property 'getState' of undefined](https://velog.io/@eve1s/react%EC%97%90%EB%9F%ACCannot-read-property-getState-of-undefined) 
- [1-8. Provider 컴포넌트를 사용하여 리액트 앱에 store 연동하기](https://redux.vlpt.us/1-8-provider.html)

<!--author-->