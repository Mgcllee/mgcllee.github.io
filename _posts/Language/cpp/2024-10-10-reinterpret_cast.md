---
title:  "[C++] reinterpret_cast 연산자"
excerpt: ""

categories: [Language, C&#47;C&#43;&#43;]
tags: [C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-10-10
last_modified_at: 2024-10-10
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

## reinterpret_cast
---

reinterpret_cast 는 **포인터** 와 관련된 형변환만 지원합니다.  
포인터가 다른 포인터 형식으로 변환하거나 정수형을 포인터로 변환할 수 있습니다.  

이 연산자는 원본 데이터를 새로운 형식으로 변환할 때, 데이터의 유효성을 따지지 않고 변환합니다.  
따라서 변환 후 데이터가 유효한지 알 수 없어 사용에 주의해야 합니다.  

예를들어, **int&#42;**, **char**, **int&#42;** 순서대로 변환한다면 처음 있던 int 값은 어떻게 될까요?  
우선, 포인터 변수는 주소값만 저장하기 때문에 (64bit 기준) 8byte 의 공간을 사용합니다.  
이를 1byte 공간인 char 형으로 변환한다면 7byte(8byte - 1byte)가 손실되고  
다시 int* (8byte) 로 변환하면 원래 데이터에서 7byte 가 **손실된 값** 이 전달됩니다.  

<br/>

## 사용 예시

```c++
int* integer = new int(97);
char* character = reinterpret_cast<char*>(integer);

// [성공]: integer : 97 character : a
printf("integer : %d character : %c", *integer, *character);


// [오류]: 데이터 손실이 발생할 수 있습니다. (8byte -> 1byte)
char ch = reinterpret_cast<char>(integer);
```

<br/>

## 참고

[[c++] reinterpret_cast, 정체불명의 모모님 블로그](https://uncertainty-momo.tistory.com/72)  
[[C++] reinterpret_cast (타입캐스트 연산자), 개발자 지망생님 블로그](https://blockdmask.tistory.com/242)  
