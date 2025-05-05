---
title:  "[C++] sstream으로 문자열 조작하기"
excerpt: ""

categories: [Language, C&#47;C&#43;&#43;]
tags: [C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2025-05-05
last_modified_at: 2025-05-05
---

<br/>

## 문자열 조작이 필요한 이유

여러 종류의 알고리즘 문제에서 자주 사용되는 것이 "문자열 조작"입니다.  
한 문자열 안에 여러 정보가 함께 주어지거나 변수와 함께 특정 형식으로 출력해야 하는 경우가 있습니다.  

예를들어, 공백으로 구분된 여러 정보가 한 개의 문자열로 주어질 때  
각 정보를 하나씩 vector에 저장하는 함수를 구현하려고 합니다.  
이 함수를 하드 코딩으로 구현하면 아래와 같이 구현할 수 있습니다.  

<br/>

```c++
#include<iostream>
#include<string>
#include <vector>

using namespace std;

int main() {
	string order = "ABC 123 3.14";
  vector<string> info;
	int start_index = 0;
	for(int end_index = 1; end_index < order.size(); ++end_index) {
	  if(order[end_index] == ' ') {
	    user_name.push_back(string(order.begin() + start_index, order.begin() + end_index));
	    start_index = end_index + 1;
	  } else if(end_index + 1 == order.size()) {
	  	user_name.push_back(string(order.begin() + start_index, order.end()));
	  }
	}

	for(auto name : user_name) {
		cout << name << endl;
	}

  /*
  [출력 결과]
  ABC
  123
  3.14
  */

  return 0;
}
```

위와 같은 코드는 가독성이 떨어져 기능 수정, 확장이 어렵다는 단점이 존재합니다.  
C++ STL에서는 sstream 헤더에서 위 코드를 개선할 수 있는 여러 기능을 제공합니다.  

sstream 헤더에는 여러 클래스가 존재하지만 이 포스트에서는 자주 사용되는  
**'istringstream, ostringstream, stringstream'**에 대해서 설명하겠습니다.  

<br/>

## istringstream

이 클래스는 생성자로 전달받은 문자열에서 공백(' ', '\n', '\t' 등)을 기준으로 나눌 수 있습니다.  
위에서 하드 코딩했던 내용을 사용하면 아래의 코드처럼 개선할 수 있습니다.  

<br/>

```c++
#include <iostream>
#include <sstream>

using namespace std;

int main() {
	istringstream iss("ABC 123 3.14");
	string alpha;
	int number;
	float PI;
	iss >> alpha >> number >> PI;
	
	cout << alpha << endl << number << endl << PI;
	return 0;
}
```

위 코든느 하드 코딩 보다 비교적 간략하고  
심지어 원하는 자료형으로 전달받을 수 있다는 장점이 있습니다.  
반드시 int, float으로 받아야 하는 것이 아니기 때문에 3개의 정보를 문자열로 받을 수 있습니다.  

<br/>

## ostringstream

이 클래스는 C++20의 format 헤더와 비슷하게 사용할 수 있습니다.  
이 클래스 객체와 **<< 연산자**로 함께 원하는 모양의 문자열을 만들 수 있습니다.  

<br/>

```c++
#include <iostream>
#include <sstream>

using namespace std;

int main() {
	ostringstream oss("ABC 123 3.14");
	oss << "ABC" << endl << 123 << endl << 3.14;
	
  cout << oss.str();
	return 0;
}
```

이 클래스 객체는 문자열만 아니라 다른 기본 자료형도 받을 수 있다는 특징이 있습니다.  

<br/>

## stringstream

이 클래스는 이전 istringstream, ostringstream의 기능을 함께 사용할 수 있는 클래스입니다.  

<br/>

```c++
#include <iostream>
#include <sstream>

using namespace std;

int main() {
  // istreamstring처럼 사용하기
	stringstream iss("ABC 123 3.14");
	string alpha;
	int number;
	float PI;
	iss >> alpha >> number >> PI;
	cout << alpha << endl << number << endl << PI << endl;
	
  // ostreamstring처럼 사용하기
	stringstream oss("ABC 123 3.14");
	oss << "ABC" << endl << 123 << endl << 3.14;
	cout << oss.str();
	return 0;
}
```

추가로 동일한 유형의 문자열이 반복적으로 주어질 경우 아래와 같이 사용할 수 있습니다.

```c++
#include <iostream>
#include <sstream>

using namespace std;

int main() {
  // istreamstring처럼 사용하기
	stringstream ss("my_name your_name our_name");
	string name;
  while(ss >> name) {
    cout << name << endl;
  }
	return 0;
}
```

<br/>

## 함께 풀면 좋은 문제

[프로그래머스 - "오픈채팅방"](https://school.programmers.co.kr/learn/courses/30/lessons/42888)  

