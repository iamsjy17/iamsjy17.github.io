---
layout: post
title: "[RxJS] Observable 생성 함수 - create"
date: 2020-09-30 18:00:00
author: Jewoo.Song
categories:
  - RxJS
tags:
  - RxJS 생성 함수
  - RxJS create
  - Observable
  - Observable 생성
---

## Observable 생성

Observable을 생성하기 위해 new 연산자로 Observable class의 생성자를 통해 생성할 수도 있지만, RxJS에서는 Observable을 생성할 수 있는 다양한 생성 함수를 제공한다.

알맞은 생성 함수를 사용해서 목적에 맞는 Observable을 생성할 수 있다. 그중 가장 기본이 되는 생성 함수인 create를 먼저 살펴보고 전반적인 Observable과 observer의 내부 동작에 대해 소개한다.

## create 함수

create 함수는 Observer가 subscribe 할 때 실행할 `onSubscription` function을 인자로 받아서 새로운 옵저버블을 생성하는 함수이다.

![img](/assets/img/rxjs/2020-09-30-inside-rxjs-creation-fn-create1.png)

create는 내부적으로 Observable 생성자를 호출하여 새로운 콜드 옵저버블을 생성한다.

```ts
//https://github.com/ReactiveX/rxjs/blob/master/src/internal/Observable.ts
static create: Function = <T>(subscribe?: (subscriber: Subscriber<T>) => TeardownLogic) => {
    return new Observable<T>(subscribe);
};
```

이렇게 생성된 옵저버블을 subscribe할 때마다 인자로 받은 Observer가 onSubscription 함수로 전달되고 onSubscription 함수에서 Observer의 `next` , `error` , `complete` 메서드를 호출한다.

### 예제 1. observer.complete()

```ts
//http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#static-method-create
const observable = Rx.Observable.create(function (observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete();

  //complete 이후는 실행되지 않는다.
  observer.next(4);
  observer.error(new Error("error!!"));
  observer.complete();
  console.log("after complete");
});

observable.subscribe(
  (value) => console.log(value),
  (err) => {},
  () => console.log("complete")
);

// Logs
// 1
// 2
// 3
// "complete"
// "after complete"
```

위 예제는 3개의 숫자를 emit 하고 complete을 호출하는 Observable이다.

한 가지 특이한 점은 complete이 호출된 이후 `next` , `error` , `complete` 는 실행되지 않는다는 것이다.

complete이 호출되면 내부적으로 unsubscribe가 호출된다. 즉, 구독이 해제되기 때문에 이후 호출되는 observer 메서드는 수행되지 않는다.

<br>

```ts
//Subscriber의 complete 함수(Observe interface의 구현 class)
//https://github.com/ReactiveX/rxjs/blob/master/src/internal/Subscriber.ts
protected _complete(): void {
  this.destination.complete();
  // subscribe의 인자로 받은 Observer의 complete 호출

  this.unsubscribe();
}

unsubscribe(): void {
  if (!this.closed) {
    this.isStopped = true;
    super.unsubscribe();
  }
}
```

<br>

unsubscribe가 호출되면 `Subscriber` 내부적으로 isStopped flag가 켜지고 이후에 동작하는 메서드는 아래와 같이 튕기게 된다.

```ts
//https://github.com/ReactiveX/rxjs/blob/master/src/internal/Subscriber.ts
next(value?: T): void {
  if (!this.isStopped) {
    this._next(value!);
  }
}

error(err?: any): void {
  if (!this.isStopped) {
    this.isStopped = true;
    this._error(err);
  }
}

complete(): void {
  if (!this.isStopped) {
    this.isStopped = true;
    this._complete();
  }
}
```

그러나 "after complete" 로그는 찍히는 것을 볼 수 있다. 즉, complete 함수를 실행했더라도 onSubscription 함수는 계속해서 실행된다. (flag에 의해 이후 observer 메서드가 수행되지 않을 뿐.)

따라서 onSubscription 함수를 구현할 때 구독이 완료된 이후 불필요한 동작이 일어나지 않도록 구현해야 한다.

<br>

### 예제 2. observer.error()

```ts
//http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#static-method-create
const observable = Rx.Observable.create((observer) => {
  observer.error("something went really wrong...");

  return () => {
    console.log("unsubscribe");
  };
});

observable.subscribe(
  (value) => console.log(value), // will never be called
  (err) => console.log(err),
  () => console.log("complete") // will never be called
);

// Logs
// "something went really wrong..."
// "unsubscribe"
```

<br>

위 예제에서 볼 수 있듯이 observer.error를 실행하면 구독을 해제하는 `TeardownLogic` (unsubscribe) 함수가 실행된다.

```ts
//https://github.com/ReactiveX/rxjs/blob/master/src/internal/Subscriber.ts
protected _error(err: any): void {
  this.destination.error(err);
  this.unsubscribe();
}
```

그러면 만약 명시적으로 observer.error()를 호출해 준 것이 아니라 onSubscription 함수 내에서 error가 발생한 상황에서는 어떻게 될까?

<br>

### 예제 3. 실행 중에 에러가 발생하는 경우

```ts
const observable = Rx.Observable.create((observer) => {
  observer.next(1);
  observer.next(2);
  observer.next(3);

  consolee.log("오타");

  observer.complete();

  return () => {
    console.log("unsubscribe");
  };
});

observable.subscribe(
  (value) => console.log(value),
  (err) => {
    console.log("error 함수 호출");
    console.log(err);
  },
  () => console.log("complete")
);

// Logs
// "1"
// "2"
// "3"
// "error 함수 호출"
// "ReferenceError: consolee is not defined"
```

<br>

onSubscription 함수 내에서 오타를 내서 강제로 에러를 발생시켰다. 이때도 observer의 error 함수를 호출해서 에러 메시지를 출력해준다.

> 그러나 unsubscribe 함수는 호출되지 않는다.

이유는 아래 코드와 같이 subscribe 함수 내에서 observer를 추가하기 전에(`subscriber.add()`) subscribe 하는 과정(`_trySubscribe()`) 에서 에러가 발생하고 이에 따라 catch 문에서 `obserber.error()` 를 호출하고 subscriber.add는 수행되지 않기 때문이다.

<br>

```ts
//https://github.com/ReactiveX/rxjs/blob/master/src/internal/Observable.ts
subscribe(
    observerOrNext?: PartialObserver<T> | ((value: T) => void) | null,
    error?: ((error: any) => void) | null,
    complete?: (() => void) | null
  ): Subscription {
    const subscriber = isSubscriber(observerOrNext) ? observerOrNext : new SafeSubscriber(observerOrNext, error, complete);

    const { operator, source } = this;
    subscriber.add(
      operator
        ? operator.call(subscriber, source)
        : source || config.useDeprecatedSynchronousErrorHandling
        ? this._subscribe(subscriber)
        : this._trySubscribe(subscriber)
    );

    return subscriber;
}

//...

protected _trySubscribe(sink: Subscriber<T>): TeardownLogic {
    try {
      return this._subscribe(sink);
			// onSubscription 메서드 내에서 에러 발생
    } catch (err) {
      if (config.useDeprecatedSynchronousErrorHandling) {
        throw err;
      } else {
        // observer.error 실행
        canReportError(sink) ? sink.error(err) : reportUnhandledError(err);
      }
    }
 }
```

사용자가 신경 쓰지 않아도 에러 발생 시에 observer.error를 호출해 주는 것은 RxJS의 좋은 기능이지만 만약 onSubscription 함수 내에서 반드시 teardownLogic이 수행되어야 하는 작업이 수행된다면 문제가 될 수 있다.

에러 발생 시에도 온전하게 subscribe/unsubscribe가 수행되도록 하기 위해서는 onSubscribe 함수 내에서 에러 처리를 하도록 하면 된다.

<br>

### 예제 4. 실행 중에 에러가 발생하는 경우 (try-catch)

```ts
const observable = Rx.Observable.create((observer) => {
  try {
    observer.next(1);
    observer.next(2);
    observer.next(3);

    consolee.log("오타");

    observer.complete();
  } catch (e) {
    observer.error(e);
  }

  return () => {
    console.log("unsubscribe");
  };
});

observable.subscribe(
  (value) => console.log(value),
  (err) => {
    console.log("error 함수 호출");
    console.log(err);
  },
  () => console.log("complete")
);

// Logs
// "1"
// "2"
// "3"
// "error 함수 호출"
// "ReferenceError: consolee is not defined"
// "unsubscribe"
```

에러 처리를 바깥에서 함으로써 내부적으로는 subscribe와 `subscriber.add()` 가 온전하게 수행되고 catch 문 내에서 `observer.error()` 를 호출해주므로 teardownLogic이 수행된다.

## 참고

- [https://www.learnrxjs.io/learn-rxjs/operators/creation/create](https://www.learnrxjs.io/learn-rxjs/operators/creation/create)
- [http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#static-method-create](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#static-method-create)
- [https://github.com/ReactiveX/rxjs/blob/master/src/internal](https://github.com/ReactiveX/rxjs/tree/master/src/internal)
- RxJS 프로그래밍 - (한빛 미디어 / 이종욱, 안재하)
