---
layout: post
title: "[알고리즘]수학#3 소수판별"
date: 2019-06-03 19:00:00
author: 송튜디오
categories: 알고리즘_수학
tags: 소수 에라토스테네스의체 소수판별
---

### 소수

소수는 자신보다 작은 두 개의 자연수를 곱하여 만들 수 없는 1보다 큰 자연수이다

- 모든 자연수는 소수의 곱으로 나타낼 수 있다는 규칙이 있다.
- 84 = 2^2 \* 3^1 \* 5^0 \* 7^1 \* 11^0 \* 13^0 \* 17^0 \* ...
  - 위 수식에서 많은 소수의 지수 부분이 0이다.
- 가분성(divisibility)
  - 어떤 수 x로 y를 나눌 수 있으려면 x를 소수의 곱으로 분할하였을 때,
  - 나열되는 모든 소수는 y를 소수의 곱으로 분할하였을 때 나열되는 모든 소수들의 부분집합이다.
  - x = 2^j0 \* 3^j1 \* 5^j2 \* 7^j3 \* 11^j4 \* .. 이고,
  - y = 2^k0 \* 3^k1 \* 5^k2 \* 7^k3 \* 11^k4 \* .. 일 때,
  - x가 y로 나누어 떨어지면 ji <= ki를 만족한다.
  - 즉, x/y를 만족하려면 모든 i에 대해 ji <= ki를 만족해야 한다.

#### 소수 판별

어떤 수 n이 소수인지 여부를 판별하는 가장 단순한 방법은 2에서 n-1까지 루프를 돌면서 나누어지는 경우가 있는지 확인하는 것이다.

```js
function primeNaive(n) {
  if (n < 2) {
    return false;
  }
  for (var i = 2; i < n; i++) {
    if (n % i == 0) return false;
  }
  return true;
}
```

#### 에라토스테네스의 접근

루프를 n이 아닌 n의 제곱근까지만 돌고도 소수를 판별할 수 있다.

```js
function primeNaive2(n) {
  if (n < 2) {
    return false;
  }
  var sqrt = Math.sqrt(n);
  for (var i = 2; i <= sqrt; i++) {
    if (n % i == 0) return false;
  }
  return true;
}
```

- 제곱근까지만 검사해도 소수를 판별할 수 있는 이유는, n을 나누는 모든 숫자 a는 그에 대한 보수 b가 반드시 존재하기 때문이다.
- b : (a\*b=n)
- 만일 a > sqrt(n) 이라면 b < sqrt(n) 이다.(sqrt(n)^2 = n)
- 따라서 n이 소수인지 판별하기 위해 a까지 검사할 필요는 없다.
- 주어진 자연수 N이 소수이기 위한 필요충분 조건은 N이 N의 제곱근보다 크지 않은 어떤 소수로도 나눠지지 않는다.
- 수가 수를 나누면 몫이 발생하게 되는데 몫과 나누는 수, `둘 중 하나`는 반드시 `N의 제곱근 이하`이기 때문이다.

#### 에라토스테네스의 체

- 에라토스테네스의 체는 소수 목록을 만드는 굉장히 효율적인 방법이다.
- 이 알고리즘은 소수가 아닌 수들은 반드시 다른 소수로 나누어진다는 사실에 기반해서 동작한다.
- 1 ~ n까지의 리스트를 만든다.
- 그리고 첫 번째, 2로 나누어지는 모든 수를 리스트에서 없앤다.
- 그 후에 2, 3, 5, 7.. 등의 소수로 나뉘는 모든 수들을 리스트에서 삭제한다.

```js
function sieveOfEratosthenes(max) {
  var flags = [].fill.call({ length: max + 1 }, true, 2);
  var prime = 2;

  while (prime <= Math.sqrt(max)) {
    crossOff(flags, prime);
    prime = getNextPrime(flags, prime);
  }
}

// 배수 제거 함수
function crossOff(flags, prime) {
  /*
    prime의 배수들을 제거해나간다. k < prime인 k에 대한 k * prime은 이전 루프에서 이미 제거되었을 것이므로
    prime * prime부터 시작한다.
    */
  for (var i = prime * prime; i < flags.length; i += prime) {
    flags[i] = false;
  }
}

// 다음 소수를 찾는 함수
function getNextPrime(flags, prime) {
  var next = prime + 1;
  while (next < flags.length && !flags[next]) {
    next++;
  }
  return next;
}
```

## 참고

- 코딩인터뷰 완전정복
- https://ko.wikipedia.org/wiki/소수_(수론)
