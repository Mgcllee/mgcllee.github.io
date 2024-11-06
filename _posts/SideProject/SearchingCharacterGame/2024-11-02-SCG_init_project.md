---
title:  "[SCG 프로젝트] MonoGame과 C# 서버로 간단한 프로그램 만들기"
excerpt: ""

categories: [SideProject, Fireboy Watergirl and Earthman]
tags: [SideProject, C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-11-02
last_modified_at: 2024-11-02
--- 

> 이 포스트는 개인 프로젝트의 기록입니다.  

C# 을 사용해 보기 위해 무엇을 만들어볼까 고민 중  
SFML처럼 Monogame이라는 오픈소스 프로젝트를 접하게 되었습니다.  

[Monogame](https://monogame.net/)은 MS(MicroSoft)의 XBOX 360 게임 개발을 위해 배포하던 SDK를
오픈소스 커뮤니티에서 크로스 플랫폼으로 확장한 시킨 프로젝트입니다.
또한 가장 큰 특징은 C#을 사용한다는 점입니다.  

크로스 플랫폼이기 때문에 플랫폼(Window, Linux, Sitch, ... etc)에 제약받지 않고  
지원되는 플랫폼 모두 개발이 가능하다는 장점도 있습니다.  

![결과](/assets/img/side_project_img/monogame_소개.png){: width="600", height="500"}  

<br/>

## 프로젝트 목표

이번 프로젝트의 목표는 C# 서버를 사용한 **스트레스 테스트**로  
클라이언트 요청을 서버가 어떻게 처리 해야할지 연습해보고자 합니다.  

요청의 종류로는 이동 및 충돌 그리고 위치 검증과  
랜덤한 위치로의 최단거리 이동을 목표로 하고 있습니다.  

<br/>

## 진행 상황

![결과](/assets/img/side_project_img/monogame_init_project.png){: width="600", height="500"}  

첫 진행 상황은 위의 그림처럼  
서버 콘솔에서 입력한 위치가 송신되어 클라이언트가 수신한 뒤 초록 원을 이동시키는 상태입니다.  

현재는 단일 클라이언트이지만  
다음 개발에서는 복수의 클라이언트 접속을 구현할 예정입니다.  

<br/>

## 소스코드 확인하기

[Github 주소](https://github.com/Mgcllee/SearchingCharacterGame/tree/d569f64c8434db5027424999ad94504e1856eca7)