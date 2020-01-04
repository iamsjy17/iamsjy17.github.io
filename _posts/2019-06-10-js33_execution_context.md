---
layout: post
title: "[Javascript] Execution Context와 Lexical Environment"
date: 2019-06-10 19:00:00
author: Jewoo.Song
categories: Javascript
tags: 실행컨텍스트 VariableObject 변수객체 전역컨텍스트 EC GO AO 스코프체인 ScopeChain LexicalEnvironment ExecutionContext EnvironmentRecord OuterReference
---

### 1. 실행 컨텍스트

자바스크립트의 핵심 개념으로 실행 가능한 코드를 형상화하고 구분하는 추상적인 개념이라 정의된다.
쉽게 풀어쓰면 `실행 가능한 코드`를 `실행하기 위해 필요한 환경`이다.

> 실행 가능한 코드?

1. 전역 코드 : 전역 영역에 존재하는 코드
2. Eval 코드 : eval 함수로 실행되는 코드
3. 함수 코드 : 함수 내에 존재하는 코드

> 코드를 실행하기 위해 필요한 환경?

자바스크립트 엔진이 코드를 실행하기 위해서는 다음과 같은 정보들이 필요하다.

1. 변수 : 전역 변수, 지역 변수, 매개 변수, 함수의 선언
2. arguments 객체
3. Scope
4. this

#### 실행 컨텍스트 스택

- 현재 실행 중인 컨텍스트에서 이 컨텍스트롸 관련 없는 코드가 실행되면 새로운 컨텍스트가 생성된다.
- 이 컨텍스트는 스택에 쌓이게 되고 컨트롤(제어권)이 넘어간다.
- 콜 스택과 유사한 개념이라고 생각하면 된다.

```js
var a = 100;
var b = 100;
var c = 100;

function foo() {
  var b = 10;
  function bar() {
    var c = 1;
    console.log(`bar@ a : ${a} , b : ${b}, c : ${c}`);
  }
  console.log(`foo@ a : ${a} , b : ${b}, c : ${c}`);
  bar();
}
console.log(`@ a : ${a} , b : ${b}, c : ${c}`);
foo();

// @ a : 100 , b : 100, c : 100
// foo@ a : 100 , b : 10, c : 100
// bar@ a : 100 , b : 10, c : 1
```

코드가 실행되면 컨텍스트 스택은 아래와 같이 형성된다.

![Alt context](/assets/img/js33/context1.png)

1. 실행 가능한 코드로 컨트롤이 이동하면 논리적 스택 구조를 가지는 빈 실행 컨텍스트 스택이 생성된다.
2. 전역 코드로 컨트롤이 이동하면 `전역 실행 컨텍스트(Global EC)`가 생성되고 스택에 쌓인다. 이 전역 실행 컨텍스트는 App 종료 시기까지 유효하다.
3. 함수를 호출하면 해당 함수의 코드로 컨트롤이 이동하고, 함수의 실행 컨텍스트가 생성되고 스택에 쌓인다.
4. 함수 실행이 끝나면 해당 함수의 실행 컨텍스트가 파기되고, 직전 실행 컨텍스트로 컨트롤이 이동한다.

> 이때 이 실행 컨텍스트는 어떠한 것들을 포함하고 있을까?

### 2. 물리적 실행 컨텍스트

> ECMA-262 Edition3의 주요 개념이고 ECMA-262 Edition5부터는 Lexical Environment가 변수 객체와 Scope Chain을 포함하는 개념이 되었다. 현재는 달라젔지만 널리 알려진 개념이고, 개념을 이해하기 위해서 더 쉬우므로 이 개념으로 설명한다.

실행 컨텍스트는 실행 가능한 코드를 형상화하고 구분하는 추상적인 개념이지만 물리적으로는 객체의 형태를 가지며 아래와 같은 3개의 프로퍼티를 가진다.

1. 변수 객체(Variable Object) : 함수 선언, arguments 객체, 지역 변수, 전역 변수 ..
2. Scope Chain : 변수 객체 + 상위 스코프
3. this : Context Object

#### 변수 객체(Variable Object)

- 실행 컨텍스트가 생성되면 자바스크립트에서는 해당 컨텍스트에서 실행에 필요한 여러 가지 정보를 담을 객체를 생성한다. 그게 바로 변수 객체이다.
- 사용자가 접근할 수 있는 영역은 아니고 자바스크립트 엔진이 사용하는 영역이다.
- 변수 객체는 다음과 같은 값들을 포함한다.

1. 변수
2. 매개 변수(parameter), 인수(arguments)
3. 함수 선언(함수 표현식은 제외된다.)

그런데 이 변수 객체는 실제로는 다른 객체를 가리키는 참조 변수이다.
전역 코드 실행 시 생성되는 전역 컨텍스트와 함수 실행 시 생성되는 함수 컨텍스트는 다르기 때문에 가리키는 객체가 다르다.(`매개 변수와 인수 정보` 포함 여부가 차이 난다.)

##### 전역 코드 실행과 전역 객체

전역 코드로 컨트롤이 이동하면 변수 객체는 유일하며 최상위에 위치하고 전역 변수, 전역 함수를 포함하는 `전역 객체(Global Object)`를 가리킨다.

![Alt context](/assets/img/js33/context2.png)

##### 함수 코드 실행과 활성 객체

함수 코드로 컨트롤이 이동하면 변수 객체는 `활성 객체(Activation Object)`를 가리킨다.
활성 객체는 arguments 객체를 추가로 가진다.

- AO란? function context에서의 VO이다.
- Caller에 의해 function context가 활성화되면 AO가 생성되고 function이 호출된다.
- 호출시 arguments 객체에 매개 변수와 인수를 담아 넘긴다.
- AO도 VO이므로 arguments + VO(변수, 함수선언)을 가진다.

> arguments 객체는 매개 변수와 인수 정보를 가지고 있다.

![Alt context](/assets/img/js33/context3.png)

#### 스코프 체인(Scope Chain)

스코프 체인은 일종의 리스트로 전역 객체와 그 위에 쌓인 함수의 스코프의 레퍼런스를 순서대로 저장한다.

- 스코프 체인은 식별자 중에서 객체(전역 객체 제외)의 프로퍼티가 아닌 식별자, 즉 변수를 검색하는 메커니즘이다.
  - 식별자 중 변수가 아닌 객체의 프로퍼티를 검색하는 메커니즘은 프로토타입 체인이다.
- AO -> AO -> GO와 같은 형태의 리스트이다.
  - 현재 실행 컨텍스트의 AO를 Head로 두고, 순차적으로 상위 컨텍스트의 AO를 가리킨다.
  - 리스트의 마지막은 GO를 가리킨다.
- 엔진은 스코프 체인을 통해 렉시컬 스코프(Lexical Scope)를 확인한다.
  - 함수 실행 중 변수를 만나면 스코프 체인 리스트에 따라서 Head에 위치한 현재 실행 컨텍스트의 AO를 검색하고 순차적으로 상위 VO(AO 혹은 GO)를 검색한다.
  - 이렇게 순차적으로 리스트를 따라가며 검색하기 때문에 `스코프 체인`이라고 불린다.
  - 마지막까지 검색에 실패하면 참조 에러가 발생한다.

예제를 다시 보자면, foo함수 실행 시 b를 참조할 때 먼저 foo의 실행 컨텍스트에서 검색하기 때문에 10이 출력되는 것이다.

```js
var a = 100;
var b = 100;
var c = 100;

function foo() {
  var b = 10;
  // ...
  console.log(`foo@ a : ${a} , b : ${b}, c : ${c}`);
  // ...
}
// ...
// foo@ a : 100 , b : 10, c : 100
```

예제에서 foo가 실행되는 시점에 실행 컨텍스트 스택은 다음과 같은 형태이다.

![Alt context](/assets/img/js33/context4.png)

#### this value

this value 프로퍼티에는 this 값이 함수 호출 패턴에 따라서 결정된다.

### 3. 물리적 실행 컨텍스트 2 (Lexical Environment와 Viriable Environment)

앞에 물리적 실행 컨텍스트에서 설명한 내용은 사실은 ES5 이전의 개념이다.
개념적으로 실행 컨텍스트를 이해하기에 용이하기에 사용하지만 ES5부터의 실행 컨텍스트는 물리적으로 다른 모양을 갖는다.

변수 객체, 활성화 객체, 스코프 체인등의 개념이 LexicalEnvironment로 변경되었다.

```js
ExcutionContext = {
    LexicalEnvironment = [Lexical Environment],
    VariableEnvironment = [Lexical Environment],
    thisbinding = [Object]
}
```

#### LexicalEnvironment

```js
LexicalEnvironment = {
    "Environment Record" = ,
    "Outer Environment Reference" = ,
}
```

Lexical Environment는 자바스크립트 코드에서 식별자 정의를 위해 사용하는 객체로 `Environment Record`와 `Outer Environment Reference`를 프로퍼티로 갖는다.

- Outer Environment Reference
  - 중첩 유효 범위를 가질 수 있는 환경에서 상위 Lexical Environment를 참조한다.
  - 즉, 외부 환경에 대한 참조를 가지고 있다.
  - 전역 환경에서는 null이다.
- Environment Record
  - 유효범위 내의 값에 식별자를 매핑한다.
  - DeclarativeER : 변수 선언 및 함수 선언을 저장한다.
  - ObjectER : with 문이나 전역 환경에서 사용된다.
  - this
- 예제 코드

```js
function foo() {
  var a = 100;
  var b = 100;
  var c = 100;
  function bar() {
    //...
  }
}
foo();
```

- foo 함수의 Execution Context의 LexicalEnvironment

```json
{
    environmentRecord: {
        a: 100,
        b: 100,
        c: 100,
        bar : function Object
    },
    outer : Outer Environment
}
```

![Alt context](/assets/img/js33/context5.png)

> DeclarativeEnvironmentRecord

- `DeclarativeER`는 ECMA 언어로 표현되는 값들과 식별자를 직접적으로 연결해주는 함수 선언, 변수 선언, catch 절과 같은 문법 요소의 효과를 정의하기 위해 사용된다.
- 쉽게 말해 식별자 정보를 `DeclarativeER`에서 찾을 수 있다.
- `DeclarativeER`는 Environment Record를 상속한 서브 클래스이며, 따라서 HasBinding 메서드를 구현하고 있다.

```json
DeclarativeEnvironmentRecord : {
    x : 10,
    y : 20,
    result : 30
}

```

> ObjectEnvironmentRecord

- 전역 코드에 대한 lexical environment는 objective environment records를 포함한다. 변수와 함수의 선언과 다르게 object environment record는 글로벌 객체도 기록한다.
- 즉, 전역 환경에서 사용되는 프로퍼티로 글로벌 객체를 매핑한다.
- object environment record는 with 문과 같이 식별자를 어떤 특정 객체 A의 속성으로 취급할 때 사용되며, 이를 위해 binding object라는 속성으로 A를 가리킨다.
- `Object Environment Record`는 Environment Record를 상속한 서브 클래스이며, 따라서 HasBinding 메서드를 구현하고 있다.
- HasBinding 메서드는 내부적으로 HasProperty 함수를 호출하고, HasProperty 함수는 프로토타입 체인을 뒤져서 식별자를 찾아내다.

> Global Environment Record의 object Environment Record

- Global Environment Record에 있는 object Environment Record의 binding object는 전역 객체를 가리킨다.
- 따라서 일반적인 Object Environment Record와는 다르게 Object, Array, Function과 같은 built-in global과 전역 코드에서의 함수 선언, 변수 선언에 의해 생성된 모든 식별자 정보를 bind object가 참조하는 전역 객체에서 찾을 수 있다.
- bind object가 전역 객체를 가리키는 바람에 declarative Environment Record와 역할이 바뀐 것 같은 모양이 되었다. (일반적인 함수 context에서의 Environment Record와 다르다.)

```json
ObjectEnvironmentRecord : {
    bindObject : globalObject
}
```

> Global Environment Record의 declarative Environment Record

- Global Environment Record에 있는 declarative Environment Record도 일반적인 Declarative Environment Record와 동작 방식이 다르며, 전역 코드에서 object Environment Record에 포함되지 않은 식별자 정보만 보유한다.
- 이유는 Declarative Environment Record의 역할을 object Environment Record가 가져가버렸기 때문이다.

#### VariableEnvironment

Lexical Environment과 대게 동일한 값을 갖는다.
그러나 만들어진 변수 선언 및 함수 선언에 대해 바인딩을 유지한다.

- LexicalEnvironment는 코드 실행 중에 실행 컨텍스트 내에서 변경될 수 있지만 VariableEnvironment는 항상 값을 유지한다.
  - LexicalEnvironment는 일시적으로 LexicalEnvironment 하위에 새로운 환경을 가리킵니다.
  - 이 새로운 환경은 임시 바인딩을 보유합니다.
  - 그리고 임시 범위를 벗어나면 VariableEnvironment가 참조하고 있는 값으로 LexicalEnvironment를 복구합니다.

> 예제는 `개발왕 김코딩`님의 블로그의 발표 자료를 인용하였습니다.
> https://wit.nts-corp.com/2014/06/20/1560

![Alt context](/assets/img/js33/context6.png)
![Alt context](/assets/img/js33/context7.png)
![Alt context](/assets/img/js33/context8.png)

#### Identifier Resolution(스코프 체인)

- 식별자와 묶여있는 값을 찾아가는 과정으로 스코프 체인의 원리이다.
- 어떤 함수 내에서 식별자를 찾는 것은 함수가 호출되어 실행될 때 일어나는 일이다.
- 스코프 체인은 Lexical Environment를 원소로 하는 단방향 링크드 리스트다.
- `Outer Environment Reference`를 통해서 스코프 체인을 연결한다.

> ES3까지는 `Scope Chain`이란 용어를 직접 명시했다면, ES5부터는 `Lexixal nesting structure` 또는 `Logical nesting of Lexical Environment Values`등으로 표현한다.

```js
var a = 100;
var b = 100;
var c = 100;
function foo() {
  var b = 10;
  function bar() {
    var c = 10;
    //...
  }
  bar();
  //...
}
foo();
//...
```

```js
GlobalEnvironment = {
  environmentRecord: {
    a: 100,
    b: 100,
    c: 100,
    foo: [Function]
  },
  outer: null
};

fooEnvironment = {
  environmentRecord: {
    b: 10,
    bar: [Function]
  },
  outer: GlobalEnvironment
};

barEnvironment = {
  environmentRecord: {
    c: 100
  },
  outer: fooEnvironment
};
```

![Alt context](/assets/img/js33/context9.png)

### 3. 실행 컨텍스트 생성 과정

실행 컨텍스트란 실행 가능한 코드를 실행하기 위해 필요한 환경이다.
`실행 가능한 코드(전역/함수/eval)`가 실행될 때 `실행하기 위해 필요한 환경(실행 컨텍스트)`는 다음과 같은 두 가지 단계로 생성된다.

1. 생성 단계(Creation Phase)
2. 실행 단계(Execution Phase)

#### Creation Phase

컴파일러는 실제로 코드를 실행하기 전 전체 코드를 두 번 실행한다.

- 첫 번째 실행에서는 모든 함수 선언을 선택하여 참조와 함께 메모리에 저장한다.
- 두 번째 실행에서는 모든 변수를 선택하고 undefined를 할당한다. 변수 선언과 함수 선언 사이에 충돌이 발생하면 해당 변수는 무시된다.

이렇게 두 번의 실행을 하면서 Creation Phase에서는 두 가지 일이 일어난다.

1. LexicalEncironment 컴포넌트 생성
2. VariableEnvironment 컴포넌트 생성

렉시컬 환경은 세 가지의 일을 한다.(VariableEnvironment도 동일하다.)

1. Environment Record(변수, 함수, 인자 생성) 초기화

   - step 1 : 매개변수가 Environment Record의 프로퍼티로 인수가 값으로 세팅된다.
   - step 2 : 대상 코드 내의 함수 선언(표현식 제외)을 대상으로 함수명이 Environment Record의 프로퍼티로 생성된 함수 객체가 값으로 설정된다.
     - `함수 호이스팅`
   - step 3 : 대상 코드 내의 변수 선언을 대상으로 변수명이 Environment Record의 프로퍼티로 추가되고, undefined 값으로 설정된다.
     - `변수 호이스팅`

2. Reference to the outer environment(스코프 체인 생성과 초기화)
3. this binding

> `함수 호이스팅`
>
> - Environment Record 초기화의 2번째 단계는 아직 코드 실행이전이다.
> - 그러나 Environment Record에 함수가 등록되어 있으므로 함수 선언식 이전에 함수 호출이 가능하다.(`함수 표현식은 일반 변수의 방식을 따른다.`)
> - 이렇게 선언문 이전에 함수 호출이 가능한 현상을 `함수 호이스팅`이라 한다.

> `변수 호이스팅`
> var 변수는 선언, 초기화, 할당 단계를 통해서 거친다.
>
> 1. 선언 : Environment Record에 변수를 등록한다. 이 ER이 포함된 실행 환경은 스코프가 참조할 수 있는 대상이 된다.
> 2. 초기화 : ER에 등록된 변수를 메모리에 할당하고, Undefined로 초기화한다.
> 3. 할당 : undefined로 초기화된 변수에 실제 값을 할당한다.
>
> - var 키워드로 선언된 변수는 선언과 초기화 단계가 동시에 이루어진다.
> - 현재 실행환경에 변수가 등록될 때 undefined로 초기화된다.
> - 때문에 변수 선언문 이전에 변수에 접근할 수 있다.(값은 undefined이다.)
> - 이렇게 선언문 이전에 변수 접근이 가능한 현상을 `변수 호이스팅`이라 한다.

#### Execution Phase

Execution Phase는 실제로 코드를 읽으면서 코드를 실행할 차례이다.
이 단계에서는 선언했던 변수들에 값이 할당된다.

### 4. 실행 컨텍스트 생성 과정의 예제

예제 코드의 실행 컨텍스트 생성 과정을 따라가보면 왜 실행 컨텍스트 스택인지 알 수 있다.

```js
var a = 100;
var b = 100;
var c = 100;

function foo() {
  var b = 10;
  function bar() {
    var c = 1;
    console.log(`bar@ a : ${a} , b : ${b}, c : ${c}`);
  }
  console.log(`foo@ a : ${a} , b : ${b}, c : ${c}`);
  bar();
}
console.log(`@ a : ${a} , b : ${b}, c : ${c}`);
foo();
```

#### 1) 전역 코드 진입

전역 코드로 컨트롤이 진입하면 전역 실행 컨텍스트가 생성되고, 앞서 설명한 `Creation Phase`단계의 동작이 수행된다.

즉, `Global Environment()` 컴포넌트가 생성된다.

- Environment Record 단계
  - Global Environment의 LexicalEnvironment의 EnvironmentRecord의 타입은 Object이다.
  - 따라서 globalObject를 바인딩 한다.
  - step 1 : 인수 값 설정은 전역 객체이므로 생략된다.
  - step 2 : 함수 foo의 선언 처리
    - foo가 전역 객체의 프로퍼티로 함수 객체가 값이 된다.
    - 생성된 함수 객체는 `[[scope]]` 프로퍼티를 가지게 된다. `[[scope]]`는 함수 객체만이 가지는 프로퍼티로 함수 객체가 실행되는 환경을 가리킨다.
  - step 3 : 변수 a, b, c의 선언 처리
    - 변수 a, b, c는 전역 객체의 프로퍼티로 추가되고 값은 undefined로 설정된다.

> `클로저`
>
> - 함수 내부의 `[[scope]]` 프로퍼티는 자신의 실행환경(Lexical Environment)과 자신을 포함하는 외부 함수의 실행환경과 전역 객체를 가리키는데 이때 자신을 포함하는 외부함수의 실행 컨텍스트가 스택에서 사라지더라도 `[[scope]]`가 가리키는 외부 함수의 실행환경은 소멸되지 않고 참조할 수 있다.
> - 즉, 외부 함수의 실행 컨텍스트가 소멸되어도 내부 함수의 스코프 체인이 외부 함수의 실행환경을 참조하므로 사용할 수 있다.

- Reference to the outer environment 단계
  - 전역 실행 컨텍스트의 outer는 null이다.
- this binding 단계
  - this value가 결정되기 이전에 this는 전역 객체를 가리킨다.
  - 함수 호출 시 `함수 호출 패턴`에 따라서 this에 할당하는 값이 결정된다.

```js
GlobalExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      bindObject : globalObject
    }
    outer: <null>,
    this: <global object>
  }
}

globalObject = {
    //초기 빌드인 객체(Math, String, Array, Bom, Dom등)...
    foo : <functionobject>,
    a : undefined,
    b : undefined,
    c : undefined
}
```

#### 2) 전역 코드 실행

##### 2-1) 변수 값의 할당

- 전역 코드가 실행되면서 a, b, c 변수에 값이 할당된다.
  - 먼저 현재 실행 컨텍스트의 Lexical Environment가 참조하고 있는 Environment Record에서(`전역 객체`) a, b, c에 해당하는 프로퍼티를 검색한다.
  - 검색이 성공하면 값을 할당한다.

```js
globalObject = {
    //초기 빌드인 객체(Math, String, Array, Bom, Dom등)...
    foo : <functionobject>,
    a : 100,
    b : 100,
    c : 100
}

```

##### 2-2) 함수 foo 코드 진입

전역 함수 foo의 코드로 진입하면 새로운 함수 실행 컨텍스트가 생성된다.
함수 foo의 실행 컨텍스트로 컨트롤이 이동하면 전역 코드와 마찬가지로 `Creation Phase` 과정이 수행된다.

- Environment Record 단계
  - step 1 : Argument 세팅.
  - step 2 : 함수 bar의 선언 처리
    - bar가 EnvironmentRecord의 프로퍼티로 함수 객체가 값이 된다.
  - step 3 : 변수 b의 선언 처리
    - 변수 b는 현재 Environment Record의 프로퍼티로 추가되고 값은 undefined로 설정된다.
- Reference to the outer environment 단계
  - 생성된 Lexical Environment의 outer는 `Global Environment`를 가리킨다.
- this binding 단계
  - this는 함수 호출 패턴에 따라서 함수 호출시에 결정된다.

```js
FunctionExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      b : undefined,
      bar : <functionobject>
    }
    outer: <Global Environment>,
    this: //<함수 호출시 결정된다.>
  }
}
```

##### 2-3) foo 함수 코드의 실행

###### 2-3-1) 변수 값의 할당

- foo 함수가 실행되면서 b 변수에 값이 할당된다.
  - 먼저 현재 실행 컨텍스트의 Lexical Environment가 참조하고 있는 Environment Record에서 b에 해당하는 프로퍼티를 검색한다.
  - 검색이 성공하면 값을 할당한다.

```js
FunctionExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      //..
      b: 10
    }
    //..
  }
};
```

###### 2-3-2) 함수 bar 코드 진입

bar의 코드로 진입하면 마찬가지로 새로운 함수 실행 컨텍스트가 생성된다.
함수 bar의 실행 컨텍스트로 컨트롤이 이동하면 `Creation Phase` 과정이 수행된다.

- Environment Record 단계
  - step 1 : Argument 세팅.
  - step 2 : 함수 없으므로 생략
  - step 3 : 변수 c의 선언 처리
    - 변수 c는 현재 Environment Record의 프로퍼티로 추가되고 값은 undefined로 설정된다.
- Reference to the outer environment 단계
  - 생성된 Lexical Environment의 outer는 foo 함수의 `Lexical Environment`를 가리킨다.
- this binding 단계
  - this는 함수 호출 패턴에 따라서 함수 호출 시에 결정된다.

```js
FunctionExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      c : undefined,
    }
    outer: <foo`s Lexical Environment>,
    this: //<함수 호출시 결정된다.>
  }
}
```

###### 2-3-3) bar 함수 코드 실행

- bar 함수가 실행되면서 c 변수에 값이 할당된다.

#### 결과

bar 함수가 실행되는 시점에(변수가 할당이 완료된 시점)에 실행 컨텍스트의 스코프 체인(`Lexixal nesting structure`)을 유사 코드로 작성하면 아래와 같다.

```js
//Global EC
GlobalExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      bindObject : globalObject
    }
    outer: <null>,
    this: <global object>
  }
}

globalObject = {
    //초기 빌드인 객체(Math, String, Array, Bom, Dom등)...
    foo : <functionobject>,
    a : 100,
    b : 100,
    c : 100
}

//foo's EC
FunctionExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      b : 10,
      bar : <functionobject>
    }
    outer: <Global Environment>,
    this: //<함수 호출시 결정된다.>
  }
}

//bar EC
FunctionExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      c : 1,
    }
    outer: <foo`s Lexical Environment>,
    this: //<함수 호출시 결정된다.>
  }
}
```

앞서 나왔던 이미지와 유사한 것을 확인할 수 있다.

![Alt context](/assets/img/js33/context9.png)

## 참고

- Learning Javascript
- Inside Javascript
- http://2ality.com/2011/04/ecmascript-5-spec-lexicalenvironment.html
- https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0
- https://meetup.toast.com/posts/129
- https://wit.nts-corp.com/2014/06/20/1560
- https://poiemaweb.com/js-execution-context
- https://homoefficio.github.io/2016/01/16/JavaScript-식별자-찾기-대모험/
