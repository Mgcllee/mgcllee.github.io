---
title:  "[알고리즘] 사이클 그래프 판별하기"
excerpt: ""

categories: [Algorithm, 경로 탐색]
tags: [Algorithm, Cycle]

toc: true
toc_sticky: true
math: true
 
date: 2024-11-05
last_modified_at: 2024-11-05
---

그래프에서 사이클을 판단하는 것은 매우 중요합니다.  
여러 그래프 탐색 알고리즘에서도 제한시간 이내에 탐색을 완료할 수 있도록  
사이클에 빠지지 않기 위해 방문을 기록하는 배열을 사용하기도 합니다.  

사이클과 관련된 알고리즘 문제에서는 대표적으로  
사이클의 **존재 여부**를 판별하는 문제와 사이클에 **포함된 노드를 활용**하는 문제가 있습니다.  

탐색하고자 하는 그래프의 방향성에 따라 그래프의 판별 방법이 다릅니다.  
무방향(양방향) 그래프의 경우, Union-Find, DFS 알고리즘을 이용해 사이클을 판단할 수 있으며,  
방향(단방향) 그래프의 경우, DFS 알고리즘을 사용해 사이클을 판단할 수 있습니다.  

<br/>

## Union-Find를 사용한 사이클 판단

Union-Find를 사용한 사이클 판단은 Union-Find 알고리즘을 변형할 필요없이 바로 사용할 수 있다는 장점이 있습니다.  
그러나 아래의 코드처럼 알고리즘이 동작할 때, 그래프의 방향성은 고려되지 않기 때문에  
방향 그래프에서는 사용할 수 없습니다.  

그래프에 있는 모든 간선을 순회하며 양쪽에 있는 노드의 부모를 찾으며  
주어진 간선으로 연결된 노드가 사이클에 속하는지 판단합니다.  

```c++
#include <iostream>
#include <vector>

using namespace std;

vector<int> parent;

int Find(int curr) {
    if(curr == parent[curr]) return curr;
    else return parent[curr] = Find(parent[curr]);
}

void Union(int a, int b) {
    a = Find(a);
    b = Find(b);

    if(a < b) {
        parent[b] = a;
    } else {
        parent[a] = b;
    }
}

int main() {
    vector<vector<int>> graph = {
        {1, 2, 5},  // 0
        {0},        // 1
        {0, 4, 5},  // 2
        {5},        // 3
        {2},        // 4
        {0, 2, 3}   // 5
    };  
    
    int Node_cnt = graph.size();

    // 각 노드의 부모는 자기 자신으로 초기화
    for(int i = 0; i < Node_cnt; ++i) {
        parent.push_back(i);
    }

    for(int start_node = 0; start_node < Node_cnt; ++start_node) {
        // 간선에 따른 그래프 연결
        for(int next : graph[start_node]) {
            if(Find(start_node) == Find(next)) {
                cout << "Find Cycle!\n";
                return 0;
            } 
            
            Union(start_node, next);
        }
    }

    cout << "doesn't find Cycle...\n";
    return 0;
}
```

Union-Find 알고리즘은 그래프 내에 사이클 여부를 빠르게 판단할 수 있다는 장점이 있지만  
만약 사이클에 속하는 노드가 무엇인지 알고 싶은 경우, Union-Find 알고리즘으로는 확인할 수 없습니다.  

<br/>

## DFS를 사용한 사이클 판단

DFS 알고리즘을 사용한 사이클 판단은 그래프 간선의 방향성과 상관없이 모두 사용할 수 있으며,  
추가로 어떤 노드가 사이클에 속해 있는지까지 판단할 수 있습니다.  
그러나 Union-Find 알고리즘을 이용한 판단과 비교해 비용이 더 필요합니다.  

```c++
#include <iostream>
#include <vector>
#include <unordered_set>
#include <stack>

using namespace std;

vector<vector<int>> graph;
unordered_set<int> cycle_nodes;

bool DFS(int curr_node, int prev_node, vector<bool>& visited) {
    visited[curr_node] = true;
    cycle_nodes.insert(curr_node);

    for (int next_node : graph[curr_node]) {
        if (false == visited[next_node]) {
            if (DFS(next_node, curr_node, visited)) {
                return true;
            }
        } 
        // 사이클 판별용
        else if (next_node != prev_node) {
            cycle_nodes.insert(next_node);
            return true;
        }
    }

    cycle_nodes.erase(curr_node);
    return false;
}

unordered_set<int> find_cycle(const vector<vector<int>>& graph) {
    vector<bool> visited(graph.size(), false);
    unordered_set<int> cycle_nodes;

    for (int i = 0; i < graph.size(); ++i) {
        if (!visited[i]) {
            visited[i] = true;
            if (DFS(i, -1, visited)) {
                return cycle_nodes;
            }
        }
    }

    return {};
}

int main() {
    // 주어진 그래프: 방향 그래프를 인접 리스트로 표현
    vector<vector<int>> graph = {
        {1, 2, 5},  // 0
        {0},        // 1
        {0, 4, 5},  // 2
        {5},        // 3
        {2},        // 4
        {0, 2, 3}   // 5
    };

    auto cycles = find_cycle(graph);
    for(int node : cycles) {
        cout << node << ", ";
    }

    return 0;
}
```

<br/>
