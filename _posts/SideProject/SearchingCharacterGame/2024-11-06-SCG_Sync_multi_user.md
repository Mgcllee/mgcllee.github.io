---
title:  "[SCG 프로젝트] 클라이언트 간 동기화 시키기(기초)"
excerpt: ""

categories: [Side Project, SearchingCharacterGame]
tags: [SideProject, C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-11-06
last_modified_at: 2024-11-06
--- 

> 이 포스트는 개인 프로젝트의 기록입니다.  

이번 개발 내용은 멀티 스레드에서 예상하지 못한 문제가 있었습니다.  
대표적으로 아래의 코드가 그와 같습니다.  

```c#
for (int i = 0; i < 2; i++)
{
    Socket new_client_socket = null;
    while (true)
    {
        new_client_socket = server.Accept();
        if (new_client_socket.Connected)
        {
            lock (lockObject)
            {
                clients_socket.Add(new_client_socket);
            }
            break;
        }
    }
    
    clients.Add(new Thread(() => client_recver(i)));
    clients[i].Start();
}
```

![결과](/assets/img/side_project_img/scg_server_log_01.png){: width="600", height="500"}  

위 코드를 실행하면서 발생한 문제점은 지역변수 i 를 스레드에 넘겨줄 때, 발생했습니다.  
clients.Add()함수를 실행하기 전, i의 값은 0이였지만  
넘겨받은 client_recver함수 안에서는 i의 값이 1로 나오는 문제가 있었습니다.  

여러 가능성을 확인하던 중 스레드 스케쥴링에 의해 for 반복문 속 ++i 연산이 먼저 실행되어  
다른 값이 나왔을 수 있음을 생각했습니다.  

따라서 이 문제를 해결하고자 지역변수 user_ticket을 선언하여 아래와 같이 수정하였습니다.  

```c#
for (int i = 0; i < 2; i++)
{
    int user_ticket = i;
    Socket new_client_socket = null;
 
    while (true)
    {
        new_client_socket = server.Accept();
        if (new_client_socket.Connected)
        {
            lock (lockObject)
            {
                clients_socket.Add(new_client_socket);
            }
            break;
        }
    } 
    clients.Add(new Thread(() => client_recver(user_ticket)));
    clients[i].Start();
}
```

<br/>

## 진행 상황

{% include embed/youtube.html id='sxRaB1sD1oc' %}

현재 모습은 위 영상처럼  
서버에서 두 개의 클라이언트 접속을 허용하고 동시에 위치를 동기화 시키는 모습을 확인할 수 있습니다.  

다음 개발에서는 클라이언트 간 충돌을 구현할 예정입니다.  

<br/>   

## 소스코드 확인하기

[Github 주소](https://github.com/Mgcllee/SearchingCharacterGame/tree/210e4774c5b078c50ec3b091291b9dad97a4b6e5)