---
title: '[React] Attempted import error'
layout: post
categories: frontEnd
tags: Web react
comments: true
---
client 폴더 내 index.js 파일에서 하위폴더인 `_reducers`에서 데이터를 가져올려고 하자 Attempted import error가 발생했다.
![attempt-import-error](/assets\img/attempt-import-error.PNG)
```
./src/index.js
Attempted import error: 'Reducer' is not exported from './_reducers'.
```
프론트엔드 서버의 index.js 파일에서 './_reducers' 경로에 있는 index.js 파일을 import 하는데에 실패했다는 내용의 에러이다. 에러 발생 원인은 reducers 폴더 내 index.js 파일에서 export 시 default를 붙였기 때문이다. 
- reducers 폴더 내 index.js파일
    ```
    import { combineReducers } from "redux";

    export default rootReducer; //다른 파일에서 리듀서를 쓸 수 있도록 익스포트
    ```
- 변경 전 프론트엔드 서버의 index.js
    ```
    import { Reducer } from './_reducers';
    ```

위와 같은 import 구분을 중괄호를 제거하고 아래와 같이 변경하였더니 정상적으로 import 되었다.
```
import Reducer from './_reducers';
```
그럼 어떤 경우에 import 할 때 중괄호를 써야하고, 어떤 경우에는 중괄호를 없이 써줘야할까? **이유는 바로 export 방식의 차이이다.**
export 방식에 따라 {}중괄호를 사용하고, 안하고가 다른데 export 시 default를 붙인 경우 import에 중괄호 없이 써줘야한다. 반대로 `export rootReducer;`와 같이 default 없이 export한 경우 import에 중괄호를 제거한 후 써줘야한다.

### 참고
[react에서 import할때 {}중괄호 유무의 의미](https://mesonia.tistory.com/135) 

<!--author-->