---
layout: post
title: "[Books] Angular Essentials"
date: 2020-05-04 1:00:00
author: Jewoo.Song
categories: Books
tags:
  - Angular Essentials
  - Angular Essentials 서평
---

![Alt angular essential](/assets/img/books/angularessential.png)

이 책을 선택한 것은 우선 Angular 관련 서적이 많이 없다.

19년 4월에 나온 책이 있긴 한데, Angular에만 집중된 책은 아니다. 그나마 최신 책이라서 선택했고 이웅모님은 이 책을 보기 전에도 알고 있었다.

https://poiemaweb.com/ 이라는 웹사이트를 운영중이신데 javascript 관련 포스트를 우연히보고 설명이 굉장히 좋아서 관련 post들을 다 본적이 있었다.

그래서 믿고 보기 시작했는데 보길 잘한 것 같다.

일단 Angular를 보기 전에 javascript, typescript에 대한 챕터가 있는데 javascript에 대해서는 처음 시작하는 사람이 다른 책을 보지 않고 시작할 수 있을 만큼 충분한 내용이 담겨있다.

typescript는 다소 설명이 간결하긴 한데 책을 이어서 보는데 무리 없을 정도의 내용은 담겨있다.

그리고 Angular에 대한 설명은 공식 가이드 문서 - Fundamentals에서 다루는 내용들 중 Angular를 이해하는데 기본적으로 꼭 필요하다고 생각이 드는 내용들은 거의 포함되어 있는 것 같다.

> Animations, NgZone에 대한 설명은 포함되지 않았다.

<br/>

좋았던 점은 Angular의 주요 내용들에 대해서 설명할 때 각각 항목 무엇인지 정의를 해주는 것이 좋았다.

디렉티브를 예를 들어보면 공식 문서에서는 디렉티브를 다음과 같이 설명한다.(설명 초반부, 이후에는 사용법이 나온다.)

```
Angular templates are dynamic. When Angular renders them, it transforms the DOM according to the instructions given by directives. A directive is a class with a @Directive() decorator.

A component is technically a directive. However, components are so distinctive and central to Angular applications that Angular defines the @Component() decorator, which extends the @Directive() decorator with template-oriented features.
```

<br/>

책에서는 디렉티브에 대해 다음과 같이 설명한다.

```
디렉티브는 DOM의 모든 것(모양이나 동작 등)을 관리하기 위한 지시이다. HTML 요소 또는 어트리뷰트의 형태로 사용하여 디렉티브가 사용된 요소에게 무언가를 하라는 지시(directive)를 전달한다.

디렉티브는 애플리케이션 전역에서 사용할 수 있는 공통 관심사를 컴포넌트에서 분리하여 구현한 것으로 컴포넌트의 복잡도를 낮추고 가독성을 향상시킨다. 컴포넌트 또한 뷰를 생성하고 이벤트를 처리하는 등 DOM을 관리하기 때문에 큰 의미에서 디렉티브로 볼 수 있다.

디렉티브는 보편적이고 애플리케이션 전역에서 공통으로 사용 가능한 고유의 관심사를 기능으로 구현한다.
즉, 컴포넌트는 뷰 단위의 관심사를 가지고 있다면 디렉티브는 DOM 요소의 공통 기능에 관심을 갖는다.

단일 책임 원칙에 의해 복합적인 기능보다는 여러 요소에서 공통적, 반복적으로 사용될 하나의 기능을 명확히 구현하는 것이 바람직하다.

디렉티브는 뷰를 가지고 있지 않기 때문에 자식을 가질 수 없다.
하지만 컴포넌트(큰 의미에서는 디렉티브)는 뷰를 가지고 다른 컴포넌트를 자식으로 가질 수 있다.
```

> 책을 보고 정리해둔 것에서 인용한 것이라서 정확히 문구가 일치하지 않을 수도 있습니다.

<br/>

그 외에도 무언가에 대한 설명이 나올 때 내부 동작이나 원리, 어떻게 매핑되는지에 대한 설명도 포함되어 있는 경우가 많아서 이해하기 좋았다.

예를 들면 Form에 대한 챕터에서도 템플릿 기반 폼과 리액티브 폼의 차이에 대해 설명할 때 NgModel이 내부적으로 어떻게 매핑되는지나 FormGroup, FormControl이 어떤 역할을 하는지 개념적으로 잘 설명되어 있다.

예제도 잘되어 있고 전반적으로 저자분이 굉장히 내용을 잘 전달해 주시려고 노력하시고 쓰셨겠구나 싶었다.

아쉬운 점을 굳이 꼽자면 일단 책 출간 당시에는 최신이었겠지만 지금 Angular9가 정식 release된 시점에서 Angular 6 기준의 책이라는 것인데,

이미 2년 가까이 지난 책을 봐도 될까? 싶을 수 있겠지만 지금 봐도 충분히 좋다고 생각한다.
