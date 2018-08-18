---
layout: post
title: "[React] create-react-app으로 React 시작하기"
date: 2018-08-18 19:00:00
author: 송타
categories: React
tags: React create-react-app
---

### create-react-app 설치

먼저 글로벌에 create-react-app 을 설치합니다.

```
yarn global add create-react-app
```

### 프로젝트 생성

```
create-react-app project-name
```

놀랍게도 이렇게하면 React 프로젝트가 생성됩니다. 갓페이스북!! 참고로 create-react-app 은 페이스북에서 제공해주는 것이랍니다.

한번 실제로 프로젝트를 만들어 보겠습니다.

```
create-react-app react-todolist
```

react-todolist 라는 프로젝트를 생성했습니다. **참고로 프로젝트 이름에 대문자는 사용할 수 없습니다.**

생성한 프로젝트는 아래와 같은 폴더 구조를 갖습니다. 딱 필수로 필요한 것들만을 포함하고 있습니다.

![react-todolist 폳더 구조](/assets/img/2018-08-18-create-react-app/폴더구조.png)

Dependency 를 보면 `react`, `react-dom`, `react-scripts` 가 포함되어 있습니다.

![react-todolist dependency](/assets/img/2018-08-18-create-react-app/dependency.png)

### 실행

```
yarn start
```

`yarn start` 명령어를 입력하니 아래와 같이 React 프로젝트가 실행되었습니다!

![react-todolist 실행화면](/assets/img/2018-08-18-create-react-app/start.png)

yarn 명령어는 아래와 같이 있습니다. 설치가 완료되면 저렇게 목록을 보여줍니다.

![yarn 명령어](/assets/img/2018-08-18-create-react-app/yarn.png)

그런데 마지막 happy hacking!?? 이건 뭘까요 ㅋㅋㅋ Happy hacking 키보드 사고싶다..

끝!!
