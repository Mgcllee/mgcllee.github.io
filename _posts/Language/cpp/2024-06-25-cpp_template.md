---
title:  "[C++] 템플릿"
excerpt: "템플릿의 개념과 사용"

categories:
  - C++
tags:
  - [C/C++]

toc: true
toc_sticky: true
 
date: 2024-06-25
last_modified_at: 2024-06-25
---

영어사전에서 **template** 의 뜻을 찾아보면 다음과 같습니다.

> **template : 형판, 견본, 본보기**

template 의 뜻인 형판처럼 C++ 에서의 template 또한 형판과 같은 역할을 합니다.  

template은 **함수 템플릿**과 **클래스 템플릿**이 존재합니다.  

<br/>

# 함수 템플릿
<br/>
예를들어 기능은 완전하게 똑같지만 **매개 변수의 자료형만 다른** 함수가 있다고 하면  

프로그래머는 동일한 함수를 자료형만 바꿔 작성해야 합니다.  

기능상 아무런 문제가 없지만 필요한 자료형이 하나씩 늘어날 때 마다 작성하기 때문에  

매우 비효율적인 작업입니다.  

<br/>

```c++
int add(int x, int y)
{
    return x + y;
}

float add(float x, float y)
{
    return x + y;
}
```

위와 같이 비효율적인 코드에서 template 기능을 사용하면 아래와 같이 변하게 됩니다.  

```c++
template<typename T>
T add(T x, T y)
{
    return x + y;
}

add(1, 2);      // ok : T가 int로 된다.
add(1.1f, 2.2f);// ok : T가 float이 된다.

add(1, 2.2f)    // error : T가 int, float 동시에 될 수 없다.
```

<br/>

위의 코드에서 T는 add() 함수가 호출될 때, 사용되는 자료형으로  

이전 비효율적인 코드에서 int, float 뿐만 아니라 그 외 + 연산자가 허용되는 자료형은  

모두 입력이 가능합니다.  

그러나 3번 째 예제인 add(1, 2.2f)는 동작이 안됩니다.  

add() 함수는 자료형이 **T 만 존재**하므로 입력된 1과 2.2f는 자료형이 다르기 때문입니다.  

만약 서로 다른 자료형을 넣고 싶으면 다음과 같이 수정하면 됩니다.

<br/>

```c++
template<typename T1, typename T2>
T1 add(T1 x, T2 y)
{
    return x + y;
}

add(1, 2.2f);   // ok : 그러나 덧셈 연산을 int자료형에서 사용하므로 반환값도 int
```

<br/>

> template를 사용할 때, class 혹은 typename을 혼용해서 사용하기도 합니다.  
> C++ 가 제작될 당시에는 class로 제작이 되었으나  
> C++ 위원회에서 기존의 class 키워드와 혼동을 막고자 typename을 추가하였습니다.  
> class와 typename은 기능적으로 똑같습니다.  

<br/>

# 클래스 템플릿
<br/>
클래스 템플릿은 함수 템플릿과 거의 같은 원리로 사용할 수 있습니다.  

단, 클래스 인스턴스를 생성할 때, **<> 기호로 자료형을 명시**해야한다는 차이점이 있습니다.  

<br/>

```c++
template<typename T>
class CAL
{
private:
    T x, y;

public:
    CAL(T in_x, T in_y) 
        : x(in_x)
        , y(in_y)
    {

    }

    T add() {
        return x + y;
    }
}

...

// int 자료형 명시
CAL<int> cal_01(1, 2);
tsd::cout << cal_01.add() << std::endl;
// 출력 : 3

// float 자료형 명시
CAL<float> cal_02(1.1f, 2.2f);
tsd::cout << cal_02.add() << std::endl;
// 출력 : 3.3
```

<br/>

# 템플릿 특수화
<br/>
템플릿 특수화는 여러 타입을 받을 수 있도록 설계 했지만  

특정 자료형에서만 다른 방식으로 동작하도록 설계하고 싶을 때 사용하는 것입니다.  

<br/>

```c++
template<typename T>
T add(T x, T y)
{
    return x + y;
}

template<>
char* add(char* x, char* y)
{
    char* buffer = new char[100]; // 필요한 공간만큼 확보

    strcpy(buffer, x);
    strcat(buffer, y);

    return buffer;
}

add(1, 2);  // ok : 위의 1번 add(T x, Ty)가 사용된다.

char a[] = "abc", b[] = "def";
add(a, b);  // ok : 위의 2번 add(char* x, char* y)가 사용된다.
```
