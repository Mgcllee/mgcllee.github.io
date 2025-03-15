---
title:  ".NET으로 서버 만들기 - 프로젝트 기본 설정하기"
excerpt: ""

categories: [Side Project, ChattingProgram]
tags: [SideProject, C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2025-03-20
last_modified_at: 2025-03-20
---

> **".NET으로 서버 만들기 시리즈"**는 처음 .NET을 접한 뒤 도전해 보는 프로젝트로  
> 기술의 내용이 부족할 수 있습니다.  

<br/>

## NDC로 접한 Orleans
---

NDC(Nexon Developers Conference)의 여러 강연을 구경하던 중 우연히 발견한  
["‘헤일로’로 부터 배우는 가상 액터 모형과 MSR의 Orleans Project"](https://youtu.be/SIOtlPWYFTw?feature=shared)라는 강연을 보게 되었습니다.  
이 강연에서 소개된 **Orleans**라는 오픈 소스 프로젝트가 흥미로워 시작하게 되었습니다.  

<br/>

Orleans는 OOP(객체지향 프로그래밍)과 유사한 부분이 있지만 OOP 보다 더욱 독립적으로  
객체를 관리하는 액터 모델이라는 프로그래밍 기법을 사용합니다.  
액터의 특징인 독립성, 액터 간 통신, 캡슐화를 간략히 설명하면 아래와 같습니다.  

액터 모델이란, 독립적으로 존재하며 다른 액터와 상태(멤버 변수)를 공유하지 않습니다.  
C++의 접근 제한자를 빌려 설명하면 모든 상태는 private으로 선언하여 **외부의 접근을 허용하지 않는 것**입니다.  

모든 상태는 외부에서 접근할 수 없으므로 현재 액터가 다른 액터의 상태를 알고 싶다면 직접 접근하는 것이 아니라,  
**액터 간 통신**으로 얻어 올 수 있습니다.  

그리고 액터 상태에 직접 접근이 불가능하기 때문에 액터 상태의 변경은  
오직 액터의 내부(private)에서 변경됩니다.  

<br/>

액터는 다른 액터와의 통신으로 자신이 소유하지 않은 상태를 가져오고  
해당 상태에 대한 변경은 상태를 소유한 액터만 변경이 가능하므로 액터들은 각자 실행된다는 것을 알게 되었습니다.  

멀티쓰레드를 토대로 설명하면, 동시에 여러 액터를 실행하기 위해 여러 쓰레드로 여러 액터를 실행하는 것과 같습니다.  

C++에서 액터 구조를 제작하려면 쓰레드를 활용해 동시성 만들거나  
비동기 기능을 사용해서 만들어야 합니다.  

그러나 Orleans의 경우, 프레임워크에서 생성된 액터들을 관리해주기 때문에 쉽게 액터를 사용할 수 있습니다.  
Orleans 프레임워크는 액터가 사용되지 않을 경우, 액터를 소멸시키고 액터가 필요할 경우 생성하는 방식으로  
여러 액터들을 동시에 관리합니다.  

Orleans 공식 문서에서는 프레임워크가 직접 생성하고 종료시키기 때문에 이를  
**Virtual Actor Model(가상 액터 모델)**이라고 부릅니다.

Orleans가 가장 필요했던 이유는 위와 같은 가상 액터 모델이
단일 코어 환경에서 동작할 수 있었기 때문입니다.  

<!-->Orleans 공식문서에서 찾아서 내용 첨부하기<-->

![Orlenas 이점 01](/assets/img/side_project_img/TrickFarm/Orleans이점_01.png){: width="600", height="600"}  
<center>[출처: MS Learn Orleans의 이점]</center>

<br/>

로컬에서 실행하는 환경은 VM Ware에 Linux Ubuntu server 22.04를 이용하여  
실행할 예정입니다.  

<br/>

### 출처
---

1. 조직의 단일 스레드 실행 가능한 Orlenas [소개](https://learn.microsoft.com/ko-kr/dotnet/orleans/benefits#single-threaded-execution-of-grains)  
