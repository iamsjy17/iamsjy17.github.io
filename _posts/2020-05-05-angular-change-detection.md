---
layout: post
title: "[Angular] Angular의 Change Detection"
date: 2020-05-05 1:00:00
author: Jewoo.Song
categories: Angular
tags:
  - Angular
  - Change Detection
  - zone
  - zone.js
  - NgZone
  - runOutsideAngular
  - markForCheck
  - detectChange
---

## 1. Change Detection이란?

Angular의 Change Detection은 Angular 프레임워크의 핵심 매커니즘이다.

프레임워크는 애플리케이션의 상태(상태와 템플릿)를 DOM에 반영해야 한다. 상태의 어떠한 변화가 발생할 때 View를 업데이트하는 것은 필수이다.

이렇게 상태 변경을 감지해서 View(Dom)과 Model을 동기화하는 이러한 메커니즘을 Change Detection이라 한다.

즉, Change Detection 이란? Model(data)에 변경되었을 때 View(DOM)를 업데이트하는 일련의 프로세스이다.

Angular뿐 아니라 각 프론트엔드 프레임워크는 이러한 메커니즘을 구현하고 있다.
예를 들어 React에 경우 Virtual DOM을 사용하고, Angular는 Change Detection을 사용한다.

대부분의 경우 애플리케이션의 성능을 향상시키기 위해 Change Detection의 원리까지 알 필요는 없지만, 만약 제대로 사용하지 않는다면 심각한 성능 저하를 발생시킬 수 있다.

> Angular에서 Change Detection 이란? 작게 보면 단순히 변경이 있는지 검사하고 만약 변경된 상태가 있다면 변경된 상태를 View에 반영하는 메커니즘이다.
> 그런데 좀 더 넓게 본다면 `자동으로 변화가 발생할 수 있는 상황을 감지하여(zone.js)` 변경 여부를 체크하고 변경된 상태가 있다면 변경된 상태를 View에 반영하는 이 일련의 프로세스 전체를 Change Detection으로 볼 수 있을 것 같다.

### 1-1. Change Detection 동작 원리

1. 개발자가 모델을 업데이트한다.(바인딩된 컴포넌트 모델을 업데이트한다.)
2. Angular가 zone을 통해서 변화가 발생할 수 있는 상황을 감지하여 Change detection을 발생시킨다.
3. Change detection가 모델이 변화가 있는지 확인하기 위해 컴포넌트 트리의 모든 컴포넌트를 탑다운 방식으로 점검한다.
4. 만약 변화가 감지되었다면 뷰를 다시 그려서 업데이트된 모델과 동기화한다.

![img](/assets/img/angular/changedetection1.png)

![img](/assets/img/angular/changedetection2.png)

그림은 애플리케이션 bootstrap 과정에서 생성된 Angular 컴포넌트 트리와 각 컴포넌트의 Change Detector(CD)를 나타낸다.

Change Detector는 현재 value의 property와 이전 value의 property를 비교하고 만약 변화가 감지되었다면 `isChanged`를 true로 세팅한다.

> Change Detection의 비교는 deep compare가 아닌 단순 === 비교이다. 따라서 value 타입인 경우 값만 같다면 변경 감지가 발생하지 않지만 참조 타입인 경우 모든 값이 같더라도 참조가 다르다면 변경 감지가 발생할 수 있다.

## 2. Zone.js

### 2-1. Zone?

Zone 이란 비동기 작업에서 지속되는 실행 컨텍스트이다. 즉, 논리적으로 연결된 여러 비동기 작업을 쉽게 할 수 있도록 도와주는 새로운 메커니즘이다.

Angular에서 Zone 이란 Change Detection 동작 원리 중 2번 `Angular가 변화가 발생할 수 있는 상황을 감지하여 Change Detection을 발생시킨다.`에 해당하는 부분으로 Angular에게 언제 Change Detection 수행하도록 해야하는지 우리가 신경쓰지 않아도 되도록 자동으로 Change Detection을 발생시켜주는 역할을 한다.

> Angular에서 Zone은 Change Detection의 트리거 역할을 한다.

#### 일반적인 Zone의 실행 과정

일반적으로 zone은 비동기 작업을 추적하고 인터셉트한다.
그리고 다음과 같은 과정을 거친다.

1. stable 상태에서 시작
2. zone 안에서 작업이 실행되면 unstable 상태로 변경
3. zone 안에서의 작업이 완료되면 다시 stable 상태로 변경

#### Zone을 사용함으로써 얻을 수 있는 이점

- Thread와 유사한 zone에 연결된 데이터들은 같은 zone 안에서는 어떤 비동기 작업도 데이터에 접근할 수 있게 된다. (데이터 접근성)
- 클린업이나, 렌더링, 테스크 검증 작업을 수행하기 위해 존안에서 실행된 비동기 작업을 자동으로 추적한다.(자동 추적)
- zone 안에서 실행된 전체 시간을 측정할 수 있다.(프로파일링)
- 최상위 영역에 예외를 전달하지 않아도, zone안에서의 예외와 Promise의 예외들을 다룰 수 있다.(예외 처리)

### 2-2. monkey patch

Angular에서의 Zone 역할을 수행하기 위해 Angular는 애플리케이션 시작 단계에서 Change detection을 위하여 몇몇 로우 레벨 브라우저 API들을 `Zone.js`를 이용하여 패치한다.

다음 이벤트 중 하나가 발생하면 zone 안에서 콜백이 실행되고 이를 통해서, Angular는 변화가 감지되었다고 간주하고 Change detection을 실행한다.

- 애플리케이션의 상태를 변경시키는 이벤트들
  - EventEmitter
  - any Browser event(click, keyup, etc.)
  - setInterval() and setTimeout()
  - HTTP requests (XMLHttpRequest)

> 전체 몽키 패치 목록
>
> - https://github.com/angular/angular/blob/376ad9c3cdf5a9432500333bce4a6fc42113210a/packages/zone.js/MODULE.md
> - 위 URL의 표에서 Macro Task 인지 Micro Task 인지 Event Task 인지에 따라 async task 스케줄링을 예측할 때 참고할 수 있다.
>
> 소스 코드
>
> - https://github.com/angular/zone.js/blob/1ba851989ccb6907df49bba37ee24ab60adf13a9/lib/browser/browser.ts#L28

### 2-3. Zone 주요 API

> Zone 관련 대부분의 설명과 예제는 아래 링크에서 인용하였습니다.
>
> - https://indepth.dev/i-reverse-engineered-zones-zone-js-and-here-is-what-ive-found/
> - https://norux.me/65

```ts
class Zone {
  constructor(parent: Zone, zoneSpec: ZoneSpec);
  static get current();
  get name();
  get parent();

  fork(zoneSpec: ZoneSpec);
  run(callback, applyThis, applyArgs, source);
  runGuarded(callback, applyThis, applyArgs, source);
  wrap(callback, source);
  fork();
}
```

#### current

- Zone에는 현재 영역(Current Zone)이라는 개념이 중요하다. 현재 역영은 모든 비동기 작업을 전달받은 비동기 컨텍스트이다.
- 이것은 현재 실행된 스택 프레임과 비동기 작업을 연결한 Zone을 의미한다.
- Zone.current라는 static getter 함수로 접근이 가능하다.

#### name

- 각 zone은 name property를 갖는다 이 name은 디버깅을 위해 사용된다.

#### run

```ts
z.run(callback, applyThis, applyArgs, source);
```

- 주어진 zone안에서 동기적으로 함수를 호출한다.
- 이 함수는 실행된 콜백의 현재 존을 z로 설정하고 콜백의 실행이 완료되고나면 현재 존을 원래의 존으로 되돌린다.
- 이때 존 안에서 실행된 콜백은 존에 들어간다고 표현한다.

#### runGuarded

```ts
z.runGuarded(callback, applyThis, applyArgs, source);
```

- 기본적으로 run 함수와 동일하다.
- 단, 런타임 에러를 캐치하고, 에러 인터셉트 메커니즘을 제공한다.
- 만약 에러를 어떠한 부모 존에서도 처리되지 않는다면 다시 에러를 throw 한다.

#### wrap

```ts
Zone.prototype.wrap = function (callback, source) {
  // ...
  var zone = this;
  return function () {
    return zone.runGuarded(callback, this, arguments, source);
  };
};
```

- 클로저에 z를 담은 새로운 함수를 만든다.
- 그리고 실행될 때 z.runGuarded(callback) 함수를 호출한다.
- 나중에 콜백으로 other.run(callback)을 받는다고 하더라도 other 존이 아닌 여전히 z 존안에서 실행된다.
- javascript의 Function.prototype.bind와 유사하다.

#### fork

```ts
z.fork(zoneSpec);
```

- zone에서 가장 많이 사용되는 기능 중 하나로 fork 함수를 통해서 새로운 zone을 생성한다.
- zone을 복제하여 새로운 자식 zone을 생성하고, 복제된 zone의 parent는 기존의 zone이 된다.
- fork 메서드에 전달되는 오브젝트는 ZoneSpec이다.
  - name은 zone의 이름으로 정의된다.
  - properties는 zone 안의 데이터들을 연결하기 위해 사용한다.
  - 다른 모든 속성들은 부모 존이 자식 존의 특정 오퍼레이션을 인터셉트할 수 있게 해주는 후킹 속성이다.
    - zone은 부모 zone에 의해 인터셉트될 수 있다.

```ts
interface ZoneSpec {
    name: string;
    properties?: {
        [key: string]: any;
    };

    onFork?: ( ... );
    onIntercept?: ( ... );
    onInvoke?: ( ... );
    onHandleError?: ( ... );
    onScheduleTask?: ( ... );
    onInvokeTask?: ( ... );
    onCancelTask?: ( ... );
    onHasTask?: ( ... );
}
```

```ts
// zoneB 생성
const zoneB = Zone.current.fork({ name: "B" });

// zoneB 안에서의 실행
// - zoneB에 들어간다.
// - zoneB의 데이터에 접근이 가능하다.
zoneB.run(callback);
```

z.run 메서드를 사용함으로써 각 함수는 사용할 zone을 직접 정의하여 run 메서드로 실행할 수 있다.
만약 z.run을 사용하지 않고 직접 실행하면 현재 실행 컨텍스트에 해당하는 존에서 실행된다.

만약 z.run을 통해 zone을 변경하지 않는다면 모든 함수는 root zone에서 실행된다.

```ts
function c() {
  console.log(Zone.current.name); // <root>
}
function b() {
  console.log(Zone.current.name); // <root>
  c();
}
function a() {
  console.log(Zone.current.name); // <root>
  b();
}
a();
```

### 2-4. Zone의 주요 동작 원리

#### 비동기 테스크에서의 Zone 유지

```ts
const zoneBC = Zone.current.fork({ name: "BC" });

function c() {
  console.log(Zone.current.name); // BC
}

function b() {
  console.log(Zone.current.name); // BC
  setTimeout(c, 2000);
}

function a() {
  console.log(Zone.current.name); // <root>
  zoneBC.run(b);
}

a();
```

zone 안에서 호출한 코드는 같은 zone 안에서 실행된다. 그리고 이 메커니즘은 비동기 함수 호출에서도 똑같이 적용된다.
위와 같은 코드가 있을 때 각 실행 컨텍스트의 zone은 다음과 같다.

- global : root zone
- a : root zone
- b : zone BC
- c : zone BC

#### 비동기 테스크에서 컨텍스트 전파

zone의 장점 중 하나는 데이터 접근성인데 이는 `Context propagation`을 통해서 이루어진다.
데이터를 zone에 넣어두면 이 zone 안에서 실행된 어떤 테스크들이라도 이 데이터에 접근할 수 있다.

새로운 zone을 fork 할 때 zoneSpec으로 데이터를 정의한다.

```ts
const zoneBC = Zone.current.fork({
  name: "BC",
  properties: {
    data: "initial",
  },
});
```

그리고 zone.get 메서드를 사용하여 이 데이터에 접근할 수 있다.

```ts
function a() {
  console.log(Zone.current.get("data")); // 'initial'
}

function b() {
  console.log(Zone.current.get("data")); // 'initial'
  setTimeout(a, 2000);
}

zoneBC.run(b);
```

그리고 fork 메서드로 생성된 자식 zone은 부모 zone의 properties를 상속한다.

```ts
const parent = Zone.current.fork({
  name: "parent",
  properties: { data: "data from parent" },
});

const child = parent.fork({ name: "child" });

child.run(() => {
  console.log(Zone.current.name); // 'child'
  console.log(Zone.current.get("data")); // 'data from parent'
});
```

#### 테스크 추적

zone은 또한 비동기 macrotask나 microtask를 추적할 수 있다.
zone은 큐안에 있는 중요 테스크들을 모두 유지한다. 큐의 상태가 바뀔 때마다 알림을 받기 위해 zoneSpec에 있는 onHasTask라는 후킹 함수를 이용할 수 있다.

```ts
onHasTask(delegate, currentZone, targetZone, hasTaskState);

type HasTaskState = {
  microTask: boolean;
  macroTask: boolean;
  eventTask: boolean;
  change: "microTask" | "macroTask" | "eventTask";
};
```

fork 메서드 호출 시 zoneSpec을 아래와 같이 지정함으로써 후킹 메서드를 실행시킬 수 있다.

```ts
const z = Zone.current.fork({
  name: "z",
  onHasTask(delegate, current, target, hasTaskState) {
    console.log(hasTaskState.change); // "macroTask"
    console.log(hasTaskState.macroTask); // true
    console.log(JSON.stringify(hasTaskState));
  },
});

function a() {}

function b() {
  setTimeout(a, 2000);
}

z.run(b);
```

위 코드의 실행 결과는 다음과 같다.

```ts
macroTask
true
{
    "microTask": false,
    "macroTask": true,
    "eventTask": false,
    "change": "macroTask"
}

// 2초 후

macroTask
false
{
    "microTask": false,
    "macroTask": false,
    "eventTask": false,
    "change": "macroTask"
}
```

단, onHasTask 후킹은 전체의 테스크 큐의 비어있거나(empty) 혹은 비어있지 않은(non-empty) 상태를 추적할 때만 사용할 수 있다. 각각의 테스크 큐를 추적할 때는 사용할 수 없다.

- empty -> non-empty
- non-empty -> empty

개별 테스크를 추적할 때는 다음과 같은 훅을 정의해서 사용한다.

- onScheduleTask: setTimeout와 같은 비동기 오퍼레이션이 탐지될 때마다 실행된다.
- onInvokeTask: setTimeout(callback) 같이 비동기 오퍼레이션으로 전달된 콜백이 동작될 때 실행된다.
- onInvoke: z.run()을 실행하여 zone에 진입할 때마다 실행된다.

### 2-5. NgZone

Angular는 `NgZone`이라는 zone을 사용하고, 이 NgZone은 오직 하나만 존재하며 이 영역 안에서 발생한 비동기 작업에 한하여 Change Detection이 실행된다.

- NgZone은 주입가능한 서비스로써 NgZone을 사용하여 Angular zone 안팎에서의 작업을 수행할 수 있다.
- UI 업데이트 또는 오류 처리가 필요없는 비동기 작업을 수행할 때 성능 최적화를 위해서 사용한다.
  - 이러한 작업은 runOutsideAngular를 통해 시작할 수 있다.

#### NgZone의 정체

NgZone 은 단지 자식 존을 복제하는 것에 대한 래퍼(wrapper)이다.

```ts
function forkInnerZoneWithAngularBehavior(zone: NgZonePrivate) {
    zone._inner = zone._inner.fork({
        name: 'angular',
        properties: <any>{},
        onInvokeTask: ()=>{
          //  setTimeout(callback) 같이 비동기 오퍼레이션으로 전달된 콜백이 동작될 때 실행된다.
        },
        onInvoke:()=>{
          //  z.run()을 실행하여 zone에 진입할 때마다 실행된다.
        },
        onHasTask:()=>{
          //  비동기 큐의 상태가 변경될 때마다 실행된다.
        },
        onHandleError:()=>{
          //  에러 발생 시 실행된다.
        }
    }
```

위 코드와 같이 fork된 zone은 `_inner` 속성에 저장되며 이 zone은 NgZone.run()을 실행할 때 사용된다.
즉, `_inner`가 ngZone이라고 불리는 zone의 참조이다.

```ts
  run<T>(fn: (...args: any[]) => T, applyThis?: any, applyArgs?: any[]): T {
    return (this as any as NgZonePrivate)._inner.run(fn, applyThis, applyArgs) as T;
  }
```

ngZone을 fork를 실행한 순간에 현재 Zone은 `_outer` 속성에 저장되며 이 zone은 NgZone.runOutsideAngular() 메서드를 실행할 때 사용된다.

```ts
export class NgZone {
// ...
  constructor(...){
    // ...
    self._outer = self._inner = Zone.current;
    // ...
    forkInnerZoneWithAngularBehavior(self);
    // ...
  }
// ...
}
```

runOutsideAngular 메서드는 성능에 영향을 끼칠 수 있는 작업을 실행할 때 사용된다.
Angular zone 밖에서 실행되게 함으로써 Change Detection을 발생시키는 것을 피할 수 있다.

```ts
runOutsideAngular<T>(fn: (...args: any[]) => T): T {
return (this as any as NgZonePrivate).\_outer.run(fn) as T;
}
```

> https://github.com/angular/angular/blob/698b0288bee60b8c5926148b79b5b93f454098db/packages/core/src/zone/ng_zone.ts

#### ngZone의 속성(Event)

|              Property               |                                                                                            Description                                                                                            |
| :---------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|          isStable: boolean          |                                                                                microtask 또는 macrotask 존재 여부                                                                                 |
|    onUnstable: EventEmitter<any>    |                                                                Angular Zone에 진입할 때 발생한다. <br/>VM Turn에서 처음 시작된다.                                                                 |
| onMicrotaskEmpty: EventEmitter<any> | 현재 VM Turn에 더 이상 큐에 대기 중인 microtask가 없을 때 발생한다. <br/> 이 이벤트는 Angular가 Change Detection을 수행하도록 합니다.<br/> 따라서 이 이벤트는 VM Turn마다 여러 번 발생할 수 있다. |
|     onStable: EventEmitter<any>     |                      마지막 onMicrotaskEmpty가 실행되어, 더 이상 microtask가 없을 때 발생한다. <br> 이것은 곧 VM turn이 전환될 것을 의미하고, 이 이벤트는 한 번만 발생된다.                       |
|     onError: EventEmitter<any>      |                                                                                     에러가 있을 때 발생한다.                                                                                      |

Angular는 Change Detection을 자동으로 발생시키기 위해서 ApplicationRef 내부에서 NgZone의 onMicrotaskEmpty를 subscribe 한다.

```ts
export class ApplicationRef {
// ...
  constructor(
      private _zone: NgZone, ...) {
  // ...
    this._zone.onMicrotaskEmpty.subscribe(
        {next: () => { this._zone.run(() => { this.tick(); }); }});
      }
// ...
}
```

> ApplicationRef의 tick 메서드는 Change Detection을 발생시킨다.
>
> https://github.com/angular/angular/blob/30d5a2ca83c9cf44f602462597a58547b05b75dd/packages/core/src/application_ref.ts#L364

#### onMicrotaskEmpty 이벤트 발생 과정

onMicrotaskEmpty 이벤트는 NgZone의 checkStable 메서드를 통해 발생한다.

```ts
function checkStable(zone: NgZonePrivate) {
  if (zone._nesting == 0 && !zone.hasPendingMicrotasks && !zone.isStable) {
    // ...
    zone._nesting++;
    zone.onMicrotaskEmpty.emit(null);
    //  ...
  }
}
```

그리고 checkStable은 다음 세 가지 zone의 후킹 메서드에서 호출된다.

- onHasTaks
- onInvokeTask
- onInvoke

### 2-6. Zone이 없는 Angular의 Change Detection

Zone과 Angular의 Change detection은 강한 연관성이 있지만, 기술적으로 부분집합적인 관계는 아니다.

> Zone은 비동기 작업을 수행할 때 Change Detection을 자동으로 발생시킨다.

Change detection은 Zone 과는 별도의 메커니즘이기 때문에 Zone과 NgZone 없이도 Change Detection을 구현할 수 있다.

#### ApplicationRef

```ts
interface ApplicationRef {
  // ...
  tick(): void;
}
```

페이지에서 실행되고 있는 Angular 애플리케이션에 대한 참조로 injectable 한 서비스이다.
ngZone을 사용하지 않는다면 비동기 작업을 수행할 때 자동으로 Change Detection이 실행되지 않는데 이때 ApplicationRef의 `tick()` 메서드를 사용하여 실행시킬 수 있다.

다시 한번 정리하자면 zone과 NgZone은 Change Detection의 일부분이 아니며 자동으로 Change Detection을 실행시키기 위한 편리한 메커니즘일 뿐이다.

## 3. Change Detection Strategies

Angular의 Chnage Detection은 인라인 캐싱을 사용하여 매우 빠르게 실행된다.
이 과정은 효율적이지만 큰 규모의 애플리케이션에서는 여전히 성능 문제가 발생할 수 있다.

따라서 Angular는 두 가지의 Change Detection 전략을 제공한다.

### 3-1. Default

Angular는 default로 ChangeDetectionStrategy.Default를 사용한다.
default는 이벤트가 발생하여 Zone이 Change Detection을 트리거 할 때마다 컴포넌트 트리의 모든 컴포넌트를 top-down 방식으로 변경된 상태가 있는지 검사한다.

보통의 경우에는 문제가 되지 않지만 대규모 애플리케이션(많은 컴포넌트로 구성된)에서는 성능 문제가 발생할 수 있다.

### 3-2. OnPush

다른 옵션으로는 ChangeDetectionStrategy.OnPush이 있다.
OnPush 컴포넌트 및 자식 컴포넌트에 대해서 불필요한 검사를 skip 할 수 있다.

```ts
@Component({
  selector: "test",
  changeDetection: ChangeDetectionStrategy.OnPush,
  // ...
})
export class Test {
  // ...
}
```

![img](/assets/img/angular/changedetection3.png)

![img](/assets/img/angular/changedetection4.png)

OnPush 전략을 사용하면 Angular는 오직 다음과 같은 상황에만 Change Detection을 발생시킨다.

- Input 참조 변경될 때
- 컴포넌트 또는 컴포넌트의 자식 컴포넌트의 이벤트 핸들러가 트리거 될 때
- 명시적으로 Change Detection이 실행될 때
- 템플릿에서 Async pipe를 통해 연결된 Observable이 새로운 value를 emit 할 때

#### Input 참조 변경

ChangeDetectionStrategy.Default를 사용할 때, Angular는 `@Input` data가 바뀌거나, 수정될 때마다 Change Detector를 실행한다.
그러나 OnPush 전략을 사용할 때 오직 새로운 참조가 @Input의 값으로 전달될 때에만 Change Detection이 트리거 됩니다.

@Input으로 primitive 타입을 받는 경우 value 타입이므로 Chnage Detection이 발생한다.

@Input으로 참조 타입인 Object나 Array을 받는 경우 property 수정이나 배열의 요소를 수정하는 것은 참조가 변경되지 않으므로 Change Detection을 발생시키지 않고,
오직 새로운 object, array의 참조를 전달할 때 Change Detection이 발생한다.

#### 이벤트 핸들러 트리거

OnPush 컴포넌트 또는 자식 컴포넌트의 이벤트 핸들러가 트리거 될 때 Change Detection이 발생한다.
이벤트 핸들러가 아닌 다음과 같은 비동기 작업은 OnPush 컴포넌트에서 Change Detection이 발생하지 않는다.

- setTimeout
- setInterval
- Promise
- this.http.get('...').subscribe()

#### 명시적 Change Detection 실행

Chnage Detection을 명시적으로 실행시키는 방법

```ts
abstract class ChangeDetectorRef {
  abstract markForCheck(): void;
  abstract detach(): void;
  abstract detectChanges(): void;
  abstract checkNoChanges(): void;
  abstract reattach(): void;
}
```

##### detectChange()

ChangeDetectorRef의 detectChange 메서드는 현재 컴포넌트와 자식 컴포넌트에 대해서 Change Detection Strategy를 적용하여 Change Detection을 발생시킨다.
detach 메서드와 함께 사용하여 로컬 Change Detection을 구현할 수 있다.

```ts
class GiantList {
  constructor(
    private ref: ChangeDetectorRef,
    public dataProvider: DataListProvider
  ) {
    ref.detach();
    setInterval(() => {
      this.ref.detectChanges();
    }, 5000);
  }
}
```

위 예제는 큰 데이터를 자주 읽어야 하는 컴포넌트 예시이다.
성능 향상을 위해 컴포넌트의 ChangeDetector를 detach 하고 5초마다 명시적으로 로컬 Change Detection을 발생시킨다.

##### ApplicationRef.tick()

ApplicationRef.tick() 메서드는 전체 애플리케이션에 대해서 Change Detection Strategy를 적용하여 Change Detection을 발생시킨다.

##### markForCheck()

만약 @Input이 변경되거나 Event가 발생하는 경우 View는 dirty flag가 켜진다.
만약 트리거가 발생하지 않더라도 dirty flag를 키고 싶은 경우 사용한다.

즉, View가 OnPush 전략을 사용하는 경우 View의 dirty flag를 켜서(checkOnce) 다시 검사할 수 있도록 한다.

ChangeDetectorRef의 markForCheck() 메서드는 Change Detection을 발생시키지는 않지만,
dirty flag를 키므로 모든 OnPush 부모(부모의 부모...) 컴포넌트들을 현재 또는 다음 Change Detection Cycle에 한 번씩 검사한다.

만약 OnPush 전략을 사용하더라도, markForCheck()가 호출된 컴포넌트에 한해서는 Change Detection을 수행한다.

![img](/assets/img/angular/changedetection5.png)

#### Async Pipe

AsyncPipe는 OnPush 전략을 사용합니다.
AsyncPipe를 subscribe 하면 가장 최근에 emit 한 값을 리턴한다.

내부적으로 AsyncPipe는 새로운 값이 emit될 때마다, `markForCheck`를 호출한다.

```ts
private _updateLatestValue(async: any, value: Object): void {
  if (async === this._obj) {
    this._latestValue = value;
    this._ref.markForCheck();
  }
}
```

### 참고

- [The Last Guide For Angular Change Detection You'll Ever Need](https://mokkapps.de/blog/the-last-guide-for-angular-change-detection-you-will-ever-need/)
- [[번역] Zones(zone.js)를 리버스 엔지니어링해서 찾은 것](https://norux.me/65)
- [[번역] 아직도 NgZone이 단순하게 Angular의 변화감지(Change Detection)를 위해서만 필요하다고 생각하시나요?](https://norux.me/66)
- [angular.io](https://angular.io/guide)
- [Angular Change Detection - How Does It Really Work?](https://blog.angular-university.io/how-does-angular-2-change-detection-really-work/)
