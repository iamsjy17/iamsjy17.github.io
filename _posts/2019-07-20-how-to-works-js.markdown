---
layout: post
title: "[Javascript] 자바스크립트의 호출 스택과 이벤트 루프"
date: 2019-07-20 15:00:00
author: 송튜디오
categories: Javascript
tags: 호출스택 이벤트루프 callstack javascriptengine eventloop taskqueue microtask requestanimationframe
---

## 자바스크립트의 호출 스택과 이벤트 루프

자바스크립트는 `single-thread`기반의 언어입니다. 즉, 자바스크립트는 하나의 호출 스택을 가집니다.
하나의 호출 스택을 가진 단일 스레드로 동작하는 자바스크립트에서 어떻게 동시성을 지원할까요?
답은 이벤트 루프입니다. 자바스크립트는 이벤트 루프 기반의 비동기 방식으로 Non-Blocking IO를 지원합니다.

`Node.js`의 특징으로 자주 소개되는 내용입니다. `Node.js`가 V8(자바스크립트 엔진)으로 빌드 된 이벤트 기반 자바스크립트 런타임이기 때문에 브라우저에서의(`크롬 V8`) 자바스크립트와 동일한 특징을 가지고 있습니다.

> Node.js는 확장성 있는 네트워크 애플리케이션(특히 서버 사이드) 개발에 사용되는 소프트웨어 플랫폼이다. 작성 언어로 자바스크립트를 활용하며 Non-blocking I/O와 단일 스레드 이벤트 루프를 통한 높은 처리 성능을 가지고 있다.

> 크롬 V8는 웹 브라우저를 만드는 데 기반을 제공하는 오픈소스 자바스크립트 엔진이다. 구글 크롬 브라우저와 안드로이드 브라우저, Node.js 등에서 사용된다.

> Non-blocking I/O(Asynchronous I/O 혹은 Non-sequential I/O): Non-blocking I/O란, 입출력 처리는 시작만 해둔 채 완료되지 않은 상태에서 다른 처리 작업을 계속 진행할 수 있도록 멈추지 않고 입출력 처리를 기다리는 방법을 말한다

### 1. 자바스크립트 엔진의 구성요소

아래는 자바스크립트 엔진의 모습입니다.

![Alt Javascript Engine](/assets/img/howtoworksjs/engine.png)

- 자바스크립트 엔진
  - Heap : 메모리 할당이 일어나는 영역
  - Call Stack : 코드 실행에 따라 호출 스택이 쌓이는 영역

### 2. 런타임

위 그림에서도 알 수 있지만 중요한 사실은 자바스크립트에는 이벤트 루프가 없습니다. V8과 같은 자바스크립트 엔진은 단일 호출 스택을 사용하고, 요청이 들어오면 그 순서에 따라 순차적으로 처리할 뿐입니다.

그러면 비동기 요청은 어떻게 처리할까요?

자바스크립트 엔진을 구동하는 런타임 환경(브라우저나 Node.js)이 담당합니다.

- 런타임 환경이 제공하는 것
  - Web APIs
    - DOM(document)
    - AJAX(XMLHttpRequest)
    - Timeout(setTimeout)
  - Event Loop

이렇게 런타임 환경과 자바스크립트 엔진이 만나서 익숙한 이벤트 루프 그림이 완성됩니다.

![Alt Javascript EventLoop](/assets/img/howtoworksjs/eventloop.png)

위 그림과 같이 우리가 비동기 호출을 위해 사용하는 Web API과 Event Loop, Task Queue는 자바스크립트 엔진 외부에 런타임 환경에 구현이 되어있습니다.

### 3. 단일 호출 스택

자바스크립트는 호출 스택이 하나입니다. 이것의 의미는 하나의 함수가 실행되면 이 함수의 실행이 끝날 때까지 다른 작업이 중간에 끼어들지 못한다는 의미입니다.

함수가 실행되면 해당 함수는 호출 스택의 가장 상단에 위치하고, 함수의 실행이 끝날 때 해당 함수는 호출 스택에서 제거됩니다.

> 호출 스택: 현재 프로그램 상에서 어디에 있는지를 기록하는 자료 구조



#### 단일 호출 스택(single-thread)의 단점

브라우저에서 호출 스택에 실행할 함수가 쌓여있는 동안은 다른 일을 할 수 없습니다. 이 상태를 `blocked`라 합니다. 이 상태에서 브라우저는 렌더링을 할 수도 없고, 다른 코드를 실행할 수도 없습니다.

- 브라우저 자체가 동작을 하지 않음.
- 매끄러운 화면 UI를 제공하지 못하게 됨.
- 프로그램 동작 중에 이러한 현상은 사용자 경험을 완전히 망가뜨리는 일이다.

```js
var count = 0;
function stack() {
  console.log(++count);
  stack();
}
stack();
```

- Stack Overflow

![Alt StackOverFlow](/assets/img/howtoworksjs/stackoverflow1.png)



- Error

![Alt StackOverFlow](/assets/img/howtoworksjs/stackoverflow2.png)



- 브라우저 동작 멈춤

![Alt StackOverFlow](/assets/img/howtoworksjs/stackoverflow.gif)

### 4. 동시성과 이벤트 루프

위 예제는 극단적으로 재귀 호출을 하는 예제이지만, 재귀 호출이 아니더라도 비슷한 현상은 충분히 발생할 수 있습니다. 호출 스택에 처리 시간이 엄청나게 오래 걸리는 함수가 들어온다면(실행된다면) 위 예제와 같이 함수가 실행되는 동안 브라우저는 아무 작업도 하지 못하는 `blocked` 상태가 되고 대기 상태가 됩니다.

함수의 내부 호출 스택이 `12540개`까지 늘어난다면 stack overflow가 발생할 수도 있고요.

> 12540개는 정확한 숫자가 아니라 제 환경에서 테스트 1회 했을 때의 결과입니다. 대충 큰 숫자라고 여기시면 될 것 같습니다.


이러한 문제를 해결하기 위해 이벤트 루프를 통한 동시성 확보를 해야 합니다.

> 적절하게 task를 쪼개서 비동기 호출을 하고, 또 중간중간 렌더링등 UI 갱신이 이루어질 수 있도록 호출 스택이 빈 상태가 되도록 해주어야 한다.



#### 이벤트 루프

이벤트 루프는 하나의 단순한 동작만을 수행합니다. 호출 스택과 Task Queue를 감시하면서, 만약 호출 스택이 비어있다면 이벤트 루프는 큐에서 첫 번째 Task를 호출 스택에 넣고 해당 Task가 수행됩니다.

![Alt Event Loop](/assets/img/howtoworksjs/eventloop2.png)

이러한 반복을 이벤트 루프에서는 `tick`이라고 합니다.

-  task queue

MDN에서 Event Loop을 보면 다음과 같이 간이 코드가 나옵니다.
task queue는 message를 기다리고 message가 들어오면 task queue에 추가합니다.

```js
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

-  event loop

그리고 이벤트 루프는 가장 오래된 메시지부터 시작해서 메시지를 처리합니다.
메시지를 처리한다는 것은 함수를 실행해서 호출 스택에 올린다는 뜻입니다.
메시지가 처리되면 또 호출 스택이 빌 때까지 기다렸다가 메시지를 처리하고 하는 동작을 반복합니다.(`tick`)

```js
while (eventLoop.waitForTask()) {
  const taskQueue = eventLoop.selectTaskQueue();
  if (taskQueue.hasNextTask()) {
    taskQueue.processNextTask();
  }
}
```

그런데 이벤트 루프가 다발적으로 발생한 메시지들을 큐에 쌓고 실행을 해주면서 동시성을 확보하는 것이지 실제로 동시에 동작이 수행되는 것은 아닙니다.

실제 실행 자체는 호출 스택에 올라가서 수행이 되므로 Run-to-completion 으로 동작합니다.


> Run-to-completion : Each message is processed completely before any other message is processed.



### 5. Task Queue vs Microtask Queue Animation vs Animation Frames

앞에 까지는 모든 비동기 동작이 Task Queue에 쌓이는 것처럼 설명을 했는데, 실제로는 여러 Queue가 존재합니다.

ES6에 들어오면서 새로운 컨셉인 `Microtask Queue`가 도입되었습니다. Microtask Queue는 Task Queue와 동일한 계층에 존재하고 프로미스의 비동기 호출 시 Microtask Queue에 쌓이게 됩니다.



#### 1) Microtask Queue vs Task Queue

다음과 같은 코드를 실행했을 때 결과는 어떻게 나올까요?

```js
console.log("script start");

setTimeout(function() {
  console.log("setTimeout");
}, 0);

Promise.resolve()
  .then(function() {
    console.log("promise1");
  })
  .then(function() {
    console.log("promise2");
  });

console.log("script end");
```

만약 모든 비동기 호출이 단일 task queue에 의해 관리되고, Event Loop는 호출 스택이 비었을 때 task queue에서 순서대로 꺼낸다면 아래와 같은 결과가 나올 겁니다.

```js
script start
script end
setTimeout
promise1
promise2
```

그러나 실제로는 아래와 같이 출력됩니다.

```js
script start
script end
promise1
promise2
setTimeout
```

이것에 대해 이해하기 위해서는 브라우저의 이벤트 루프가 `task`와 `microtask`를 어떻게 다루는지 알아야 합니다.

- 브라우저의 이벤트 루프 우선순위
  - 이벤트 루프는 실행 순서를 보장하는 여러 queue에서 어떤 task를 꺼내서 실행시킬지 결정한다.
  - 이를 통해 브라우저는 우선순위가 높은 task를 먼저 실행하도록 할 수 있다.
  - microtask는 일반 task보다 높은 우선순위를 가지고 있다.

앞에 예제 코드를 좀 더 자세히 살펴보겠습니다.

> https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/ 에서 애니메이션으로 실행 순서를 볼 수 있습니다.

```js
//1. script 실행 (log)
console.log("script start");

//2. script 실행 (setTimeout callback task queue에 등록)
setTimeout(function() {
  //9. Task 실행
  console.log("setTimeout");
}, 0);

//3. script 실행 (Promise then callback Microtask queue에 등록)
Promise.resolve()
  .then(function() {
    // 6. MicroTask 실행
    console.log("promise1");
  }) // 7. script 실행 (Promise then callback Microtask queue에 등록)
  .then(function() {
    // 8. MicroTask 실행
    console.log("promise2");
  });

//4. script 실행 (log)
console.log("script end");
//5. Stack의 모든 Task 실행완료
```



#### 2) Microtask Queue vs Task Queue vs Animation Frames

Microtask 외에도 Queue는 또 있습니다. 바로 requestAnimationFrame에 의해 등록되는 `Animation Frames`입니다.

예제를 일부만 수정해서 Animation Frame을 추가해보았습니다.

```js
//1. script 실행 (log)
console.log("script start");

//2. script 실행 (setTimeout callback task queue에 등록)
setTimeout(function() {
  //11. Task 실행
  console.log("setTimeout");
}, 0);

//3. script 실행 (Promise then callback Microtask queue에 등록)
Promise.resolve()
  .then(function() {
    // 7. MicroTask 실행
    console.log("promise1");
  }) // 8. script 실행 (Promise then callback Microtask queue에 등록)
  .then(function() {
    // 9. MicroTask 실행
    console.log("promise2");
  });

//4. script 실행 (AnimationFrame Animation frames에 등록)
requestAnimationFrame(function() {
  //10. Animation Frame 실행
  console.log("animation");
});

//5. script 실행
console.log("script end");
//6. Stack의 모든 Task 실행완료
```

결과는 다음과 같습니다.

```js
script start
script end
promise1
promise2
animation
setTimeout
```

정리하자면 자바스크립트는 비동기 작업을 수행할 때 Web API를 통해 여러 queue에 등록된 작업들을 우선순위에 따라 꺼내서 처리합니다.



##### 이벤트 루프의 우선순위

1. 호출 스택의 작업을 처리한다.
2. 호출 스택이 비어있다면 microtask queue를 확인하고 작업이 있다면 microtask queue의 task를 작업을 호출 스택으로 넣고 처리한다.
3. 만약 microtask가 비어있다면 Animation Frames를 확인하고 브라우저 렌더링이 발생한다.
4. 1, 2, 3번 작업이 완료되었다면 task queue를 확인하고 작업이 있다면 task queue의 작업을 호출 스택으로 넣고 처리한다.

> Animation Frame은 Vsync에 맞춰서 호출되므로 task보다 후에 호출될 수도 있습니다.
> Input과 같은 Event 처리는 Microtask, task, Animation Frame보다 높은 우선순위를 가집니다. 다음 글에서 설명하도록 하겠습니다.

![default](/assets/img/howtoworksjs/eventloop3.png)

> 이러한 동작들은 브라우저마다 호출 순서가 다를 수 있습니다.
> promise가 ECMA Spec이므로 브라우저마다 처리하는 방식이 다르기 때문입니다.
> 특정 브라우저에서는 promise를 microtask로 처리하는 것이 아니라 task로 처리합니다.
> 이 글에서는 크롬이 올바른 스펙이라고 가정하고 크롬의 동작에 대해서만 다루겠습니다.
> 자세한 비교를 보시려면 아래 링크를 참고해주세요.
> https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/



## 참고

- http://sculove.github.io/blog/2018/01/18/javascriptflow/
- https://blog.risingstack.com/writing-a-javascript-framework-execution-timing-beyond-settimeout/
- https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop
- https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf
- https://blog.sessionstack.com/how-javascript-works-event-loop-and-the-rise-of-async-programming-5-ways-to-better-coding-with-2f077c4438b5
- https://meetup.toast.com/posts/89
- https://ko.wikipedia.org/wiki/크롬_V8
- https://ko.wikipedia.org/wiki/Node.js
- https://tech.peoplefund.co.kr/2017/08/02/non-blocking-asynchronous-concurrency.html#fn:2
