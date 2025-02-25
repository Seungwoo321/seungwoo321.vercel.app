---
title: 브라우저 렌더링 과정
date: '2022-01-24'
tags: ["Frontend", "JavaScript"]
categories: Web
permalink: /blog/:year/:month/:day/:title/
---

브라우저 렌더링 과정은 크게 다음 4단계로 설명 할 수 있습니다.

<!--more-->

## 1. 파싱

- 렌더링 엔진은 HTML 과 CSS 을 파싱해서 DOM 및 CSSOM 트리를 생성합니다.
  - 바이트를 문자열로 변환하고, 토큰을 식별 한 후 노드로 변환하고 DOM 트리를 빌드 합니다. (바이트 → 문자 → 토큰 → 노드 → 객체 모델)
  - DOM 및 CSSOM 은 서로 독립적인 데이터 구조 입니다.

## 2. 렌더 트리

- DOM 및 CSSOM 트리를 결합해서 렌더 트리를 구성합니다.
  - 스크립트, 메타 태그, display: none 속성을 설정한 노드 등은 렌더 트리에 포함되지 않습니다.
  - 렌더 트리는 각 렌더 객체의 콘텐츠 및 스타일 정보를 모두 포함합니다.
  - 웹킷은 render object 로 구성되어 있는 render tree 라는 용어를 사용하고, 모질라 파이어폭스의 게코는 frame tree 라고 부르고 각 요소를 frame 이라고 합니다.

## 3. 배치 (Layout)

- 뷰포트 내에서 요소의 정확한 위치와 크기를 찾아내 기하학적 형태를 계산하여 레이아웃 트리를 만듭니다.
  - display: none 속성이 적용된 요소는 포함되지 않으나, visibility: hidden 속성이 적용된 요소는 포함됩니다.
  - 이와 비슷하게 ::before 와 같은 의사 클래스(pseudo class)의 콘텐츠는 DOM에는 포함되지 않지만 레리아웃 트리에는 포함됩니다.
  - 게코는 리플로우 (reflow) 라고 부릅니다.

## 4. 페인트 (Paint)

- 각 노드를 화면의 실제 픽셀로 변환하여 렌더링 하는 과정입니다.

> DOM 또는 CSSOM이 수정되어 화면에 다시 렌더링 할 필요가 있는 픽셀을 파악하려면 이 프로세스를 다시 반복해야합니다. 위 단계를 수행할 때 걸리는 총 시간을 최소화하면 가능한 한 빨리 화면에 렌더링 할 수 있으며, 초기 렌더링 후 화면 업데이트 사이의 시간을 줄여 줍니다.

<img src="/assets/images/posts/2022/01/24/high-level-flow.png" alt="high-level-flow" />

<small>
    (이미지 출처: ryanseddon 슬라이드)
</small>

---

## Reference

- [https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction?hl=ko](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction?hl=ko)
- [https://ryanseddon.com/browser/jsconfeu15/](https://ryanseddon.com/browser/jsconfeu15/)
- [https://speakerdeck.com/ryanseddon/how-the-browser-actually-renders-a-website](https://speakerdeck.com/ryanseddon/how-the-browser-actually-renders-a-website)
- [브라우저는 어떻게 동작하는가?](https://d2.naver.com/helloworld/59361)
- [최신 브라우저의 내부 살펴보기 3 - 렌더러 프로세스의 내부 동작](https://d2.naver.com/helloworld/5237120)
