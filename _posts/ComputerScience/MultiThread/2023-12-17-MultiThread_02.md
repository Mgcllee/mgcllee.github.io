---
title:  "[멀티 쓰레드] 멀티 쓰레드 프로그래밍"
excerpt: "멀티 쓰레드 프로그래밍의 개념을 확인합니다."

categories: [Computer Science, Multi Thread]
tags: [MultiThread, C&#47;C&#43;&#43;, CS, OS]

toc: true
toc_sticky: true
 
date: 2023-09-11
last_modified_at: 2024-08-08
---

## 운영체제와 Multi thread programming
멀티쓰레드가 C\++언어에 정식 채택된 버전은 C++11이였습니다. (ISO/IEC 14882:2011)

멀티쓰레드는 **Windows와 Linux에서의 특징**은 다음과 같습니다.

|              |                                             Windows                                             | Linux                                                                                                                     |
| :----------: | :---------------------------------------------------------------------------------------------: |
| **사용방법** |                                WIN32 lib에서 지원되는 API를 사용                                | pthread API사용(ex. gcc-pthread text.cpp로 사용)                                                                          |
|   **특징**   |                                Windows는 Multithread에 특화된 OS                                | Pthread는 POSIX Thread의 약자입니다.                                                                                      |
|  **쓰레드**  | Process를 구성하는 원소(최초는 1개 시작)<br/>OS가 직접 스케쥴링 멀티 Core는 여러 개를 동시 실행 | 쓰레드라는 개념이 없음(모든 것은 Process)<br/>Code, Data를 공유하는 Process를 생성할 수 있다.(타 OS의 Thread와 차이 없음) |

<br/>

## C++11의 thread method
C++11 thread class가 지원하는 Method는 다음과 같습니다.

|             이름             |                            기능                             |
| :--------------------------: | :---------------------------------------------------------: |
|           .join()            |               Thread의 종료까지 기다리는 함수               |
|         .joinable()          |                Thread의 종료를 확인하는 함수                |
|          .get_id()           | kenel에서 Thread의 id를 가져오는 함수 (고유 ID를 가져온다.) |
|          .detach()           |             부모 Thread 객체에서 분리하는 함수              |
|   .hardware_concurrency()    |           현재 실행환경의 Core 수를 알려주는 함수           |
|         this_thread          |               Thread의 this를 의미하는 메소드               |
|    .this_thread::get_id()    |                   자신의 Thread ID를 반환                   |
|  .this_thread::sleep_for()   |       정해진 시간동안 이 Thread의 실행을 멈추는 함수        |
| .this_thread::sleep_untill() |       정해진 시간까지 이 Thread의 실행을 멈추는 함수        |
|    .this_thread::yield()     |      다른 Thread에게 실행 시간을 양보 (시점 예측 불가)      |

<br/>

## Data Race의 개념
하나(공유)의 메모리를 여러 쓰레드에서 <span style="color:black;background-color:#fff5b1"> __읽고 쓰는 순서에 따라서 결과가 달라질 수 있습니다.__ </span>
>단, 여러 쓰레드 중 최소한 하나는 Write를 해야 Data Race가 발생한다.

코어 1개에서 실행하여도 Data Race가 발생할 수 있습니다.
근본적인 문제는 <span style="color:black;background-color:#fff5b1"> **Context switching 시점** </span> 으로 코어의 수와 상관없이
연산 중간에 context switching이 발생할 경우 결과값이 달라질 수 있습니다.
>1 Core환경에서 실행하기 위해서는 프로그램 실행과 동시에
일시정지(입력을 받는 등의 작업으로 실행정지)하고<br/>
’작업관리자’ → ‘자세히’ → ‘해당 프로그램 우클릭’ → ‘선호도 설정’ 에서<br/>
Core 1개를 제외한 나머지는 모두 해제합니다.
(단 Context switching이 정상인 경우 문제가 없어 보일 수 있음)

<br/>

코드에 직접 어셈블리 언어를 작성하여도 CPU에서는 명령어를 한 클럭에 실행시키지 않습니다.
>interept가 발생해야 Context switch 발생
명령어가 종료되면 interept 발생하므로 만약 Core 1개에서
’_asm add sum, 2;’ 와 같이 작성할 경우 Data Race는 발생하지 않습니다.

이 문제는 공유 메모리에 여러 쓰레드가 동시에 접근하는 것이 원인 입니다.
따라서 여러 쓰레드가 동시에 접근하는 것을 막아 Data Race를 해결해야 합니다.

<br/>

## MUTual EXclusion
공유 자원 접근 순서에 따라 실행 결과가 달라지는 프로그램 코드 구역을 **임계구역**(Critical Section)
이라고 합니다. Data Race 해결 방법 중 하나인 **상호배제**(Mutual Exclusion)는
한 쓰레드가 공유 자원에 접근하였으면 <span style="color:black;background-color:#fff5b1">**다른 쓰레드는 접근을 막는 것**</span> 입니다.
(C++11에서) 상호배제 객체로 mutex가 존재하며 lock, unlock을 사용하여 상호배제할 수 있습니다.

<br/>


## Lock and Unlock
Lock을 사용할 경우 한번에 하나의 쓰레드만 실행하고 
다른 쓰레드는 Queue에 순서를 저장하고 스핀하며 대기하기 때문에 
멀티 쓰레드의 목적인 병렬성과 성능이 저하됩니다.

코드에서의 사용방법은 다음과 같습니다.
<pre>
<code>
mutex g_mutex;
int global_value;

// 생성된 쓰레드에서 사용하는 함수
void Funcion()
{
		// 임계구역의 시작입니다.
		g_mutex.lock();    

		// lock과 unlock 사이 임계구역입니다.
		// 1개의 쓰레드에서만 접근이 가능합
		global_value += 1; 

		// 임계구역의 끝으로 unlock을 호출해야 다른 쓰레드에서 lock을 사용합니다.
		g_mutex.unlock();  
}
</code>
</pre>

>Mutex 객체는 Local이 아닌 Global Valiable로 선언해야 합니다.
각 쓰레드에 Mutex 객체가 따로 존재할 경우 상대의 Mutex객체를 확인할 수 없어
Mutex 객체의 기능인 동시 접근 불가를 정상적으로 실행할 수 없습니다.

<br/>

## Peterson lock
2개의 쓰레드를 사용하여 알고리즘을 사용할 때, 매개변수로 Lock과 Unlock을 구현하는 방법입니다.
<pre>
<code>
volatile int victim = 0;
volatile bool flag[2] = (false, false};

Lock (int myID)
{
		int other = 1 - myID;
		flag [myID] = true;
		victim = myID;
		while (flag(other] & victim == myID) ()
}

Unlock(int myID)
{
		flag [myID] = false;
}
</code>
</pre>
실행결과는 정상적으로 동작하지 않습니다.

<br/>
