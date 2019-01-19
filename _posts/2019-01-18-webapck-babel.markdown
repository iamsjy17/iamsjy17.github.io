---
layout: post
title: "[Webpack] Babel7 과 Webpack4 설정을 통해 ES6 모듈 사용하기"
date: 2019-01-19 19:00:00
author: 송튜디오
categories: webpack
tags: Webpack Webpack4 babel babel7 ES6 Javascript
---

ES6 import.. export.. 문법을 브라우저에서 사용하고 싶지만 안타깝게도 아직까지 대부분의 브라우저에서 es6 모듈을 지원하지 않습니다.
때문에 webpack과 babel이 필요합니다.

Babel은 최신 자바스크립트를 ES5 문법으로 변환해주는 역할을 합니다.
그리고 webpack은 모듈 번들러입니다. 간단하게 말하자면 여러 파일들을 모아서 하나의 파일로 만들어줍니다.

웹팩과 바벨을 설정하는 것을 처음부터 진행해 보도록 하겠습니다.

### 프로젝트 생성

```bash
mkdir webpack_babel_skeleton
cd webpack_babel_skeleton
npm init -y
```

여기까지 진행하면 아래와 같은 파일이 생깁니다.
//package.json

```json
{
  "name": "webpack_babel_skeleton",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

### 설치

#### babel

ES6에 새롭게 추가된 기능들을 변환하기 위해서는 @babel/polyfill을 설치해야 합니다.

```bash
npm install @babel/cli --save-dev
npm install @babel/core --save-dev
npm install @babel/preset-env --save-dev
npm install babel-loader --save-dev
npm install @babel/polyfill --save-dev
```

#### webpack

```bash
npm install webpack --save-dev
npm install webpack-cli --save-dev
```

#### web-doc-server

web-doc-server는 개발 과정에서 확인할 수 있도록 웹서버를 제공해줍니다.
소스를 수정하고 브라우저에서 바로 바로 확인할 수 있어서 매우 편리합니다.

```bash
npm install webpack-dev-server --save-dev
```

### 설정

#### package.json

build와 devserver에 올리는 용도로 script에 아래와 같이 추가해줍니다.

```js
//package.js
...
  "scripts": {
    "build": "webpack -w",
    "devserver": "webpack-dev-server -w"
  }
...
```

#### webpack.config.js

프로젝트 루트에 webpack.config.js를 만들고 아래와 같이 설정해주세요.
index.js를 entry로 src폴더 밑에 js파일들을 하나로 합쳐서 dist/ 폴더 밑에 bundle.js로 떨궈주도록 설정하였습니다.

```js
//webpack.config.js
const path = require("path");

module.exports = {
  entry: ["@babel/polyfill", "./src/test.js"],
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist/")
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        include: [path.resolve(__dirname, "src/js")],
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env"]
          }
        }
      }
    ]
  },
  devtool: "source-map",
  mode: "development"
};
```

#### .babelrc

프로젝으 루트에 .babelrc 파일을 만들고, 브라우저는 최근 2버전까지만 지원하도록 설정하였습니다.

```json
//.babelrc
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "browsers": ["last 2 versions"]
        }
      }
    ]
  ]
}
```

### 예제

폴더 구조를 아래와 같이 만들어주세요.

```
root
- src
  - test.js
  - lib.js
- dist
- node_modules
- .babelrc
- package.json
- webpack.config.js
- index.html
```

#### Entry

```js
// src/test.js
import "@babel/polyfill";
import Util from "./util";

class Test {
  constructor(num) {
    this.num = num;
  }
  Print() {
    console.log(Util.Print(this.num));
  }
}

window.Test = Test;
```

#### util

```js
// src/util.js
const Util = {
  Print(num) {
    console.log(num);
  }
};
export default Util;
```

#### index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <script type="text/javascript" src="/dist/bundle.js"></script>
  </head>
  <body>
    <script type="text/javascript">
      var test = new Test(10);
      test.Print();
    </script>
  </body>
</html>
```

#### 실행

##### build

build를 실행하면 dist 폴더 밑에 bundle.js가 생성된 것을 보실 수 있습니다.

```bash
npm run build
```

##### devserver

devserver로 실행하면 실제로 예제 결과를 localhost:8080 에서 확인할 수 있습니다.

```bash
npm run devserver
```

### Github

https://github.com/iamsjy17/webpack_babel_skeleton

끝!!
