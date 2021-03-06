---
title: '[CS] CS 면접 준비 관련 사이트 모음'
layout: post
categories: cs
tags: cs
comments: true
---

**# CS 면접 준비 관련 깃허브**

[백엔드 개발자로 입사를 준비하며 받았던 질문, 예상했던 질문, 인터넷 참고한 질문(CC BY-NC)](https://github.com/ksundong/backend-interview-question)

[Interview Question for Beginner](https://github.com/JaeYeopHan/Interview_Question_for_Beginner)

[Next Tech Interview](https://www.interviewbit.com/)

[기술 면접 대비를 위한 기본 개념 정리](https://github.com/WeareSoft/tech-interview)

[ 신입 개발자 전공 지식 & 기술 면접 백과사전](https://github.com/gyoogle/tech-interview-for-developer)

**# 면접 후기**

[[2020년 하반기 면접 후기] 네이버 공채 면접 후기 ! (1차/2차 면접)](https://frogand.tistory.com/70)

## 면접 질문
**Q. 재귀 호출을 사용할 떄 생길 수 있는 단점은 무엇인가?**  
A. 재귀 호출을 하려면 함수를 반복해서 호출해야 하는데, 매번 호출할떄마다 시간과 스택 공간에서 오버헤드가 발생합니다. 재귀 호출을 헷갈리는 사람이 많기 떄문에 재귀 호출 함수는 문서화나 디버깅, 유지보수가 어려울 수 있습니다.

**Q. 모바일 기기용 프로그래밍과 컴퓨터용 프로그래밍은 어떻게 다른가요?**
A. 모바일 프로그래밍은 일반 프로그래밍과 여러 면에서 다릅니다. 모바일 기기는 보통 안드로이드나 IOS처럼 모바일용으로 특화된 운영체제를 사용합니다.  
이런 운영체제는 Desktop이나 Server 운영체제와는 다른 파일 시스템 액세스, 메모리 액세스, 애플리케이션간 통신 패러다임 같은 특징이 있습니다.  
모바일 프로그래밍 에서는 전력 소모에 더 신경을 써야하고, 저장 공간이나 네트워크 대역폭에도 제약이 더 심한 편입니다.  
네트워크 연결이 중간중간 끊어질수도 있고 대역폭도 크게 달라질 수 있습니다. 대부분 모바일 기기에서는 터치스크린과 마이크를 주 입력 장치로 사용하기 떄문에 작은 화면, 손가락으로 다루기 좋은 위젯, 제스처, 음성인식 사용을 최적화하고  
화면을 누르는 소프트 키보드로는 불편한 텍스트 입력을 최소화 하는 것이 좋은 사용자 인터페이스 디자인을 구성하는 방법입니다. 모바일 기기는 가속도계, GPS 위치 서비스, 알림, 연락처 DB 같은 자원의 가용성 면에서  
모바일이 아닌 기기에 비해 일관성이 높은 편이나, 이런 자원에 대한 접근은 접근 권한 모형에 의해 제한될 수도 있습니다. 애플리케이션 배포는 보통 온라인 애플리케이션 스토어를 통해서만 이루어지지만,  
저희의 경우 기업 관계자만 설치할 수 있도록 별도의 URL을 제공하여 어플리케이션을 설치 할 수 있는 APK를 제공하고 있습니다.