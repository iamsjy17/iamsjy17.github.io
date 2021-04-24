---
layout: post
title: "[Angular] Angular 11 업데이트 (9 ~ 11까지의 주요 변경사항 및 주요 기능들)"
date: 2021-04-25 1:00:00
author: Jewoo.Song
categories: Angular
tags:
  - Angular release notes
  - Angular 9
  - Angular 10
  - Angular 11
  - Angular 11 Update
  - Ivy
---

Angular 8에서 Angular 11로 업데이트를 하기 전에 미리 변경점이나 확인해야 할 점, 주요 기능들을 정리했습니다.

## 1. Angular 8 → 11 업데이트시 주요 변경사항

### [Angular 9](https://v9.angular.io/guide/updating-to-version-9)

- Default 컴파일러 `Ivy`로 변경 ([Ivy 호환성 가이드 참고](https://angular.io/guide/ivy-compatibility))
- Default로 [AOT mode](https://v9.angular.io/guide/aot-compiler)로 컴파일
- TypeScript 3.5 지원 중단 → TypeScript 3.7 버전 업데이트
- tslib `peer dependency`로 변경
- `entryComponents` 제거
- ModuleWithProviders 제네릭과 함께 사용
- esm5, fesm5 → esm2015, fesm2015로 entrypoints 변경 ([Angular Package Format](https://docs.google.com/document/d/1CZC2rcpxffTDfRDs6p1cfbmKNLA6x5O-NtkJglDaBVs/preview))
- [CSS Binding 규칙](https://angular.io/guide/attribute-binding#styling-precedence) 변경

### [Angular 10](https://v10.angular.io/guide/updating-to-version-10)

- TypeScript 3.7 지원 중단 → TypeScript 3.9 버전 업데이트
- `number` 타입의 Input field는 값이 변경될 때마다 단 한 번의 `valueChanges` event를 발생시킨다.
- `minLength`, `maxLength` validators 는 오직 length 프로퍼티가 있는 경우에만 동작한다.

### [Angular 11](https://v10.angular.io/guide/updating-to-version-10)

- IE 9, 10, IE 모바일 지원 중단
- TypeScript 3.9 지원 중단 → TypeScript 4.0 버전 업데이트
- `relativeLinkResolution`의 기본값이 `legacy` 에서 `corrected`로 변경됨.
- `shouldReuseRoute` 의 future, curr snapshots 인수 순서가 뒤바뀐 오류 수정.
- `DatePipe`는 ms를 반올림하지 않는다.
- `@angular/core/testing`의 async → `waitForAsync`로 변경

<br/>

## 2. Angular 9 ~ 11 주요 기능

> Angular 9 ~ 11 release notes를 보고 사용해보면 좋을 것 같은 기능들을 정리한 내용입니다.

### [Angular 9](https://blog.angular.io/version-9-of-angular-now-available-project-ivy-has-arrived-23c97b63cfa3)

#### 더 나아진 디버깅

Ivy는 디버깅 툴을 좀 더 다양하게 제공한다. Dev 모드로 실행할 때 디버깅을 위한 ng 객체를 제공한다.

- 컴포넌트, 디렉티브와 같은 Angular 인스턴스에 접근 가능
- 인스턴스의 메소드를 직접 실행하거나 상태 변경 가능
- Change Detection 의 결과를 확인하기 위해서 applyChanges 함수를 수동으로 실행 가능(Change Detection Trigger)

![img](/assets/img/angular/angular11update5.png)

<br/>

Ivy 이전에는 발생하는 에러의 실행 스택이 크게 도움이 되지 않았다.

![img](/assets/img/angular/angular11update2.png)

Ivy에서는 함수 실행 스택이 좀 더 유용한 단위로 표시되기 때문에 수정이 필요한 부분으로 바로 이동이 가능하다.

![img](/assets/img/angular/angular11update3.png)

템플릿 instruction으로 바로 이동할 수 있는 좀 더 유용한 스택을 볼 수 있다.

![img](/assets/img/angular/angular11update4.png)

#### TypeScript 3.7 - Optional Chaning

옵셔널 체이닝을 사용해서 각 참조가 유효한지 확인하지 않고 깊은 곳에 있는 property에 접근할 수 있다.

- `?.` 연산자는 reference error를 발생시키는 대신에 `undefined`를 리턴한다.
- 함수 호출에 사용할 때에도 함수가 존재하지 않으면 `undefined`를 리턴한다.

```tsx
const adventurer = {
  name: "Alice",
  cat: {
    name: "Dinah",
  },
};

const dogName = adventurer.dog?.name;
console.log(dogName);
// expected output: undefined

console.log(adventurer.someNonExistentMethod?.());
// expected output: undefined
```

### [Angular 10](https://blog.angular.io/version-10-of-angular-now-available-78960babd41)

딱히 없어 보임...

### [Angular 11](https://blog.angular.io/version-11-of-angular-now-available-74721b7952f7)

#### 핫 모듈 갱신 기능 지원

핫 모듈 갱신 기능을 활성화하기 위해서 환경설정이나 프로젝트 코드 수정 없이 ng serve 명령으로 간단하게 활성화 가능하게 변경되었다.

```bash
ng serve --hmr
```

개발하는 동안 컴포넌트, 템플릿, 스타일 코드를 변경해도 실행 중인 애플리케이션에 화면 새로고침 없이 즉시 반영된다.

<br/>

## 3. Angular 11 Update

> 업데이트 Check List 중 수정사항이 있거나 확인해봐야 하는 항목 위주로 작성하였습니다.

![img](/assets/img/angular/angular11update1.png)

```bash
ng update @angular/core@8 @angular/cli@8

ng update @angular/core@9 @angular/cli@9

ng update @angular/core@10 @angular/cli@10

ng update @angular/core @angular/cli
```

### 주요 Check List

- [x] Styling Binding 규칙 변경사항 확인
- [x] `entryComponents` 제거
- [x] `TestBed.get` → `TestBed.inject` 대체
- [x] `minLength` , `maxLength` validator가 사용되는 경우 length property 확인
- [x] async pipe를 사용할 때 같은 값이 emit 되는 경우에도 렌더링 업데이트가 필요한 경우가 있었는지 확인
- [x] async pipe에서 undefined 로 비교하는 경우가 있는지 확인
- [x] `keyvalue` pipe 동작 확인
- [x] asyncValidator가 validation 동작을 완료한 후 `statusChange` 이벤트를 발생시키도록 변경됨. 동작 확인.
- [x] RouteReuseStrategy의 `shouldReuseRoute` 인자 순서가 변경 됨으로써 동작 달라지는 부분 없을지 확인

<br/>

## 참고

- [Angular 8.2 -> 11.0 Check List](https://update.angular.io/?l=3&v=8.2-11.0)
- [Angular 9 Update Guide](https://v9.angular.io/guide/updating-to-version-9)
- [Angular 10 Update Guide](https://v10.angular.io/guide/updating-to-version-10)
- [Angular 11 Update Guide](https://angular.io/guide/updating-to-version-11)
- [Angular 9 Release Notes](https://blog.angular.io/version-9-of-angular-now-available-project-ivy-has-arrived-23c97b63cfa3)
- [Angular 10 Release Notes](https://blog.angular.io/version-10-of-angular-now-available-78960babd41)
- [Angular 11 Release Notes](https://blog.angular.io/version-11-of-angular-now-available-74721b7952f7)
- [Optional Chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)
