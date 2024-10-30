---
title:  "[C++] dynamic_cast 연산자"
excerpt: ""

categories: [Language, C&#47;C&#43;&#43;]
tags: [C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-10-12
last_modified_at: 2024-10-12
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

이번 포스트에서는 dynamic_cast 연산자에 대해서 알아보겠습니다.  

<br/>

## 업캐스팅과 다운캐스팅
---

dynamic_cast를 사용하기 위한 사전 지식인 **업캐스팅과 다운캐스팅**에 대해서 설명하겠습니다.  

```c++
class Car {}
class Bus : public Car {}

int main()
{
    Bus bus;

    // [업 캐스팅]: 파생(자식) 클래스의 객체를 기본(부모) 클래스의 객체처럼
    Car* pcar = &bus; 

    Car car;

    // [다운캐스팅]: 기본(부모) 클래스의 객체를 파생(자식) 클래스의 객체처럼 (업 캐스팅의 반대)
    Bus* pbus = (Bus*)&car;
}
```

파생 클래스의 포인터를 기본 클래스의 포인터로 변환하는 것을 업캐스팅  
기본 클래스의 포인터에서 파생 클래스의 포인터로 변환하는 것을 다운캐스팅이라고 합니다.  

업캐스팅, 다운캐스팅은 C++의 특징인 다형성을 이용해서 코드의 재사용률을 높이기 위해서입니다.  
여기서 dynamic_cast 는 **안전한 다운 캐스팅** 을 위해서 사용됩니다.  

<br/>

## dynamic_cast
---

```c++
class Car {}
class Bus : public Car {}

int main()
{
    Bus bus;
    Car* pcar = new Car();

    // dynamic_cast<바꾸려는 데이터 타입>(변환 대상)
    Bus* pbus = dynamic_cast<Bus*>(pcar);
}
```

사용 방법은 다른 변환 연산자처럼 **데이터 타입**, **변환 데이터** 가 필요합니다.  
dynamic_cast는 안전한 다운 캐스팅을 위해서 런타임에 실제로 변환이 가능한지 검사합니다.  
때문에 런타임 비용이 다른 변환 연산자에 비해서 높습니다.  

| 결과               | 반환 내용                   |
| ------------------ | --------------------------- |
| 성공               | 변환된 데이터를 반환합니다. |
| 포인터 변환 실패   | nullptr 반환                |
| 레퍼런스 변환 실패 | 예외 처리                   |
