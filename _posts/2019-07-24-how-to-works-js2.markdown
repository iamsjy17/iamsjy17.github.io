---
layout: post
title: "[Javascript] 비동기 스케줄링과 Frame의 LifeCycle"
date: 2019-07-20 15:00:00
author: Jewoo.Song
categories: Javascript
tags: 호출스택 이벤트루프 callstack javascriptengine eventloop taskqueue microtask requestanimationframe beginframe 비동기 비동기스케줄링
---

## 비동기 스케줄링

이전에 포스팅했던 호출 스택과 이벤트 루프에서 자바스크립트에서 동시성 확보를 위해 비동기 호출과 이벤트 루프를 통한 스케줄링을 하는 것에 대한 내용을 다뤘습니다.

> [자바스크립트의 호출 스택과 이벤트 루프](https://iamsjy17.github.io/javascript/2019/07/20/how-to-works-js.html)

이번에는 Input Event 등의 Event와 Task, MicroTask, Animation Frame 과의 우선순위와 여러 가지 실험? 들 그리고 Frame의 라이프 사이클에 대해서 다뤄보겠습니다.

### 1. 비동기 task 우선순위

각 비동기 task 들과 Input Event의 우선순위를 알아보기 위한 예제입니다.

```js
function onKeyDown(){
    console.log(`onKeyDown`);
     
    requestAnimationFrame(function() {
        console.log(`animation`);
    });

    Promise.resolve()
      .then(function() {
        console.log(`promise`);
      });

    setTimeout(function() {
        console.log(`setTimeout`);
    }, 0);
}
window.addEventListener('keydown', onKeyDown);
```

결과는 아래와 같습니다.

```js
onKeyDown
promise
animation
setTimeout
```

onKeydown(onKeydown가 호출 스택에 올라갔을 때)에서 비동기 호출을 하므로, 위 예제는 적절하지 않습니다.

<br/>

> 제대로 테스트를 해보려면 onKeydown에서 다시 key 입력이 들어올 만큼 `충분히 오랜 시간 script가 호출되고` keydown이 연속해서 들어온다면 한 번의 onKeydown이 끝났을 때에 이벤트 루프와 큐가 어떻게 동작할지를 테스트해봐야 합니다.

<br/>

첫 번째 onKeydown 처리 전에 다시 keydown event가 발생할 때의 이벤트 루프를 확인해보도록 하겠습니다.

#### 1-1. 테스트 예제

onKeyDown에서 엄청나게 오랜 시간이 걸리는 작업을 수행하도록 하여 Call Stack에 함수가 수행되고 있는 중에 Input Event가 여러 번 들어올 수 있도록 예제를 만들어 보았습니다.

> 대략 190ms의 시간이 걸리는 무식한 테스트입니다.

```js
var keydownCount = 0;
var rafCount = 0;
var promiseCount = 0;
var settimeoutCount = 0;
function onKeyDown(){
    var start = performance.now();    
    console.log(`onKeyDown ${++keydownCount}`);
     
    requestAnimationFrame(function() {
        console.log(`animation ${++rafCount}`);
    });

    Promise.resolve()
      .then(function() {
        console.log(`promise ${++promiseCount}`);
      });

    setTimeout(function() {
        console.log(`setTimeout ${++settimeoutCount}`);
    }, 0);

    for(var i =0; i < 400000; i++){
        JSON.stringify(([].slice().concat([1])).splice(0,1));
    }  
    var end = performance.now();
    console.log(end-start);
}
window.addEventListener('keydown', onKeyDown); 
//189.81499999972584
```

#### 1-2. 2번의 키 입력이 들어온 상황

위와 같은 예제가 있을 때 먼저 예상을 해보도록 하겠습니다.
첫 번째 키 입력이 들어오고 onKeydown이 호출되고, 아직 호출 스택에서 수행하는 도중에 keydown Event가 다시 들어온 상황입니다. 

아래와 같은 순서로 비동기 Task들이 등록된 상태에서 Call Stack이 Idle 상태가 되었을 때 어떤 Task가 먼저 들어올까요?

1. onKeyDown 실행
2. requestAnimationFrame 등록
3. Promise 등록
4. setTimeout 등록
5. onKeyDown 아직 실행 중
   - keydown Event 발생
   - onKeydown 등록
6. `누가 실행될까?`

<br/>


#### 1) onKeyDown 실행

![medium](/assets/img/howtoworksjs/asyncPriority1.png)

<br/>


#### 2) requestAnimationFrame 등록

![medium](/assets/img/howtoworksjs/asyncPriority2.png)

<br/>


#### 3) Promise 등록

![medium](/assets/img/howtoworksjs/asyncPriority3.png)

<br/>


#### 4) setTimeout 등록

![medium](/assets/img/howtoworksjs/asyncPriority4.png)

<br/>

#### 5) onKeyDown 아직 실행 중

- keydown Event 발생
- onKeydown 등록

![medium](/assets/img/howtoworksjs/asyncPriority5.png)

<br/>


#### 6) 비동기 Task 실행

모든 Task Queue에 하나씩 비동기 Task들이 들어온 채로 콜스택이 Idle 상태가 되었을 때 아래와 같은 순서로 Task가 실행됩니다.

```js
onKeyDown 1
promise 1
onKeyDown 2
promise 2
animation 1
animation 2
setTimeout 1
setTimeout 2
```

테스트 결과를 정리하자면 Input Event와 promise(MicroTask)의 우선순위는 같습니다.

<br/>

> Input Event, Micro Task > Animation Frame > Task 순으로 비동기가 실행된다.

<br/>
<br/>


#### 1-3. 반복 테스트

이번에는 키를 10초가량 입력을 했을 때 어떻게 될지 테스트를 해보았습니다.
결과는 10초가량의 입력이 끝난 후에도 Event Queue 남은 Input Event를 처리하느라 13초가량의 시간이 추가로 걸렸습니다.
그리고 이 총 23초의 시간 동안 settimeout과 animation frame은 실행되지 않고 모든 Input과 promise가 실행된 후 실행되었습니다.

`명백한 stavation 상태에 빠지게 됩니다.`

Event Handler에서 무거운 작업을 수행한다면 Rendering이 수행되지 않는다는 것을 눈으로, 수치로 확인할 수 있습니다.

> test demo는 1.5 배속입니다.

![default](/assets/img/howtoworksjs/asynctest.gif)


재귀 호출로 Stack Overflow가 발생하는 것도 아니고, 동일한 시점에 각 비동기 Task를 하나씩 쌓고 있는데 왜 이런 현상이 발생할까요? 
앞에서 말했던 대로 비동기 Task들의 우선순위 차이 때문에 이런 stavation 상태가 됩니다.

#### 테스트에서 중요한 두 가지

1. Vsync 내에 실행이 완료되지 못하더라도 Input Event는 막히지 않습니다.  
  - queue에 쌓인 Event들은 결국 다 실행이 됩니다.
2. 각 비동기 Task의 우선순위가 다릅니다.
  - 비동기 스케줄링을 잘 못한다면 심각하게 frame이 drop 될 수 있습니다.


### 2. Life of a frame

Input Event가 rAF 보다 높은 우선순위를 가지는 것은 브라우저 프레임에서 이벤트의 라이프 사이클을 보면 알 수 있습니다.

브라우저는 Input Event가 있다면 Input Event를 먼저 처리하고 requestAnimationFrame을 호출합니다. 


- Life of a frame

![default](/assets/img/howtoworksjs/AframeinChromium.png)

- Event Dispatch Diagram
<br/>
![default](/assets/img/howtoworksjs/Eventdispatchdiagram.png)


먼저 Input을 처리하고 렌더링을 한다는 것이니까 합리적이라고 볼 수 있습니다.
그러나 위에서 실험해본 상황과 같이 극단적인 상황에서는 브라우저가 계속해서 Input Event만 처리하는 상황이 오게 됩니다.

### 3. 결론

이러한 현상을 해결하는 가장 좋은 방법은 이런 상황을 만들지 않는 것입니다.
Input Event Handler에서 오랜 시간 걸리는 작업을 하지 않는 것이 좋습니다.

그런데 만약 해야 한다면 Event Handler에서 바로 모든 작업을 해주는 것이 아니라 작업을 쪼개서 동기, 비동기로 적절하게 우선순위를 조정하여 호출해주어야 합니다.

브라우저가 렌더링 할 수 있는 시간을 줍시다! 

다음 포스팅에서는 이런 상황에서 어떻게 비동기로 작업을 호출하는 것이 효율적일지에 대해서 포스팅해보도록 하겠습니다. 

## 참고

- https://medium.com/@paul_irish/requestanimationframe-scheduling-for-nerds-9c57f7438ef4
- https://docs.google.com/drawings/d/1bUukRm-DV34sM7rL2_bSdxaQkZVMQ_5vOa7nzDnmnx8/edit
- https://docs.google.com/presentation/d/1e-aNC_urs4BiAilWPK3MYcf6UZu8rhluGUttNOy6iVY/edit#slide=id.g1459cdb6e2_0_17

