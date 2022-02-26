---
title: '[CS] FTP(File Transfer Protocol) Active와 Passive의 차이'
layout: post
categories: cs
tags: cs
comments: true
---

오늘은 FTP Active와 Passive의 차이에 대해서 정리해보려 한다.

## FTP(File Transfer Protocol)란? 
- File Transfer Protocol의 약자로 파일을 전송하는 통신 규약
  - FTP 서버에 파일들을 업로드, 다운로드 할 수 있도록 해주는 프로콜
  - FTP 서버와 FTP 클라이언트 간의 통신에서 이루어짐
- FTP는 Actvie, Passive 2개의 모드가 존재
  - 각각의 모드에서는 2개 또는 2개 이상의 포트가 연결을 맺고 데이터를 전송하는데 사용됨
  - 기본 동작모드로 Active 모드를 사용
  - 사용되는 포트의 종류는 2개가 있음
    - Command(명령) 포트: 연결 제어하는 역할, 접속 시에 사용됨 ex) 21 port
    - Data 포트: 데이터를 전송하는 역할 ex) Active:20 port, Passive:1024~65535 port
- FTP는 TCP 기반으로 만들어져 있음
- ack(Acknoledement)
  - 어떠한 컴퓨터가 네트워크를 통해 일련의 자료를 다른 컴퓨터로 성공적으로 전송했을 때, 전송을 받은 컴퓨터가 전송을 해 준 컴퓨터에게 보내는 신호.
  - 이는 수신측 컴퓨터가 "준비 완료"의 뜻을 알리는 신호이기도 함.
  - 긍정 응답 문자라고도 하며, 이와의 반대로는 부정 응답 문자(negative acknowledgement code, NACK)라고 부름

## Active 모드
- 아래 사용된 명령(Command)포트와 Data 포트는 **서버의 설정에서 임의 수정하여 사용 가능**
- Acitve 모드는 클라이언트가 서버에 접속을 하는 것이 아닌 **서버가 클라이언트에 접속하는 것**
![ftp_actvie.PNG](/assets\img/ftp_actvie.PNG)
### 동작방식
1. Client는 Server의 21번 포트로 접속 → **자신이 사용할 두번째 포트(5151 port)를 서버에 미리** 알려준다.
2. 서버는 클라이언트의 요청에 응답(acks)
3. Server의 20번 Data Port는 Client가 알려준 두번째 포트(5151 Port)로의 접속을 시도
4. Client가 Server의 요청에 응답(acks)

### 주의사항
- FTP Client에 방화벽이 설치되어 있어 외부에서의 접속을 허용하지 않는다면, FTP 접속이 정상적으로 이루어지지 않음. 접속이 되더라도 데이터 목록을 받아오지 못할 수 있음.
   - 즉, Client 측의 방화벽에 20 port가 차단되어 있는 경우, 데이터 채널 연결 불가능  
- Server와 Client에 **방화벽 설정 필요**  
  - Server: 20 Port에 대한 아웃바운드 허용(OutBound, 내 서버→외부서버로 보내는 요청)  
  - Client: 20 Port에 대한 인바운드 허용(InBound, 외부 서버→내서버로 들어오는 요청)

## Passive 모드
- Active 모드의 단점을 해결하기 위해 등장
- 아래 사용된 명령(Command)포트와 Data 포트는 **서버의 설정에서 임의 수정하여 사용 가능**
   - Data Port 번호를 지정하지 않은 경우 1024~65535 Port 중에서 사용 가능한 임의의 포트를 사용함
   - 포트의 번호를 지정할 때는 10001 ~ 10005와 같이 범위 지정도 가능
![ftp_passive.PNG](/assets\img/ftp_passive.PNG)
### 동작방식
1. Client가 Command Port(21)로 접속을 시도(Passvie Mode로 연결)
2. Server에서는 **사용할 두번째 포트(1234 Port)를 Client에게 알려줌**
3. Client는 다른 Port(5151)를 열어 서버가 알려준 포트(1234)로 접속 시도
4. Server가 Client의 요청에 응답(acks)

Active Mode와의 차이점 - 데이터채널 연결 요청 시 Active Mode에서 사용했던 20번 port가 아닌 1024번 이후의 임의의포트를 데이터 채널 포트로 사용됨(여기서는 5151 Port로 사용)

### 주의사항  
- Server측의 Data 채널 포트가 막혀있는 경우 데이터 채널 연결 불가
  - 데이터 채널 포트의 범위는 지정할 수 있고, 별도로 지정하지 않은 경우 1024~65535번의 포트를 사용하게 됨
- Server에 INBOUND 모두 허용하는 **방화벽 설정 필요**
  - Server 측에서 Data 채널 Port 범위를 지정하여 특정 범위의 Port만 허용해주면, 모든 포트를 허용해야하는 문제를 어느정도 해결할 수 있음

### 참고
- [응답 문자](https://ko.wikipedia.org/wiki/%EC%9D%91%EB%8B%B5_%EB%AC%B8%EC%9E%90)
- [[IT 기초 지식] FTP(File Transfer Protocol) 정리](https://milhouse93.tistory.com/168)
- [FTP Active와 Passive 차이](https://madplay.github.io/post/ftp-active-passive)