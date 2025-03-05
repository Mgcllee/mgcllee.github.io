---
title:  "[네트워크] EPoll의 개념"
excerpt: ""

categories: [Network]
tags: [Network, socket]

toc: true
toc_sticky: true
math: true

date: 2024-11-25
last_modified_at: 2024-11-25
---

> 이 포스트는 ["게임 서버 프로그래밍 교과서"](https://product.kyobobook.co.kr/detail/S000001792817?LINK=NVB&NaPm=ct%3Dm3lamecg%7Cci%3De1bf25f4bf80022ba0f2750e1fc4a02f3a415449%7Ctr%3Dboksl1%7Csn%3D5342564%7Chk%3D8e4035492f1c600c6594d4fb77b82e721ba004ec)를 참고하여 작성된 포스트입니다.  

<br/>

[이전 포스트](https://mgcllee.github.io/posts/Network_NonBlocking_Socket/)에서 Poll() 함수를 언급을 했었습니다.  
포스트의 마지막에 설명한 Poll() 함수의 단점으로 반복문을 사용한 소켓 전체 순회가 있었습니다.  
이 전체 순회로 인해 실시간 처리가 중요한 게임 서버에서는 적합하지 않을 수 있습니다.  

따라서 처리 속도가 중요한 게임 서버에서는 전체 순회 대신  
**I/O 가능 상태인 소켓만 감지하여 사용자에게 알려주는 EPoll() 함수**가 더 적합합니다.  
이번 포스트에서는 EPoll() 함수에 대해서 알아보겠습니다.  

<br/>

## EPoll

EPoll() 함수는 여러 소켓 중 I/O 가능이 되는 순간의 소켓을 **내장된 큐에 푸시(push)**합니다.  
사용자는 EPoll() 에서 이러한 이벤트 정보를 팝(pop)하여 정보를 처리할 수 있습니다.  
이러한 과정 덕분에 소켓이 1만개 이상이라고 하더라도 그중 I/O 가능이 된 소켓만 바로 얻을 수 있습니다.  
즉, Select(), Poll() 함수와 다르게 반복문을 1만 번 순회하지 않아도 됩니다.  

> EPoll은 리눅스와 안드로이드 계얼에서만 사용이 가능합니다.  

<br/>

```c++
epoll = new epoll(); // (1)

for(auto socket : sockets) {
    epoll.add(socket, GetUserPtr(socket)); // (2)
}

events = epoll.wait(100ms); // (3)

for(auto event : events) { // (4)
    socket = event.socket;
    userPtr = event.userPtr;
    
    type = event.type;
    if(type == receive_event) {
        result = socket.recv();
        
        if(result > 0) {
            Process(userPtr, socket, result); // (5)
        }
    }
}
```

이 의사 코드는 아래와 같은 순서로 진행됩니다.  

1. epoll 객체를 생성합니다.  
2. I/O 상태 감지 대상인 소켓들을 epoll 객체에 전달합니다.  
   이때, 소켓뿐만 아니라 사용자 객체 등 다른 값을 함께 전달할 수 있습니다.  
3. epoll 에서 이벤트를 꺼내오는 함수(wait)를 실행합니다.  
   이 함수는 사용자가 원하는 시간까지만 블로킹되며, 그 전에 이벤트가 생기면 즉시 반환됩니다.  
4. 각 이벤트에 대해 반복문을 돌며 이벤트에 저장된 정보를 꺼내 옵니다.  
5. 이벤트에서 꺼낸 정보를 토대로 원하는 처리를 수행합니다.  

<br/>

EPoll 대신 Select를 사용했다면 전달 받은 소켓 전부를 순회하면서 확인해야 합니다.  
하지만 EPoll을 사용하면 I/O 가능인 상태의 소켓에 대해서만 반복문을 실행하면 됩니다.  

그러나 실제 실행 환경에서는 소켓 버퍼로 인한 송수신 중단이 거의 일어나지 않기 때문에  
**대부분의 소켓이 송수신 가능 상태**입니다.  
따라서 필요 이상의 반복문을 수행해야 하기 때문에 **불필요한 CPU 연산**이 발생됩니다.  

이 문제를 개선하기 위해서 사용되는 것이 엣지 트리거(Edge Trigger)입니다.  

<br/>

## 레벨 트리거(Level Trigger)와 엣지 트리거(Edge Trigger)

레벨 트리거와 엣지 트리거는 전자공학에서 나온 용어로 추정됩니다.  
전기 회로에서 레벨(Level)은 전압이 들어와 있는 상태이고 엣지(Edge)는 전압에 **변화**가 있음을 의미합니다.  

![트리거](/assets/img/Network/LevelTrigger_EdgeTrigger.png){: width="600", height="600"}

<br/>

EPoll() 에서 의미하는 레벨 트리거는 소켓이 I/O 가능한 상태를 의미합니다.  
엣지 트리거는 소켓이 I/O 가능이 아니였는데, **이제 I/O 가능**이 되었다는 의미입니다.  

즉, 레벨 트리거는 I/O 가능 상태(값이 1이라면)라면 항상 epoll에서 꺼내지만  
엣지 트리거는 I/O 가능이 된 순간(0에서 1로 변한 순간)에만 epoll에서 꺼냅니다.  
엣지 트리거를 사용하면 불필요한 CPU 연산을 줄일 수 있지만 사용할 때 주의해야할 점도 있습니다.  

UDP 소켓의 수신 버퍼에 데이터그램 2개가 저장되어 있을 때,  
recv() 함수로 데이터그램 1개를 꺼내면 수신 버퍼에는 아직 1개의 데이터그램이 남아있는 상태입니다.  
그러나 epoll에서 엣지 트리거만 인식한다면 데이터그램을 꺼낸 뒤 I/O 상태 변화가 없으므로  
epoll은 데이터그램이 남아 있어도 알려주지 않기 때문에  
결국 남은 데이터그램은 영원히 꺼낼 수 없습니다.  

따라서 엣지 트리거를 사용하기 위해서는 아래의 과정을 반드시 실행해야 합니다.  

1. 소켓은 논블로킹으로 설정된 상태  
2. I/O 호출을 1회만 실행하는 것이 아니라 **'would block'**이 발생할 때까지 반복  

<br/>

epoll 은 connect()와 accept()의 I/O 가능 이벤트도 받을 수 있습니다.  
connect() 함수는 send 이벤트와 동일하게, 그리고 accept() 는 recv 이벤트와 동이랗게 취급됩니다.  

예를들어, 리스닝(listening) 소켓에서 recv 이벤트가 발생할 경우, accept() 함수를 호출하면  
새로운 연결에 대한 소켓을 받을 수 있습니다.  

```c++
for(auto event : events) {
    socket = event.socket;
    userPtr = event.userPtr;
    
    type = event.type;
    if(type == receive_event) {
        
        if(socket == listen_socket) {// 현재 소켓이 리스닝 소켓이라면
            new_socket = socket.accept();
        } 
        else { // 일반 소켓의 수신
            result = socket.recv();
            // ...
        }
    }
}
```