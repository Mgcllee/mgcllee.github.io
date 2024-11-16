---
title:  "[CS] 프로세스의 개요"
excerpt: ""

categories: [컴퓨터 구조]
tags: [컴퓨터 구조]

toc: true
toc_sticky: true
 
date: 2024-11-13
last_modified_at: 2024-11-13
---

## 프로세스의 정의

프로세스는 일반적으로 CPU(프로세서)에 의해 처리되는 사용자 프로그램,  
시스템 프로그램 등 **실행중인 프로그램**을 의미합니다.  

프로세스는 실행중인 프로그램이라는 정의 외 여러 정의가 존재합니다.  

* PCB(Process Control Block)을 가진 프로그램  
* 메인메모리에 적재된 프로그램  
* 운영체제가 관리하는 실행 단위  

이 외에도 여러 정의가 존재하지만 가장 중요한 것은 역시, 실행중이라는 의미입니다.  

<br/>

## 프로세스 제어 블록

PCB는 운영체제가 프로세스에 대한 정보를 저장하는 공간의 이름입니다.  
모든 프로세스는 생성될 때, PCB가 생성되고 프로세스가 종료될 때, PCB가 제거됩니다.  

운영체제가 관리하는 프로세스의 정보는 아래와 같습니다.  

|저장 정보|설명|
|---|---|
|현재 상태|프로세스의 준비, 대기, 실행 등의 상태 표시|
|포인터|부모/자식 프로세스 포인터, 현재 메모리 포인터, 자원 메모리 포인터|
|고유 식별자|다른 프로세스와 식별할 수 있는 식별자|
|스케줄링 및 우선순위|운영체제가 사용하는 스케줄링 정보와 실행 우선순위|

이 외에도 CPU 레지스터 정보, 주기억장치 관리 정보 등 여러 정보들이 존재합니다.  

<br/>

## 프로세스 상태 전이

프로세스 상태 전이는 프로세스가 메인메모리에 적재되어 있는 시간동안 상태가 변화하는 것을 의미합니다.  
예를들어 프린트 지시, 외장메모리 복사 등 시간이 오래 걸리는 작업을 수행하는 동안  
해당 프로세스가 CPU(프로세서)를 독점하면 다른 프로세스는 오랜 시간동안 대기해야 합니다.  

따라서 이러한 비효율적 작업처리를 개선하고자  
프로세스의 상태를 제출, 접수, **준비, 실행, 대기**로 나눠 상태에 따라 CPU를 사용합니다.  

![상태전이](/assets/img/CS/프로세스상태전이.png){: width="500", height="500"}  

|상태|설명|
|---|---|
|제출|사용자가 시스템에 작업을 제출할 상태|
|접수|제출된 작업이 디스크 할당 위치에 저장된 상태|
|**준비**|프로세스가 CPU를 할당받기 위해 준비하는 상태<br/>준비상태인 프로세스들은 준비큐에 존재|
|**실행**|준비상태의 프로세스가 CPU를 할당받아 실행되는 상태<br/>CPU의 사용은 할당된 시간만큼 사용하고 반납<br/>I/O작업이 필요하면 대기상태로 전이|
|**대기**|프로세스가 외부 작업을 대기하고 있는 상태<br/>작업이 끝나면 준비상태로 전이|