---
layout: post
title: "[Angular] Eslint 설정하기 (vscode Prettier 연동)"
date: 2020-04-18 18:00:00
author: Jewoo.Song
categories: Angular
tags: eslint prettier angular typescript airbnb
---

Typescript 팀에서 공식적으로 더 이상 TSLint를 지원하지 않고, ESLint의 Typescript 지원을 개선하는데 집중할 것이라고 말했습니다.

> https://medium.com/palantir/tslint-in-2019-1a144c2317a9

현재 사용하는 데는 문제는 없지만 장기적으로 본다면 TSLint를 ESLint로 마이그레이션 하는 것이 좋습니다.
Angular + VsCode + ESLint + Prettier 세팅과 Airbnb 스타일 가이드 적용까지 해보도록 하겠습니다.

### 프로젝트 생성

먼저 적용할 프로젝트를 생성합니다.

```bash
ng new angular-skeleton
```

만들어진 프로젝트는 기본적으로 tslint가 적용되어 있습니다.

_아마 장기적으로는 Angular도 기본 구성자체를 eslint로 바꾸지 않을까 생각합니다._

### 설치

```bash
# ESLint 관련 패키지
npm install --save-dev eslint eslint-plugin-import

# Airbnb 패키지
npm install --save-dev eslint-config-airbnb-base

# Typescript ESLint 패키지
npm install --save-dev @typescript-eslint/eslint-plugin @typescript-eslint/parser

# Prettier 관련 패키지
npm install --save-dev prettier eslint-config-prettier eslint-plugin-prettier
```

- eslint: ESLint 라이브러리
- eslint-config-airbnb-base: airbnb style guide 적용
- eslint-plugin-import: ES2015 import/export문 관련 lint 룰 적용
- @typescript-eslint/eslint-plugin: Typescript에 관련된 ESLint 룰 추가
- @typescript-eslint/parser: typescript 파싱
- prettier: Prettier 라이브러리
- eslint-config-prettier: Prettier와 충돌되는 ESLint 규칙을 Disable
- eslint-plugin-prettier: Prettier를 ESLint 룰으로 적용

### 세팅

#### .eslintrc.json

eslint가 동작하도록 하기 위해서 프로젝트 루트에 .eslintrc.json을 추가합니다.
그리고 기존에 .tslintrc.json이 있다면 제거합니다.

Airbnb 규칙은 엄격하므로 사용하는 스타일에 따라서 룰을 수정하여 사용하시면 됩니다.

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
    "ecmaVersion": 2020,
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
    "import/extensions": "off",
    "import/prefer-default-export": "off",
    "class-methods-use-this": "off"
  }
}
```

#### vscode .setting.json

Save, Paste 등의 동작을 수행할 때 eslint + prettier를 자동으로 돌리도록 설정합니다.

```json
{
  "[typescript]": {
    "editor.defaultFormatter": "dbaeumer.vscode-eslint",
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
    },
    "editor.formatOnSave": true,
    "editor.formatOnPaste": true
  }
}
```

#### .prettierrc.json

여기까지의 설치/세팅은 prettier 룰을 ESLint 룰과 함께 사용하도록 합니다.

prettier 룰 중 ESLint 룰을 덮어썼을 때 문제가 될만한 것들은 수정해 줍니다.

```json
{
  "singleQuote": true,
  "jsxBracketSameLine": true
}
```

### 실행

#### script 세팅

lint 명령어를 통해서 eslint를 실행시킬 수 있도록 script를 추가해 줍니다.

```json
{
  "scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "test": "ng test",
    "lint": "eslint src/**/*.ts",
    "lint:fix": "eslint --fix src/**/*.ts"
  }
}
```

#### 실행

```bash
npm run lint

npm run lint:fix
```

## 참고

- https://dev.to/dreiv/using-eslint-and-prettier-with-vscode-in-an-angular-project-42ib
- https://pravusid.kr/javascript/2019/03/10/eslint-prettier.html
- https://ivvve.github.io/2019/10/09/js/ts/typescript-eslint&airbnb/
- https://medium.com/@pks2974/tslint-%EC%97%90%EC%84%9C-eslint-%EB%A1%9C-%EC%9D%B4%EC%82%AC%ED%95%98%EA%B8%B0-ecd460a1e599
