---
title:  "[RHTF] C#, C++ 프로젝트에서 Protobuf 적용하기"
excerpt: ""

categories: [Side Project, Rail Hero]
tags: [SideProject, gRPC, protobuf]

toc: true
toc_sticky: true
 
date: 2024-07-19
last_modified_at: 2024-07-19
---

개인 프로젝트에서 C++, C# 서버와 Unity, Unreal Engine 기반 클라이언트 사이 통신을 위해  

gRPC인 **Protobuf** 를 연습하고 있습니다.  

이번 포스트에서는 C#, C++를 사용하는 프로젝트에서 Protobuf 를 적용하는 방법에 대해 설명하겠습니다.  

<br/>

## C# 프로젝트에 Protobuf 적용하기  

---

C# 의 경우 MS 에서 .Net Framework를 위해 직접 만든 언어이므로  

MS 의 IDE인 Visual Studio 2022 를 활용해서 적용하는 방법이 제일 간단합니다.  

적용 방법은 다음 순서와 같습니다.  

<br/>

1. 사용하려는 Protobuf 버전을 확인하기  
2. Nuget Manager 에서 **Google.Protobuf** 패키지 설치하기  
3. 프로젝트에서 사용하기  

<br/>

위의 과정을 차례대로 살펴보겠습니다.  

먼저, Protobuf는 Google 의 오픈소스 프로젝트로 현재도 꾸준한 업데이트가 진행 중에 있습니다.  

따라서 여러분은 Protobuf 를 프로젝트에 **적용하기 전 어떤 버전이 필요한지 확인** 해야 합니다.  

Protobuf 는 **Protoc** 를 기준으로 [각 언어의 버전](https://protobuf.dev/support/version-support/)을 확인할 수 있습니다.  

이때, Protoc 는 .proto 파일을 유저가 원하는 언어로 변환하기 위한 protobuf 전용 컴파일러 입니다.  

저의 경우 protoc 25.1 버전을 사용하기 때문에 그에 대응하는 C#의 protobuf 3.25.1 버전을 사용합니다.  

<br/>

![protobuf version check image](/assets/img/side_project_img/cs_nuget_manager_03.png)

<br/>

사용할 버전을 결정했으면, Visual Studio 2022 로 이동하여  

솔루션 탐색기에서 적용할 프로젝트에 우클릭한 뒤 "Manage Nuget Packages..." 메뉴를 선택합니다.  

<br/>

![vs2022 nuget manager](/assets/img/side_project_img/cs_nuget_manager_01.png)

<br/>

다음으로 "Browse" 메뉴에서 "protobuf" 를 검색하면 다음과 같은 리스트가 나옵니다.  

<br/>

![searching protobuf](/assets/img/side_project_img/cs_nuget_manager_02.png)

<br/>

그러면 위의 이미지처럼 우측의 메뉴에서 원하는 Protobuf 버전을 선택한 뒤 프로젝트에 적용하면 됩니다.  

마지막으로 Protoc 에서 출력된 .cs 파일을 프로젝트에 포함하면 바로 사용할 수 있습니다.  

<br/>

## C++ 프로젝트에서 Protobuf 적용하기

---

C++ 는 **vcpkg** 라는 기술을 활용해 프로젝트에 적용하겠습니다.  

vcpkg 는 설치를 위한 실행파일이 없고, **github** 를 사용해 설치해야 합니다.  

먼저, [vcpkg github 페이지](https://github.com/microsoft/vcpkg)로 이동해 git clone을 진행합니다.  

<br/>

![git clone vcpkg](/assets/img/side_project_img/cpp_vcpkg_protobuf_02.png)

<br/>

그리고 **시스템 환경 변수 편집** 에서 Path 에 **[설치 위치]\vcpkg** 를 등록합니다.  

![vcpkg 환경 변수 등록](/assets/img/side_project_img/cpp_vcpkg_protobuf_03.png)

<br/>

다음으로 cmd 창에서 vcpkg search protobuf 를 입력해 현재 vcpkg 에서 지원하는 protobuf 버전을 확인합니다.  

<br/>

![vcpkg protobuf version](/assets/img/side_project_img/cpp_vcpkg_protobuf_05.png)  
<center>[[vcpkg 공식 사이트](https://vcpkg.io/en/package/protobuf)에서 지원하는 버전 확인]</center>  

<br/>

![vcpkg searching protobuf](/assets/img/side_project_img/cpp_vcpkg_protobuf_04.png)  
<center>[cmd 창에서 현재 vcpkg 에 있는 protobuf 확인하기]</center>  

<br/>

만약, 원하는 버전이 4.25.1 이 아니라면  

**git log** 명령어와 **git checkout** 명령어를 이용해 다른 버전을 사용할 수 있습니다.  

<br/>

![vcpkg searching protobuf](/assets/img/side_project_img/cpp_vcpkg_protobuf_06.png)  

<br/>

마지막으로 **vcpkg install protobuf** 라고 입력하면 현재 커밋에 해당하는 protobuf 버전을 설치합니다.  

이제 Visual Studio 2022 에서 C++ 프로젝트의 설정창으로 이동하여 외부 참조자를 연결하면 됩니다.  

이때, Release Mode 와 Debug Mode 에서 폴더 위치와 .lib 파일이 다르다는 것을 확인하고 연결해줘야 합니다.  

<br/>

### Debug Mode
![protobuf project](/assets/img/side_project_img/cpp_vcpkg_protobuf_07.png)  
<center>[Debug Mode 에서 폴더 위치 연결하기]</center>  

![protobuf project](/assets/img/side_project_img/cpp_vcpkg_protobuf_09.png)  
<center>[Debug Mode 에서 libprotobufd.lib 파일 적용하기]</center>  

<br/>

### Release Mode
![protobuf project](/assets/img/side_project_img/cpp_vcpkg_protobuf_08.png)  
<center>[Release Mode 에서 폴더 위치 연결하기]</center>  

![protobuf project](/assets/img/side_project_img/cpp_vcpkg_protobuf_10.png)  
<center>[Release Mode 에서 libprotobuf.lib 파일 적용하기]</center>  

<br/>

마지막으로 Protoc 에서 출력된 .pb.h, .ph.cc 2개의 파일을 프로젝트에 포함하면 바로 사용할 수 있습니다.  



