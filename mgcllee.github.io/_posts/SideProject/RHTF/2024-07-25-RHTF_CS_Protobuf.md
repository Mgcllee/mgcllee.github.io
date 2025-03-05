---
title:  "[RHTF] Unity 프로젝트에 Protobuf 적용하기"
excerpt: ""

categories: [Side Project, Rail Hero]
tags: [SideProject, gRPC, protobuf]

toc: true
toc_sticky: true
 
date: 2024-07-25
last_modified_at: 2024-07-25
---

## Unity 에서 Protobuf 적용하기

이전 포스트인 ["언리얼에 Protobuf 적용시키기"](https://mgcllee.github.io/posts/RHTF_FirstNet/)는 잘 보셨나요?  

이번 포스트에서는 Unity 프로젝트에서 Protobuf 를 적용히시키는 방법을 알아보겠습니다.  

Unity 는 **C#** 을 주로 사용하기 때문에 C++ 과 달리 매우 쉽게 적용시킬 수 있습니다.  

먼저, [Nuget Packages 사이트](https://www.nuget.org/packages)로 이동하여, 다음 4가지를 다운받습니다.  

<br/>
<br/>

![Nuget_01](/assets/img/side_project_img/cs_protobuf_01.png)  
<center>[Google.Protobuf Nuget Package]</center>

<br/>
<br/>

![Nuget_02](/assets/img/side_project_img/cs_protobuf_02.png)  
<center>[System.Memory Nuget Package]</center>

<br/>
<br/>

![Nuget_03](/assets/img/side_project_img/cs_protobuf_03.png)  
<center>[System.Runtime.CompilerServices.Unsafe Nuget Package]</center>

<br/>
<br/>

![Nuget_04](/assets/img/side_project_img/cs_protobuf_04.png)  
<center>[System.Buffers Nuget Package]</center>

<br/>
<br/>

위의 4가지 패키지를 모두 다운한 뒤 .nupkg 확장자 파일에서 확장자를 .zip 으로 변경해줍니다.  

그리고 압축을 해제하면 여러 폴더가 나오는데 여기서 **"lib\netstandared2.0\"** 폴더에 있는 **.dll** 파일을  

유니티의 **"Asset\Plugin\"** 폴더에 넣어 줍니다.  

> 만약 Plugin 폴더가 없을 경우, 사용자가 직접 만들어도 괜찮습니다.  

위의 과정으로 준비 과정은 모두 끝났습니다.  

이제 Unity 프로젝트에 **Protoc** 로 빌드된 .cs 파일을 포함시켜 사용하면 됩니다.  