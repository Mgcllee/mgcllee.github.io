---
title:  "[C++] 조건문 속 변수 선언의 주의점(스코프)"
excerpt: ""

categories: [Language, C&#47;C&#43;&#43;]
tags: [C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-11-08
last_modified_at: 2024-11-08
---

[알고리즘 문제](https://www.acmicpc.net/problem/2607) 중, if()문의 조건식 안에 변수를 선언해 풀고자 한 문제가 있었습니다.  
제가 작성한 코드는 아래와 같습니다.  

```c++
for(auto it = target.begin(); it != target.end(); ++it) {
    if(int t = original.find(*it) == string::npos) {
        next_target += *it;
    } else {
        something_deleted = true;
        original.erase(original.begin() + t);
    }
}
```

<br/>

위 코드는 MSVC기준 컴파일 오류가 발생하지 않은 코드입니다.  
그러나 조건문의 지역변수인 t의 값이 else문으로 넘어가지 않았습니다.  
따라서 original의 문자를 제거하는 동작이 예상과 다르게 동작하였습니다.  

이 문제를 해결하기 위해 여러 가능성을 확인하던 중,  
연산자 우선순위가 다르다는 것을 기억하였고 위 코드는 연산자 우선순위에 의해 아래와 같이 실행되었습니다.  

```c++
if(int t = (original.find(*it) == string::npos)) {
    next_target += *it;
} else {
    something_deleted = true;
    original.erase(original.begin() + t);
}
```

<br/>

따라서 t의 값은 1(true), 0(false) 값만 나왔기 때문에 결과값이 예상과 달랐던 것 입니다.  
이 문제를 해결하기 위해 변수 t를 조건문 밖에서 선언하도록 수정하였고, 원하는 결과값을 출력할 수 있었습니다.  

```c++
int t = original.find(*it);
if(t == string::npos)) {
    next_target += *it;
} else {
    something_deleted = true;
    original.erase(original.begin() + t);
}
```

<br/>

## 참고

[Cppreference.com의 Block scope 설명](https://en.cppreference.com/w/cpp/language/scope)  

![블록스코프](/assets/img/Cpp/scope_block.png){: width="600", height="600"}  
<center>[Cppreference.com의 Block scope 설명]</center>
