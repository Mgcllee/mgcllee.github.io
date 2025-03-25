---
title:  "채팅 프로그램을 만들자 - vector::reserve 메서드와 vector::emplace_back 메서드"
excerpt: ""

categories: [Side Project, ChattingProgram]
tags: [SideProject, C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2025-04-07
last_modified_at: 2025-04-07
---

C++20 기반 개발 중 vector에서 reserve를 사용하지 않으면  
emplace를 해도 공간이 확보되지 않는다는 문제점을 발견
컴파일과 실행은 정상이지만 Client 클래스의 SOCKET 멤버가 정상적으로 사용되지 않는다는 문제점

이를 검증하여 포스팅할 필요를 느낌