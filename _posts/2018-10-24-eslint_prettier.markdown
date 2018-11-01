---
layout: post
title: "[React] Eslint 설정하기 (vscode Prettier 연동)"
date: 2018-10-24 19:00:00
author: 송튜디오
categories: React
tags: eslint prettier .eslintignore eslint-disable NODE_PATH
---

안녕하세요. 송튜디오입니다.

혹시 eslint 설정하고 config 나 serviceWorker 와 같은 linting 이 불필요한 파일에서 에러가 나서 해결방법을 찾고 계신가요?
아니면 NODE_PATH 를 설정했는데 eslint 에서 인식을 못해서 고생하시나요?

저는 그랬습니다..

해결했다!! 하고 새 프로젝트 만들 때 또 찾아보고 몇 번 반복하다가 포스팅을 남깁니다.

### Eslint + prettier 설정

#### eslint 설치

우선 vscode 의 market place 에서 eslint 를 찾아서 설치합니다.

#### .eslintrc 파일 추가

eslint 를 적용하기 위해 프로젝트 루트 경로에 `.eslintrc.json` 파일을 추가합니다.
바로 빨갛게 오류가 잔뜩 생기는 것을 볼 수 있으실 거에요. 일단은 무시하고 넘어갑시다.

#### prettier 설치

다음으로 market place 에서 prettier 를 설치합니다.
이 부분은 사실 필수는 아니에요. 그런데 `prettier`는 코드 포맷팅을 도와주는 plugin 으로 설치하시면 매우 편리합니다!

#### eslint 와 prettier 연동

`prettier-eslint` 를 추가합니다.

```bash
yarn add --dev prettier-eslint
```

그리고 설정을 수정하여 저장할 때마다 자동으로 prettier 를 실행하도록 하고, 또 eslint 규칙에 따라서 formatting 하도록 설정합니다.

```json
{
  "editor.formatOnSave": true,
  "javascript.format.enable": false,
  "prettier.eslintIntegration": true
}
```

#### airbnb 추가

```
yarn add eslint-config-airbnb
```

airbnb 는 가장 유명한? javascript 코딩 규칙입니다. 많은 곳에서 사용되고 있어서 이 규칙에 익숙해 지시면 _올바른_(튀지않는) 코딩 스타일을 익힐 수 있습니다.

그런데 매우 매우 까다로워서 사용하시다가 이건 좀 심하다 싶으신 것들은 꺼주시면 됩니다^^

#### .eslintrc 파일 수정

그럼 .eslintrc 파일을 수정하여 airbnb 스타일을 사용하도록 합니다.
아래 rules 아래에 규칙들을 추가하여 on/off 할 수 있습니다.

```json
{
  "parser": "babel-eslint",
  "extends": "airbnb",
  "plugins": ["react", "jsx-a11y", "import"],
  "rules": {
    "react/jsx-filename-extension": 0
  },
  "env": {
    "browser": true,
    "node": true
  }
}
```

#### linting 에서 파일 제외하기

에러가 무지막지하게 뜨시나요..?
eslint 가 적용되지 않아도 되는 파일까지 포함되었다면 제외해줍니다.

```
/* eslint-disable */
```

linting 이 불필요한 파일들에 맨 위에 위와같이 주석을 추가해주면 제외됩니다.
제가 제대로 설정을 못한건지 .eslintignore 파일을 추가해서 .gitignore 파일 처럼 만들어 보았는데 적용이 안되더라고요..
해매다가 주석 한 줄 추가하니까 똭! 해결되었습니다.

#### NODE_PATH 인식

`../../../components/header`와 같이 상대경로의 depth 가 깊어지면 가독성도 좋지 않고 실수할 여지도 많아집니다. 그렇기 때문에 NODE_PATH 를 지정하여 `src` 경로로부터 절대경로로 사용을 합니다.

`components/header` 이런식으로 사용이 가능하니까 아주 깔끔해집니다.
_하는 방법은 리액트 시작하기 2 포스팅에 있습니다._

문제는 이렇게 경로를 지정했고! 잘 동작하는데! eslint 는 오류로 인식을 한다는 겁니다.

`.eslintrc`파일에 아래와 같이 추가하시면 해결됩니다.

```json
{
  ...
  "settings": {
    "import/resolver": {
      "node": {
        "moduleDirectory": ["node_modules", "src"]
      }
    }
  },
  ...
}
```

마음의 평안을 얻으셨길 바래요~~
