---
title: 'React 스크립트 실행 오류'
layout: post
categories: frontEnd
tags: Web react
comments: true
---
```
create-react-app : 이 시스템에서 스크립트를 실행할 수 없으므로 ~\AppData\Roaming\npm\creat
e-react-app.ps1 파일을 로드할 수 없습니다. 자세한 내용은 about_Execution_Policies(https://go.microsoft.
com/fwlink/?LinkID=135170)를 참조하십시오.
위치 줄:1 문자:1
+ create-react-app my-app
+ ~~~~~~~~~~~~~~~~
    + CategoryInfo          : 보안 오류: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

1) windows PowerShell 프로그램을 관리자 권한으로 실행합니다.  
2) 아래와 같은 명령어를 작성하면 컴퓨터의 실행정책이 어떻게 설정되어 있는지 확인할 수 있습니다.  
    ```
    Get-ExecutionPolicy
    ```  

   별다른 설정 변경을 하지 않았다면, Windows 10 PC에서는 Restricted로 보입니다. 저도 Get-ExecutionPolicy에 아무런 argument를 입력하지 않았기 떄문에 아래처럼 기본값이 보이고 있습니다.

   ![script-excution-error-now](/assets\img/script-excution-error-now.png)

3) 권한이 RemoteSigned 가 아니라면 Set-ExecutionPolicy RemoteSigned 를 입력합니다.  
    ```
    Set-ExecutionPolicy RemoteSigned
    ```
    ![script-excution-error-now2](/assets\img/script-excution-error-now2.png)
  - **remoteSigned란?**  
  이 값은 최신 Windows Server 버전(Windows Server 2012 R2 이후)의 Powershell 실행정책 기본값입니다.  
  해당 로컬 컴퓨터에서 에서 작성된 모든 스크립트는 실행이 가능하며, 인터넷에서 다운로드(IE, 크롬, 파이어폭스, 아웃룩 등)한 스크립트는 인증기관이 발행한 코드로 서명되어야만 실행이 가능합니다. 인터넷 이외의 소스로부터 다운로드 받거나 서명은 되었지만 악의적인 목적이 있는 스크립트는 위험이 있을 수도 있습니다.  
  Microsoft Windows PowerShell 팀에서 권장할 만큼 가장 많이 설정되는 값이며, 보안과 편리함의 균형을 어느정도 확보할 수 있습니다. 하지만 스크립트가 **반드시 실행되어야 하는 컴퓨터에서만 사용하는 것이 바람직**합니다.

<!--author-->