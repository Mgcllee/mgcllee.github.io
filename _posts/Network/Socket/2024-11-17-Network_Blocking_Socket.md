---
title:  "[네트워크] 블로킹 소켓"
excerpt: ""

categories: [Network, Network Socket]
tags: [Network, socket]

toc: true
toc_sticky: true
math: true

date: 2024-11-17
last_modified_at: 2024-11-17
---

> 이 포스트는 ["게임 서버 프로그래밍 교과서"](https://product.kyobobook.co.kr/detail/S000001792817?LINK=NVB&NaPm=ct%3Dm3lamecg%7Cci%3De1bf25f4bf80022ba0f2750e1fc4a02f3a415449%7Ctr%3Dboksl1%7Csn%3D5342564%7Chk%3D8e4035492f1c600c6594d4fb77b82e721ba004ec)를 참고하여 작성된 포스트입니다.  

<br/>

## 블로킹

블로킹(Blocking)이란 스레드가 대기하는 현상을 의미합니다.  
예를들어, 파일을 읽는 함수를 호출하면  
스레드는 처리 요청을 운영체제에 요청하고 응답이 올 때까지 대기합니다.  
이처럼 스레드가 대기하는 현상을 모두 블로킹이라고 합니다.  

블로킹 상태에 있는 스레드를 waitable 상태라고 하며,  
이 상태에서는 스레드가 CPU를 사용하지 않아 CPU의 사용률이 0%로 감소합니다.  

운영체제에 요청한 작업이 완료되어 이에 대한 응답을 받으면  
블로킹 되어있던 스레드는 running 상태로 변환되고 블로킹된 다음의 코드를 실행합니다.  

블로킹은 소켓 통신에서도 사용됩니다.  
소켓 통신 중인 2대의 컴퓨터에서 송신하는 컴퓨터는 없고 수신하는 컴퓨터만 있다면  
수신할 수 있는 데이터가 생길 때까지 waitable 상태가 유지됩니다.  

<br/>

## TCP 소켓 연결 및 송신

TCP는 연결 지향형 프로토콜입니다.  
또한 1 대 1 통신만 허용하기 때문에 TCP 소켓 1개는 오직 1개의 EndPoint만 통신할 수 있습니다.  
아래의 코드는 TCP 소켓을 이용한 통신을 표현한 의사 코드입니다.  

> TCP에 대한 내용은 [다른 포스트](https://mgcllee.github.io/posts/TCP_IP/)에서 설명하겠습니다.  

```c++
s = socket(TCP);
s.bind(any_port);
s.connect("55.66.77.88:5959");
s.send("Hello");
s.close();
```

<br/>

### 1번: s = socket(TCP)

s에 TCP 소켓 핸들을 생성하여 전달합니다.  
s는 소켓이 생성만 되었을 뿐, 아직 통신을 할 수 없습니다.  

### 2번: s.bind(any_port)

bind 함수를 사용해서 현재 컴퓨터의 Port 중 사용 가능한 빈 포트를 사용하도록 설정합니다.  
빈 포트가 없다면, 이미 사용중인 포트를 공유하여 사용할 수 있습니다.  

TCP 통신은 송신자와 수신자 모두의 포트를 알아야 하기 때문에 bind 함수를 통해 송신자의 포트를 설정합니다.  

### 3번: s.connect("55.66.77.88:5959")

connect 함수를 통해 상대방과 연결을 시도합니다.  
이때, 상대방과 연결이 성공할 때 까지 **블로킹**이 발생합니다.  
연결이 성공하면 connect 함수는 리턴합니다.  
그러나 상대방이 존재하지 않을 경우(전원 꺼짐 등) 마찬가지로 함수를 반환합니다.  
이 함수의 반환값을 통해서 네트워크 연결이 성공했는지 여부를 판단할 수 있습니다.  


### 4번: s.send("Hello")

send 함수는 상대방을 향해 전달받은 데이터를 전달합니다.  

함수의 내부에서는 데이터를 운영체제에게 넘기고 운영체제는 이 데이터를 **송신 버퍼**에 전달합니다.  
운영체제는 데이터가 송신 버퍼에 저장되면 send 함수에게 완료됨을 알리고  
send 함수는 리턴합니다.  

여기서 데이터는 송신 버퍼에 담겨 있는 상태이기 때문에  
send 함수가 리턴 되었다고 반드시 상대방이 수신을 완료된 것은 아닙니다.  

### 5번 s.close()

TCP 소켓을 닫습니다.  
이 함수로 TCP 연결이 완전히 해제됩니다.  

<br/>

## 블로킹과 소켓 버퍼

의사 코드의 4번째 함수의 실행을 설명할 때, 상대방이 수신하지 않더라도 send 함수가 반환한다고 했습니다.  
어째서 수신을 하기 전에 함수가 반환되는지 알기 위해서는  
**송신 버퍼와 수신 버퍼**에 대해서 알아야 합니다.  

송신 버퍼는 일련의 바이트 배열(byte array)로 Queue의 작동 방식처럼 FIFO(First In First Out) 형태로 작동합니다.  
아래는 송신 버퍼가 4byte 인 소켓을 표현한 것입니다.  

![블로킹 소켓 통신](/assets/img/Network/BlockingSocket_TCP_Send_01.png)  

![블로킹 소켓 통신](/assets/img/Network/BlockingSocket_TCP_Send_02.png)  

<br/>

위의 과정에서 블로킹이 발생하는 조건은 송신 버퍼에 **자리가 없을 때** 입니다.  

[실제 송신 버퍼](https://mgcllee.github.io/posts/MTU_MSS/)는 이보다 크기 때문에 더 많은 데이터를 담기 때문에  
위 의사코드처럼 작은 데이터("Hello")와 같은 것은 바로 반환됩니다.  

<br/>

## 네트워크 연결 받기 및 수신

지금까지 상대방 컴퓨터에 네트워크 연결을 시도하고 데이터를 송신하는 방법을 알아보았습니다.  
이번에는 데이터를 수신받는 입장에서 네트워크 연결을 수락하고 데이터를 받는 방법에 대해 알아보겠습니다.  

아래의 의사 코드는 네트워크 연결을 수신하는 컴퓨터의 의사 코드입니다.  

```c++
s = socket(TCP);
s.bind(5959);
s.listen();
s2 = s.accept();
while(true) {
    data = s2.recv();
    if(data.length() <= 0) break;
    print(data);
}
s2.close();
```

<br/>

의사 코드의 진행은 아래와 같습니다.  

### 1번: s = socket(TCP)

TCP 소켓을 생성합니다.  

### 2번: s.bind(5959)

TCP 포트 5959번을 점유합니다.  
만약, 5959번 포트가 이미 점유 상태라면 실패할 수 있습니다.  

### 3번: s.listen()

listen() 함수로 이 소켓은 TCP 연결을 받는 역할을 시작합니다.  

### 4번: s2 = s.accept()

누군가 TCP 연결로 들어올 때까지 대기합니다.  
즉, 누군가가 이 프로그램의 주소(55.66.77.88)과 포트번호(5959)로 접속을 시도하면  
accept() 함수는 5959번호가 아닌 **새로운 포트번호로 연결된 새로운 소켓**을 반환합니다.  
따라서 기존 소켓인 s가 아니라 새로운 소켓인 s2를 사용해야 합니다.  

### 6번: data = s2.recv()

연결된 새로운 소켓에서 데이터를 수신합니다.  
만약, 수신 버퍼에서 수신할 데이터가 없다면 수신할 데이터가 발생할 때까지 블로킹됩니다.  

### 7번: if(data.length() <= 0) break

소켓 통신에서 받은 데이터가 0일 경우, 상대방이 TCP 연결을 끝냈음을 의미하고  
음수인 경우는 소켓 연결에 문제가 발생하였음을 알려줍니다.  
따라서 0 이하인 경우 반복문을 종료합니다.  

### 10번 s2.close()

TCP 소켓을 닫습니다.  
이 함수로 TCP 연결이 완전히 해제됩니다.  

<br/>

소켓 통신의 수신 과정은 송신 과정과 매우 유사합니다.  

단, 버퍼를 사용하는 순서에서는 차이가 존재합니다.  
송신 과정에서 송신 버퍼는 사용자가 push() 하고 운영체제가 pop() 을 실행하고  
수신 과정에서 수신 버퍼는 운영체제가 push() 하고 사용자가 pop() 을 실행합니다.  

소켓의 수신 버퍼 안에는 수신되는 데이터가 존재한다면 계속해서 채워 줍니다.  
따라서 수신 버퍼를 방치한다면 버퍼가 꽉 채워져 데이터를 더 이상 받을 수 없는 상태가 됩니다.  
반대로 수신 버퍼가 완전히 비어있으면 수신 함수에서 블로킹이 발생합니다.  

<br/>