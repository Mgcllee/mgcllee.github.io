---
title:  "[C++] find와 bianry_search (STL 구현 확인하기)"
excerpt: ""

categories: [Language, C&#47;C&#43;&#43;]
tags: [C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
math: true
 
date: 2024-10-09
last_modified_at: 2024-10-09
---

백준의 [Sort 마스터 배지훈의 후계자](https://www.acmicpc.net/problem/20551)라는 문제는 정렬과 탐색으로 간단히 풀 수 있는 문제였습니다.  
저는 algorithm 라이브러리의 sort, find 함수를 통해 문제를 풀고자 하였습니다.  

<br/>

## 첫 번째 시도 (실패)

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int main() {
	cin.tie(0)->sync_with_stdio(false);
	cout.tie(0);

	int A, M;
	cin >> A >> M;
	
	int* arr = new int[A];
	for(int i = 0, key; i < A; ++i) {
		cin >> arr[i];
	}

	sort(arr, arr + A);

	int quest;
	for(int i = 0; i < M; ++i) {
		cin >> quest;
		int target_index = it = find(arr, arr + A, quest) - arr;
		if(A == target_index) {
			cout << "-1\n";
		} else {
			cout << target_index << '\n';
		}
	}

	return 0;
}
```

그러나 이 풀이는 **시간 초과** 가 되었습니다.  
코드를 개선하기 위해서 문제를 다시 확인하던 중 정렬된 데이터들에서 원하는 데이터를 찾기 때문에  
이진 탐색(binary search) 또한 가능하다는 것을 확인했고 코드를 수정하였습니다.  

<br/>

## 두 번째 시도 (성공)

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int main() {
	cin.tie(0)->sync_with_stdio(false);
	cout.tie(0);

	int A, M;
	cin >> A >> M;
	
	int* arr = new int[A];
	for(int i = 0, key; i < A; ++i) {
		cin >> arr[i];
	}

	sort(arr, arr + A);

	int quest;
	for(int i = 0; i < M; ++i) {
		cin >> quest;
		if(false == binary_search(arr, arr + A, quest)) {
			cout << "-1\n";
		} else {
			cout << lower_bound(arr, arr + A, quest) - arr << '\n';
		}
	}

	return 0;
}
```

<br/>

## 오답 사유

C++ STL 의 algorithm 라이브러리는 어떠한 함수든 반복자를 넘겨주면 최적화된 로직으로  
연산을 수행할 줄 알았지만 아니였습니다.  

[cppreference.com](https://en.cppreference.com/w/cpp/algorithm/find)을 참고하면 find 함수의 구현은 다음과 같습니다.  

```c++
template<class InputIt, class T = typename std::iterator_traits<InputIt>::value_type>
constexpr InputIt find(InputIt first, InputIt last, const T& value)
{
    for (; first != last; ++first)
        if (*first == value)
            return first;
 
    return last;
}
```

find 함수는 first 부터 last 까지 **순차 탐색** 으로 구현되어 있었습니다.  
최악의 경우 시간복잡도가 $$O(N^2)$$ 이고 주어진 배열과 찾고자 하는 숫자의 개수가 $$10^5$$ 일 때,  
제한 시간에 끝나지 않을 수 있습니다.  

> 데이터의 개수가 $$ 10^9 $$ 이면 연산이 **1초** 가 소요된다고 할 때,  
> 순차 탐색을 이용하면 $$O((2 \times 10^5)^2) = O(4 \times 10^{10})$$ 이므로 약 40초가 소요됩니다.  

<br/>

반면 이진 탐색은 $$O(log(N))$$ 의 속도를 보여주기 때문에 제한 시간 안에 문제를 해결할 수 있었습니다.  