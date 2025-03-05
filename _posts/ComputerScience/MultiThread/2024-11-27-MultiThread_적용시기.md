---
title:  "[멀티 쓰레드] 멀티 쓰레드 프로그래밍은 언제 해야 할까?"
excerpt: ""

categories: [Computer Science, Multi Thread]
tags: [MultiThread, C&#47;C&#43;&#43;, CS, OS]

toc: true
toc_sticky: true
 
date: 2024-11-27
last_modified_at: 2024-11-27
---

> 이 포스트는 개인 학습을 기록한 내용을 담고 있어 추후 수정될 수 있습니다.  
> 이 포스트는 ["게임 서버 프로그래밍 교과서"](https://product.kyobobook.co.kr/detail/S000001792817?LINK=NVB&NaPm=ct%3Dm3lamecg%7Cci%3De1bf25f4bf80022ba0f2750e1fc4a02f3a415449%7Ctr%3Dboksl1%7Csn%3D5342564%7Chk%3D8e4035492f1c600c6594d4fb77b82e721ba004ec)를 참고하여 작성된 포스트입니다.  

멀티 쓰레드는 여러 가지 일을 한 번에 처리할 수 있다는 엄청난 장점이 있지만  
쓰레드를 생성하고 일반 싱글 쓰레드처럼 사용하면 최악의 경우,  
**싱글 쓰레드 프로그램보다 못한 결과**가 나올 수 있습니다.  

따라서 멀티 쓰레드를 적용하기 전 "왜 멀티 쓰레드가 필요한가?" 에 대한 대답이 분명해야 합니다.  

멀티 쓰레드를 사용하는 대표적인 상황은 아래와 같습니다.  

1. **오래 걸리는 일 하나**와 **빨리 끝나는 일 여럿**을 함께 처리해야 할 때  
2. 어떤 **긴 처리**를 진행하는 동안 **다른 짧은 일**을 처리해야 할 때  
3. 기기에 있는 CPU를 100% 사용해야 할 때  

<br/>

## 오래 걸리는 일 하나와 빨리 끝나는 일 여럿을 함께 처리

이러한 경우의 예시로 게임 클라이언트에서 로딩 화면의 모습이 있습니다.  

![오래걸리는일짧은일여럿](/assets/img/MultiThread/오래걸리는일짧은일여럿.png){: width="400", height="400"}  

클라이언트 프로그램을 최초로 실행하고 게임에 입장하기 전, 여러 캐릭터와 배경을 구성하는 리소스 정보를  
가져와야 하기 때문에 보조 메모리에서 많은 양의 데이터를 읽어야 합니다.  
클라이언트는 쉴새없이 일하는 중이지만 게임 유저 입장에서는 클라이언트가 일시정지한 것처럼  
화면이 멈춰있을 수 있습니다.  

따라서 유저에게 클라이언트가 무엇인가를 작업하고 있다는 것을 알려주기 위해서 로딩화면을 띄워줍니다.  

<br/>

만약 멀티 쓰레드가 아니라 싱글 쓰레드로 리소스 읽기와 로딩화면 갱신을 할 경우 코드는 아래와 같습니다.  

```c++
void load_scene() {
    render_scene();
    load_texture();
    render_scene();
    load_animation();
    render_scene();
    load_sound();
    render_scene();
}
```

모든 리소스의 크기가 작으면 이와 같은 함수가 의도대로 동작할 수 있지만  
리소스의 크기가 매우 크다면 render_scene() 함수가 매끄럽게 동작하지 않을 수 있습니다.  
추가적으로 load_scene() 함수가 출력과 리소스 읽기 2가지 일을 한 번에 처리하고 있어  
코드의 가독성이 떨어진다는 단점이 있습니다.  

이 문제를 해결하는 방법 중 하나가 바로 멀티 쓰레드를 이용한 실행 분리 입니다.  

```c++
bool is_still_loading = true;

void render_loading_scene() {
    while(is_still_loading) {
        render_scene();
    }
}

void load_resource() {
    is_still_loading = true;

    load_texture();
    load_animation();
    load_sound();

    is_still_loading = false;
}
```

위 두 함수를 두 개의 쓰레드로 실행하면 화면을 지속적으로 갱신하면서 리소스를 가져올 수 있습니다.  
따라서 멀티 쓰레드를 이용하면 리소스 로딩이 진행되는 동안 부드러운 화면 렌더링이 가능합니다.  

<br/>

## 긴 처리 진행중 다른 짧은 일 처리

게임 서버 개발에서도 멀티 쓰레드가 필요한 경우가 있습니다.  
유저의 로그아웃으로 유저 정보를 디스크에 저장해야하는 등 처리가 오래 걸리는 작업이 있을 때 입니다.  

게임 서버가 유저의 정보를 저장하는데 소요되는 시간이 1만분의 1초(1/10'000 초)라고 한다면  
유저 한 명의 정보를 저장하는데 1만분의 1초가 걸린다는 의미입니다.  
게임 서버는 동시에 여러 클라이언트의 접속을 허용하므로 1만개의 클라이언트가 서버에 접속해 있다면  
서버에 접속한 모든 유저의 정보를 저장하는데 1초가 걸립니다.  

또한 게임 서버는 유저 정보 저장뿐만 아니라 물리 충돌과 같은 복잡한 연산 등  
여러가지 일을 동시에 진행하기 때문에 1만분의 1초를 매번 기다릴 여유가 없습니다.  

따라서 이 문제를 해결하기 위해  
게임 서버에서는 정보를 저장하면서 다른 작업도 함께 처리할 수 있어야합니다.  

<br/>

## 기기 CPU를 모두 활용

근래의 CPU들은 단일 코어 제품이 아닌 여러 개의 코어를 가지고 있습니다.  

![CPU사양예시](/assets/img/MultiThread/CPU사양예시.png){: width="450", height="450"}  

> 코어와 쓰레드의 개수가 다른 이유는 한 코어에서 2개의 쓰레드를 담당하기 때문입니다.  

<br/>

그러나 멀티 쓰레드가 아닌 싱글 쓰레드를 사용할 경우, CPU의 성능 중 아주 일부분만 사용하는 것입니다.  

만약, 1부터 1억까지의 정수 중 소수를 판별하고 출력하는 프로그램을  
싱글 쓰레드로 만든다면 아래의 의사 코드와 같습니다.  

```c++
#define MAX 100'000'000

vector<int> prime_number;

for(int n = 1; n <= MAX; ++n) {
    if(is_prime(n)) {
        prime_number.push_back(n);
    }
}

for(int pn : prime_number) {
    cout << pn << endl;
}
```

이 코드를 위의 사진에 있는 CPU처럼 4개의 코어를 갖는 CPU에서  
실행한다면 나머지 3개의 코어는 아무 일을 하지 않는 상태가 됩니다.  
실행 시간이 목표하는 시간 내 끝난다면 상관없지만  
게임 서버와 같은 빠른 응답이 필요한 프로그램에서는 엄청난 손해입니다.  

<br/>

4개의 코어로 구성된 CPU의 성능을 100% 사용하기 위해서는 아래와 같이 수정해야 합니다.  

```c++
#define MAX 100'000'000

mutex g_mutex;
vector<int> prime_number;

void thread_func(int start, int end) {
    for(int n = start; n <= end; ++n) {
        if(is_prime(n)) {
            const lock_guard<mutex> lock(g_mutex);
            prime_number.push_back(n);
        }
    }
}

thread t1(thread_func, 1, MAX / 4);
thread t2(thread_func, MAX / 4 + 1, MAX / 2);
thread t3(thread_func, MAX / 2 + 1, MAX * (3.f/4.f));
thread t4(thread_func, MAX * (3.f/4.f), MAX);

t1.join();
t2.join();
t3.join();
t4.join();

for(int pn : prime_number) {
    cout << pn << endl;
}
```
