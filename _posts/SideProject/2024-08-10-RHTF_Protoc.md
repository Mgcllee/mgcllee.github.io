---
title:  "[RHTF] Protobuf 컴파일러 Protoc"
excerpt: ""

categories:
  - SideProject
tags:
  - [SideProject, protobuf, C#, C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-08-10
last_modified_at: 2024-08-10
---

Protoc 는 Protobuf 파일을 입력과 함께 전달받은 명령어로 특정 언어로 컴파일된 파일을 출력합니다.  

이번 포스트에서는 Protoc 를 설치하는 방법에 대해서 알아보겠습니다.  

Protoc 를 설치하는 방법은 vcpkg, github 등 여러가지 방법이 있습니다.  

<br/>

## vcpkg 를 사용해서 protoc 설치하기

> vcpkg search protobuf

cmd 창에서 위의 명령어를 입력하면, 현재 사용가능한 protobuf의 버전과 설명이 출력됩니다.  

> vcpkg install protobuf:x64-windows

위의 명령어를 통해 protobuf 의 windows x64 버전을 설치합니다.  
(명령어를 통해 x64 버전 외 다른 버전도 사용할 수 있습니다.)  

이후 설치가 완료되면 vcpkg 폴더에서 다음의 경로로 이동합니다.  

<br/>

![protoc.exe](/assets/img/side_project_img/protoc_01.png)  

<br/>

마지막으로 **protoc.exe** 사용을 위해 **위의 파일 경로를 시스템 환경 변수에 등록**하면 됩니다.  

<br/>

## Github 를 사용해서 protoc 설치하기

![protoc github](/assets/img/side_project_img/protoc_02.png)  

<br/>

[Protobuf Github](https://github.com/protocolbuffers/protobuf/releases) 사이트로 이동하여 원하는 버전의 protoc 압축파일을 다운 받습니다.  

<br/>

![protoc install](/assets/img/side_project_img/protoc_03.png)  

<br/>

다운받은 압축파일을 해제하고 bin 폴더에 들어가면 **protoc.exe** 파일을 확인할 수 있습니다.  

마지막으로 vcpkg에서 **시스템 환경 변수**에 등록한 것과 같이 등록하면 됩니다.  

<br/>

## 설치 방법 고르기

vcpkg 를 통해 설치하는 것과 Github 를 통해 직접 설치하는 방법에 protoc 의 차이점은 없지만  

vcpkg 를 사용할 경우, vcpkg 를 통해 버전 관리가 편리하다는 장점이 있습니다.  

그러나 사용할 protoc의 버전이 **고정**되어 있을 경우, Github 를 통해서 직접 설치하는 방법도 편리합니다.  

