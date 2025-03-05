---
title:  "[네트워크] C#과 C++ 사이 통신하기"
excerpt: ""

categories: [Network, Protocol Buffers]
tags: [Network, C#, C&#47;C&#43;&#43;, gRPC, protobuf]

toc: true
toc_sticky: true
 
date: 2024-07-02
last_modified_at: 2024-07-02
---

# ProtoBuf 적용 과정

C#, C++ 으로 작성된 프로그램 사이 소켓 통신을 하기 위해 ProtoBuf 를 사용하게 되었습니다.  

Protobuf 란 Google 에서 제작한 오픈소스 기술로 

플랫폼과 언어의 제한 없이 통신 프로토콜이나 데이텅 저장을 사용할 때, 구조화 된 데이터를 전환하게 해 주는 방법입니다.  
(gRPC, ProtoBuf 와 관련된 이론은 추후 포스팅 하겠습니다.)  

현재 코드에서는 C#, C++ 각각에서 모두 사용할 수 있도록  

.proto 파일을 protoc 컴파일러로 .cs 파일과 .pb.h, .pb.cc 파일을 추출하였습니다.  

추출된 파일을 각 프로젝트에 연결하여 .proto 파일에서 만들었던 message 를 사용할 수 있었습니다.  

<br/>

# 현재 의문점

[현재 코드](https://github.com/Mgcllee/RHTF/blob/main/RHTF_Server/RHTFStageServer/Main.cpp)의 76, 77번 째 줄에서  

loginf_pack.ParseFromArray(&buffer, sizeof(S2CPCLoginUserRes)) 를 사용하면 S2CPCLoginUserRes 의 필드 값이 변환 되지 않고  

loginf_pack.ParseFromString(buffer); 를 사용하면 정상적으로 변환 됩니다.  

ParseFromArray 함수와 ParseFromString 함수의 공식 문서 설명은 다음과 같습니다.  

> ParseFromArray : Parse a protocol buffer contained in an array of bytes.  
> ParseFromString : Parses a protocol buffer contained in a string.  

string 또한 1byte 크기의 char 들로 구현된 자료구조로 알고 있어서  

아직 두 함수의 차이점을 확인하지 못 하였습니다.  

<br/>

# GitHub
[GitHub 소스코드](https://github.com/Mgcllee/RHTF/tree/main/RHTF_Server)  
