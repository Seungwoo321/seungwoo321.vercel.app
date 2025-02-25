---
title: 어떻게 const로 선언한 객체의 속성값이 바뀌는 것일까
date: '2022-01-31'
tags: ["Frontend", "JavaScript"]
categories: JavaScript
permalink: /blog/:year/:month/:day/:title/
---

const 키워드를 사용하여 선언된 변수는 블록 범위의 상수입니다. 상수의 값은 재할당할 수 없으며 다시 선언할 수도 없습니다. 그러나 상수가 객체 또는 배열인 경우 해당 속성이나 항목을 업데이트하거나 제거할 수 있습니다.

<!--more-->

## 단지 변수 식별자를 재할당할 수 없다는 의미

const 선언은 값에 대한 읽기 전용 참조를 만듭니다. 보유하고 있는 값이 변경 불가능하다는 의미는 아닙니다. 단지 변수 식별자를 재할당할 수 없다는 의미입니다. 예를 들어 콘텐츠가 객체인 경우 객체의 콘텐츠(예: 속성)가 변경될 수 있음을 의미합니다.

> 출처: [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const#description](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const#description)

## C를 생각해보면 배열도 포인터

C를 생각해 보면 배열도 포인터입니다. 상수 배열은 포인터 값이 변경되지 않는다는 것을 의미할 뿐 실제로 해당 주소에 포함된 데이터는 자유롭게 사용할 수 있습니다.

> 출처: [https://stackoverflow.com/a/23436796](https://stackoverflow.com/a/23436796)

## pass by reference vs pass by value

JavaScript에서는 숫자(Number)나 문자열(String)과 같은 원시 타입(primitive types)은 참조에 의한 전달(pass by reference)이 아니라 값에 의한 전달(pass by value)이기 때문에 Vue 3.0에서는 어디서나 변수를 반응성으로 만들 수 있도록 객체 안에 값을 감싸는 ref 함수를 제공하고 있습니다.

<img src="/assets/images/posts/2022/01/31/pass-by-reference-vs-pass-by-value-animation.gif" alt="pass-by-reference-vs-pass-by-value"/>

> 출처: [https://v3.ko.vuejs.org/guide/composition-api-introduction.html#ref가-있는-반응성-변수](https://v3.ko.vuejs.org/guide/composition-api-introduction.html#ref%E1%84%80%E1%85%A1-%E1%84%8B%E1%85%B5%E1%86%BB%E1%84%82%E1%85%B3%E1%86%AB-%E1%84%87%E1%85%A1%E1%86%AB%E1%84%8B%E1%85%B3%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC-%E1%84%87%E1%85%A7%E1%86%AB%E1%84%89%E1%85%AE)

## 할 수 있는 것과 할 수 없는 것

**다음을 할 수 없습니다:**

- 상수 값 재할당
- 상수 배열 재할당
- 상수 객체 재할당

**하지만 다음은 할 수 있습니다:**

- 상수 배열 변경
- 상수 객체 변경

> 출처: [https://stackoverflow.com/a/68609276](https://stackoverflow.com/a/68609276)

## 정리

숫자나 문자열과 같은 원시 타입은 메모리상에 저장된 실제 값이 변수에 전달이 되는 것이라서 값을 변경할 수 없고 객체나 배열인 경우에는 참조에 의한 전달로 주솟값이 전달되는 것이라서 변수에 할당된 주솟값을 변경할 수는 없지만 메모리에 저장된 객체의 속성이나 배열의 항목은 변경할 수 있는 것입니다.
