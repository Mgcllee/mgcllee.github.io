---
title:  "[MultiCore] C++ std::async 사용해보기"
excerpt: ""

categories:
  - MultiCore
tags:
  - [MultiCore, C/C++]

toc: true
toc_sticky: true
 
date: 2024-08-04
last_modified_at: 2024-08-04
---

## async 소개

C++에서 스레드를 사용한 작업은 직접 스레드를 생성하고 종료시켰습니다.  

하지만 std::async 에 어떤 함수를 전달한다면, 이전 스레드 관리 방법과 다르게  

알아서 스레드를 만들어 Thread Pool 로 객체를 관리하고  

해당 함수를 비동기적으로 실행한 뒤 결과값을 future 에 전달합니다.  

## async 사용법

```c++
void for_print(std::string in)
{
	for (int i = 0; i < 4; ++i)
	{
		printf("[%s] : %d\n", in.c_str(), i);
	}
}

std::future<void> A = std::async(std::launch::async, for_print, "Func A");
std::future<void> B = std::async(std::launch::deferred, for_print, "Func B");
```

async 는 **실행할 함수**를 필수 사항으로 입력받고  

선택 사항으로 함수의 실행 방법과 매개 변수를 전달 받습니다.  

> std::future<T> std::async(**실행 방법, 실행 함수, 매개 변수**)  

### 실행 방법
1. **std::launch::async**: 바로 쓰레드를 생성해서 인자로 전달된 함수를 실행합니다.  
2. **std::launch::deferred**: future 의 .get() 함수가 호출되었을 때 실행합니다. (새로운 쓰레드를 생성하지 않습니다.)

### 실행 함수
1. async로 실행할 함수의 이름을 전달합니다.  

### 매개 변수
1. **실행 함수가 받을** 매개 변수들을 전달합니다.

### 반환 객체 std::future<T>
1. **(반환 객체).wait()**: 결과를 받을 때 까지 대기합니다.  
2. **(반환 객체).get()**: 결과를 받습니다.  

> std::future<T>().get() 은 객체를 **이동**시켜 반환하므로 .get() 을 중복실해하면 오류가 발생합니다.  

<br/>

## 오류 검출

```c++
auto D = std::async(std::launch::async, for_print, "Func D");
std::future<int> E;

try {
	printf("User Get Function Start!\n");
	D.get();
	E.get();
}
catch (const std::future_error& e)
{
	std::cout << "Caught a future error with code \"" << e.code()
		<< "\"\nMessage: \"" << e.what() << "\"\n";
}
```

async 객체를 실행하는 중 오류가 발생했을 때 future_error 객체를 반환합니다.  

future_error 객체의 .code() 메소드와 .what() 메소드를 통해 오류 번호와 원인을 알 수 있습니다.  

<br/>

## 일정 시간마다 상태 확인하기

```c++
bool check_prime(int in)
{
	for (int i = 2; i < in; ++i)
		if (in % i == 0)
			return false;
	return true;
}

// ...

int num = 444444443;
std::future<bool> F = std::async(check_prime, num);

std::chrono::milliseconds du_time(100);
while (F.wait_for(du_time) != std::future_status::ready)
{
	printf(".");
}
```

비동기로 실행되는 객체인 만큼 일정한 시간마다 객체의 상태를 확인할 수 있습니다.  

.wait_for(std::chrono) 메소드를 사용하면 전달받은 std::chrono 시간마다 객체의 현재 상태를 반환합니다.  

위 코드는 F 객체가 완료될 때 까지 "printf(".")"를 실행하는 코드입니다.  

<br/>
