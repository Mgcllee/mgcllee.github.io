---
title:  "[Algorithm] Union-Find 알고리즘"
excerpt: ""

categories: [Algorithm]
tags: [Algorithm, Union-Find, 서로소]

toc: true
toc_sticky: true
math: true
 
date: 2024-09-19
last_modified_at: 2024-09-19
---

## 서로소 집합을 찾기

서로 중복되지 않는 부분 집합들을 구현할 때, 사용되는 것이 Union-Find 알고리즘입니다.  
예를들어, 여러 사람의 관계가 그래프의 노드(사람)과 엣지(관계)로 표현될 때  
관계 그래프의 총 개수를 확인할 수 있습니다.  
이때, 반드시 확인해야 할 것은 두 집합이 반드시 **상호 배타적인 부분 집합(서로소 집합)** 이여야 한다는 점입니다.  
사람 관계 그래프에서는 A 그룹과 B 그룹 모두 속해서는 안된다. 라는 규칙이 있어야  
상호 배타 부분 집합이 성립되어 Union-Find 알고리즘을 사용할 수 있습니다.  
<br/>

![서로소](/assets/img/Algorithm/DisjointSets.png){: width="440" height="180" }  
<center>[A집합과 B집합은 서로소이다.]</center>

<br/>

## 동작 원리

Union-Find 알고리즘의 동작은 합치기(Union)와 찾기(Find)의 반복으로 동작합니다.  
먼저 합치기와 찾기 동작이 각각 어떻게 수행되는지 살펴보겠습니다.  
<br/>

### 합치기(Union)

> union(A, B)

합치기 연산은 원소 A가 속한 집합과 B가 속한 집합을 **하나로 합치는** 연산입니다.  
<br/>

### 찾기(Find)

> find(C)

찾기 연산은 원소 C가 어떤 집합 **소속** 인지 찾는 연산입니다.
<br/>
<br/>

마지막으로 모든 원소에 대해 합치기와 찾기 연산을 반복적으로 수행해주면  
서로소 집합을 찾을 수 있습니다.  
<br/>

## 코드 구현

```c++
// 각 원소를 인덱스로 표현하고 어느 집합 소속인지 표현하는 배열
int parent[MAX];

int Find(int C) {
    if(parent[C] == C) 
        return C;
    else 
        return parent[C] = Find(parent[C]);
}

void Union(int A, int B) {
    A = Find(A);
    B = Find(B);

    if(A < B)
        praent[B] = A;
    else
        praent[A] = B;
}
```

<br/>

이 코드 속 Find 함수의 "return parent[C] = Find(parent[C]);" 내용을  
"return Find(parent[C]);" 로 작성하여도 출력은 동일하지만 속도에서 차이가 발생합니다.  
Find 함수가 각 원소를 지나가면서 기록하기 때문에 이후 같은 값에 접근하면  
새롭게 탐색할 필요 없이 **압축된 경로** 를 바로 반환할 수 있어  
비교적 빠른 속도로 출력할 수 있습니다.  
<br/>

## 참고

[공부하는 식빵맘님 블로그](https://ansohxxn.github.io/algorithm/unionfind/)  
[나동빈님 블로그](https://blog.naver.com/ndb796/221230967614)  
[박지훈님 블로그](https://velog.io/@kimdukbae/Union-Find-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)  
