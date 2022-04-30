---
layout: post
title: "[Svelte] 스벨트는 어떻게 상태 변경을 DOM에 반영할까? (스벨트의 동작 원리)"
date: 2022-04-30 14:00:00
author: Jewoo.Song
categories: Svelte
tags:
  - Svelte
  - Svelte 동작 원리
  - Svelte 메커니즘
  - 스벨트 동작 원리
  - Svelte 컴파일
  - Svelte 컴파일 결과물 분석
  - 스벨트 컴파일
---

스벨트 공식 문서의 설명을 보면 스벨트는 React, Vue, Angular와 같은 기존 프레임워크에서 브라우저에서 하는 많은 작업을 컴파일 단계로 옮겼다고 합니다.

컴파일 단계에서 Virtual DOM diffing과 같은 기술을 사용하는 `대신 앱의 상태가 변경될 때 DOM을 직접 업데이트하는 코드`로 만들어 준다고 합니다.

어떻게 프레임워크가 없는 작은 js로 컴파일 되는데, Virtual DOM 보다 Incremental DOM 보다 빠르고, Store, Reactivity, Binding (단방향, 양방향) 을 모두 지원하는 코드가 다 포함될까?

그래서 컴파일된 js 파일을 뜯어보기로 했습니다.

## 앱 만들기

### 프로젝트 세팅

> Rollup bundler와 Webpack bundler 중 선택할 수 있다.

```bash
npx degit sveltejs/template svelte-app
#npx degit sveltejs/template-webpack hello-svelte
```

### **Vscode Svelte plugin 설치 (Svelte for VS Code)**

![svelte](/assets/img/svelte/svelte1.png)

### Svelte 실행

```bash
npm i
npm run dev
```

![svelte](/assets/img/svelte/svelte2.png)

## 컴파일 전/후 코드 비교

최대한 간단하게 $(반응성)을 포함하고 있는 코드를 분석해 보겠습니다.

### 컴파일 전

**App.svelte**

```html
<script>
  let count = 0;
  $: doubled = count * 2;

  function handleClick() {
    count += 1;
  }
</script>

<button on:click="{handleClick}">
  Clicked {count} {count === 1 ? 'time' : 'times'}
</button>

<p>{count} doubled is {doubled}</p>
```

**main.js**

```tsx
import App from "./App.svelte";

const app = new App({
  target: document.body,
});

export default app;
```

### 컴파일 후

> production 모드로 빌드 한 후 실행한다. 진짜 코딱지만한 번들이 매우 빠르게 만들어졌다..

![svelte](/assets/img/svelte/svelte3.png)

코드 보기에 용이하도록 dev 모드로 실행한 후 코드 분석해 보겠습니다.

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />

    <title>Svelte app</title>

    <script defer src="/build/bundle.js"></script>
  </head>

  <body></body>
</html>
```

**bundle.js**

```js
(function (l, r) {
  if (!l || l.getElementById("livereloadscript")) return;
  r = l.createElement("script");
  r.async = 1;
  r.src =
    "//" +
    (self.location.host || "localhost").split(":")[0] +
    ":35729/livereload.js?snipver=1";
  r.id = "livereloadscript";
  l.getElementsByTagName("head")[0].appendChild(r);
})(self.document);
var app = (function () {
  "use strict";

  //...

  function flush() {
    //...
  }
  function update($$) {
    if ($$.fragment !== null) {
      $$.update();
      run_all($$.before_update);
      const dirty = $$.dirty;
      $$.dirty = [-1];
      $$.fragment && $$.fragment.p($$.ctx, dirty);
      $$.after_update.forEach(add_render_callback);
    }
  }

  //...
  function make_dirty(component, i) {
    if (component.$$.dirty[0] === -1) {
      dirty_components.push(component);
      schedule_update();
      component.$$.dirty.fill(0);
    }
    component.$$.dirty[(i / 31) | 0] |= 1 << i % 31;
  }

  function init(
    component,
    options,
    instance,
    create_fragment,
    not_equal,
    props,
    append_styles,
    dirty = [-1]
  ) {
    //...
  }

  class SvelteComponent {
    $destroy() {
      destroy_component(this, 1);
      this.$destroy = noop;
    }
    $on(type, callback) {
      const callbacks =
        this.$$.callbacks[type] || (this.$$.callbacks[type] = []);
      callbacks.push(callback);
      return () => {
        const index = callbacks.indexOf(callback);
        if (index !== -1) callbacks.splice(index, 1);
      };
    }
    $set($$props) {
      if (this.$$set && !is_empty($$props)) {
        this.$$.skip_bound = true;
        this.$$set($$props);
        this.$$.skip_bound = false;
      }
    }
  }

  //...

  const file = "src/App.svelte";

  function create_fragment(ctx) {
    //...
  }

  function instance($$self, $$props, $$invalidate) {
    //...
  }

  class App extends SvelteComponentDev {
    constructor(options) {
      super(options);
      init(this, options, instance, create_fragment, safe_not_equal, {});

      dispatch_dev("SvelteRegisterComponent", {
        component: this,
        tagName: "App",
        options,
        id: create_fragment.name,
      });
    }
  }

  const app = new App({
    target: document.body,
  });

  return app;
})();
//# sourceMappingURL=bundle.js.map
```

다소 무식한 방법이지만 일단 bundle.js에 있는 모든 함수에 브레이킹 포인트를 찍고 플로우를 먼저 확인하고, 그 이후 주요 함수들을 추려보겠습니다.

## 앱 생성

먼저 앱 생성 부분을 보겠습니다.

App class는 SvelteComponent를 상속받습니다.

```js
const app = new App({
  target: document.body,
});
```

```js
class App extends SvelteComponentDev {
  constructor(options) {
    super(options);
    init(this, options, instance, create_fragment, safe_not_equal, {});

    dispatch_dev("SvelteRegisterComponent", {
      component: this,
      tagName: "App",
      options,
      id: create_fragment.name,
    });
  }
}
```

```js
class SvelteComponent {
  $destroy() {
    destroy_component(this, 1);
    this.$destroy = noop;
  }
  $on(type, callback) {
    const callbacks = this.$$.callbacks[type] || (this.$$.callbacks[type] = []);
    callbacks.push(callback);
    return () => {
      const index = callbacks.indexOf(callback);
      if (index !== -1) callbacks.splice(index, 1);
    };
  }
  $set($$props) {
    if (this.$$set && !is_empty($$props)) {
      this.$$.skip_bound = true;
      this.$$set($$props);
      this.$$.skip_bound = false;
    }
  }
}
```

### init

App class 생성자에서 init 함수가 실행됩니다.

이때 options는 App 생성자에 넣어주고 instance와 create_fragment, safe_not_equal는 상위 스코프에서 접근합니다.

```js
function init(
  component,
  options,
  instance,
  create_fragment,
  not_equal,
  props,
  append_styles,
  dirty = [-1]
) {
  const parent_component = current_component;
  set_current_component(component);
  const $$ = (component.$$ = {
    //...
  });
  //...
  $$.ctx = instance
    ? instance(component, options.props || {}, (i, ret, ...rest) => {
        const value = rest.length ? rest[0] : ret;
        if ($$.ctx && not_equal($$.ctx[i], ($$.ctx[i] = value))) {
          if (!$$.skip_bound && $$.bound[i]) $$.bound[i](value);
          if (ready) make_dirty(component, i);
        }
        return ret;
      })
    : [];
  $$.update();

  $$.fragment = create_fragment ? create_fragment($$.ctx) : false;
  //...
}
```

init 함수 내에서 app instance의 $$ 프로퍼티가 만들어지고 이 프로퍼티는 핵심 역할을 합니다.

```js
//init function
const $$ = (component.$$ = {
  fragment: null,
  ctx: null,
  // state
  props,
  update: noop,
  not_equal,
  bound: blank_object(),
  // lifecycle
  on_mount: [],
  on_destroy: [],
  on_disconnect: [],
  before_update: [],
  after_update: [],
  context: new Map(
    options.context || (parent_component ? parent_component.$$.context : [])
  ),
  // everything else
  callbacks: blank_object(),
  dirty,
  skip_bound: false,
  root: options.target || parent_component.$$.root,
});
```

### instance

`$$.ctx`에 instance가 할당됩니다.

```js
//init function

$$.ctx = instance ? instance(component, options.props || {}, (i,ret,...rest)=>{
            const value = rest.length ? rest[0] : ret;
            if ($$.ctx && not_equal($$.ctx[i], $$.ctx[i] = value)) {
                if (!$$.skip_bound && $$.bound[i])
                    $$.bound[i](value);
                if (ready)
                    make_dirty(component, i);
            }
            return ret;
}
```

instance 함수에서 doubled와 count의 관계와 업데이트가 결정되어 있습니다.

`$$.update`에 state 업데이트 함수가 할당됩니다.

```js
function instance($$self, $$props, $$invalidate) {
  let doubled;
  let { $$slots: slots = {}, $$scope } = $$props;
  validate_slots("App", slots, []);
  let count = 0;

  function handleClick() {
    $$invalidate(0, (count += 1));
  }

  //...

  $$self.$$.update = () => {
    if ($$self.$$.dirty /*count*/ & 1) {
      $$invalidate(1, (doubled = count * 2));
    }
  };

  return [count, doubled, handleClick];
}
```

### fragment

`$$.fragment`에 dom 모양이 결정되어서 할당됩니다.

```js
$$.fragment = create_fragment ? create_fragment($$.ctx) : false;
```

```js
function create_fragment(ctx) {
  //...
  const block = {
    c: function create() {
      button = element("button");
      t0 = text("Clicked ");
      t1 = text(/*count*/ ctx[0]);
      t2 = space();
      t3 = text(t3_value);
      t4 = space();
      p = element("p");
      t5 = text(/*count*/ ctx[0]);
      t6 = text(" doubled is ");
      t7 = text(/*doubled*/ ctx[1]);
      add_location(button, file, 9, 0, 105);
      add_location(p, file, 13, 0, 198);
    },
    //...
  };

  return block;
}
```

## 런타임 동작

## state와 dom 업데이트

**before**
![svelte](/assets/img/svelte/svelte4.png)

**update**
![svelte](/assets/img/svelte/svelte5.png)

버튼이 클릭되었을 때 count는 1이 되고, doubled는 2가 되고 렌더링은 어떻게 업데이트되는지 보겠습니다.

### invalidate 함수

**handleClick**

```js
function handleClick() {
  $$invalidate(0, (count += 1));
}
```

넣어주었던 clickHandler가 내부에서 invalidate 함수를 실행해줍니다.

> invalidate 함수는 instance 생성 시점에 결정되어서 들어간다.

```js
$$.ctx = instance ? instance(component, options.props || {}, (i,ret,...rest)=>{
            const value = rest.length ? rest[0] : ret;
            if ($$.ctx && not_equal($$.ctx[i], $$.ctx[i] = value)) {
                if (!$$.skip_bound && $$.bound[i])
                    $$.bound[i](value);
                if (ready)
                    make_dirty(component, i);
            }
            return ret;
) : [];
```

현재 컨텍스트의 값과 업데이트한 값이 같은지 판단하고, 다르다면 dirty flag를 세워줍니다.

이 과정에서 렌더링와 상태를 업데이트할 컴포넌트가 dirty_components에 추가되고 해당 컴포넌트에서 업데이트할 상태 값의 index가 비트 연산으로 플래그가 세워집니다.

```js
function make_dirty(component, i) {
  if (component.$$.dirty[0] === -1) {
    dirty_components.push(component);
    schedule_update();
    component.$$.dirty.fill(0);
  }
  component.$$.dirty[(i / 31) | 0] |= 1 << i % 31;
}
```

### flush

flag가 세워진 후 다음 microtask 까지 기다린 후 flush 함수가 실행됩니다.

먼저 dirty_components에 추가된 컴포넌트들을 순회하며 각자 observe 하고 있는 변수들을 업데이트합니다.

```js
function flush() {
  const saved_component = current_component;
  do {
    while (flushidx < dirty_components.length) {
      const component = dirty_components[flushidx];
      flushidx++;
      set_current_component(component);
      update(component.$$);
    }
    //...
  } while (dirty_components.length);
  //...
}
```

```js
function update($$) {
  if ($$.fragment !== null) {
    $$.update();
    //...
  }
}
```

반응형 변수(`$`)에 대해 다시 invalidate를 반복하며 dirty flag를 세워줍니다.

```js
$$self.$$.update = () => {
  if ($$self.$$.dirty /*count*/ & 1) {
    $$invalidate(1, (doubled = count * 2));
  }
};
```

<br>
<br>

현재 microtask 내에 모든 dirty flag가 업데이트된 후 각 fragment의 update(p) 함수를 실행해 줍니다.

```js
function update($$) {
  if ($$.fragment !== null) {
    $$.update();
    //...
    $$.fragment && $$.fragment.p($$.ctx, dirty);
    $$.after_update.forEach(add_render_callback);
  }
}
```

여기서 비트 연산된 dirty 결과에 따라서 각 dom 요소들을 직접 업데이트합니다.

```js
p: function update(ctx, [dirty]) {
                if (dirty & /*count*/
                1)
                    set_data_dev(t1, /*count*/
                    ctx[0]);
                if (dirty & /*count*/
                1 && t3_value !== (t3_value = (/*count*/
                ctx[0] === 1 ? 'time' : 'times') + ""))
                    set_data_dev(t3, t3_value);
                if (dirty & /*count*/
                1)
                    set_data_dev(t5, /*count*/
                    ctx[0]);
                if (dirty & /*doubled*/
                2)
                    set_data_dev(t7, /*doubled*/
                    ctx[1]);
},
```

## 정리

### 런타임 동작 방식 정리

![svelte](/assets/img/svelte/svelte6.png)

1. 동작 발생 시 해당 동작에 대한 handler가 수행된다.
2. handler 내부에서 invalidate 함수가 실행된다.
3. 업데이트가 필요한 값들을 가지고 있는 컴포넌트들이 dirty_components에 추가된다.
4. 해당 컴포넌트 내 값 갱신이 필요한 상태 값들의 dirty flag를 세워준다.
5. 다음 microtask까지 수행되는 동작에 대해 1~4 반복
6. flush 함수가 실행된다.

   1. 업데이트할 value가 trigger 하는 value에 대해 invalidate 함수가 실행된다. 즉, dirty flag를 세워준다. (3~4)
   2. beforeUpdate 콜백 실행 (부모 → 자식)
   3. 렌더링을 업데이트한다.
      1. 현재 컴포넌트에 세워진 dirty flag의 상태에 따라서 필요한 값들만 업데이트한다.
      2. 이때 이 업데이트는 직접 dom을 업데이트하는 것이다.
   4. bind 콜백 실행 (자식 → 부모)
   5. afterUpdate 콜백 실행 (부모 → 자식)
      최초 onMount 가 수행되기 전 수행된 afterUpdate는 (자식 → 부모)

### 결론

> 결과적으로 Svelte가 말하는 `컴파일 단계에서 Virtual DOM diffing과 같은 기술을 사용하는 대신 앱의 상태가 변경될 때 DOM을 직접 업데이트하는 코드로 만들어 준다고 한다.` 는 진짜다.

기존 프레임워크와 뭐가 다를까요?

트리쉐이킹이 되겠지만 프레임워크를 동작하기 위한 프레임워크 코어 코드들이 로드되고 컴파일된 앱단의 코드들은 이 프레임워크에 효율적으로 명령할 수 있는 코드들을 만들어냅니다.

예시) Angular ivy(incremental DOM) 앱 빌드 결과

> 앱 빌드는 프레임워크 코드를 효율적으로 사용하도록 지시된 결과물이다.

```js
// out-tsc/app/src/app/app.component.js
import { Component } from "@angular/core";
import * as i0 from "@angular/core";
export class AppComponent {
  constructor() {
    this.title = "compile-test";
  }
}
AppComponent.ngComponentDef = i0.ɵɵdefineComponent({
  type: AppComponent,
  selectors: [["app-root"]],
  factory: function AppComponent_Factory(t) {
    return new (t || AppComponent)();
  },
  consts: 2,
  vars: 1,
  template: function AppComponent_Template(rf, ctx) {
    if (rf & 1) {
      i0.ɵɵelementStart(0, "h1");
      i0.ɵɵtext(1);
      i0.ɵɵelementEnd();
    }
    if (rf & 2) {
      i0.ɵɵselect(1);
      i0.ɵɵtextInterpolate(ctx.title);
    }
  },
  styles: [""],
});
/*@__PURE__*/ i0.ɵsetClassMetadata(
  AppComponent,
  [
    {
      type: Component,
      args: [
        {
          selector: "app-root",
          template: ` <h1>{{title}}</h1> `,
          styleUrls: ["./app.component.css"],
        },
      ],
    },
  ],
  null,
  null
); //# sourceMappingURL=app.component.js.map
```

이에 비해 svelte는 모든 상태와 동작이 정의된 코드가 생성됩니다.

실제 프로덕션 레벨에서 사용하면 어떨지 모르겠지만(튜토리얼 깨는 중),

스벨트의 동작 방식이나 store 사용법 반응형 변수들($, 이거 뭐라고 불러야 할지 애매해서 계속 괄호 치게 됨..) 을 템플릿 내에서 사용할 때 auto unsubscribe 되는 것, binding 등등 꽤 괜찮은 것 같습니다.

> 몇몇 문법들은 거부감이 들기도 한다..

이후에 뭐라도 개발해 보면서 사이드 이펙트는 어떻게 관리해야 할지, 구조를 어떻게 잡아야 할지 고민해 볼 기회가 있으면 다시 관련 글을 써보도록 하겠습니다!

## 참고

- [https://svelte.dev/](https://svelte.dev/)
- [https://velog.io/@thyoondev/Svelte-VSCode에서-스벨트-시작하기](https://velog.io/@thyoondev/Svelte-VSCode%EC%97%90%EC%84%9C-%EC%8A%A4%EB%B2%A8%ED%8A%B8-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)
- [https://alexband.tistory.com/58](https://alexband.tistory.com/58)
