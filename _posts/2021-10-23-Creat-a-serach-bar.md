---
title: '[Jekyll] SimpleJekyllSearch를 사용하여 서치바(Search Bar) 구현하기'
layout: post
categories: frontEnd
tags: Blog jekyll Web Markdown HTML
comments: true
---

**이 글의 목적**
- Hydejack 테마를 사용한 블로그 환경에서 Jekyll SimpleJekyllSearch를 사용하여 서치바(Search Bar)를 구현할 수 있다.

## 개요
깃헙 블로그를 시작하기 전에는 엑셀을 통해 개발공부와 관련된 내용들을 기록했었다. 그래서 엑셀을 사용할 때는 Ctrl + F하면 간단하게 찾기를 할 수 있으니 그렇게 썼었다. 그런데 데스크탑, 노트북, 본가 PC를 번갈아서 쓰면서 컴퓨터를 바꿀때마다 가장 최신버전의 엑셀을 옮겨줘야 하는 작업이 불편해졌고, 요즘 개발자들은 GitHub Pages를 통한 개인 블로그 또는 다른 사이트 운영을 통해 자신의 포트폴리오를 생성하는 것은 기본이라는 것을 알게 되었다. 그래서 나는 올 하반기부터 GitHub 블로그를 시작했고 점점 익숙해지고있다!(익숙해진거 맞지?ㅎ)  

무튼... 본론으로 돌아가자면 GitHub 블로그를 시작한지 이제 2달이 막 지났는데 포스트의 양이 늘어나면서 내가 적었던 글을 주변 개발자 친구들한테 공유해줄 때 글 찾기가 힘들어서 매번 "아 이거썼었는데;; 잠깐만 기다려봐!"하고 글을 계속 찾는 내 자신을 발견했다. 아니 서치바 없는 블로그라니 이게 무슨 블로그야! 서치바를 추가해야겠어라고 다짐하고 본격적인 서치바 생성에 들어갔다.
  
회사 선배와 개발관련 얘기를 나누다 글을 계속 찾는 나를 보고 SearchBar 추가에 좋은 라이브러리인 [SimpleJekyllSearch](https://github.com/christian-fei/Simple-Jekyll-Search)를 추천해주셨다. 정말 너무 좋은 선배님이다^_^ 설치는 라이브러리 제공 사이트 내 [가이드](https://github.com/christian-fei/Simple-Jekyll-Search) 파일과 [Jekyll에 검색 페이지 추가하기](https://jamesu.dev/posts/2021/01/03/adding-search-page-on-jekyll/) 글을 참조하면서 진행했다.

## SimpleJekyllSearch 연동하기
**1. 연동에 필요한 파일은 총 2개**  
SimpleJekyllSearch를 블로그에 연동하기 위해 가장 중요한 파일은 아래 2개 파일이다.
- `simple-jekyll-search.min.js` 또는 `simple-jekyll-search.js` [링크](https://github.com/christian-fei/Simple-Jekyll-Search/tree/master/example/js)
- `search.json` [링크](https://github.com/christian-fei/Simple-Jekyll-Search/blob/master/example/search.json)

첫번째 파일인 `simple-jekyll-search.min.js` 파일은 검색 데이터로부터 **검색 기능**을 사용하기 위한 라이브러리 파일이고, 두번째 파일인 `search.json` 파일은 검색 데이터를 **준비**하기 위한 파일이다.
그렇다면 첫번째 파일인 `*.min.js` 파일과 `*.js` 파일의 차이점이 뭘까? `*.min.js`가 `*.js` 파일의 압축버전이라고 생각하면 된다.  

두 파일의 생성 위치는 개발자 나름인데, 사용하는 블로그 테마에 따라서 어느정도의 폴더 역할에 따라 위치는 지켜야 하므로 자신이 생각하는 적당한 곳에 생성하면된다. 나같은 경우는 hydejack 테마를 사용하고 있으며 두 파일을 아래의 경로에 생성했다.
- `simple-jekyll-search.min.js`: `assets/js`
- `search.json`:`assets/json` 

**2. 검색 데이터 준비를 위한 json 파일 만들기**  
Jekyll로 만든 블로그는 별도의 서버를 갖고 있지 않은 정적 사이트이기 때문에 검색을 하기 위해서는 미리 생성된 데이터가 필요하다. 이 역할을 수행하는 파일이 바로 `search.json` 파일이다.
~~~json
---
layout: none
---
[{% raw %}
{% for post in site.posts %}
   {
    "title"      : {{ post.title | jsonify }},
    "categories" : "{{ post.categories | join: ' > ' }}",
    "tags"       : "{{ post.tags | join: ', ' }}",
    "date"       : "{{ post.date | date: '%Y.%m.%d' }}",
    "urlString"  : "{{ post.url }}",
    "url"        : "{{ post.url | prepend: site.baseurl }}"
    }{% unless forloop.last %},{% endunless %}
{% endfor %}{% endraw %}
]`
~~~
위 파일을 보면 **Jekyll의 Liquid 문법으로 json 데이터를 생성**하고 있다. 뜻은 전체 포스트를 순회하면서 제목, 카테고리, 태그, url, 날짜 등을 객체로 구성하여 배열로 만든다. 파일 생성 후 데이터가 정상적으로 생성되었는지 확인하기 위해 search.json이 저장된 경로로 접속해보자. 예를 들어 나의 경우 `http://localhost:4000/assets/json/search.json`로 접속해서 데이터 정상 생성을 확인할 수 있었다.

**3.검색 레이아웃 만들기**  
나는 Nav Bar에 있는 Search버튼을 누르면 서치 홈페이지로 이동 후, 그 홈페이지 내에서 검색기능을 구현하는 페이지를 만들고 싶었다. 그래서 가장 먼저 검색 레이아웃을 만들었다. 경로는 `_layouts` 폴더 아래에 `search.html`이라는 이름으로 만들었다.
```html
<div class="common-header search">
  <div class="search">
    <i class="fas fa-search fa-fw"></i>
    <input id="search-input" 
      type="search" 
      tabindex="1" 
      spellcheck="false" 
      placeholder="Search"
    >
  </div>
</div>

<ul id="results-container"></ul>
```
기본적인 소스는 [라이브러리 예시](https://github.com/christian-fei/Simple-Jekyll-Search/blob/master/example/_layouts/default.html)에서 볼 수 있다. 하지만 진짜 못생김ㅎ  
![simple-jekyll-search-ex](/assets\img/simple-jekyll-search-ex.PNG)  
예시 소스의 기본 레이아웃이다. 입력하는 부분과 그 출력값을 나타내주는 두 부분이 큰 틀인데, 예시에서는 레이아웃이 꾸밈이 없이 최소한으로 되어있다. 그래서 나는 jamesu님의 레이아웃 디자인을 가져왔다.

**3.검색 레이아웃 만들기**  

### 참고
- [Jekyll에 검색 페이지 추가하기](https://jamesu.dev/posts/2021/01/03/adding-search-page-on-jekyll/)
- [SimpleJekyllSearch](https://github.com/christian-fei/Simple-Jekyll-Search)
- [[GithubPages] 하루만에 만드는 깃허브 블로그 05.게시물 검색기능 추가하기](https://khw11044.github.io/blog/githubpages/2020-12-26-making-blog-05/)