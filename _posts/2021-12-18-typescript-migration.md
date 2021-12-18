---
layout: post
title: "[Typescript] plotta.js Typescript 마이그레이션하기 1 - 단순 마이그레이션"
date: 2021-12-18 12:00:00
author: Jewoo.Song
categories: Typescript
tags:
  - Typescript
  - Typescript Migration
  - Typescript 마이그레이션
  - plotta.js
  - javascript 오픈소스
  - Typescript 오픈소스
---

이번에 Plotta.js를 타입스크립트로 마이그레이션 하는 작업은 몇 가지 단계를 두고 할 계획입니다.

1. 단순 마이그레이션
2. noImplicitAny 설정
3. 리팩터링

그중 단순 마이그레이션 과정을 정리한 글입니다.

점진적 마이그레이션을 할 때는 모듈 단위로 진행하는 것이 이상적입니다. 하나의 모듈에 타입 정보를 추가한다고 할 때, 해당 모듈뿐 아니라 해당 모듈이 임포트 하는 모듈에서 비롯되는 타입 오류 또한 발생합니다.

그래서 다른 모듈을 의존하지 않는(아무것도 임포트 하지 않는) 최하단 모듈부터 작업을 시작해야 합니다.

그래서 먼저 각 모듈 간 의존성을 시각화하는 것 부터 시작하겠습니다.

### 1. 의존성 시각화

의존성 시각화는 `[dependency-cruiser](https://www.npmjs.com/package/dependency-cruiser)`라는 라이브러리를 사용해서 진행했습니다.

#### 설치

```bash
npm i -g dependency-cruiser
```

#### 실행

```bash
depcruise --include-only "^src" --output-type dot src | dot -T svg > dependencygraph.svg
```

실행 시 오류가 발생한다면 [https://graphviz.gitlab.io/download/](https://graphviz.gitlab.io/download/) 를 먼저 설치해야 합니다.

[https://github.com/sverweij/dependency-cruiser/issues/385](https://github.com/sverweij/dependency-cruiser/issues/385)

```bash
brew install graphviz
```

#### 결과

![dependencygraph.svg](/assets/img/typescript/dependencygraph.svg)

결과가 이쁘게 잘 나왔습니다. 이 중 최하단 모듈 세 개부터 먼저 마이그레이션 해보도록 하겠습니다.

- util.js
- drawHelper.js
- platform.js

### 2. 타입스크립트 설치 및 환경세팅

#### 타입스크립트 설치 및 Webpack 세팅

먼저 타입스크립트를 설치하고 웹팩에 ts-loader를 추가합니다.

```bash
npm i --save-dev typescript ts-loader
```

**tsconfig.json**

```json
{
  "compileOnSave": false,
  "compilerOptions": {
    "outDir": "./dist/tsc",
    "noImplicitAny": false,
    "module": "es6",
    "target": "es5",
    "allowJs": true,
    "sourceRoot": "/src",
    "sourceMap": true,
    "moduleResolution": "node"
  }
}
```

**webpack.config.js**

```jsx
const config = {
  //...
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: "ts-loader",
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: [".tsx", ".ts", ".js"],
  },
  //...
};
```

#### ESLint, Prettier 세팅

기존에도 ESLint 설정이 되어있었는데 Typescript ESLint 관련 패키지를 추가하고 Prettier 관련 패키지도 추가했습니다.

혹시 참고하시는 분이 계시다면 Prettier나 ESLint 관련 설정은 선택적으로 설치해 주시면 될 것 같습니다!

```bash
# ESLint 관련 패키지
npm i --save-dev eslint eslint-plugin-import

# Airbnb 패키지
npm i --save-dev eslint-config-airbnb-base

---------------기존 세팅---------------

# Typescript ESLint 패키지
npm i --save-dev @typescript-eslint/eslint-plugin @typescript-eslint/parser

# Prettier 관련 패키지
npm i --save-dev prettier eslint-config-prettier eslint-plugin-prettier
```

**.eslintrc.json**

```json
{
  "env": {
    "browser": true,
    "amd": true,
    "node": true
  },
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "extends": [
    "airbnb-base",
    "plugin:prettier/recommended",
    "prettier/@typescript-eslint",
    "plugin:@typescript-eslint/recommended"
  ],
  "parserOptions": {
    "sourceType": "module",
    "project": "./tsconfig.json"
  },
  "settings": {
    "import/resolver": {
      "node": {
        "extensions": [".js", ".jsx", ".ts", ".tsx"]
      }
    }
  },
  "rules": {
    //...
  }
}
```

**.prettierrc.json**

```json
{
  "singleQuote": true,
  "jsxBracketSameLine": true
}
```

### 3. 마이그레이션하기

글 초반에 말했듯이 이번 타입스크립트 마이그레이션은 몇 가지 단계를 두고 진행하려 합니다.

첫 번째로는 noImplicitAny: false로 세팅한 후 최소한의 수정으로 단순 마이그레이션만 하는 것으로 진행하였습니다.

#### JSDoc

기존 프로젝트에서 사용하던 JSDoc 스타일 주석들 중 타입을 명시한 주석들은 필요가 없어젔습니다.

#### Property 선언

자바스크립트 클래스에서 할당된 프로퍼티들은 타입스크립트에서는 타입을 지정하여 선언을 해야 합니다.

**canvasHelper.js**

```tsx
class CanvasHelper {
  constructor(canvas, dpr) {
    this.presentationCanvas = canvas;
    this.backgroundCanvas = document.createElement("canvas");
    //...
  }
  //...
}
```

**canvasHelper.ts**

```tsx
class CanvasHelper {
  presentationCanvas: HTMLCanvasElement;
  backgroundCanvas: HTMLCanvasElement;
  //...

  constructor(canvas: HTMLCanvasElement, dpr: number) {
    this.presentationCanvas = canvas;
    this.backgroundCanvas = document.createElement("canvas");
    //...
  }
  //...
}
```

#### Bottom Up 마이그레이션

![dependencygraph.svg](/assets/img/typescript/dependencygraph.svg)

프로젝트 내 모든 파일을 마이그레이션 하지 않더라도 의존성에 따라 최하단 모듈부터 바텀업으로 진행하면 빌드가 정상적으로 됩니다. 그래서 운영 중인 프로젝트이고, 규모가 크다면 순차적으로 하는 것이 좋습니다.

지금 프로젝트는 그렇게 규모가 크지는 않지만 한 번에 날 잡고 진행하기가 어려워서,, 시간 날 때 조금씩 마이그레이션 하고 중간중간 실행되는지 빌드 확인하면서 진행하기 위해 바텁업으로 마이그레이션을 진행했습니다.

**마이그레이션 순서**

- 최하단 모듈
  - util.js/platform.js/drawHelper.js/viewModelHelper.js
- view
  - canvasHelper.js/osWorker.js/osCanvasHelper.js/canvasHelperFactory.js
  - graphCanvas.js/viewModel.js
  - graphView.js
- model
  - tics.js/axis.js
  - config.js/lineData.js
  - graphModel.js
- entry
  - plotta.js

몇 년 동안 방치했던 코드이다 보니 리팩터링하고 싶은 곳이 많이 보였지만 참고 any도 남발하면서 1단계 마이그레이션을 마첬는데요. 굉장히 불편하네요,,

작업했던 [PR](https://github.com/iamsjy17/Plotta.js/pull/33) 입니다.

다음은 noImplicitAny 옵션을 키고 any를 제외하고 타입들을 제외할 예정입니다!

### 참고

- [https://www.netlify.com/blog/2018/08/23/how-to-easily-visualize-a-projects-dependency-graph-with-dependency-cruiser/](https://www.netlify.com/blog/2018/08/23/how-to-easily-visualize-a-projects-dependency-graph-with-dependency-cruiser/)
- Effective Typescript - Item 62. 의존성 관계에 따라 모듈 단위로 전환하기
