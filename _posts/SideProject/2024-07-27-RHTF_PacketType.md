---
title:  "[RHTF] Protobuf 패킷 타입 구분하기"
excerpt: ""

categories:
  - SideProject
tags:
  - [SideProject, protobuf, socket]

toc: true
toc_sticky: true
 
date: 2024-07-27
last_modified_at: 2024-07-27
---

## 현재 문제점

[문제의 코드](https://github.com/Mgcllee/RHTF/blob/e0c7f2f64956a9e6e1287bb2668f2395d3c7f062/RHTF_Server/RHTFStageServer/worker.cpp#L126)에서는 패킷을 모두 protobuf가 아니라 구조체로 작성했을 때, 유효했습니다.  

그러나 이 프로젝트에서는 Protobuf를 사용한 패킷 통신이기 때문에 문자열을 그대로 해석하면 정상적인 해석이 불가능합니다.  

<br/>

```c++
void process_packet(int c_id, char* packet)
{
    switch (packet[1])
    {
    case User::NUM::C2SLoginUserReq:
    {
        // ...
    }
    break;
    case User::NUM::S2SLoginUserReq:
    {
        // ...
    }
    break;
    }
}
```

```proto
syntax = "proto3";

package User;

enum NUM {
	C2SLoginUserReq     = 0;
	S2SLoginUserReq     = 1;
}

message C2SPCLoginUserReq {
    NUM Type            = 1;
	uint32 UserID       = 2;
}

message S2CPCLoginUserRes {
    NUM Type            = 1;
	uint32  UserID		= 2;
	string  UserName	= 3;
	uint32  UserLevel	= 4;
}
```

<br/>

## 해결 방법

따라서 위의 문제점을 해결하고자 위의 .proto 파일을 다음과 같이 변경하였습니다.  

<br/>

```proto
syntax = "proto3";

package User;

enum NUM {
	C2SLoginUserReq = 0;
	S2SLoginUserReq = 1;
}

message Default {
	uint32 Size = 1;
	NUM Type	= 2;
}

message C2SPCLoginUserReq {
	uint32	Size	= 1;
	NUM		Type	= 2;
	uint32 UserID	= 3;
}

message S2CPCLoginUserRes {
	uint32	Size		= 1;
	NUM		Type		= 2;
	uint32  UserID		= 3;
	string  UserName	= 4;
	uint32  UserLevel	= 5;
}
```

<br/>

모든 패킷이 크기와 타입 정보 갖도록 수정하였습니다. 또한 크기와 타입만 갖고 있는 Default message 를 통해서  

현재 수신한 패킷의 타입을 확인할 수 있도록 C++ 코드를 아래와 같이 수정하였습니다.  

<br/>

```c++
void process_packet(int c_id, char* packet)
{
    User::Default buffer;
    buffer.ParseFromString(packet);
    User::NUM packet_type = buffer.type();

    switch (packet_type)
    {
    case User::NUM::C2SLoginUserReq:
    {
        // ...
    }
    break;
    case User::NUM::S2SLoginUserReq:
    {
        // ...
    }
    break;
    default:
    {
        printf("Invalid packet! [%d]\n", packet_type);
    }
    break;
    }
}

```

<br/>

## 개선할 점

protobuf의 모든 메시지에 size, type을 작성하는 것은 새로운 메시지를 생성할 때 마다  

작성해줘야 하기 때문에 비효율적이라고 생각됩니다.  

따라서 이 점을 개선하기 위해 좀 더 연구가 필요합니다.
