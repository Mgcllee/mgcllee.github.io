---
title:  "[C++] static_cast 연산자"
excerpt: ""

categories: [Language, C&#47;C&#43;&#43;]
tags: [C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-10-01
last_modified_at: 2024-10-01
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

이번 포스트에서는 static_cast 연산자에 대해서 알아보겠습니다.  

<br/>

# static_cast
---

```c++
static_cast<데이터 타입>(대상);
```

static_cast 는 대상이 주어진 데이터 타입으로 변환 가능할 때, 변환을 실행합니다.  
int 에서 float 형으로, enum 에서 int 형 등 변환을 시킬 수 있습니다.  

기본 자료형 외에도 배열을 포인터로 변환하거나 반대로 포인터를 배열로 변환할 수 있습니다.  

그러나 포인터 타입(int*, float*, ... etc)의 변환은 컴파일러에서 허용하지 않습니다.  
예외적으로 상속 관계의 클래스 포인터는 가능합니다.  
이 예외를 알아보기 전 먼저 **업캐스팅** 과 **다운캐스팅** 에 대해 알아보겠습니다.  

```c++
class Car {}
class Bus : public Car {}

int main()
{
    Bus bus;

    // [업 캐스팅]: 파생(자식) 클래스의 객체를 기본(부모) 클래스의 객체처럼
    Car* pcar = &bus; 

    Car car;

    // [다운캐스팅]: 업 캐스팅의 반대
    Bus* pbus = (Bus*)&car;
}
```

파생 클래스의 포인터를 기본 클래스의 포인터로 변환하는 것을 업 캐스팅  
기본 클래스의 포인터에서 파생 클래스의 포인터로 변환하는 것을 다운 캐스팅이라고 합니다.  

**static_cast 는 업 캐스팅, 다운 캐스팅** 을 사용할 수 있습니다.  
그러나 다운 캐스팅은 안전하게 동작하지 않을 수 있기 때문에 만약 안전하게 동작해야 한다면  
dynamic_cast 라는 연산자를 사용하는 것이 좋습니다.  

<br/>

# 예시 코드

```c++
// 정수 -> 실수 변환(역변환 가능)
int i_num = 10;
float f_num = 3.14f;

int float_to_int_num = static_cast<int>(f_num);
float int_to_float_num = static_cast<float>(i_num);


// 배열 -> 포인터 변환(역변환 가능)
int arr[5]{1, 2, 3, 4, 5};
int* p_arr;
p_arr = static_cast<int*>(arr);

for(int i = 0; i < 5; ++i) {
    cout << p_arr[i] << ", ";
}


// 열거형 -> 정수 변환(역변환 가능)
enum STATE {
    DIRTY = 10,
    CLEAN,
    EMPTY
}

STATE curr_state = DIRTY;
int new_state = static_cast<int>(curr_state);

// 위 코드의 Car, Bus 클래스를 그대로 사용합니다.
// [업 캐스팅]: 파생(자식) 클래스의 객체를 기본(부모) 클래스의 객체처럼
Car* pcar = &bus; 

// [안전하지 않음, 다운캐스팅]: 업 캐스팅의 반대
Bus* pbus = (Bus*)&car;
// Bus 클래스의 메서드 혹은 필드에 접근할 경우, 유효한 값이 아닐 수 있음.
```

<br/>

# 결론

static_cast 는 기존 C 언어에서 명시적 타입 변환을 대신해 사용할 수 있습니다.  
그리고 C 언어의 타입 변환 기능에 더해 컴파일 타임에서 변환 가능 여부를 확인하기 때문에  
변환을 안전하게 실행할 수 있습니다.  
또한 static_cast 라고 코드에 작성하기 때문에 이전보다 가독성이 올라간다는 장점도 있습니다.  

마지막으로 업 캐스팅과 다운 캐스팅을 사용할 수 있지만  
다운 캐스팅의 경우, 부모 클래스 인스턴스가 자식 클래스로 변환되는 것이기 때문에  
변환된 인스턴스에서 메서드에 접근할 경우, 유효하지 않은 결과를 얻을 수 있습니다.  
<br/>
