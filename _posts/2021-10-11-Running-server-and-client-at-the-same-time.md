---
title: 'Concurrently를 이용한 백서버와 클라이언트서버 동시 실행시키기'
layout: post
categories: server
tags: server
comments: true
---

개발하면서 매번 터미널 두개 켜서 하나는 server run 시키고, 다른 하나는 react run 시키고 두 번 해줘야 하는게 정말 불편했습니다. 그러던 와중에 알게된 것은 바로 **Concurrently**를 이용해서 백서버와 클라이언트서버를 동시에 실행시키는 것! 그래서 이번 글에서는 어떻게 Concurrently를 활용해서 동시에 실행시킬 수 있는지 알려드리겠습니다. 우선 사용을 위해서는 Concurrently 라이브러리를 다운받으셔야합니다.
```
npm install concurrently --save
```
설치 후 사용방법은 package.json 파일 내 "scripts" 부분에 실행할 명령어를 넣고 concurrenlty를 적은 뒤 켜고 싶은 것들을 차례대로 적어주시면 됩니다.
```
"dev": "concurrently   \"npm run backend\" \"npm run start --prefix client\""
```

## 참고
- [concurrently-npm](https://www.npmjs.com/package/concurrently) 

<!--author-->