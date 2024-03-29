---
layout: post
title: "[알고리즘] 알고리즘에 필요한 기초 수학 정리"
date: 2019-05-09 19:00:00
author: Jewoo.Song
categories: 알고리즘_수학
tags: 수학 알고리즘 에라토스테네스의체 에라토스테네스 순열 소수
---

# 수학

알고리즘 문제 풀이에 많이 등장하는 수학 문제들과 유용하게 사용할 수 있는 몇 가지 수학들입니다.

### 확률

- AnB의 확률
  - A에 속해있으면서도 B에 속해 있을 확률
  - P(AnB) = P(B\|A)P(A)
  - 예) 1~10까지의 수 중 5이하 이면서, 짝수일 확률
  - 5이하일 확률 = 50%, 5이하 숫자 중 짝수일 확률 = 40%
  - 1/2 \* 2/5 = 1/5
  - 베이즈 정리
    - P(AnB) = P(B\|A)P(A) = P(A\|B)P(B)
    - P(A\|B) = P(B\|A)P(A) / P(B)
- AuB의 확률
  - P(AuB) = P(A)+P(B)-P(AnB)

#### 독립성과 상호 배타성

- 독립성
  - A와 B가 독립사건(한 사건의 발생과 다른 사건의 발생 사이에 아무런 관계가 없는 경우)이라면, A가 B에 아무런 영향을 끼치지 않는다.
  - P(B\|A) = P(B)
  - P(AnB) = P(A)P(B)
- 상호 배타성(mutual exclusivity)
  - A와 B가 상호 배타적(한 사건이 일어난 경우 다른 사건은 발생할 수 없는 경우)라면,
  - P(AnB) = 0
  - P(AuB) = P(A) + P(B)
- 독립성과 상호 배타성은 완전히 다른 개념이다.
  - 상호 배타성은 한 사건이 발생하면 다른 사건이 발생할 수 없다는 관계가 존재한다.
  - 독립성은 한 사건의 발생 여부가 다른 사건에 아무런 영향을 미치지 않아야 한다.
  - 따라서 두 사건의 확률이 0보다 큰 경우 두 조건을 동시에 만족시킬 수 없다.
  - 두 사건 중 하나의 확률이 0이라면, 두 사건은 독립적이면서 상호 배타적이다.

### 1부터 n가지의 합

- 1 + 2 + 3 + .... + n 은 짝은 값과 큰 값을 하나의 쌍으로 만들어서 생각해 볼 수 있다.
  - n이 짝수라면 1과 n, 2와 n-1의 순으로 짝을 짓는다.
    - 합이 n+1이 되는 쌍이 n/2개 나온다.
    - => (n+1)\*n/2
  - n이 홀수라면 0과 n, 1과 n-1의 순으로 짝을 짓는다.
    - 합이 n이 되는 쌍이 (n+1)/2개 나온다.
    - => n\*(n+1)/2
- 1 + 2 + 3 + .... + n = n\*(n+1)/2

### 2의 승수의 합

- 2^0 + 2^1 + 2^2 + ... 2^n

  - 값을 2진수로 생각해 보면 간단하다.
  - 0부터 n까지의 2의 승수의 합 = 2^(n+1) - 1

| 승수        | 2진수 | 10진수 |
| ----------- | :---: | -----: |
| 0           | 00001 |      2 |
| 1           | 00010 |      4 |
| 2           | 00100 |      8 |
| 3           | 01000 |     16 |
| 4           | 10000 |     32 |
| 합 : 2^5 -1 | 11111 |     64 |

### 로그의 밑

log_2를 log_10으로 바꿀 수 있는 방법은 로그의 법칙을 통해 증명할 수 있다.

- 로그의 법칙
  - log_b(k) log_x(k)가 있을 때,
  - log_b(k) = c
  - b^c = k
  - log_x(b^c)
  - c \* log_x(b) = log_x(k)
  - c = log_x(k) / log_x(b)
  - 따라서,
  - log_2(p)를 log_10으로 변환 방법은 아래와 같다.
  - log_10(p) = log_2(p) / log_2(10)

### 순열

중복되는 문자가 없을 때, 길이가 n인 문자열을 재배열하는 방법은 n!개이다.

- 첫 번째 위치에 놓을 수 있는 문자 n개
- 두 번째 위치에 놓을 수 있는 문자 n-1개 ...
- n! = n*(n-1) * (n-2) \* (n-2)

총 n개의 문자로 길이가 k인 문자열을 만들때는 다음과 같다.

- n!/(n-k)! = n \* (n-1) \* (n-2) .... \* (n-k+1)

### 조합

n개의 문자의 집합이 있을 때, 이 중에서 k개의 문자를 선택하여 새로운 집합을 만들 때 만들 수 있는 부분집합의 수 구하기.

- n개의 원소 중 크기가 k인 부분집합의 개수(순서에 상관없이)
- nCk
- 길이가 k인 부분 문자열의 개수 n!/(n-k)!
- 길이가 k인 부분집합을 재배치하여 만들 수 있는 문자열의 개수는 k! 이므로,
- 각 부분 문자열 리스트에는 각각의 부분집합이 k!번 반복 사용되었다.
- n!/k!(n-k)!

## 참고

- 코딩인터뷰 완전정복
