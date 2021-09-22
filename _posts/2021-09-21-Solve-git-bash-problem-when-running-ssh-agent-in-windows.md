---
layout: post
title: 'window 환경에서 ssh-agent 실행 시 git bash 무반응 문제'
categories: git
tags: Git ssh-agent git-bash
comments: true
related_posts:
 - _posts/2021-09-15-SSH를-이용해-GITHUB-연결하기.md
---

오늘의 삽질 하나 추가

## 문제
SSH Agent를 Background에 실행시키기위해 아래 명령어를 입력하였다.  
```
eval "$(ssh-agent -s)"
```
그런데 명렁어를 입력한 다음 줄에 커서가 깜빡이면서 아무런 반응이 일어나지 않았다.

## 시도해본 것들
### 1. 창 종료 후 재시도
 Git Bash 창 강제 종료 후 새로운 Git Bash 관리자 권한으로 실행 후 다시 입력해보고 계속 반복하였으나 현상은 똑같았다.

### 2. 구글링
그래서 구글에 Gib Bash 멈춤, 무반응, 커서 깜빡임 등 다양하게 쳐보면서 구글링을 계속 이어갔지만 마땅한 해결책을 찾을 수 없었다. 흐아 너무 스트레스,,,,  

그러던 와중에 갑자기 컴퓨터가 너무 느려지다 못해 구글이 강제로 꺼지는 등 이상한 현상을 발견했다. 작업관리자를 켜봤더니 백그라운드에서 ssh-agent 여러개가 이미 실행되고 있었고 CPU가 99%와 100%를 왔다갔다했다ㄷㄷ
  
### 3. ssh-add 하기
이상함을 감지하고 명령어는 정상 실행됐으나 Git bash가 결과를 출력하지 못하는구나 라는 생각이 들면서 다음 스텝인 ssh-add를 실행했다.
```
ssh-add ~/.ssh/id_ed25519
```
위 명령어를 사용하던 중에 아래 오류를 만났다.  
`Could not open a connection to your authentication agent.`

위 메시지의 발생 원인은 어떤 `ssh-agent`를 사용해야 할 지 모르기 때문이다. 머신에서 한 번도 설정하지 않은 경우에 발생할 수 있다. 그럼 SSH Agent가 Background에서 정상 실행되지 않은거네,,,,? 그럼 이건 패스

## 해결책
진짜 얻어걸린 해결책,,,

백그라운드에서 ssh-agent가 1개도 수행되고 있지 않은 클린한 상태에서
1. Git Bash에 `eval "$(ssh-agent -s)"` 입력하기
2. 작업관리자에서 실행중인 ssh-agent 우클릭 후 작업 끝내기
3. Git Bash로 돌아가면 무반응상태에서 나의 ssh-agent pid가 출력된다.  
   ![ssh-agent-pid](/assets\img/ssh-agent-pid.png)

^^,,,, 이것 때문에 내 소중한 시간을 펑펑썼다니,,, 행복하다^^,,,, 

## 참고
- [꿀벌개발일지::ssh-add 에서 authentication agent 문제](https://ohgyun.com/483) 

<!-- author -->