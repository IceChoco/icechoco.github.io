---
title: 'Hydejack 테마 Custmomization'
layout: post
categories: frontEnd
tags: Web jekyll
comments: true
---

Hydejack 테마 Custmomization에 관한 내용입니다.

## 사용자 지정 CSS 추가
### above-the-fold contents?
아무런 행동을 하지 않은 기본 화면을 UX 업계에서는 **‘Above the Fold’**라고 지칭합니다.

지금 우리의 사회와 생활에 크게 밀접해 있는 웹이 사용자들에게 효율적으로 전달되기를 연구하는 것처럼, 과거 신문이 미디어 산업을 주도할 때 신문이 사용자들에게 어떻게 하면 관심을 끌고 잘 전달됨으로써 구매로 이어질 수 있을지에 대한 많은 에디터들의 고민이 있었습니다.

그때 가판대에 접혀져 놓인 신문이 소비자에게 제일 먼저 보이는 영역을 ‘Above the Fold’라고 일컬었고, 지금 **웹 사이트에서 스크롤로 가려지지 않은 채 사용자에게 제일 먼저 보이는 영역** 또한 ‘Above the Fold’라 불리고 있으며 반대 의미인 스크롤로 가려진 아래 영역은 ‘Below the Fold’라 합니다.

- 관련사이트 - [Above The Fold란?](https://www.beusable.net/blog/?p=1724)   

### CSS 추가 방법
Hydejack 테마에 CSS를 커스텀하는 가장 빠르고 안전한 방법은 _sass/my-inline.scss 파일과 _sass/my-style.scss 파일을 통한 방법입니다. 만약 해당 폴더 및 파일이 없다면 생성하시면 됩니다.  

above the fold 영역에 대한 css 규칙은 my-inline.scss에 기술합니다. 그 외 영역에 대한 css 규칙은 my-style.scss에 기술합니다.
단, 이 구분은 no_inline_css가 활성화된 경우 효과가 없습니다.

![custom-css-file-path](/assets\img/custom-css-file-path.png)

<!--author-->