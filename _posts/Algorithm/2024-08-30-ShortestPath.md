---
title:  "[Algorithm] 최단 경로 알고리즘"
excerpt: ""

categories:
  - Algorithm
tags:
  - [Algorithm, Dijkstra, FloydWarshall]

toc: true
toc_sticky: true
math: true
 
date: 2024-08-30
last_modified_at: 2024-08-30
---

> **이것이 취업을 위한 코딩 테스트다 with 파이썬 (나동빈 저)** 를 참고해 작성한 포스트입니다.

<br/>

## 가장 빨리 도달하는 방법

최단 경로 알고리즘은 이름 그대로 목적지까지 가장 짧은 경로를 찾는 알고리즘입니다.  
최단 경로 유형의 문제로는 "한 지점에서 특정 지점까지의 최단 경로를 구해야 하는 경우",  
"모든 지점에서 다른 모든 지점까지의 모두 최단 경로를 구해야 하는 경우" 등 여러 가지가 있습니다.  
<br/>

최단 경로 알고리즘 문제를 풀 때 자료구조는 그래프로 표현합니다.  
그래프의 노드를 방문할 수 있는 지점으로 설정하고 노드를 연결하는 간선을 이동 비용으로  
설정할 수 있기 때문입니다.  
<br/>

최단 경로 알고리즘의 경우 다익스트라(Dijkstra) 최단 경로 알고리즘, Floyd Warshall 알고리즘,  
벨만 포드 알고리즘 등 여러 알고리즘이 존재하며  
이번 포스트에서는 자주 사용되는 다익스트라 최단 경로 알고리즘을 설명하겠습니다.  
<br/>

## 다익스트라(Dijkstra) 최단 경로 알고리즘

다익스트라 알고리즘은 한 지점에서 특정 지점까지의 최단 경로를 구해야 하는 경우에 사용되는 알고리즘입니다.  
그래프 자료구조를 사용하고 간선에는 **음이 아닌 값** 으로 표현됩니다.  
<br/>

알고리즘의 실행 순서와 코드는 다음과 같습니다.  

1. 출발 노드를 설정합니다.  
2. 최단 거리 테이블을 초기화합니다.  
3. **방문하지 않은 노드 중 최단 거리가 가장 짧은 노드를 선택** 합니다.  
4. 선택된 노드에 연결된 간선을 이용해 최단 거리 테이블을 갱신합니다.  
5. 위 3, 4번 과정을 반복합니다.  
<br/>

```c++
void dijkstra(int start)
{
    distance[start] = 0;
    visited[start] = true;

    // next[0]: 노드 번호, next[1]: 간선에 저장된 이동 비용
    for(vector<int> next : graph[start])
    {
        distance[next[0]] = next[1];
    }

    for(int node = 0; node < node_cnt; ++node)
    {
        // 최단 거리가 가장 짧은 노드를 현재 노드로
        int curr = get_smallest_node();
        visited[curr] = true;

        // 현재 노드에 연결된 모든 노드까지의 비용 확인
        for(vector<int> next : graph[curr])
        {
            // next 노드에 대한 새 비용 저장
            int cost = distance[curr] + next[1];

            // 새 비용이 기존 비용보다 작은 경우 갱신
            if(cost < distance[next[0]])
            {
                distance[next[0]] = cost;
            }
        }
    }
}
```

<br/>

매번 최단 거리가 가장 짧은 노드(get_smallest_node())를 구해야 하고  
구해진 노드에 연결된 모든 노드를 순차 탐색하므로 노드의 개수가 N 일때, 시간 복잡도가 $$O(N^2)$$ 가 됩니다.  
여기서 **최단 거리가 가장 짧은 노드** 연산의 시간 복잡도를 줄이는 방법으로  
힙 자료구조를 사용한 우선순위 큐를 사용해 개선할 수 있습니다.  
<br/>

```c++
void dijkstra(int start)
{
    priority_queue<pair<int, int>> pq;
    pq.push({0, start});

    while (false == pq.empty()) {
        int curr_cost = -pq.top().first;
        char curr_node = pq.top().second;
        pq.pop();

        if (curr_cost > distances[curr_node])
        {
            continue;
        }

        for (auto next : graph[curr_node]) {
            char next_node = next.first;
            int next_cost = curr_cost + next.second;

            if (next_cost < distances[next_node]) {
                distances[next_node] = next_cost;
                pq.push({-next_cost, next_node});
            }
        }
    }
}
```

<br/>

C++ 의 우선순위 큐는 최대힙을 사용하기 때문에  
최단 거리가 가장 짧은 노드를 저장하기 위해서는 push 전 $$-cost$$ 처럼 부호를 변경해  
가장 짧은 노드가 먼저 힙에서 나올 수 도록 하였습니다.  
개선된 다익스트라 알고리즘의 시간 복잡도는 E가 간선 수, N 이 노드의 수 일때  
$$O(Elog(N))$$ 가 될 수 있습니다.  
<br/>
