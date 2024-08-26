---
title:  "[RHTF] C#, C++에서 protobuf의 oneof 키워드 사용하기"
excerpt: ""

categories:
  - SideProject
tags:
  - [SideProject, protobuf, C#, C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-08-01
last_modified_at: 2024-08-01
---

## 문제의 코드

개인 프로젝트를 간단히 소개하면 통신하는 프로그램은 유저의 최초 접속부터 매치 메이킹까지 진행하는 C# 메인 서버와  

게임 스테이지를 전문적으로 관리하는 C++ 스테이지 서버로 이루어져 있고  

클라이언트는 PC에서 UE5, 모바일에서 Unity 로 되어있습니다.  

[GitHub Repo](https://github.com/Mgcllee/RHTF/tree/3263700637dbd68c715b03f655bb81038c1a0e7e)  

> **다양한 기술에 도전해보고 싶어 프로젝트에 많은 기술을 넣게 되었습니다.**

이번 포스트의 내용은 protobuf의 **oneof** 키워드를 사용한 메시지 전달 방법 입니다.  

<br/>

## UE5 Client(C++) 에서 Main server(C#) 로 송/수신 하기  

```c++
// ...

// UE5 Client 일부 코드

User::C2SPCLoginUserReq* login_info = new User::C2SPCLoginUserReq();
login_info->set_userid(9785);

User::PacketType packet;
packet.set_allocated_c2sloginuserreq(login_info);

uint8* buffer = new uint8[sizeof(User::C2SPCLoginUserReq)];
packet.SerializeToArray(buffer, sizeof(packet));

int32 bytesent = 0;
bool send_ret = socket->Send(buffer, sizeof(User::C2SPCLoginUserReq), bytesent);

//...
```

<br/>

[이전 커밋](https://github.com/Mgcllee/RHTF/tree/3263700637dbd68c715b03f655bb81038c1a0e7e)에서 위의 코드로 Stage server(C++)와 유효한 통신을 했습니다.  

그러나 문제는 Main server(C#)과 UE5 Client(C++) 사이 유효한 통신이 안된다는 문제점이 있었습니다.  

먼저, 문제점을 해결하기 위해 **이전 커밋**에서 유효한 통신을 주고 받은 UE5 Client(C++)가 아닌 Main server(C#) 에 문제가 있다고 판단했습니다.  

<br/>

```c#
// ...

// Main server(C#) 일부 코드

byte[] data = new byte[test_client.ReceiveBufferSize];
int bytesRead = stream.Read(data.ToArray(), 0, data.Length);

PacketType message;
using (MemoryStream ms = new MemoryStream(data.ToArray(), 0, bytesRead))
{
    message = PacketType.Parser.ParseFrom(ms);
}

switch (message.TypeCase)
{
    case PacketType.TypeOneofCase.C2SLoginUserReq:
        // 클라이언트의 로그인 요청
        break;
    
    // 그 외 다양한 메시지들...
    
    default:
        // ...
        break;
}

// ...
```

<br/>

위의 두 코드를 그대로 실행시킬 경우,  

![Debug_01](/assets/img/side_project_img/debug_protobuf_error_01.png)  
<center>[while문 1회차]</center>  

**protoc**에서 출력된 .cs 파일의 내부 함수에서 수신받은 메시지를 잘 받아  

유효한 데이터 처리를 하는 것 처럼 보이지만 반복문을 이어서 진행하면 아래와 같이 message의 태그가 0이 되어 오류가 발생합니다.  

![Debug_02](/assets/img/side_project_img/debug_protobuf_error_02.png)  
<center>[while문 2회차]</center>  

디버깅 결과 중단점 진입 직전 코드인 **MemoryStream** 부분과 **Parser** 부분이 유효한 데이터 변환을 못 한다고 판단했습니다.  

<br/>

## 해결 방법

혹시 UE5 C++ 에서 FSocket 을 사용하는 것이 문제일 가능성도 있다고 판단해 이전에 유효한 통신을 한 Stage server(C++)에  

UE5 C++ 의 메시지 송신 부분만 사용해 통신을 하던 중 위와 같은 동일한 문제가 발생한다는 것을 알게 되었습니다.  

그리고 이전 **tag zero 버그** 외에 디버거에서 **송신 부분의 직렬화 문제일 수 있다 라는 알림** 을 토대로  

C++의 송신 부분을 수정해야 한다고 판단했습니다.  

<br/>

문제점을 해결하기 위해 [Protobuf 공식 문서의 oneof 설명](https://protobuf.dev/programming-guides/proto3/#oneof-features)을 참고한 결과  

oneof를 사용한 메시지를 설정할 때, **set_allocated_~() 함수로 메시지를 설정하는 것이 아니라 mutable_~() 함수로** 포인터를 받아  

메시지를 설정해야 한다는 것을 알게 되었습니다.  

수정한 코드는 아래와 같습니다.  

```c++
// ...

// 수정된 UE5 Client 일부 코드

User::PacketType packet;

User::C2SPCLoginUserReq* login_info = packet.mutable_c2sloginuserreq();
login_info->set_userid(9785);

std::string buffer_str;
packet.SerializeToString(&buffer_str);

const uint8* buffer = reinterpret_cast<const uint8*>(&buffer_str);

int32 bytesent = 0;
bool send_ret = socket->Send((uint8*)buffer_str.c_str(), buffer_str.length(), bytesent);

// ...
```

## 결과 

이전에 set_allocated_~() 함수를 사용해 송/수신을 했던 모든 C++를 수정한 결과  

Main server(C#), Stage server(C++), UE5 Client(C++) 사이 유효한 메시지 송/수신이 가능하게 되었습니다.  

> 이번 코딩을 통해 **디버깅의 중요성**을 다시금 알게 되었습니다.  