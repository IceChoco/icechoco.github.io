---
title: 'Redux 설치하기'
layout: post
categories: frontEnd
tags: Web react Redux
comments: true
---

Redux를 자신이 개발할 어플리케이션에 설치하고 설정하는 방법을 기록하겠습니다. 우선 다운 받아야 할 dependency들이 redux, react-redux, redux-promise, redux-thunk까지 총 4개가 있습니다. 이 중에서 redux-promis, redux-thunk는 redux의 미들웨어입니다.
```
npm install redux react-redux redux-promise redux-thunk --save
```
## redux-promise, redux-thunk가 필요한 이유
redux-promise, redux-thunk는 redux를 잘 쓸 수 있게 도와주는 역할을 하는 미들웨어입니다. redux는 redux store 안에 모든 state를 관리하고 있습니다. 이 store 안에 있는 state를 변경하기 위해서는 dispatch를 이용해서 action을 통해 변경시킬 수 있습니다. 이렇게 store가 데이터를 받을 때 항상 객체 형식으로 데이터를 받는 것이 아니라 때에 따라 promise, functions 형식으로 받기도 합니다. 그런데, redux 기초편에서 봤듯 우리는 reudx store는 객체 형식으로만 데이터를 받을 수 있다고 배웠습니다.

**redux-thunk**는 dispatch한테 어떻게 function 형식을 받는지 그 방법을 알려주고, **redux-promise**는 dispatch한테 promise가 왔을 때 어떻게 해야하는지 store에게 알려주는 역할을 합니다. 즉 이 2가지를 다운받지 않고 promise, functions 형식을 받아 redux를 사용하면 에러가 발생합니다.

## combineReducers가 하는 역할
<p align="center"><img src="/assets\img/combine-reducer.PNG" height="300px" width="500px"></p>
store에는 reducer가 여러가지 있을 수 있습니다.이 reducer는 어떻게 state가 변하는 지를 보여준 다음, 변한 마지막 값을 리턴해주는 일을 합니다.
state는 유저, 구독 등 여러가지에 대한 state가 있을 수 있으므로 각각의 state에 따라 reducer도 나눠져 있습니다. redux에 내장되어 있는 combineReducer는 여러개의 reducer들을 Root Reducer에서 하나로 합쳐주는 역할을 합니다.

<!--author-->