---
layout: post
title: "[알고리즘]수학#1 최대공약수와 최소공배수"
date: 2019-06-01 19:00:00
author: 송튜디오
categories: 알고리즘_수학
tags: gcd lcm 최대공약수 최소공배수
---

### 최대공약수와 최소공배수

#### x와 y의 최대공약수(greatest common divisor)

적어도 하나가 0이 아닌 정수들의 최대공약수는 공약수 가운데 가장 큰 하나다.

- 지수를 이용한 방법

  - 두 수의 공통인 지수 중에서 작은쪽의 곱이 최대공약수이다.
  - gcd(x, y) = 2^min(j0, k0) \* 3^min(j1, k1) \* 5^min(j2, k2) \* ...

- 유클리드 호제법

  - m>n인 두 양의 정수 m과 n에 대해서 m이 n의 배수이면 gcd(m, n)=n이고,
  - 그렇지 않으면 gcd(m, n)= gcd(n, m%n)이다.
  - 예)
  - 상기한 1071과 1029에 대한 예시를 위와 같은 표현으로 적어보면 아래와 같다.
  - gcd(1071, 1029) = gcd(1029, 42) = gcd(42, 21) = gcd(21, 0) = 21

- Recursion 관점에서의 접근
  - if n == 0 return n : Base case
  - gcd(m, n) = gcd(n, m%n) : Recursion case

```js
function gcd(m, n) {
  if (m < n) {
    var temp = m;
    m = n;
    n = temp;
  }
  if (n == 0) return m;
  else return gcd(n, m % n);
}
```

#### x와 y의 최소공배수(least common multiple)

최소공배수는 양의 공배수 가운데 가장 작은 하나이다.

- lcm(x, y) = 2^max(j0, k0) \* 3^max(j1, k1) \* 5^max(j2, k2) \* ...
- 최소 공배수를 구하는 것은 최대공약수를 구하는 것과 관계가있다.
  - 두 정수 N1 과 N2의 최대공약수는
  - N1 = gcd(N1, N2) X n1
  - N2 = gcd(N1, N2) X n2 일때
  - lcm(N1, N2) = gcd(N1, N2) X n1 X n2
  - 이때, n1 = N1 / gcd(N1, N2) 이고, n2 = N2 / gcd(N1, N2)이므로 간단하게 다음과 같은 식으로도 구할 수 있다.
  - lcm(N1, N2) = N1 X N2 / gcd(N1, N2)
  - 따라서 최대공약수를 구하면 최소공배수를 구할 수 있다.

```js
function lcm(m, n) {
  return (m * n) / gcd(m, n);
}
```

## 참고

- 코딩인터뷰 완전정복
-
