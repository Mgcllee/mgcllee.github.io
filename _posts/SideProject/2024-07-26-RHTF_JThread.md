---
title:  "[RHTF][기록] thread에서 .join()의 중요성"
excerpt: ""

categories:
  - SideProject
tags:
  - [SideProject, thread, jthread, C++20]

toc: true
toc_sticky: true
 
date: 2024-07-26
last_modified_at: 2024-07-26
---

## 자동으로 .join() 해줄 것이라는 착각

RHTF 프로젝트를 진행하면서, Stage Server에서 사용하는 멀티스레드를  

기존에 사용해오던 thread class가 아니라, **C++20의 jthread**를 사용해 보았습니다.  

jthread는 기존 thread와 달리 join() 함수가 자동으로 처리되는 함수라고 [소개](https://en.cppreference.com/w/cpp/thread/jthread)되어 있습니다.  

**자동으로 join()** 이라는 말에 현혹되어 저도 모르게 메인 함수에서 생성한 자식 스레드를 조인하지 않고  

다음과 같이 [코드를 작성](https://github.com/Mgcllee/RHTF/blob/4afb0acf3e321f8ea296bb7f3c2dd82b58a67e07/RHTF_Server/RHTFStageServer/Main.cpp#L42)했습니다.  

## 문제점 확인하기

<br/>

![Miss_01](/assets/img/side_project_img/RHTF_Miss_01.png)  
<center>[문제점이 발견된 코드]</center>

<br/>

위의 코드에서 잘못된 점이 보이시나요?  

메인 함수에서 **생성된 자식 스레드**가 일하는 중에 메인 함수에서  

closesocket(), WSACleanup() 함수를 호출해서 다음과 같은 오류가 발생했습니다.  

<br/>

![Miss_02](/assets/img/side_project_img/RHTF_Miss_02.png)  
<center>[GetLastError() 함수로 출력된 오류 번호]</center>

<br/>

참고로 Visual C++ 에서 사용하는 GetLastError() 함수는 오류 코드만 출력하기 때문에 사용자가 직접 무슨 내용인지 찾아봐야 합니다.  

여기서 사용하기 좋은 기능이 **"Tool 메뉴 -> Error Lookup 선택 -> 출력된 오류 코드 검색"** 을 통해  

출력된 오류 코드의 의미를 확인할 수 있습니다.  

<br/>

![VC2022_ErrorLookup](/assets/img/side_project_img/RHTF_Miss_03.png)  
<center>[오류 코드 내용 찾는 방법 1]</center>

<br/>


![VC2022_ErrorLookup](/assets/img/side_project_img/RHTF_Miss_04.png)  
<center>[오류 코드 내용 찾는 방법 2]</center>

<br/>

## 문제점 해결하기

다시 본론으로 돌아와 **"스레드 종료 또는 응용 프로그램 요청 때문에 I/O 작업이 취소되었습니다."** 라는 설명처럼  

메인 스레드의 끝에 있는 closesocket(), WSACleanup() 함수로 인해서 소켓이 닫혀버렸고  

그로 인해서 GetQueuedCompletionStatus() 함수에서 FALSE 를 반환하는 문제점이 있었던 것 입니다.  

따라서 이 문제점을 해결하기 위해 아래와 같은 코드를 메인 함수에 추가했습니다.  

```c++
std::vector<std::thread> worker_threads;
int thread_cnt = std::thread::hardware_concurrency();

for (int i = 0; i < thread_cnt; ++i)
{
    worker_threads.emplace_back(worker_thread, h_iocp);
}

for (auto& th : worker_threads)
{
	th.join();
}

closesocket(g_s_socket);
WSACleanup();
```

위 코드 덕분에 메인 함수는 closesocket(), WSACleanup() 함수를 실행하기 전  

worker_thread 에 담겨 있는 자식 스레드가 join 될 때 까지 대기하게 되어  

worker_thread 에 담겨 있는 자식 스레드가 정상적으로 실행됩니다.