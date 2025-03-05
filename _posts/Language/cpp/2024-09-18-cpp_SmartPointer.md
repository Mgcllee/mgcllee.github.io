---
title:  "[C++] 스마트 포인터"
excerpt: ""

categories: [Language, C&#47;C&#43;&#43;]
tags: [C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-09-18
last_modified_at: 2024-09-18
---

> 이 포스트는 개인적으로 C++ 스마트 포인터를 공부하며 기록한 내용으로 이후 변경될 수 있습니다.  

## 메모리 낭비하기

C++ 는 new 연산자로 메모리에 동적으로 할당하고  
사용이 끝나면 delete 연산자로 사용중인 메모리를 해제합니다.  
<br/>
그러나 코드가 복잡해지면 delete 를 너무 일찍 사용해서 이미 해제된 메모리에 접근하거나,  
delete 자체를 잊어버려 메모리가 해제되지 않고 프로그램이 종료될 때 까지 할당된 상태인 일이 일어날 수 있습니다.  
<br/>
할당된 메모리가 해제되지 않을 경우, 메모리 전체에서 할당 가능한 용량이 작아져서  
현재 실행 중인 프로그램 실행에 영향을 주거나 최악의 경우, 다른 프로그램까지 영향을 받을 수 있습니다.  
이처럼 메모리가 낭비되어 사용되지 못하는 것을 **메모리 릭(Memory Leak)** 이라고 합니다.  
<br/>

```c++
/*
    Visual Studio C++ 에서 메모리 릭 확인하기
*/
#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>

#include <iostream>

int main() {
	_CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);
	
	int* p1 = new int{};
	int* p2 = new int[3] {};
	
	delete p1;
	// p2 를 해제하지 않음
	return 0;
}
```

![Leak](/assets/img/Cpp/메모리_릭_확인.png)  
<center>[출력창에서 메모리 릭 발견 결과 출력]</center>

<br/>

자바에서는 메모리 릭을 방지하기 위해서 가비지 컬렉터(Garbage Collector) 라는 것을 사용합니다.  
동적 할당한 메모리를 직접 해제할 필요 없이 사용이 끝난 메모리는 가비지 컬렉터가 자동으로 메모리를 해제합니다.  
<br/>
그러나 시스템에서 할당된 메모리를 사용하고 있는지 확인하기 때문에 많은 자원을 사용한다는 단점이 있습니다.  
그래서 C++ 에서는 가비지 컬렉터 대신에 **스마트 포인터** 를 사용해 메모리 릭을 방지합니다.  
<br/>

## 스마트 포인터

스마트 포인터란 사용이 끝난 메모리를 자동으로 해제 해주는 클래스 템플릿입니다.  
스마트 포인터의 특징은 아래와 같습니다.  

* C++11 에서 채택되어 memory 헤더에 포함  
* 메모리 릭에 대한 안정성 보장  
* 힙 메모리에 생성됩니다.  

<br/>

스마트 포인터의 종류는 다음 3가지가 있습니다.  

* shared_ptr : 참조 횟수가 기록되는 스마트 포인터  
* unique_ptr : 하나의 소유자만 허용되는 스마트 포인터  
* weak_ptr : shared_ptr 과 함께 사용될 수 있는 특별한 포인터  
<br/>

## shared_ptr

C++ 에서 **참조 카운팅** 방식의 스마트 포인터로  
shared_ptr 을 참조 횟수가 0이 되면 메모리 할당을 해제하는 원리로 동작합니다.  
shared_ptr 은 기존 포인터 변수처럼 new, *, -> 등의 연산자를 사용하면서  
가르키는 대상을 **자동으로 소멸** 시켜준다는 점이 기존 포인터와의 차이점입니다.  
<br/>

shared_ptr 은 포인터이기 때문에 참조 객체를 가르키는 포인터를 갖고 있지만 위에서 언급한 참조 카운팅을 위한  
동적 변수 포인터도 갖고 있어야 하기 때문에 2개의 포인터를 사용합니다.  
2개의 포인터는 서로 다른 위치에 저장될 수 있어 메모리 호출을 2번해야 할 수 있습니다.  
<br/>
이를 방지하기 위해 C++ 에서는 두 포인터를 단일 메모리 블록에 생성할 수 있도록  
**make_shared** 를 사용해서 shared_ptr 을 생성할 수 있도록 하였습니다.  
<br/>

```c++
#include <iostream>
#include <memory>
using namespace std;
int main() {
	
	shared_ptr<int> p1(new int(10));
	shared_ptr<int> p2 = make_shared<int>(10);

	cout << *p1 << ", " << p2.get();
	return 0;
}
```

<br/>

## unique_ptr

unique_ptr 은 이중 참조를 허용하지 않고 하나의 포인터 변수만을 허용하는 스마트 포인터 입니다.  
unique_ptr로 할 수 있는 작업은 shared_ptr로 할 수 있지만 특수한 목적을 위해 사용됩니다.   
<br/>

첫 번째는 unique_ptr 포인터가 shared_ptr 포인터에 비해 경량이므로 성능이 좋습니다.  
두 번째는 함수 내에서 지역 변수로 포인터 변수가 필요할 때 unique_ptr을 사용하면  
함수 종료 시 해당 객체도 소멸되기 때문입니다.  
이 방법은 메모리 릭이 없는 안전한 프로그래밍을 할 수 있다는 장점이 있습니다.  
<br/>

unique_ptr도 shared_ptr과 마찬가지로 new 로 선언을 할 수도 있지만 make_unique를 사용하는 것이 권장됩니다.  
<br/>

```c++
auto p1 = std::make_unique<int>(10);
```

사용법은 shared_ptr과 동일하므로 생략하겠습니다.​
<br/>

## 참고

[[MSLearn]CRT 라이브러리로 메모리 누수 찾기](https://learn.microsoft.com/ko-kr/cpp/c-runtime-library/find-memory-leaks-using-the-crt-library?view=msvc-170)  
[JK 농원 (조경수) & SW 엔지니어](https://m.blog.naver.com/dorergiverny/223088954671)  
