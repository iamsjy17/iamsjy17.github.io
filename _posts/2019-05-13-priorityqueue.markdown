---
layout: post
title: "[자료구조] 우선순위 큐"
date: 2019-05-13 19:00:00
author: 송튜디오
categories: 자료구조
tags: 힙 이진힙 우선순위큐 큐 priorityqueue
---

## 우선순위 큐

- 힙의 응용분야로 우선순위의 개념을 큐에 도입한 자료 구조로 데이터들이 우선순위를 가지고 있고, 우선순위가 높은 데이터가 먼저 나간다.
- 배열, 연결리스트, 힙으로 구현가능하며 이중에서 힙이 삽입/삭제시 O(logN)으로 가장 효율적이다.
- 운영 체제에서의 작업 스케쥴링등에 사용된다.
- 최대 우선순위 큐
  > 최소 우선순위 큐는 최대 우선순위 큐의 반대이다.
  - 두 가지 연산을 만족한다.
    - Insert(x) : 새로운 원소 x를 삽입한다.
    - ExtractMax() : 최대값을 삭제하고 반환한다.
  - Max Heap을 이용하여 최대 우선순위 큐를 구현한다.

### Insert (삽입)

> 최대 우선순위 큐를 예로한다.

- 삽입
  - 우선순위 조건을 만족하도록 유지하고 새로운 데이터를 삽입한다.
  - 최대 값 우선
  - 새로 삽입된 원소가 제대로 된 자리를 찾을 때까지 부모 노드와 교환해 나간다.
  - 최소힙에 원소를 삽입할 때는 언제나 트리의 밑바닥부터 삽입을 시작한다. (밑바닥 가장 오른쪽)
  - O(logN)

```js
MAX_HEAP_INSERT(A, key)
{
    heap_size = heap_size+1;
    A[heap_size] = key;
    i = heap_size;
    while(i>1 and A[PARENT(i)] < A[i]){
        exchange A[i] and A[PARENT(i)];
        i = PARENT(i); //PARENT = A[i/2]
    }
}
```

### ExtractMax (삭제)

- 최대값 뽑아내기
  - 최대값을 사용하는 것은 가장 위 원소를 사용하면된다.
  - 그러나 삭제하는 것은 까다롭다.
  - 최대 원소 제거 후 힙에 있는 가장 마지막 원소(밑바닥 가장 오른쪽)와 교환한다.
    - 힙의 기본 성질인 Complete Binary Tree를 만족하면서 원소를 삭제하기 위해서는 맨 마지막 노드 밖에 없다.
  - 마지막 원소와 교환하면, 최대힙의 성질이 깨지게 된다.
    - 그리고 최대힙의 성질을 만족하도록, 해당 노드를 자식 노드와 교환해 나감으로써 밑으로 내보낸다.
    - 반복적 Heapify 호출(max-heapify)
  - O(logN)

```js
HEAP_EXTRACT_MAX(A)
{
    if heap_size < 1
        then error "heap underflow"

    max = A[1];
    A[1] = A[heap_size];
    heap_size--;
    MAX_HEAPIFY(A, 1);
    return max;
}
```

## 참고

- [권오흠 유튜브](https://www.youtube.com/watch?v=2bp2ZSS3O0g&list=PL52K_8WQO5oUuH06MLOrah4h05TZ4n38l&index=15)
- [Heee's Development Blog](https://gmlwjd9405.github.io/)
