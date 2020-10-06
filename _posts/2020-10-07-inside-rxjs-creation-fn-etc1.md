---
layout: post
title: "[RxJS] Observable 생성 함수2 - of, from, fromEvent"
date: 2020-10-07 2:00:00
author: Jewoo.Song
categories:
  - RxJS
tags:
  - RxJS 생성 함수
  - RxJS of
  - RxJS from
  - RxJS fromEvent
  - Observable
  - Observable 생성
---

## 1. of

of 함수는 입력받은 인자를 순서대로 발행하는 옵저버블을 생성한다.

```tsx
of<T>(...args: (SchedulerLike | T)[]): Observable<T>
```

<br>

![img](/assets/img/rxjs/2020-10-07-inside-rxjs-creation-fn-etc1-1.png)

<br>

```tsx
//https://github.com/ReactiveX/rxjs/blob/master/src/internal/observable/of.ts

export function of<T>(...args: Array<T | SchedulerLike>): Observable<T> {
  const scheduler = popScheduler(args);
  return scheduler
    ? scheduleArray(args as T[], scheduler)
    : internalFromArray(args as T[]);
}
```

코드에서 보면 알 수 있듯이 실제 값을 발행할 때는 입력받은 인자를 배열로 변환하여 순서대로 발행한다.

그리고 마지막 인자가 RxJS의 Scheduler 타입이면 해당 스케줄러를 적용한다. (스케줄러가 없다면 일반적인 배열 iteration과 같이 동기로 값을 발행한다.)

```tsx
//https://github.com/ReactiveX/rxjs/blob/2c2fc8ef2d0484af252f848c268a584671e67fd2/src/internal/scheduled/scheduleArray.ts#L5

export function scheduleArray<T>(
  input: ArrayLike<T>,
  scheduler: SchedulerLike
) {
  return new Observable<T>((subscriber) => {
    let i = 0;
    return scheduler.schedule(function () {
      if (i === input.length) {
        subscriber.complete();
      } else {
        subscriber.next(input[i++]);
        if (!subscriber.closed) {
          this.schedule();
        }
      }
    });
  });
}
```

<br>

> 스케줄러가 없는 경우 내부적으로 fromArrayLike 함수를 사용한다. 즉, from과 동일하게 동작한다.

```tsx
export function fromArrayLike<T>(array: ArrayLike<T>) {
  return new Observable((subscriber: Subscriber<T>) => {
    for (let i = 0; i < array.length && !subscriber.closed; i++) {
      subscriber.next(array[i]);
    }
    subscriber.complete();
  });
}
```

<br>

### Example

```tsx
//https://www.learnrxjs.io/learn-rxjs/operators/creation/of

import { of } from "rxjs";

const source = of(1, 2, 3, 4, 5);
const subscribe = source.subscribe((val) => console.log(val));

//1
//2
//3
//4
//5
```

## 2. from

from 함수는 다음과 같은 데이터 타입의 객체들을 옵저버블로 변환한다.

- Observable
- Array
- Iterable
- Promise
- String
- 배열 유사 타입(Array-like object)

이외의 다른 데이터 타입 객체는 옵저버블로 변환하지 못하고 TypeError를 발생시킨다.

```tsx
export function from<T>(
  input: ObservableInput<T>,
  scheduler?: SchedulerLike
): Observable<T> {
  return scheduler ? scheduled(input, scheduler) : innerFrom(input);
}
```

첫 번째 인자인 input은 옵저버블로 변환 가능한 데이터 타입을 넣고, 두 번째는 SchedulerLike 타입을 넣는다.

<br>

![img](/assets/img/rxjs/2020-10-07-inside-rxjs-creation-fn-etc1-2.png)

<br>

### 예제 1. Array

```tsx
from([1, 2, 3, 4, 5]).subscribe(
  (val) => console.log(val),
  null,
  () => console.log("complete")
);

//1
//2
//3
//4
//5
//complete
```

from이 정상적으로 완료되면 complete이 호출된다.

```tsx
//https://github.com/ReactiveX/rxjs/blob/2c2fc8ef2d0484af252f848c268a584671e67fd2/src/internal/observable/from.ts#L175

export function fromArrayLike<T>(array: ArrayLike<T>) {
  return new Observable((subscriber: Subscriber<T>) => {
    for (let i = 0; i < array.length && !subscriber.closed; i++) {
      subscriber.next(array[i]);
    }
    subscriber.complete();
  });
}
```

<br>

### 예제 2. Promise

resolve가 호출될 때 observer의 next, complete가 호출되고, reject가 호출될 때 observer의 error가 호출된다.

```tsx
from(
  new Promise((resolve, reject) => {
    resolve("resolve");
  })
).subscribe(
  () => {
    console.log("next");
  },
  (e) => {
    console.log(e);
  },
  () => {
    console.log("complete");
  }
);

// next
// complete
```

```tsx
from(
  new Promise((resolve, reject) => {
    reject(new Error("reject"));
  })
).subscribe(
  () => {
    console.log("next");
  },
  (e) => {
    console.log(e);
  },
  () => {
    console.log("complete");
  }
);

// Error: reject
```

<br>

## 3. fromEvent

지정한 이벤트 target element와 특정 타입의 이벤트를 발행하는 옵저버블을 생성한다.

```tsx
fromEvent<T>(
	target: FromEventTarget<T>,
	eventName: string,
	options?: EventListenerOptions | ((...args: any[]) => T),
	resultSelector?: (...args: any[]) => T
): Observable<T>
```

- target : 아래 타입의 이벤트 타겟 객체를 넣어서 이벤트 핸들러와 연결한다.
  - DOM EventTarget
  - Node.js EventEmitter
  - JQuery-like event target
  - NodeList or HTMLCollection
- eventName : 타겟이 발행할 이벤트 이름
- options : Optional. Default는 `undefined`이고 addEventListener에 전달할 옵션 객체를 지정한다.
- resultSelector : Optional. Default는 `undefined`이다. (type: `(...args: any[]) => T`)

<br>

![img](/assets/img/rxjs/2020-10-07-inside-rxjs-creation-fn-etc1-1.png)

<br>

### 예제 1. 기본 예제

```tsx
fromEvent(document, "click").subscribe((event) => console.log(event));
```

Observable이 subscribe될 때마다 fromEvent 함수의 `target` 인자로 지정한 element의 `eventName` 타입의 이벤트 핸들러가 등록된다.

그리고 해당 이벤트가 발생할 때 등록된 옵저버블에 등록된 next 함수의 첫 번째 인자로 값이 방출된다.

<br>

### 예제 2. 기본 예제(unsubscribe)

해당 이벤트 핸들러는 옵저버블이 unsubscribe 되기 전까지 유지된다. 따라서 명시적으로 unsubscribe를 호출해 주어야 한다.

```tsx
const sub = fromEvent(document, "click").subscribe((event) =>
  console.log(event)
);

// 이벤트 핸들러 해제
sub.unsubscribe();
```

옵저버블이 unsubscribe 될 때 해당 이벤트 핸들러가 제거된다. (removeEventHandler가 호출된다.)

```tsx
//https://github.com/ReactiveX/rxjs/blob/master/src/internal/observable/fromEvent.ts
export function fromEvent<T>(
  target: FromEventTarget<T>,
  eventName: string,
  options?: EventListenerOptions | ((...args: any[]) => T),
  resultSelector?: (...args: any[]) => T
): Observable<T> {
  //...
  return new Observable<T>((subscriber) => {
    const handler = (...args: any[]) =>
      subscriber.next(args.length > 1 ? args : args[0]);

    if (isEventTarget(target)) {
      target.addEventListener(
        eventName,
        handler,
        options as EventListenerOptions
      );
      return () =>
        target.removeEventListener(
          eventName,
          handler,
          options as EventListenerOptions
        );
    }

    //..
  });
}
```

> 만약 `이벤트 핸들러`가 동작하는 동안 에러가 발생하는 경우에는 어떻게 될까?

<br>

### 예제 3. 에러가 발생하는 경우

```tsx
fromEvent(document, "click").subscribe((event: any) => {
  console.log(event.noneFunction());
  //일부러 에러를 발생시킴.
});
```

등록된 이벤트 핸들러가 수행되는 도중에 에러가 발생하는 경우 내부적으로 unsubscribe가 호출되어서 unsubscribe 함수인 removeEventListener가 호출된다.

## 참고

- 참고
  - [https://rxjs.dev/api/index/function/of](https://rxjs.dev/api/index/function/of)
  - [https://rxjs.dev/api/index/function/from](https://rxjs.dev/api/index/function/from)
  - [https://rxjs.dev/api/index/function/fromEvent](https://rxjs.dev/api/index/function/fromEvent)
  - [https://www.learnrxjs.io/learn-rxjs/operators/creation/of](https://www.learnrxjs.io/learn-rxjs/operators/creation/of)
  - [https://github.com/ReactiveX/rxjs/blob/master/src/internal/observable/of.ts](https://github.com/ReactiveX/rxjs/blob/master/src/internal/observable/of.ts)
  - [https://github.com/ReactiveX/rxjs/blob/master/src/internal/observable/from.ts](https://github.com/ReactiveX/rxjs/blob/master/src/internal/observable/from.ts)
  - [https://github.com/ReactiveX/rxjs/blob/master/src/internal/observable/fromEvent.ts](https://github.com/ReactiveX/rxjs/blob/master/src/internal/observable/fromEvent.ts)
  - RxJS 프로그래밍 - (한빛 미디어 / 이종욱, 안재하)
