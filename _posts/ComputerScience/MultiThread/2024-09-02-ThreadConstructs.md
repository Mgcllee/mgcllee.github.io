---
title:  "[멀티스레드] C++에서 Thread 생성하기"
excerpt: ""

categories: [CS, MultiThread]
tags: [MultiThread, C&#47;C&#43;&#43;, CS, OS]

toc: true
toc_sticky: true
 
date: 2024-09-02
last_modified_at: 2024-09-02
---

[Clean Code](https://mgcllee.github.io/categories/cleancode/)를 읽으면서 작성 했던 프로젝트를 수정하던 중  
스레드가 전역 함수를 받고 함수 내부에서 전역 변수에 접근하는 것을 보고서  
전역으로 사용할 필요가 없는 변수와 함수를 클래스에 담아 스레드에서 실행할 수 있도록 수정하고자 하였습니다.  
<br/>

스레드가 클래스를 담도록 스레드의 생성자를 확인하던 중  
생성자가 실행할 전역 함수만 전달받는 것이 아니라 여러 생성자가 있다는 것알 알게 되었습니다.  
따라서 스레드 생성자들과 사용방법에 대해 정리하고자 합니다.  
<br/>

## 스레드 생성자들

스레드의 생성자는 총 6가지로 다음과 같습니다.  

|형식|설명|
|---|---|
|std::thread t1 | t1 is not a thread |
|std::thread t2(function_1, n + 1) | pass by value|
|std::thread t3(function_2, std::ref(n)) | pass by reference|
|std::thread t4(std::move(t3)) | t4 is now running f2(). t3 is no longer a thread|
|std::thread t5(&foo::method, &foo_class_instance) | t5 runs foo::bar() on object f|
|std::thread t6(b_class_instance) | t6 runs baz::operator() on a copy of object b|

[[출처]cppreference.com](https://en.cppreference.com/w/cpp/thread/thread/thread)  
<br/>

스레드는 t1 처럼 생성자에 아무것도 전달하지 않는 경우를 제외하고는  
생성되자마자 전달 받은 함수를 실행합니다. 위의 생성자를 차례대로 살펴보겠습니다.  
<br/>

### 선언만 하기

> std::thread t

t1 ㄴ은 생성자에게 무엇을 실행할지 알려주지 않아 스레드가 실행되지 않습니다.  
<br/>
<br/>

### 매개 변수가 있는 함수 실행하기

> std::thread t2(function_1, n + 1)

t2 는 function_1 을 실행하고 매개변수로 n + 1 을 전달합니다.  
<br/>
<br/>

### 매개 변수가 참조자인 함수 실행하기

> std::thread t3(function_2, std::ref(n))

t3 는 참조자(Reference)를 매개변수로 받는 function_2 를 실행합니다.  
<br/>
<br/>

### 다른 스레드를 스레드에게 넘겨주기

> std::thread t4(std::move(t3))

t4 는 move 함수로 스레드를 전달받아 이어서 실행합니다.  
이때, 실행을 하고 있던 t3 는 더 이상 실행되지 않습니다.  
<br/>
<br/>

### 클래스 메서드 실행하기

> std::thread t5(&foo::method, &foo_class_instance) 

t5 의 경우, foo 클래스의 메서드와 인스터스를 전달합니다.  
메서드 내부에서 foo 클래스의 필드 혹은 다른 메서드 호출도 가능합니다.  
만약, 메서드에 매개 변수가 있을 경우, 3번째 위치(foo_class_instance 뒤)부터 입력해 전달할 수 있습니다.  
<br/>
<br/>

### 클래스 연산자를 이용해 실행하기

> std::thread t6(b_class_instance)

t6 의 경우, b 클래스의 인스턴스만 전달하지만  
실행되는 함수는 operator() 메서드이므로 b 클래스에서 operator() 메서드를 선언해줘야 합니다.  
<br/>

## 수정된 코드

여러 생성자 중 클래스의 메서드와 클래스 인스턴스, 매개 변수를 전달하는 방법으로  
코드를 다음과 같이 수정하였습니다.  
<br/>

```c++
PlayerManager* player_manager = new PlayerManager();
PartiesManager* parties_manager = new PartiesManager(player_manager);

for (int i = 0; i < num_threads; ++i) {
  worker_threads.emplace_back(
      &PacketWorker::worker_thread,
      new PacketWorker(player_manager, parties_manager), h_iocp);
}

for (auto& each_thread : worker_threads) {
  each_thread.join();
}
```

[원본 코드](https://github.com/Mgcllee/PokeHunter/blob/33102b4255965654129af61a1015601e5772fe60/PokeHunter_Server/IOCPServer/PacketWorker.cpp#L197)