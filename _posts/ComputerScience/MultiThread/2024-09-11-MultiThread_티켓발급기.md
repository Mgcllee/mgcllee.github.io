---
title:  "[멀티 쓰레드] Data Race 없는 티켓 발급기"
excerpt: ""

categories: [Computer Science, Multi Thread]
tags: [MultiThread, C&#47;C&#43;&#43;, CS, OS]

toc: true
toc_sticky: true
 
date: 2024-09-11
last_modified_at: 2024-09-11
---

> 이 포스트는 개인 연구를 기록 글로 부정확할 수 있습니다.

## 문제의 코드 

```c++
void PacketWorker::init_new_client_ticket() {
  int new_player_ticket = get_new_client_ticket();

  // ...
}

int PacketWorker::get_new_client_ticket() {
  // TODO: make new client ticket
  return ++user_ticket;
}
```

<br/>

[위 코드](https://github.com/Mgcllee/PokeHunter/blob/f55dfcb26e4dfe95dd4deed97fe2813522de2eed/PokeHunter_Server/MainServer/PacketWorker.cpp#L91)는 멀티 쓰레드 환경에서 실행되는 게임 서버 코드의 일부입니다.  
서버에 새로운 클라이언트가 접속하면 고유 번호를 발급해주는 함수를 만드는 것이 목적이였습니다.  
<br/>

그러나 CPU 에서는 함수의 시작, 복합 연산자 계산 그리고 반환까지 한 번에 실행되지 **않을 수 있습니다.**  
위 코드를 그대로 사용했을 때, 1~3번 클라이언트에게 동시에 티켓을 발급한다면 아래처럼 실행될 수 있습니다.  
이때, 클라이언트 번호와 동일한 번호의 쓰레드가 각 클라이언트를 담당한다고 가정하겠습니다.  
<br/>

![DataRace01](/assets/img/MultiThread/DataRace_01.png)  

<center>[실제 실행과 다른 결과가 나올 수 있습니다.]</center>

<br/>

1. 1, 2, 3번 클라이언트가 동시에 서버로 접속했습니다.  
2. 1번 클라이언트의 티켓 발급을 위해 1번 쓰레드가 get_new_client_ticket() 함수를 호출합니다.  
3. 1번 쓰레드가 지금까지 작업 내용을 스택에 작성하는 동안 2번 쓰레드로 **Context Switch 가 발생합니다.**  

> Context Switch는 운영체제에서 효율적인 동시 작업을 위해 사용하는 기술로  
> 응용 프로그래머는 Context Switch 의 시점을 파악 혹은 조작할 수 없습니다.  

4. 2번 쓰레드는 user_ticket 의 값을 읽던 중 **Context Switch 가 발생합니다.**  
5. 3번 쓰레드가 user_ticket 복합 연산(++) 결과를 반환 받던 중 **Context Switch 가 발생합니다.**  
6. 2번 쓰레드가 실행되어 **이전에 읽어뒀던 user_ticket** 값에 ++ 연산을 실행합니다.  
7. 1번 쓰레드가 다시 실행되면서 2번 혹은 3번 쓰레드에 의해 증가된 user_ticket에 ++ 연산을 실행합니다.  

<br/>

이 순서도는 Data Race가 발생하는 최악의 경우들 중 하나입니다.  
동시에 접속하는 모든 클라이언트가 동일한 티켓을 받거나 제대로된 값을 받을 수도 있습니다.  
<br/>

## DataRace 일으켜보기

```c++
// volatile 은 Visual C++ 에서 최적화를 막습니다.
volatile int SUM = 0;

void Worker() {
  for (int i = 0; i < 1'000'000; ++i) {
    ++SUM;
  }
}
```
<br/>

Worker 함수를 4개의 쓰레드로 실행하면 SUM 의 결과가 4'000'000 이라는 값이 나와야 합니다.  
그러나 실제 결과는 4백만에 근접하지도 않고 실행할 때 마다 다른 결과를 출력합니다.  
<br/>

![DataRace_02](/assets/img/MultiThread/DataRace_02.png)  

<br/>

## atomic 사용하기

이번에는 C++ 의 atomic 을 사용해 thread safe 하게 값을 연산하도록 코드를 수정해 보았습니다.  
<br/>

```c++
volatile atomic<int> SUM = 0;

void Worker() {
  for (int i = 0; i < 1'000'000; ++i) {
    SUM.fetch_add(1);
  }
}
```

<br/>

![DataRace_02](/assets/img/MultiThread/DataRace_02.png)  

<br/>

드디어 SUM 결과가 정상적으로 출력되었습니다.  
그러나 정확성은 올라갔지만 **연산 시간이 급증** 하게 되었습니다.  
한 쓰레드가 atomic 변수에 접근하게 되면 다른 쓰레드는 접근할 수 없기 때문에  
쓰레드의 대기 시간이 길어져 전체 연산 시간이 증가했기 때문입니다.  
<br/>

그리고 처음 언급한 문제의 코드에서는 SUM 의 결과값도 가져와야 하므로  

```c++
SUM.fetch_add(1);
int ticket = SUM.load();
```

<br/>

## mutex 사용하기

이처럼 수정해도 .fetch_add 와 .load 메서드 사이 **Context Switch 가 발생하지 않을 거라는 보장이 없습니다.**  
따라서 atomic 변수 대신에 mutex 를 사용하여 아래처럼 수정하게 되었습니다.  

```c++
volatile int SUM = 0;

mutex ll;

void Worker() {
  for (int i = 0; i < 1'000'000; ++i) {
    ll.lock();
    ++SUM;
    int ticket = SUM;
    ll.unlock();
  }
}
```

<br/>

이 코드는 정상적인 결과가 출력되지만 mutex 를 사용했기 때문에  
atomic 변수를 사용했을 때처럼 실행 시간이 길어진다는 단점이 있습니다.  
<br/>
