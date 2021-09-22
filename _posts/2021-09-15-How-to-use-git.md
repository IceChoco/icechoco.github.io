---
title: 'Git 사용법'
layout: post
categories: git
tags: Git Web Jekyll
comments: true
---
안녕하세요. 아이스초코입니다. 오늘은 Github 블로그 만들기에 필요한 Git 사용법에 대해 소개해드리겠습니다.
  
git이 이미 설치되어있는지 확인하기 위해서는 터미널에서 `git --version`을 치면됩니다. 버전이 나오는 경우 이미 설치되어있는 것이고, 안나오는 경우 설치되어 있지 않은 것 입니다.

**용어**  
-------------
Working Directory: 로컬PC에서 작업한 그대로의 상태  
Staging Area: git add 후 커밋 대기중인 상태  
git local repository: 내 로컬 PC에 위치한 repository  
git remote repository: github에 위치한 repository  
SSH: Secure Shell
* * *  

**Git과 GitHub의 차이**  
-------------
Git: 소스코드를 관리할 수 있는 툴  
GitHub: Git으로 관리하고 있는 소스들을 올릴 수 있는 클라우드 서비스

**Git 파일 추적(Track)과 커밋(Commit)**  
-------------  
Git에서 파일 버전을 관리하기 위해서는 해당 파일을 <span style="color:red">**추적(Track)**</span>해야 합니다.  
Git에서 추적한다는 의미는 그 파일을 관리한다는 의미입니다.  
* Track한다 = staging area에 올라갔다  

프로젝트 작업 폴더에는 Tracked 파일과 Untracked 파일이 존재합니다.  
1. Tracked(관리대상이 된 파일, 추적되고 있는 상태)  
  - Unmodified: 파일 내용 수정이 안된 상태  
  - Modified : 파일 내용이 수정된 상태  
  - Staged : 커밋으로 Git 저장소에 기록될 준비가 된 상태  
2. Untracked(관리대상이 아닌 파일, 즉 추적이 안 된 상태)  
3. Commit(커밋): 파일을 Git 저장소에 기록하는 것, 어떤 작업인지에 대한 메시지를 기록  

<!--author-->