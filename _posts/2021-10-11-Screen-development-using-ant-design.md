---
title: 'CSS Framework, Ant Design을 활용한 효율적인 화면개발하기'
layout: post
categories: frontEnd
tags: Web react
comments: true
---

원래는 모든 것을 CSS를 활용해서 하나하나 만들 수 있지만, 그렇게 되면 기능을 신경 쓰는 것 이외에 너무 많은 시간을 CSS에 투자해야합니다. 그래서 기능 개발에 초점을 맞추기 위해 CSS 프레임 워크를 사용해보도록 하겠습니다. 요즘 많은 기업들이 실무에서도 CSS 프레임워크가 잘 되어 있어 많이 사용하고 있습니다.

가장 유명한 프레임워크는 Mateiral UI, React Bootstrap, Semantic UI, Ant Design, Materialize가 있습니다. 저는 그 중 Ant Design을 활용해서 개발해보도록 하겠습니다. Ant Design은 중국에서 개발되었으며 많은 기능들이 들어있기 때문에 사이즈가 큰 편입니다. 하지만 스타일이 굉장히 깔끔하고 엔터프라이즈 환경에서도 어울리는 디자인 생성이 가능합니다. 가장 큰 장점은 사용하기가 굉장히 편합니다. Material UI도 좋은 프레임워크지만 처음 사용할 때 어려움이 있습니다.

가장먼저 npm을 사용하여 Ant Design을 설치합니다.
```
npm install antd --save
```
그 다음 사용할 곳에 아래 import문을 추가하면 ant Design을 사용할 수 있습니다.
```
import 'antd/dist/antd.css'; // or 'antd/dist/antd.less'
```

### 참고  
[Ant Design 공식 홈페이지](https://ant.design/) 

<!--author-->