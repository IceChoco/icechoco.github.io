---
title: 'Create-React-App으로 React 시작하기'
layout: post
categories: frontEnd
tags: Web react
comments: true
---

기존에는 리액트 앱을 처음 실행하기 위해서는 webpack이나 babel 같은 것들을 설정하기 위해서 굉장히 많은 시간이 소요되었습니다. 하지만 지금은 따로 설정하지 않아도 Create-React-App이라는 Command를 이용하여 react, webpack 등을 바로 설정할 수 있습니다.

## babel
**babel**이란 자바스크립트 ES6, ES7 등 해마다 새로운 메소드를 추가한 버전이 있습니다. 추가한 메소드들이 최신 브라우저에서는 잘 수행되나 오래된 브라우저에서 수행이 되지 않는 경우가 있습니다. 최신 자바스크립트 문법을 사용하여도 구형 브라우저에서도 수행될 수 있게 ES6문법을 ES5로 바꿔주는 역할을 하는 것이 babel의 역할입니다.

## webpack
**webpack**이란 예전에 웹사이트를 만들 때에는 js파일, css, html과 같이 간단하게 만들었으나 점점 웹사이트의 사이즈가 커졌습니다. 그러면서 js, html 파일 뿐만 아니라 라이브러리 및 프레임워크를 많이 사용하면서 구조가 복잡해졌습니다. 복잡해진 구조들을 webpack을 이용해서 bundle(묶어주는) 시켜주는 것이 webpack의 역할입니다.  

webpack은 src 폴더 안의 영역만 관리를 해주며, public 폴더 안의 소스까지 관리를 해주지 않습니다. 그래서 이미지 파일과 같은 특정 파일을 webpack에 넣고 싶을 때는 src 폴더 안에 넣어 webpack이 모아주는 역할을 할 수 있도록 해주면 됩니다.

터미널에 아래 스크립트를 수행하여 다운을 받을 수 있습니다.
```
npx create-react-app .
```
.의 의미는 내가 있는 디렉토리 안에다가 react를 설치하겠다는 의미입니다.

## Create-React-app
저희는 react를 설치하고자 하는 client 경로에서 `npx create-react-app .` 명령어를 통해 다운 받았습니다. 하지만 이전에는 아래와 같은 명령어를 통해 컴퓨터 안에 다운을 받았습니다.
```
npm install -g create-react-app
```
그래서 컴퓨터 안에 받은 것을 활용해서 react를 실행했었습니다. 하지만 이제는 global로 다운을 받을 필요가 없이 npx를 이용해서 다운을 받지 않고 노드 레지스트리에 있는 react-creat-app을 가져와서 이용할 수 있습니다. 그럼 npm과 npx의 차이가 뭘까요? 
### NPM: node package manager
(1) 레지스트리와 같은 저장소 역할을 한다. 라이브러리를 담고 있는 곳이 레지스트리이다.  
(2) npm에 관한 것은 'package.json' 파일에 정의되어 있다.  
>ex) dependencies는 어떤걸쓰고, 버전은 몇을 쓰고 있는지, script는 어떤 것을 써야하는지 등  

(3) local vs Global  
  - npm install을 사용하여 다운받으면 자동적으로 로컬에 다운을 받게 되고, node_modules 폴더에 저장된다.
  ```
  npm install create-react-app
  ```
  - `-g`라는 플래그를 함께 사용하면 Global로 다운받게 되고, `bin/` 폴더에 저장된다. 리눅스는 `/usr/local/bin` 폴더 안에, Windows는 `%AppData%/npm`에 저장된다  
  ```
  npm install -g create-react-app
  ```
### NPX
  - 장점
    1. Disk Space를 낭비하지 않을 수 있다.
    2. 항상 최신 버전을 사용할 수 있다.



## 참고
- [노드 리액트 기초 강의 #16 Create-React-App](https://www.youtube.com/watch?v=P4gXaWZz30Q) 

<!--author-->