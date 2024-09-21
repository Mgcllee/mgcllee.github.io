---
title:  "[RHTF] Unreal Engine 프로젝트에 Protobuf 적용하기"
excerpt: ""

categories: [SideProject, Rail Hero]
tags: [SideProject, gRPC, protobuf]

toc: true
toc_sticky: true
 
date: 2024-07-22
last_modified_at: 2024-07-22
---

## Unreal Engine 5.3.2 에서 Protobuf 적용하기
언리얼 엔진의 경우 여러 Plugin 이 존재하지만 gRPC 혹은 Protobuf 를 제공하는 Plugin 을 찾지 못해  

직접 C++ 버전의 Protobuf 를 적용시키고자 시작하게 되었습니다.  

사용하는 파일은 이전 포스트인 **[RHTF] C#, C++ 프로젝트에서 Protobuf 적용하기** 환경에서 그대로 사용합니다.  

C++ 프로젝트로 생성된 UE5 프로젝트 솔루션에서  
"[Your_Project_Name]/Source" 위치에 ProtobufCore 라는 이름의 폴더를 생성해 줍니다.  
(폴더 이름이 반드시 ProtobufCore 일 필요는 없지만, Protobuf 와 관련된 이름을 추천합니다.)  

<br/>

![Make Folder "ProtobufCore"](/assets/img/side_project_img/ue5_protobuf_01.png)  

<br/>

다음으로 ProtobufCore 안에 "Include", "Lib" 폴더를 생성하고  
언리얼 엔진 프로젝트 빌드에 Protobuf 도 함께 빌드함을 알려주기 위해 "ProtobufCore.Build.cs" 파일을 생성합니다.  

<br/>

![make .build.cs file](/assets/img/side_project_img/ue5_protobuf_02.png)  

<br/>

생성한 Lib 폴더에 protobuf 빌드에 필요한 라이브러리 파일을 넣으면 됩니다.  

vcpkg 를 통해 설치된 x64 버전의 protobuf 에서 **"libprotobuf.lib"** 파일을 가져옵니다.  

vcpkg 는 설치된 모듈 폴더로 구분하지 않고 한 폴더에 저장하므로  

**"libprotobuf.lib"** 파일은 **[vcpkg 폴더]\vcpkg\installed\x64-windows\lib** 폴더에 있습니다.  

추가적으로 동일한 위치에 있는 **"abseil_dll.lib"** 폴더도 함께 넣습니다.  

<br/>

![add library files](/assets/img/side_project_img/ue5_protobuf_03.png)  

<br/>

다음으로 이전에 생성한 Include 폴더 안에 protobuf 에서 사용하는 include 폴더를 복사합니다.  

원본 위치는 **[vcpkg 폴더]\vcpkg\installed\x64-windows\include** 에 있고  

복사할 대상은 **absl** 폴더와 **google** 폴더 2개입니다.  

<br/>

![add include folder](/assets/img/side_project_img/ue5_protobuf_04.png)

<br/>

마지막으로 Lib, Include 폴더와 함께 생성한 ProtobufCore.Build.cs 내부를 채우겠습니다.  

<br/>

```c#
using System.IO;
using UnrealBuildTool;

public class ProtobufCore : ModuleRules
{
    public ProtobufCore(ReadOnlyTargetRules Target) : base(Target)
    {
        Type = ModuleType.External;

        // UE 빌드 시스템에게 어떤 폴더를 포함시킬지 알려줍니다.
        PublicSystemIncludePaths.Add(Path.Combine(ModuleDirectory, "Include"));
        PublicIncludePaths.Add(Path.Combine(ModuleDirectory, "Include"));
        
        // library 파일을 추가합니다.
        PublicAdditionalLibraries.Add(Path.Combine(ModuleDirectory, "Lib", "libprotobuf.lib"));
        PublicAdditionalLibraries.Add(Path.Combine(ModuleDirectory, "Lib", "abseil_dll.lib"));
        
        PublicDefinitions.Add("GOOGLE_PROTOBUF_NO_RTTI=1");
    }
}
```

<br/>

그리고 새로운 위치인 **"[Your_Project_Name]/Source/[Your_Project_Name]"** 위치에 생성되어 있는 파일  

[Your_Project_Name].Build.cs 파일의 내부를 다음과 같이 수정합니다.  

<br/>

```c#
// Private 폴더에 있는 .cpp 파일에서 사용한다.
PrivateDependencyModuleNames.AddRange(new string[] { "ProtobufCore" });
```

<br/>

이제 Protobuf 를 사용할 준비는 끝났습니다.  

언리얼 엔진 프로젝트는 기본 C++ 프로젝트와 달리 프로젝트 설정 중 일부가 없어서  

C++ 프로젝트와 다른 방법으로 Protobuf 를 적용해야 합니다.  
