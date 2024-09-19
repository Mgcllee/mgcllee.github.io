---
title:  "[Algorithm][BackTracking] 백트래킹 알고리즘"
excerpt: ""

categories:
  - Algorithm
tags:
  - [Algorithm]

toc: true
toc_sticky: true
 
date: 2024-07-11
last_modified_at: 2024-07-11
---

백트래킹 기법은 이름처럼 **탐색 중 조건에 부합하지 않으면 되돌아 가는 기법**입니다.  

대표적인 예시 문제로 N-Queen 문제가 있습니다.  

N x N 체스판에서 N개의 퀸을 서로 이동이 겹치지 않도록 최대한 퀸을 배치하는 문제입니다.  

문제를 풀기위한 방법은 아래와 같습니다.  

<br/>

1. N 번째 퀸을 체스판에 배치합니다.  
2. 배치된 퀸을 두고 다른 퀸과 이동경로가 겹치는지 확인 후 문제가 없을 때,  
    다음 퀸을 배치합니다. 만약 **문제가 있으면 이전으로 되돌아갑니다.**  

<br/>

앞선 설명처럼 이 기법은 조건에 부합하지 않으면 직전 상태로 돌아가는 특성이 있습니다.  

이 특성을 구현하기 위해서 **깊이 우선 탐색**이 사용됩니다.  

DFS로 탐색 진행 중 문제 조건에 부합하지 않는 위치에 도착하면 이 전 위치로 되돌아 갑니다.  

이러한 탐색을 반복해 답을 구하는 것이 백트래킹 입니다.  

N-Queen 문제의 풀이 코드는 아래와 같습니다.  

<br/>

```c++
bool not_cross(int curr)
{
	for(int next = 0; next < curr; ++next)
	{
        // 새로운 퀸 배치(curr)가 기존의 값들과 1개라도 충돌할 경우
		if(col[next] == col[curr]
		|| std::abs(col[next] - col[curr]) == (curr - next))
		return false;
	}

	return true;
}

void DFS(int curr)
{
	if(curr == N)
		result += 1;
	else
	{
		for(int next = 0; next < N; ++next)
		{
			col[curr] = next;

            // 다음 칸이 문제 조건에 맞는지 확인하는 작업(not_cross() 함수)
			if(not_cross(curr)) 
				DFS(curr + 1);
		}
	}
}
```

<br/>

# 연습 문제
[[Silver I] 단풍잎 이야기](https://www.acmicpc.net/problem/16457)  
[[Gold IV] N-Queen](https://www.acmicpc.net/problem/9663)  
