---
title: 'Mongo DB 관련 오류'
layout: post
categories: database
tags: database React
comments: true
---

MongoDB 관련하여 발생할 수 있는 오류 List 및 해결방법 입니다.

## 1. MongoServerError: bad auth : Authentication failed.
```
PS C:\Ara\react-project\to-do-list> npm run start

> to-do-list@0.1.0 start C:\Ara\react-project\to-do-list
> node index.js

MongoServerError: bad auth : Authentication failed.
    at MessageStream.messageHandler (C:\Ara\react-project\to-do-list\node_modules\mongodb\lib\cmap\connection.js:467:30)
    at MessageStream.emit (events.js:400:28)
    at processIncomingData (C:\Ara\react-project\to-do-list\node_modules\mongodb\lib\cmap\message_stream.js:108:16)
    at MessageStream._write (C:\Ara\react-project\to-do-list\node_modules\mongodb\lib\cmap\message_stream.js:28:9)
    at writeOrBuffer (internal/streams/writable.js:358:12)
    at MessageStream.Writable.write (internal/streams/writable.js:303:10)
    at TLSSocket.ondata (internal/streams/readable.js:726:22)
    at TLSSocket.emit (events.js:400:28)
    at addChunk (internal/streams/readable.js:290:12)
    at readableAddChunk (internal/streams/readable.js:265:9) {
  ok: 0,
  code: 8000,
  codeName: 'AtlasError'
}
```

저의 경우는 index.js 파일 내 MongoDB 설정 시 패스워드 양 옆에 <>를 제거하지 않아 발생한 에러였습니다.  
설정을 `my_login_id:<my_password>`로 했었으나, `my_login_id:my_password`와 같이 <>를 제거해야 합니다.

<!--author-->