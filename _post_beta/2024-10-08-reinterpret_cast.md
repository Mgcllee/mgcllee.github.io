---
title:  "[C++] reinterpret_cast 연산자"
excerpt: ""

categories: [Language, C&#47;C&#43;&#43;]
tags: [C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-10-08
last_modified_at: 2024-10-08
---

자료형을 변환할 때, C 언어는 **암시적 형변환과 명시적 형변환** 2가지가 존재합니다.  
간략하게 설명하면 암시적 형변환은 컴파일러가 대신 수행하는 형변환이고  
명시적 형변환은 반대로 프로그래머가 직접 형변환을 수행하는 것입니다.  

```c++
int i_num = 3.14;                   // 암시적

double d_num_01 = double(i_num);    // 명시적
double d_num_02 = (double)i_num;    // 명시적
```

그렇다면 C++ 는 형 변환 연산자가 무엇이 있을까요?  
C++ 에서 사용하는 형변환 연산자는 아래의 4가지가 있으며 모두 다른 역할을 수행합니다.  

```c++
static_cast<data_type>(Data);
const_cast<data_type>(Data);
dynamic_cast<data_type>(Data);
reinterpret_cast<data_type>(Data);
```

이번 포스트에서는 reinterpret_cast 연산자에 대해서 알아보겠습니다.  

<br/>

# **reinterpret_cast**
---

reinterpret_cast 는 포인터/참조와 관련된 형변환만 지원합니다.

```c++
int* integer = new int(97);
char* character = reinterpret_cast<char*>(integer);

// 실행결과 : integer : 97 character : a
printf("integer : %d character : %c", *integer, *character);
```

단, 변환 대상에 상수값이 들어갈 수 없습니다.  
때문에 상수값을 전달하고 싶을 경우 ‘const_cast’를 사용해야 전달할 수 있습니다.  

<br/>
