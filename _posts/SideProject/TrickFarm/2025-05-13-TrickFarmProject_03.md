---
title:  "[Docker] Windows에서 Linux 기반 .NET 프로그래밍하기 (Dev Container)"
excerpt: ""

categories: [Side Project, Trick Farm]
tags: [.NET, Orleans, C#, Linux]

toc: true
toc_sticky: true
 
date: 2025-05-13
last_modified_at: 2025-05-13
---

# OS 제약 없는 개발 가이드

개발 환경과 다른 운영체제를 바탕으로 .NET 프로그램을 만드는 방법을 알려드려요.

이러한 방법으로 Windows에서 Linux 전용 프로그램을 개발할 수 있어요.

<br/>

## 'Visual Studio Code'에서 Dev Container 사용 방법  

'Visual Studio Code'(이하 VSCode)과 'Visual Studio 2022'(이하 VS2022)는 MicroSoft에서 제작한 IDE로  
두 프로그램 모두 Linux 프로그램을 개발할 수 있지만 방법 과정에서 차이가 나타나요.  

우선 VSCode의 경우, Dev Container라는 확장 앱을 사용합니다.  

![DevContainer Ex](/assets/img/side_project_img/TrickFarm/DevContainer소개.png){: width="550", height="550"}  

[이 확장앱](https://code.visualstudio.com/docs/devcontainers/create-dev-container)은 사용자가 설정한 dockerfile을 바탕으로 컨테이너를 만들어줍니다.  
VS Code가 설치되어 있는 상태에서 Dev Container를 설치하는 방법은 아래와 같습니다.  

1. Windows 전용 Docker 앱인 Docker Desktop 설치  

2. VS Code의 확장앱 검색에서 'Dev Container' 검색 후 설치  

3. 'Dev Container' 설치 후 사이드바에 'Remote Explorer'로 이동  

4. 'Remote Explorer' 창에서 Dev Container 추가를 위해 아래와 같이 '+' 기호 클릭  

![컨테이너 생성하기 01](/assets/img/side_project_img/TrickFarm/DevContainer만들기_01.png){: width="300", height="300"}  

5. 컨테이너를 생성할 때, 컨테이너의 타겟을 '현재 폴더', '새로운 컨테이너', 'GitHub 저장소' 등에서 원하는 것을 선택  

![컨테이너 생성하기 02](/assets/img/side_project_img/TrickFarm/DevContainer만들기_02.png){: width="500", height="200"}  

6. '새로운 컨테이너' 옵션을 선택하면 운영체제, 사용할 언어와 버전, 함께 사용할 데이터베이스, 이외 옵션 등을 함께 선택  

![컨테이너 생성하기 03](/assets/img/side_project_img/TrickFarm/DevContainer만들기_03.png){: width="400", height="400"}  

<br/>

설정이 모두 완료되면 'Remote Explorer' 창에서 아래의 사진처럼 새로운 컨테이너가 생성된 모습을 확인할 수 있습니다.  
그리고 설치되어 있는 'Docker Desktop'의 Container 메뉴를 보면 생성한 컨테이너를 확인할 수 있습니다.  

![컨테이너 생성하기 04](/assets/img/side_project_img/TrickFarm/DevContainer만들기_04.png){: width="300", height="300"}  

![컨테이너 생성하기 05](/assets/img/side_project_img/TrickFarm/DevContainer만들기_05.png){: width="550", height="550"}  

<br/>

## 컨테이너 접속하는 방법과 나가는 방법

Dev Container로 생성한 컨테이너로 접속하는 방법으로 2가지가 있습니다.  
아래의 사진처럼 'Remote Explorer'에 있는 화살표 아이콘을 클릭하여 접속하거나  
VS Code 상단의 명령줄에서 '> Dev Containers: Open ~'를 입력하는 방법이 있습니다.  

![컨테이너 생성하기 07](/assets/img/side_project_img/TrickFarm/DevContainer만들기_07.png){: width="300", height="300"}  

![컨테이너 생성하기 06](/assets/img/side_project_img/TrickFarm/DevContainer만들기_06.png){: width="550", height="300"}  

<br/>

이후 컨테이너 안에서 작업이 모두 완료되어 컨테이너를 나가고 싶을 경우,  
VS Code 상단의 명령줄에서 '> Dev Containers: Close ~'를 입력하면 현재 접속한 컨테이너를 닫을 수 있습니다.  

![컨테이너 생성하기 06](/assets/img/side_project_img/TrickFarm/DevContainer만들기_08.png){: width="550", height="300"}  

<br/>

> 이 포스트는 .NET Orleans를 배우면서 작성한 내용입니다.
> 오류나 틀린 부분이 있을 경우 언제든지 댓글 혹은 메일로 지적해주시면 감사하겠습니다!
