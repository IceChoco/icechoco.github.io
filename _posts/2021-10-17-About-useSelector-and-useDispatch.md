---
title: '[React] useSelector, useDispatch를 사용하여 state에 접근하기'
layout: post
categories: frontEnd
tags: Web react
comments: true
---

**이 글의 목적**
- useSelector, useDispatch를 사용하여 state에 접근할 수 있다.

## react-redux
설치방법 및 redux에 관한 내용은 [Redux 기초 | IceChoco](https://icechoco.github.io/react/2021-10-11-Redux-Basics/)글을 참조하세요.
- state 조회하기: useSelector 사용
- action 발생시키기: useDispatch 사용

## useSelector
 - connect 함수를 이용하지 않고 리덕스의 state를 조회할 수 있다.
```
  import { useSelector } from "react-redux";
  const user = useSelector(state => state.user)
```

## useDispatch
 - 생성한 action을 useDispatch를 통해 발생시킬 수 있다.
 - 만들어둔 액션생성 함수를 import 한다.
 ```
 import { change_user } from '../modules/user' import { useDispatch } from 'react-redux' const User = () => { ... const dispatch = useDispatch(); dispatch(change_user(user)); ... }
 ```
 ```
 // 위에서 dispatch한 change_user는 아래와 같이 정의된 액션 생성 함수이다. export const change_user = createAction(CHANGE_USER, user => user);
 ```

### 참고
- [[React] useSelector, useDispatch로 state에 접근하기](https://juhi.tistory.com/23) 

<!--author-->