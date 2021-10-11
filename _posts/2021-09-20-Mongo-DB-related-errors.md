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

## 2. MongooseServerSelectionError: Could not connect to any servers in your MongoDB Atlas cluster.
본가에서 몽고DB를 mongoose를 이용해서 연결 후 정상 확인까지 했으나, 집에와서 run을 해줬더니
```
PS C:\Ara\react-project\to-do-list> npm run start

> to-do-list@0.1.0 start C:\Ara\react-project\to-do-list
> node index.js

MongooseServerSelectionError: Could not connect to any servers in your MongoDB Atlas cluster. One common reason is that you're trying to access the database from an IP that isn't whitelisted. Make sure your current IP address is on your Atlas cluster's IP whitelist: https://docs.atlas.mongodb.com/security-whitelist/
    at NativeConnection.Connection.openUri (C:\Ara\react-project\to-do-list\node_modules\mongoose\lib\connection.js:796:32)
    at C:\Ara\react-project\to-do-list\node_modules\mongoose\lib\index.js:330:10
    at C:\Ara\react-project\to-do-list\node_modules\mongoose\lib\helpers\promiseOrCallback.js:32:5
    at new Promise (<anonymous>)
    at promiseOrCallback (C:\Ara\react-project\to-do-list\node_modules\mongoose\lib\helpers\promiseOrCallback.js:31:10)
    at Mongoose._promiseOrCallback (C:\Ara\react-project\to-do-list\node_modules\mongoose\lib\index.js:1151:10)
    at Mongoose.connect (C:\Ara\react-project\to-do-list\node_modules\mongoose\lib\index.js:329:20)
    at Object.<anonymous> (C:\Ara\react-project\to-do-list\index.js:17:10)
    at Module._compile (internal/modules/cjs/loader.js:1072:14)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1101:10) {
  reason: TopologyDescription {
    type: 'ReplicaSetNoPrimary',
    servers: Map(3) {
      'todolist-shard-00-00.hunw7.mongodb.net:27017' => [ServerDescription],
      'todolist-shard-00-01.hunw7.mongodb.net:27017' => [ServerDescription],
      'todolist-shard-00-02.hunw7.mongodb.net:27017' => [ServerDescription]
    },
    stale: false,
    compatible: true,
    heartbeatFrequencyMS: 10000,
    localThresholdMS: 15,
    setName: 'atlas-5hich8-shard-0',
    logicalSessionTimeoutMinutes: undefined
  }
}
```
이러한 에러가 발생했습니다. 구글링을 해보니 IP address가 제대로 할당이 안된 경우 발생할 수 있다는 글을 발견했습니다. 그래서 mongoDB사이트에서 'Network Access'로 들어가서 ADD IP ADDRESS - ADD CURRENT IP ADDRESS 버튼을 눌러 저의 현재 IP를 등록하게 하고 Confirm 버튼을 눌러줬습니다.  
confirm 버튼을 누르게 되면 pending 표시가 뜨고, 작업 완료 후 Active 상태로 변경됩니다. 그럼 이때 다시 터미널로 와서 기존 서버를 종료한 뒤 다시 run을 수행해봤습니다.
![mongodb-connected](/assets\img/mongodb-connected.png)  
그랬더니 연결이 성공했을 때 제가 출력하도록 해놓은 메시지인 'MongoDB connected...'를 정상 출력하였습니다.

### 참고
- [mongoose이용하여 mongoDB연결시 에러](https://velog.io/@kjy5947/mongoose%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-mongoDB%EC%97%B0%EA%B2%B0%EC%8B%9C-%EC%97%90%EB%9F%AC) 


<!--author-->