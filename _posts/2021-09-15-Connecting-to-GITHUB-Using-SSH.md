---
title: 'SSH를 이용해 GITHUB 연결하기'
layout: post
categories: git
tags: Git Web Jekyll
comments: true
---

지금까지 github에서 소스를 클로닝(Cloning) 할 때, HTTP 연결을 통해 하였다. Cloning 하는데에 별문제 없이 잘 처리되었지만, 비번을 바꾸거나 할 때 처럼 인증 정보가 변경 되면, 재로그인을 하거나 Windows 자격 증명에서 Github 자격 증명을 갱신해주어야 한다. 비밀번호가 변경되더라도 인증정보를 갱신해야하는 번거로움 없는 SSH키를 이용한 Github와 연동하기를 시도해보았다.

## SSH KEY 생성하기  
1. CMD창을 연다
2. 아래 텍스트에서 이메일 주소를 바꾼 뒤 붙여넣기 한다
    ```
    ssh-keygen -t ed25519 -C "dkfk2685@naver.com"
    ```
3. `Enter file in which to save the key (경로):`라는 메시지가 뜬다. 확인 후 엔터를 쳐주고 다음 메시지가 나와도 계속 엔터를 쳐주면 key 생성이 완료된다.
   ![generate-new-ssh-key](/assets\img/generate-new-ssh-key.png)
4. 자신의 홈 디렉토리 아래 .ssh 폴더 내 id_ed25519, id_ed25519.pub 2개의 파일이 생성되어 있다.  
  - 홈디렉토리 예시 C:\Users\dkfk2\.ssh

## 나의 SSH Key를 SSSH Agent에 추가하기
1. git bash에 아래 텍스트를 붙여넣기하여 백그라운드에서 ssh-agent를 실행한다.
    ```
    eval "$(ssh-agent -s)"
    ```
2. SSH Private Key를 SSH-AGENT에 넣는다.  
    아까 생성된 Key 2개 중 id_ed25519는 Private Key, id_ed25519.pub는 public Key 이다.
    ```
    ssh-add ~/.ssh/id_ed25519
    ```
    ![ssh-agent-add](/assets\img/ssh-agent-add.png)

## public SSH Key를 gitHub에 추가하기
1. git bash에 아래 텍스트를 붙여넣기하여 public SSH Key를 클립보드로 복사한다.
    ``` 
    clip < ~/.ssh/id_ed25519.pub
    ```
2. github에 들어가서 우측 상단 계정 버튼 클릭 - Settings 클릭한다.
3. 좌측 목록에서 SSH and SPG Keys를 클릭한다.
4. 초록색 버튼의 New SSH Key를 클릭한다.
5. key 영역에 클립보드에 복사된 SSH public key를 붙여넣기한다.
6. 자신이 원하는 title을 입력하고 Add SSH Key를 클릭한다.  

SSH Key가 정상적으로 등록되어 컴퓨터와 gitHub 서버가 안전하게 통신할 수 있게 되었다.

## 로컬에서 remote repository로 push하기
  로컬 터미널에서 아래 명령어를 순차적으로 입력하면 to-do-list remote 레파지토리에 push가 정상적으로 완료된다.  
  ``` 
  echo "# to-do-list" >> README.md
  git remote add origin https://github.com/IceChoco/to-do-list.git  
  git branch -M main  
  git push -u origin main
  ```

## 추천문서
- [Generating a new SSH key and adding it to the ssh-agent - GitHub Docs](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) 
- [GitHub 접속 용 SSH 키 만드는 방법 - LainyZine](https://www.lainyzine.com/ko/article/creating-ssh-key-for-github/#%EA%B3%B5%EA%B0%9C%ED%82%A4%EB%A5%BC-github-%EA%B3%84%EC%A0%95%EC%97%90-%EB%93%B1%EB%A1%9D%ED%95%98%EA%B8%B0)
- [SSH 키로 깃헙에 연동하기 (Connecting to GitHub with SSH)- Game Programmer Life](https://kindtis.tistory.com/606)

<!--author-->
