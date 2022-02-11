---
layout: post
title: "[Typescript] plotta.js Typescript 마이그레이션하기 2, 3 - noImplicitAny 옵션 설정 및 리팩터링"
date: 2022-02-12 21:00:00
author: Jewoo.Song
categories: Typescript
tags:
  - Typescript
  - Typescript Migration
  - Typescript 마이그레이션
  - noImplicitAny
  - plotta.js
  - javascript 오픈소스
  - Typescript 오픈소스
---

타입스크립트 1단계로 단순 마이그레이션은 끝냈고, 다음 단계는 noImplicitAny 설정을 키고 any를 제거하고 타입들을 정리하는 단계입니다.

> noImplicitAny : 암묵적으로 `any` 로 추론되는 표현식과 선언에 오류를 발생시킨다.

1. 단순 마이그레이션
2. **noImplicitAny 설정**
3. **리팩터링**

### 2. **noImplicitAny 타입 에러** 수정

noImplicitAny 타입 에러 수정을 위해 기존에 any 타입으로 지정되어 있던 혹은 지정되어 있지 않던 타입들을 모두 `interface` 또는 `type` 으로 정의해 주었습니다.

이 과정에서는 최대한 리팩터링을 배제하고 실제 사용되는 property들을 타입으로 정리하여 모듈 구조상 최하단부터 오류들을 수정했습니다.

### 3. 리팩터링

noImplicitAny 타입 에러를 수정하는 단계에서 몇몇 리팩터링이 필요한 요소들을 발견했습니다.

#### 입력 데이터와 모델 통합

기존에는 입력 데이터와 모델이 미묘하게 다른 부분이 있었습니다.

입력받을 때에는 config 하위에 axis, border, tics... 단위로 묶어서 받고, 사용할 때에는 사용하기 좋은 형태로 모델을 정의되어 있었는데 타입 스크립트로 마이그레이션 하면서 모델과 입력 데이터의 타입을 통합하고, 외부에서 참조 가능하도록 export 하였습니다.

**입력 데이터**

```json
{
  "axis": {
    "x": {...},
    "y": {...}
  }
}
```

**모델**

```json
{
  "axisX": {...},
  "axisY": {...}
}
```

#### ViewModel 구조 정리

기존에는 viewModel 내에 실제 데이터를 나타내는 drawData가 있고 viewModel은 drawData를 업데이트하는 메서드를 포함하고 있었고, viewModelHelper는 viewModel 메서드 내부에서 사용하는 일종의 private 메서드들이 포함되어 있었습니다.

이 구조가 직관적이지 않고 viewModel과 viewModelHelper의 역할과 분명하게 분리되지 않았습니다. 그래서 viewModel은 interface로 정의하여 명확하게 뷰를 나타내는 모델로 정의하고, viewModelHelper는 모델에 연산을 하여 뷰모델을 업데이트하는 역할을 하도록 역할을 분리하였습니다.

#### 불필요하게 class interface로 전환

기존에 GraphConfig, Axis, Tics 등은 각각 0~2개 정도의 메서드를 가진 클래스 타입이었습니다.

대부분의 메서드가 모델을 업데이트하는 메서드였고 오히려 복잡도가 올라가게 되어서 interface로 변경하고 그에 맞춰서 graphModel 내에서 사용하는 쪽을 수정하였습니다.

#### To do

- Draw, View 추상화
  - svg, canvas 등 렌더링 메서드 확장 가능하도록
- 그래프 타입 확장을 위한 Draw 내부 타입 추가 및 렌더링 추상화
  - Bar, Dot, Line 등 그래프 타입 확장 가능하도록
- TSDoc 정리

### 정리

- 생각보다 타입스크립트 마이그레이션이 수월했다면 noImplicitAny 옵션이 켜져 있는지 확인하자.
- noImplicitAny 설정이 없다면 타입 선언과 관련된 실제 오류가 드러나지 않는다.
- [작업한 PR](https://github.com/iamsjy17/Plotta.js/pull/37)
