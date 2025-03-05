---
title:  "[네트워크] C#, C++ 상호 변환 제작기"
excerpt: ""

categories: [Network, Protocol Buffers]
tags: [Network, C#, C&#47;C&#43;&#43;, gRPC, protobuf]

toc: true
toc_sticky: true
 
date: 2024-07-01
last_modified_at: 2024-07-01
---

개인 프로젝트에서 C#, C++ 로 각각 작성된 프로그램 사이 통신을 처리할 때 문제가 있었습니다.  

패킷을 만들 때, C#과 C++는 구조체의 형식이 다르기 때문에 송수신만 한다고 해결될 문제가 아니였습니다.  

따라서 최초로 시도한 해결 방법은 C++ 로 구조체를 만들고 C# 구조체로 변경해주는  

**파일 변환 프로그램**을 직접 만들고자 하였습니다.  

그러나 이 프로그램의 문제점은 C++ 의 구조체에서 1byte씩 압축하기 위해 사용한 **#pragma pack (push, 1)**를  

C# 은 단순한 키워드가 아닌 구조체 마다 선언이 필요함에 있었습니다.  

<br/>

이 문제를 해결하고자 최초에는 C++ 에서 작성되있던 struct 를 class 로 변경하고  

클래스 위에 

> [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Ansi, Pack = 1)]

키워드를 넣어 1byte 씩 압축하고자 하였습니다.  

그리고 필드의 특성에 맞춰  

> [MarshalAs(UnmanagedType.ByValTStr, SizeConst = RULE.CHAR_SIZE)]

위와 같은 마샬링 또한 붙일 수 있도록 프로그램을 만들었습니다.   

<br/>

1byte 압축과 필드 마샬링을 진행하였지만, C# 에서는 정상적으로 동작하지 못하였습니다.  

그 이유를 찾아보니, 1byte 압축과 마샬링을 C++ 의 public, int 등의 키워드를 찾아 붙이는 것으로 제작했지만  

원치 않는 곳에 작성될 수 있다는 문제점이 새롭게 생겼습니다.  

<br/>

여기서 저는 이 문제점을 해결하기 위해  

C++ 와 C# 사이 변환을 찾아 보던 중 gRPC, ProtoBuf 라는 기술을 찾게 되었습니다.  

그 중 ProtoBuf는 이 프로그램의 목적인 **언어에 제약 받지 않는 통신**이 가능한 이였습니다.  

따라서 위의 프로그램을 포기하고 ProtoBuf를 사용해 기존에 필요하던 작업을 할 수 있게 되었습니다.

<br/>

ps. 이후 gRPC, ProtoBuf 관련 포스팅에서 후기를 남기겠습니다.  

<br/>

# 자료
[변환 프로그램으로 출력된 .cs 파일](https://github.com/Mgcllee/PokeHunter_LoginServer/blob/main/winform_dummy_client/protocol.cs)  
[변환 프로그램 Git](https://github.com/Mgcllee/PokeHunter_LoginServer/tree/main/send_protocol_file)  