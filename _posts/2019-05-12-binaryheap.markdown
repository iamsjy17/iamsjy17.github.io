---
layout: post
title: "[알고리즘] 힙과 힙 정렬"
date: 2019-05-12 19:00:00
author: 송튜디오
categories: 알고리즘_정렬
tags: 힙 이진힙 heap binaryheap 힙정렬
---

## 힙(heap)

- Complete Binary Tree의 일종이면서 Heap의 특성을 만족해야한다.
  - Max Heap Property : 부모는 자식보다 크거나 같다.
  - Min Heap Property : 부모는 자식보다 작거나 같다.
- 우선순위 큐에서 사용되는 자료구조이다.
- 여러 개의 값들 중에서 최대값이나 최소값을 빠르게 찾아내도록 만들어진 자료구조이다.
- 힙 트리는 중복된 값을 허용한다.
- 힙은 느슨한 정렬 상태를 유지한다.
  - 큰 값이 상위 레벨에 있고, 작은 값이 하위 레벨에 있다는 정도
  - 때문에 같은 데이터로 힙 특성을 만족하는 여러 힙이 만들어 질 수 있다.
- 힙은 일차원 배열로 표현이 가능하다.
  - Compete Binary Tree이기 때문에 가능하다.
  - 루트 노드 A[1]
  - A[i]의 부모 = A[i/2]
  - A[i]의 왼쪽 자식 = A[2i]
  - A[i]의 오른쪽 자식 = A[2i+1]

### 힙의 종류

> Heap Sort에 활용하기에는 Max Heap이 더 간편하다.

- 최대 힙(max heap)
  - 부모 노드의 키 값이 자식 노드의 키 값보다 크거나 같은 완전 이진 트리
  - 부모 노드 key >= 자식 노드 key
- 최소 힙(min heap)
  - 부모 노드의 키 값이 자식 노드의 키 값보다 작거나 같은 완전 이진 트리
  - 부모 노드 key <= 자식 노드 key

### Heapify(Max Heap)

> Min Heap은 조건만 반대로 하면 된다.

- 힙 특성을 만족시키기 위한 연산
- Complete Binary Tree일 때, 오직 루트 노드만 Heap Property를 만족하지 않는 상태에서 전체 트리를 힙으로 만드는 동작
  - 왼쪽 자식 트리, 오른쪽 자식 트리는 그 자체로 Heap일 때.
- 기본 동작
  - 루트 노드와 두 자식들 중 큰 값과 위치를 교환한다.
  - 교환한 자식 노드를 루트로 하는 힙을 다시 Heapify한다.
  - 리프 노드에 도달하거나 두 자식 노드가 더 작은 경우 끝난다.
- Heapify의 시간 복잡도는 이진트리의 높이와 같다.
  - h = logn
  - O(logn)

```js
//특정 노드를 기준으로 Heapify 연산을 수행하는 함수
MAX_HEAPIFY(A, i)
{
    if there is no child of A[i]
        return;
    k <- index of the biggest child of i
    if A[i] >= A[k]
        return;
    exchange A[i] and A[k];
    MAX_HEAPIFY(A, K);
}
```

### 힙 정렬

#### 힙 생성

- 먼저 주어진 Complete Binary Tree를 힙으로 만든다.
- 밑바닥 가장 오른쪽 노드부터 왼쪽 위 방향으로 차례대로 최초로 Leaf노드가 아닌 노드부터 Heapify 연산을 시작한다. n/2 index부터 시작한다.
- 시간 복잡도 : O(n)
  - 1/2 n번 Heapify 연산
  - Heapify = O(logn)
  - 1/2 \* n \* logn
  - O(nlogn)
  - 그러나 Heapify연산은 루트 노드에 한해서만 logn이고, 그 외에 노드에 한해서는 logn보다 작은 시간에 수행된다.
  - 따라서 O(n)으로 표기할 수 있다.
  - 힙 생성 알고리즘이 O(logn)이던 O(n)이던 힙정렬 전체의 시간 복잡도에는 영향을 주지 못한다.
    - 2 \* O(logn) = O(logn)
    - O(n) + O(logn) = O(logn)

```js
BUILD_MAX_HEAP(A)
{
   heap-size[A] <- length[A
   for i <- length[A/2] downto 1
        do MAX_HEAPIFY(A, i)
}
```

#### 힙 정렬 과정

> 힙은 느슨한 정렬 상태를 유지하기 때문에 힙이 만들어 젔다고해서 모든 요소가 정렬되어있지는 않다. 따라서 힙 정렬 과정이 필요하다.

1. 주어진 데이터를 힙으로 만든다.
2. 힙에서 최대값(루트)를 가장 마지막 값과 바꾼다.
3. 힙의 크기가 1줄어든 것으로 간주한다. 즉, 가장 마지막 값은 힙의 일부가 아닌 것으로 간주한다.
4. 루트 노드에 대해서 HEAPIFY(1)한다.
5. 2~4번 반복한다.

- 시간 복잡도 : O(nlogn)

```js
HEAPSORT(A)
{
    BUILD_MAX_HEAP()                // O(n)
    for i <- heap_size downto 2 do  // n-1
        exchange A[1] <-> A[i]      // O(1)
        heap_size <- heap_size -1   // O(1)
        MAX_HEAPFIY(A, 1)           // O(logn)
}
```

## 참고

- [권오흠 교수님 유튜브](https://www.youtube.com/watch?v=ihyg2OR8IR0&list=PL52K_8WQO5oUuH06MLOrah4h05TZ4n38l&index=13&t=0s)
- [Heee's Development Blog](https://gmlwjd9405.github.io/)
