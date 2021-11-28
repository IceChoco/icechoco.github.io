---
title: '함수형 컴포넌트 vs 클래스 컴포넌트'
layout: post
categories: frontEnd
tags: Web react
comments: true
---

안녕하세요. 오늘은 React Component의 두 종류인 함수형 컴포넌트와 클래스 컴포넌트에 대해서 포스팅하겠습니다.
## Class Component
![class-component](/assets\img/class-component.PNG)  
- state와 Lifecyle method와 같은 더 많은 기능들을 사용 할 수 있습니다. 다만 코드가 좀 더 길어지고, 복잡해지며, 성능적인 면에서 느립니다.

## Functional Component
![functional-component](/assets\img/functional-component.PNG)
- state와 Lifecyle method를 사용할 수 없어 제공하는 기능들이 한정됩니다. 하지만 v16.8 이후 Hook를 이용하여 보완되었습니다.
- 코드가 짧아지고 간단해져 선언하기가 편합니다.
- 메모리를 클래스 컴포넌트보다 덜 사용해서 성능이 좋고 빠릅니다.
- 배포시, 결과물의 파일크기가 클래스 컴포넌트보다 작습니다.

Class component에서 사용 가능하고 functional Component에서 사용할 수 없는(v16.8 전) 기능에 대해 자세히 알아보기 위해 우선 아래 라이프 사이클 다이어그램을 보겠습니다.

## Lifecycle
![React-lifecycle-methods-diagram](/assets\img/React-lifecycle-methods-diagram.PNG)
  
class 컴포넌트를 이용하여 구현한 경우, 리액트에서 처음에 페이지를 킬 때 어떤 순서로 시작이 되는지 알려주는 다이어그램입니다. 하지만 functional 컴포넌트의 경우 위 라이프 사이클의 어느 기능도 사용할 수가 없습니다. 그러다보니 functional 컴포넌트는 빠르긴 하지만 너무 많은 기능이 안되다 보니 대부분 class형 컴포넌트를 사용했습니다. 그러다가 리액트 16.8 버전에서 Hook를 발표했습니다. 이 이후부터는 functional 컴포넌트에서도 라이프사이클과 state를 사용할 수 있게 되었습니다.
- **Class component VS Hook를 활용한 Functional Component**
![class-component-vs-functional-component](/assets\img/class-component-vs-functional-component.PNG)
  
    - Constructor: Constructor를 이용하여 state를 부여해줍니다.
    - rendor: Dom에 알맞게 넣어 화면에 렌더링해줍니다.

### 참고
- [React lifecycle methods diagram](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/) 


<!--author-->