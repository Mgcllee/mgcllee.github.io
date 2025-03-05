---
title:  "[프로그래머스] JadenCase 문자열 만들기"
excerpt: ""

categories: [CodingTest, Programmers]
tags: [CodingTest, Programmers]

toc: true
toc_sticky: true
math: true
 
date: 2024-09-24
last_modified_at: 2024-09-24
---

> 코딩테스트 연습 - JadenCase 문자열 만들기 [바로가기](https://school.programmers.co.kr/learn/courses/30/lessons/12951)  

<br/>

## 문제 설명

<br/>

![01](/assets/img/Programmers/JadenCase.png){: width="550" height="450" }  

<br/>

## 잘못된 문제 접근

문제를 처음 풀 때, 마지막 제약 조건을 확인하지 못해서 C++ stringstream 을 활용한 풀이로 접근했었습니다.  
stringstream 은 문자열 속 공백 개수와 상관없이 공백이 존재한다면  
그곳을 기준으로 **공백이 아닌 부분만** 반환하기 때문에 첫 시도에서 실패했었습니다.  
<br/>

```c++
#include <string>
#include <sstream>
#include <vector>

using namespace std;

string solution(string s) {
    stringstream ss(s);
    string buf, answer = "";
    while(ss >> buf) {
        for(char& c : buf) {
            c = tolower(c);
        }
        if(isalpha(buf[0])) {
            buf[0] = toupper(buf[0]);
        }
        answer += (buf + " ");
    }
    answer.pop_back();
    return answer;
}
```

<br/>

## 옳게된 문제 풀이

제약 조건을 확인해 보면 문자열의 최대 길이는 200 이므로  
시간 복잡도가 $$O(N^3)$$ 이내 완료되는 알고리즘을 선택하면 되므로  
저는 **순차 탐색** 을 사용했습니다.  
<br/>

```c++
#include <string>
#include <vector>

using namespace std;

string solution(string s) {
    if(isalpha(s[0])) {
        s[0] = toupper(s[0]);
    }
    
    for(int i = 1; i < s.length(); ++i) {
        if(s[i - 1] == ' ' && isalpha(s[i])) {
            s[i] = toupper(s[i]);
        } else if(isalpha(s[i])) {
            s[i] = tolower(s[i]);
        }
    }
    return s;
}
```