---
layout: post
title: "[React] GitHub Pages에 React App(SPA) 호스팅 하기"
date: 2018-11-04 19:00:00
author: Jewoo.Song
categories: React
tags: GitHub Page SPA React router react-router
---

이번 포스팅의 주제는 GitHub Page 에 SPA React App 을 배포하기입니다.
프로젝트를 할 때마다 호스팅을 한다면 자금이 만만치 않고, 로컬 서버를 돌리기에도 쉽지가 않죠. 그런데 고맙게도 GitHub Pages 로 무료로 호스팅을 할 수 있습니다.

GitHub Pages 를 이용해서 무료로 호스팅하는 두 가지 방법 중 프로젝트 사이트를 만드는 방법에 대해서 설명드리겠습니다. _다른 한 가지가 제 블로그 같이 개인 사이트를 만드는 거에요!_

로컬에서 구동할 수 있는 리액트 앱 프로젝트가 있다고 가정하고 설명을 진행하겠습니다.

> 만약 프로젝트가 없다면 [React] React 프로젝트 시작하기 (라이브러리 추가 및 환경설정) 포스팅을 보고 만드시고 시작하셔도 됩니다.

### React App 호스팅하기

#### gh-pages 설치

먼저 gh-pages package 를 설치합니다.

```bash
yarn add gh-pages
```

#### package.json 수정

그리고 호스팅할 프로젝트의 주소를 "homepage"에 지정합니다.
그리고 deploy 시 현재 프로젝트의 빌드 결과물을 해당 주소에 deploy 할 수 있도록 명령어를 지정합니다.

```json
{
  ...
  "homepage": "https://사용자아이디.github.io/프로젝트명",
  ...
  "scripts": {
    ...
    "predeploy": "yarn run build",
    "deploy": "gh-pages -d build"
  },
}
```

#### deploy

설정을 마첬으면 이제 build 와 deploy 를 수행합니다.

```bash
yarn run deploy
```

여기까지 해서 일단 호스팅하는 것은 완료입니다.
본인의 github 계정에 들어가 보면 gh-pages branch 가 현재 repository 추가 된 것을 보실 수 있을겁니다.

**그런데, 안타깝게도 GitHub-Pages 는 SPA 를 지원하지 않습니다!!**
그래서 router 를 사용하여 SPA 로 동작하도록 하기 위해서는 몇 가지 설정(트릭?)이 필요합니다.

### SPA React App 호스팅하기

설명은 create-react-app 으로 프로젝트를 생성하였다고 가정하고 하도록 하겠습니다.

#### 404.html 추가

먼저 프로젝트 루트의 public 폴더에 404.html 을 추가합니다.
이유는 GitHub pages 가 SPA 를 지원하지 않기 때문에 `iamsjy17.github.id/planet/Demo` 와 같은 URL 을 인식하지 못하고 모두 404 page 로 뜨게 됩니다. 따라서 저희는 404.html 을 만들어서 해당 URL 을 쿼리로 만들어서 리다이렉션을 하도록 하겠습니다.

_아래 소스는 MIT license 의 오픈 소스입니다. 프로젝트 포함시 라이센스를 명시해주세요._

```html
<!DOCTYPE html>
<html>
  <head>
      <meta charset="utf-8">
      <title>Single Page Apps for GitHub Pages</title>
      <script type="text/javascript">
          // Single Page Apps for GitHub Pages
          // https://github.com/rafrex/spa-github-pages
          // Copyright (c) 2016 Rafael Pedicini,  licensed under the MIT License
          //  ---------------------------------------------- ------------------------
          // This script takes the current url and  converts the path and query
          // string into just a query string, and then  redirects the browser
          // to the new url with only a query string  and hash fragment,
          // e.g. http://www.foo.tld/one/two?a=b& c=d#qwe, becomes
          // http://www.foo.tld/?p=/one/two&  q=a=b~and~c=d#qwe
          // Note: this 404.html file must be at least  512 bytes for it to work
          // with Internet Explorer (it is currently >  512 bytes)

          // If you're creating a Project Pages site  and NOT using a custom domain,
          // then set segmentCount to 1 (enterprise   users may need to set it to > 1).
          // This way the code will only replace the  route part of the path, and not
          // the real directory in which the app  resides, for example:
          //  https://username.github.io/repo-name/one/two?  a=b&c=d#qwe becomes
          // https://username.github.io/repo-name/? p=/one/two&q=a=b~and~c=d#qwe
          // Otherwise, leave segmentCount as 0.
          var segmentCount = 1;

          var l = window.location;
          l.replace(
              l.protocol + '//' + l.hostname + (l.port ?   ':' + l.port : '') +
              l.pathname.split('/').slice(0, 1 +  segmentCount).join('/') + '/?p=/' +
              l.pathname.slice(1).split('/').slice  (segmentCount).join('/').replace(/&/g,  '~and~') +
              (l.search ? '&q=' + l.search.slice(1) .replace(/&/g, '~and~') : '') +
              l.hash
          );

      </script>
  </head>
  <body>
  </body>
</html>
```

#### index.html 수정

404.html 을 통해서 쿼리로 변경된 URL 을 index.html 에서 받게됩니다. 이 때 index.html 에서 해당 쿼리를 해석하여 올바른 URL 로 변경하도록 스크립트를 추가합니다.

_아래 소스는 MIT license 의 오픈 소스입니다. 프로젝트 포함시 라이센스를 명시해주세요._

```html
<head>
...
  <!-- Start Single Page Apps for GitHub Pages -->
  <script type="text/javascript">
    // Single Page Apps for GitHub Pages
    // https://github.com/rafrex/spa-github-pages
    // Copyright (c) 2016 Rafael Pedicini, licensed under the MIT License
    // ----------------------------------------------------------------------
    // This script checks to see if a redirect is present in the query string
    // and converts it back into the correct url and adds it to the
    // browser's history using window.history.replaceState(...),
    // which won't cause the browser to attempt to load the new url.
    // When the single page app is loaded further down in this file,
    // the correct url will be waiting in the browser's history for
    // the single page app to route accordingly.
    (function (l) {
      if (l.search) {
        var q = {};
        l.search.slice(1).split('&').forEach(function (v) {
          var a = v.split('=');
          q[a[0]] = a.slice(1).join('=').replace(/~and~/g, '&');
        });
        if (q.p !== undefined) {
          window.history.replaceState(null, null,
            l.pathname.slice(0, -1) + (q.p || '') +
            (q.q ? ('?' + q.q) : '') +
            l.hash
          );
        }
      }
    }(window.location))
  </script>
  <!-- End Single Page Apps for GitHub Pages -->

</head>
```

#### basename 설정

마지막으로 App.js 또는 Root.js 의 BrowserRouter 에 basename 을 지정합니다. 예를들어 `package.json`에 아래와 같이 설정했다면

```json
"homepage": "https://iamsjy17.github.io/planet",
```

basename 은 아래와 같이 지정해주시면 됩니다.

```js
basename = "/planet";
```

env 파일에 따로 빼주셔도 됩니다.

```js
<Provider store={store}>
  <BrowserRouter basename={process.env.PUBLIC_URL}>
    <App />
  </BrowserRouter>
</Provider>
```

참고: https://github.com/rafrex/spa-github-pages

GitHub Pages 에 SPA 호스팅하기 끝!!
