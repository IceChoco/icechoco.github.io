---
title: 'Redux 기초'
layout: post
categories: frontEnd
tags: Web react Redux
comments: true
---

Redux는 상태관리 라이브러리(predictable state container) 입니다. 그렇다면 여기서 말하는 state가 뭘까요? React에서는 Props와 State가 있습니다. 이 두가지를 비교하면서 state에 대해 알아보겠습니다.
## Props VS State
### Props
- Props는 Properties의 줄임말입니다.
- 부모 컴포넌트와 자식 컴포넌트가 있는 경우 데이터를 서로 주고 받아야겠죠? 두 컴포넌트간의 데이터를 주고 받을 때는 Props를 이용해야 합니다.
- **단방향 데이터 흐름**: child component에 전달되는 값 또는 method. 부모 컴포넌트에서 자식 컴포넌트로 즉 위에서 아래로만 데이터 전달이 가능합니다. 반대로 child component에서 parent component로 전달은 불가능합니다.
- **read donly**: child component로 전달받은 props 값은 변하지 않습니다다. 만약 부모로부터 받은 값을 수정하고 싶다면 수정된 값으로 부모 컴포넌트에게서 다시 받아야 합니다.
- **소스코드 예시**
      
    - parent component

    ```
    import React, {component} from 'react';
    import Hello from './Hello';

    class App extends Component{
        rendor(){
            return(
                <Hello name="홍길동"> //Hello라는 자식 컴포넌트에게 name이라는 변수를 전달한다.
            );
        }
    }
    ```
    - child component

    ```
    import React, {component} from 'react';

    class Hello extends Component{
        rendor(){
            return(
                <div>
                    안녕하세요. {this.props.name}님!!
                </div>
            );
        }
    }
    ```

### State
- **컴포넌트가 가지고 있는 값**: parent component에서 child component로 보내는게 아닌 그 component 자체 안에서 데이터를 전달할 수 있습니다.
- state는 변할 수 있습니다.
- state가 변경되면, component가 re-rendering 됩니다. (즉, update 라이프 사이클이 수행됩니다)
- <span style="color:red">state의 변경은 반드시 setState()함수를 이용</span>해줘야 합니다. 절대 `this.state.term = '검색어'`와 같이 직접 할당할 수 없습니다.
- **소스코드 예시**

```
import React, {Component} from 'react';

class App extends Component{
    constructor(props){
        super(props);

        this.state = { //초기화
            term: "",
        }
    }

    rendor(){
        return(
            <div>
                <input
                    onchange={(e) => this.setState({term: e.target.value})} //수정
                >
                </input>
                <div>InputText1 {this.state.term}</div> //읽기
            </div>
        );
    }
}
```
<p align="center"><img src="/assets\img/without-redux.PNG"></p>
그래서 Redux는 이러한 state를 관리해주는 툴입니다. 만약 아래처럼 Comments 라는 컴포넌트 안에 A라는 컴포넌트가 있고, 또 그 안에 B라는 컴포넌트가 있다고 해봅시다.  
<p align="center"><img src="/assets\img/redux-exam.PNG" height="200px" width="600px"></p>
B 컴포넌트에서 Action이 발생했다면 commments 컴포넌트에 어떤 액션이 일어났는지 알려줘야합니다. 이때 Redux가 없다면 B컴포넌트 → A컴포넌트 → Comments 컴포넌트로 하나하나 단계적으로 타고 올라가야합니다. 
<p align="center"><img src="/assets\img/redux-store.PNG" height="200px" width="600px"></p>
그런데 이때 상위 컴포넌트로 이동하는게 아니라 Redux 스토어에 저장을 해놓게되면 여러 컴포넌트를 거쳐 올라가지 않아도 B컴포넌트 → Store로 직접 접근이 가능하며, 반대로 store → B컴포넌트로도 직접 접근이 가능합니다. 여러개의 컴포넌트를 왔다갔다 해야하는 과정이 빠지니 훨씬 편하게 State관리를 할 수 있습니다.

## Redux 데이터 Flow
<p align="center"><img src="/assets\img/redux-data-flow.PNG" height="200px" width="600px"></p>
Redux는 철저하게 한뱡향 데이터 플로우를 가집니다(strict unidirectional data flow). 리액트 컴포넌트에서 시작되어 Action → Reduecer → Store를 거쳐 다시 리액트 컴포넌트로 돌아옵니다.

### Action
Action은 무엇이 일어났는지 설명하는 객체입니다. 형식이 객체여야만 redux store가 내용을 받을 수 있습니다.
```
{type: 'LIKE_ARTICLE', articleId: 42} //articleId 42번을 좋아요 했다.
{type: 'FETCH_USER_SUCCESS', response: {id:3, name: 'Mary'}} //이름이 Mary이고 id가 3번인 유저를 가져오는 것을 성공했다.
{type: 'ADD_TODO', text: 'Read the Redux docs.'} //이 텍스트를 todo List에 add했다.
```

### REDUCER
이 액션을 함으로 인해서 A였던 값이 B로 변했다는 것을 설명해줍니다. 자세하게는 이전 State와 action object를 받은 후에 next state를 return 해주는 역할을 합니다.
```
{previousState, action} => nextState
```

### STORE
Application의 state을 감싸주는 역할을 합니다. 이 Store안에는 여러가지의 많은 메소드가 있습니다. 이 메소드를 이용하여 State를 관리할 수 있습니다.

<!--author-->