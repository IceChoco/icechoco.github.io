---
title: '[jekyll] jekyll 실행 시 cannot load such file -- webrick (LoadError) 에러 조치 방법'
layout: post
categories: frontEnd
tags: Web jekyll
comments: true
---

데스크탑 컴퓨터를 최근에 포맷 후 개발환경 셋팅을 다시 하는 중이다. 포맷 이후 노트북에서만 작업하다가 데탑에서 블로그 포스팅 작업을 하려고 `bundle exec jekyll serve`로
로컬에서 jekyll 서버를 구동하려고했다. 근데 아래와 같이 webrick을 로드하지 못했다는 오류가 발생했다.

![jekyll-webrick-error](/assets\img/jekyll-webrick-error.png)

이런 경우 `bundle add webrick`으로 webrick을 추가해주고 다시 실행하면 된다.  
이유는 ruby 3.0.0부터 webrick이 기본으로 포함된 gem에서 빠졌기 때문이다.

### 참고
- [jekyll 실행 시킬 때 `require': cannot load such file -- webrick (LoadError) 오류가 난다면 bundle add webrick](https://junho85.pe.kr/1850)
- [Jekyll serve fails on Ruby 3.0 #8523](github.com/jekyll/jekyll/issues/8523)
- [Ruby 3.0.0 Released](https://www.ruby-lang.org/en/news/2020/12/25/ruby-3-0-0-released/)