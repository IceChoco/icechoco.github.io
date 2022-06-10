---
title: 'CORS(Cross-Origin Resource Sharing) 에러'
layout: post
categories: frontEnd
tags: Web react
comments: true
---
```
Access to XMLHttpRequest at 'http://localhost:5000/api/hello' from origin 'http://localhost:3000' has been blocked
by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```
제가 설정한 서버의 포트는 5000번이고, 클라이언트의 포트는 3000번입니다. 이렇게 서버와 클라이언트의 포트가 다를 때 따로 설정을 안해주면 CORS 정책에 의해 막히게 됩니다. CORS 정책이란 오리진(서버, 클라이언트 각각을 말함)이 다른데 리소스를 쉐어링 할 때 적용되는 정책입니다.
- 임의로 proxy 설정하기  
  첫번쨰로, npm을 사용하여 http-proxy-middleware 모듈을 설치합니다.
  ```
  npm install http-proxy-middleware --save
  ``` 
  그 다음 클라이언트의 src 폴더 아래에 setupProxy.js 파일을 만들어서 아래 내용을 넣어줍니다.
  ```
  const { createProxyMiddleware } = require('http-proxy-middleware');

  module.exports = function (app) {
      app.use(
          '/api',
          createProxyMiddleware({
              target: 'http://localhost:5000', //프론트엔드 3000번에서 줄 때 타겟을 5000번으로 주겠다.
              changeOrigin: true,
          })
      );
  };
  ```
  위와같이 프록시를 설정해주면 아까 발생하던 CORS에러가 더이상 발생하지 않고, 제가 서버에서 정상적으로 /api/hello를 통해 수신받았을 때 출력하도록 해놓은 메시지인 '안녕하세요 저 프론트엔드인데요'를 정상출력하였습니다.

### 참고
[Proxying API Requests in Development | Create React App](https://create-react-app.dev/docs/proxying-api-requests-in-development)