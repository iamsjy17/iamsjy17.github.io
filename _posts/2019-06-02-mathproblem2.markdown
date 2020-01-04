---
layout: post
title: "[알고리즘] 수학#2 Fibonacci Number"
date: 2019-06-02 19:00:00
author: Jewoo.Song
categories: 알고리즘_수학
tags: Memoization Fibonacci Recursion 재귀함수 DynamicProgramming
---

## Fibonacci Number

피보나치 수는 첫째 및 둘째 항이 1이며 그 뒤의 모든 항은 바로 앞 두 항의 합인 수열이다.

### 재귀함수 사용

- Recursion 관점에서의 접근
  - f0 = 0 : Base case
  - f1 = 1 : Base case
  - fn = fn-1 + fn-2 if n>1 : Recursion case

```js
function fibonacci(n) {
  if (n < 2) return n;
  else return fibonacci(n - 1) + fibonacci(n - 2);
}
```

- 재귀함수의 시간 복잡도

  - 다수 호출로 이루어진 재귀함수에서의 수행시간은 보통 O(분기^깊이)로 표현된다.
  - f(n)의 시간 복잡도
    - 2^0 + 2^1 + 2^2 + 2^3 + 2^4 + ... 2^n-1 => 2^n -1
    - 2의 승수의 합
    - O(2^n)

> 일반적인 재귀함수 호출방식의 피보나치 수의 계산은 많은 중복 계산이 필요하다.
> Memoization으로 중복 계산을 피할 수 있다.

![Alt fibonaci](/assets/img/2019-05-08-bigo/fibonacci3.png)

### Memoization

- Memoization을 활용하여 중복 계산을 피할 수 있다.
- 중복 계산 결과를 캐싱함으로써 중복 계산을 피한다.

```js
f(n, r){
    if(n <= 0)
        return 0;
    else if(n == 1 || n == 2)
        return r[n] = 1;
    else if(r[n])
        return r[n];
    return f(n-1) + f(n-2);
}
```

## 참고

- 코딩인터뷰 완전정복
- [엔지니어대한민국 유튜브](https://www.youtube.com/watch?v=VcCkPrGaKrs&list=PLjSkJdbr_gFYSUYfnF_OGXtnGs2d3vWg7&index=2&frags=wn)
- https://ko.wikipedia.org/wiki/피보나치_수
