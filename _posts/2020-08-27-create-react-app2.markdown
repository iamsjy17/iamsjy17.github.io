---
layout: post
title: '[React] React 프로젝트 시작하기 (라이브러리 추가 및 환경설정)'
date: 2018-08-27 19:00:00
author: 송타
categories: React
tags: React create-react-app
---

지난 번에 `create-react-app`으로 프로젝트를 시작하는 것을 포스팅 했었는데요, 이번에는 좀 더 구체적으로 프로젝트를 세팅 해보겠습니다.

### 프로젝트 환경설정 eject

```
yarn eject
```

`eject`명령어로 프로젝트 환경설정 파일들을 프로젝트 루트로 옮겨줍니다.  
그런데 해당 프로젝트를 벌써! git repogitory 로 관리하고 있다면 안될 수 있습니다.(untracked 파일이나 커밋하지 않은 수정 내역이 있는 경우)
실패해도 당황하지말고 정리하고 다시해주세요.

`eject` 명령어를 실행하면 `config` 폴더가 새로생긴 것을 볼 수 있습니다.

### Sass 관련 모듈 설치 및 설정

#### dependency 추가

```
yarn add sass-loader node-sass classnames
```

1. `sass-loader` : webpack 에서 Sass 파일을 읽어오는 말그대로 `loader`입니다.
2. `node-sass` : Sass 로 작성된 코드들을 CSS 로 변환해주는 역할을 합니다.
3. `classnames` : 여러개의 클래스를 한번에 주고 싶을 때 편리하게 할 수 있도록 도와주는 라이브러리입니다.

#### sass loader 설정

`config`폴더 밑에 webpack.config.--.js 를 수정해줍니다. 저는 공부용이기 때문에 dev 파일만 수정하도록 하겠습니다.

```javascript
            {
            test: /\.scss$/,
            use: [
              require.resolve('style-loader'),
              {
                loader: require.resolve('css-loader'),
                options: {
                  importLoaders: 1,
                  modules: true,
                  localIdentName: '[path][name]__[local]--[hash:base64:5]'
                },
              },
              {
                loader: require.resolve('postcss-loader'),
                options: {
                  // Necessary for external CSS imports to work
                  // https://github.com/facebookincubator/create-react-app/issues/2677
                  ident: 'postcss',
                  plugins: () => [
                    require('postcss-flexbugs-fixes'),
                    autoprefixer({
                      browsers: [
                        '>1%',
                        'last 4 versions',
                        'Firefox ESR',
                        'not ie < 9', // React doesn't support IE8 anyway
                      ],
                      flexbox: 'no-2009',
                    }),
                  ],
                },
              },
              {
                loader: require.resolve('sass-loader'),
                options: {
                  includePaths: [paths.styles]
                }
              }
            ],
          },
```

`css-loader` 설정을 복사하여 아래에 붙여넣고 위와 같이 수정해줍니다.

1. `style-loader` : 스타일을 불러와 웹페이지에서 활성화하는 역할을합니다.
2. `css-loader` : css 파일에서 import/ url 등을 사용할 수 있도록해줍니다.
3. `postcss-loader` : 모든 웹브라우저에서 입력한 css 구문이 제대로 동작할 수 있도록 `-webkit`, `-mos` `ms`등의 접두사를 붙여줍니다.

수정한 내역은 아래와 같습니다.

1. option 수정
   - `modules : true` : CSS Module 을 활성화시켜 줍니다.
   - `localIdentName` : CSS Module 에서 사용할 클래스네임의 형식을 결정합니다. 네임스페이스를 꼬이지 않도록 도와줍니다.
2. loader 추가
   - `includePaths` : css 파일 import 시에 경로입력시 디렉터리 구조에 따라 길어지는 것을 간소화하기 위해 includePaths 를 지정해 줍니다. 여기서 `[paths.styles]`는 paths.js 에 미리 지정해둔 경로입니다.

#### paths 수정

config 폴더 밑에 `paths.js` 파일이 있습니다. includePaths 등에서 경로를 간소화시켜서 사용하기 위해서 이 파일에 경로를 입력합니다.

```javascript
module.exports = {
  ...,
  styles: resolveApp('src/styles')
};
```

주의할 점이 있습니다. 혹시 `vscode`를 사용하신다면 추가로 `jsconfig.json`을 루트 경로(package.js 가 있는 경로)에 추가해주어야 합니다.
그리고 아래와 같이 입력해주세요.

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "*": ["*", "src/*"]
    }
  }
}
```

### Redux

#### redux dependency 추가

기본적으로 리덕스를 사용하기 위해서 redux, react-redux 를 추가해 줍니다.

```
yarn add redux react-dedux
```

#### Immutable 추가

`Immutable`은 리액트에서 중요한 객체 불변성을 편하게 유지할 수 있도록 도와주는 라이브러리입니다.

```
yarn add immutable
```

#### redux-actions 추가

매번 action type 을 추가 할 때마다 action 생성함수를 만드는 것이 상당히 번거롭습니다. redux-actions 는 action 생성함수를 편하게 만들 수 있도록 도와주는 라이브러리입니다.

```
yarn add redux-actions
```

### Router

SPA, Single Page Application 개발을 위해서는 라우팅 기능이 필요합니다. 리액트 자체에는 이런 기능을 내장하고 있지 않기 때문에 라이브러리를 사용합니다.

#### react-router 추가

리액트 애플리케이션에서 라우팅 용도로 가장 많이 사용되는 라이브러리입니다.

```
yarn add react-router-dom
```

#### NODE_PATH 설정

SPA 는 여러개의 Page 를 가지고 있기 때문에, 폴더 구조가 복잡해 질 수 있습니다. 뎁스가 깊은 폴더 내에 있는 Component 에서 다른 Component 를 참조한다면 `../../../../../` 와 같은 소스가 많이 생겨나겠죠?

그래서 NODE_PATH 를 지정해서 절대경로로 import 해줄 수 있도록 합니다.

package.json 을 열어서 아래와 같이 수정해 주면 src 폴더를 기준으로 불러올 수 있습니다.

```json
...
  "scripts": {
    "start": "NODE_PATH=src react-scripts start",
    "build": "NODE_PATH=src react-scripts build",
    ...
  }
```

### 코드 스플리팅 설정

코드 스플리팅이란 말 그대로 코드를 나누는 것입니다. 웹페이지에서 모든 소스를 한번에 불러온다면 로딩시간이 그 만큼 길어지겠죠?
가장 이상적인 것은 사용자에게 제공될 첫 페이지를 만들 수 있는 소스들을 불러오고 나머지를 필요할 때 불러오는 것 입니다.
물론 시점을 잘 맞춰야합니다.

#### vendor 설정

코드 스플리팅을 하기 위해서 먼저 vendor 를 설정합니다. 주로 서드파티 라이브러리들을 vendor 로 분리 함으로써 여러분의 소스를 수정하고 소스를 업데이트할 때 그 파일 크기를 최소화할 수 있습니다.

`webpack.config.dev.js`에서 entry 를 찾아서 다음과 같이 오브젝트 형태로 수정해줍니다.
이를 통해서 전역적으로 사용하는 라이브러리들은 app 이 아닌 vendor 에 번들링됩니다.

```javascript
  ...
  entry: {
    app: [
      require.resolve('react-dev-utils/webpackHotDevClient'),
      paths.appIndexJs
    ],
    vendor: [
      require.resolve('./polyfills'),
      'react',
      'react-dom',
      'react-router-dom'
    ]
  },
  ...
```

#### CommonsChunkPlugin 설정

이 플러그인은 vendor 로 분리된 곳에 들어간 내용들이 app 에 중복으로 들어가지 않도록 합니다.
이 것도 마찬가지로 config 에 추가해주시면 됩니다.
`InterpolateHtmlPlugin`으로 검색해서 그 위에 추가해주시면 됩니다.

```javascript
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      filename: 'vendor.js'
    }),
    new InterpolateHtmlPlugin(env.raw),
    ...
  ]
```

여기 까지하면 기본적으로 개발할 준비는 다 됬습니다.
_위에 항목들은 모두 필수는 아닙니다. 그냥 이전 포스팅처럼 create-react-app 으로 생성만 하고 시작하셔도 되요!!_

다음에는 webpack 에 대해서 공부를 좀 더해서 자세하게 각 항목들에 대해서 포스팅 해보도록 하겠습니다. ^^
