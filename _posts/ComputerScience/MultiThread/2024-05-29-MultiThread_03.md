---
title:  "[멀티 쓰레드] 멀티 쓰레드의 성능 확인"
excerpt: "멀티 쓰레드 프로그래밍의 성능을 올릴 수 있는 방법을 알아봅니다."

categories: [Computer Science, Multi Thread]
tags: [MultiThread, C&#47;C&#43;&#43;, CS, OS]

toc: true
toc_sticky: true
 
date: 2024-05-29
last_modified_at: 2024-08-08
---

Multi thread program에서 thread design만큼 중요한 것이 **성능 비교 테스트와 VCS**입니다.<br/>
이전에 정리했던 것 처럼 thread의 실행 시점 예측은 불가능합니다.<br/>
따라서 매번 적용한 코드가 정상적으로 동작하는지 확인해야 합니다.<br/>

> 💡 Multi thread program은 디버깅이 어렵고, 어떤 부분이 문제인지 확인하기 어렵습니다.
> 따라서 VCS를 통해 정상 작동하던 Version으로 이동 후 재작성하는 것이 안전합니다.
<br/>
<br/>

Visual Studio에서는 Debug Mode와 Release Mode가 존재합니다.<br/>
Multi thread program의 목적은 오직 성능 향상이기 때문에 <br/>
오류를 찾기 위해 여러 기능이 함께 실행되는 Debug Mode가 아니라 더 좋은 성능을 보여주는<br/>
Release Mode로 실행해야 합니다.<br/>

> 💡 Thread의 증가가 반드시 성능의 증가로 이어지지 않습니다.
> 스택의 증가로 캐시미스가 증가하고 캐시 적중률이 급격하게 저하되어 성능이 저하될 수 있습니다.
<br/>

# Visual Studio Compiler에서 Multithreading 최적화

volatile이라는 키워드는 C++에서 지원하는 키워드로<br/>
컴파일러의 최적화를 방지해서 반드시 메모리를 읽고 변수를 레지스터에 할당하지 않습니다.<br/>
그리고 프로그래머가 작성한 코드 그대로 Read/Write의 순서를 지킵니다.<br/>

> 💡 컴파일러의 최적화 여부는 C++코드로는 확인할 수 없습니다.
> Visual Studio의 경우 This Assembly라는 기능을 사용하여
> 실제로 컴파일러가 프로그래머의 C++코드를 어떻게 최적화 했는지
> Assembly 언어로 확인할 수 있습니다.
> (Visual studio 는 Multithread를 고려하지 않고 최적화하기 때문에.)
<br/>

> 💡 Volatile의 **선언 위치**에 따라 적용되는 대상이 달라질 수 있습니다.
> 1. **volatile int*** value; 의 경우 포인터가 가르키는 메모리가 volatile 입니다. **(X → O)**
> 2. int* **volatile** value; 의 경우 포인터 자체가 volatile 입니다. **(O → X)**
<br/>

# Atomic

Data Race는 여러 쓰레드가 한 공유 메모리에 동시에 접근하여 최소 1개의 수정이 이루어져<br/>
발생하는 문제였습니다. 이 문제를 방지하고자 Mutex의 Lock, Unlock을 사용하였지만<br/>
unlock을 해주지 않을 경우 다른 쓰레드가 무한히 대기하는 상태가 있을 수 있어 위험합니다.<br/>
이 문제를 변수 수준에서 해결하는 방법으로 Atomic 객체가 존재합니다.<br/>
<br/>

Atomic 객체는 동기화 기법을 사용하지 않고 **Read/Write를 동시에 원자적으로 처리**할 수 있습니다.<br/>

```cpp
atomic<int> sum;

void Add_01()
{
		// += 라는 하나의 Method로 진행하기 때문에 Atomic하게 사용할 수 있습니다.
		sum += 2;
}

void Add_02()
{
		// 대입 연산자를 진행하는 중 다른 쓰레드가 개입할 수 있습니다.
		sum = sum + 2;
}
```

단, 위의 연산처럼 sum의 연산 방법에 따라 원자적으로 처리가 불가능할 수도 있습니다.<br/>

> 💡 **Cache Thrashing과 False Sharing 현상**
> 하나의 데이터가 여러 개의 Cache Line에 걸쳐 최악의 경우<br/>
> 캐시 적중과 미스도 아닌 상태 (데이터의 일부가 잘린채 올라온 상태)가 될 수 있습니다.<br/>
> 해결 방법으로는 C++11의 **alignas 기능**을 사용하는 방법이 있습니다.<br/>
> <br/>
> 이 문제로 Multi thraed program에서 Data Race만 문제가 아니라는 것을 볼 수 있습니다.<br/>
> 따라서 Thread Design 단계에서 메모리의 위치도 고려 대상입니다.<br/>

# Atomic programming

## 1. Programming에서 Atomic의 의미

“원자의” 라는 뜻의 Atomic은 Programming 분야에서 **“더 이상 분할이 불가능한 명령어”** 라는 의미입니다. <br/>
Atomic 개념은 주로 **Multithreading**에서 Data Race를 방지하면서 공유 데이터의 일관성을 위해 사용됩니다.<br/>
Multithreading을 사용하는 프로세스가 발생시킬 수 있는 모든 경우의 수를 프로그래머가 알 수 없기 때문에<br/>
Atomic을 사용하는 것이 바람직합니다.<br/>

## 2. 이상적인 Atomic programmings

1. 쓰레드에서 메모리의 접근이 순간적로 실행되면서, 쓰레드의 연산이 겹쳐지지 않습니다.<br/>
2. 연산의 실행 순서가 정해지면 모든 쓰레드는 동일한 순서로 실행합니다.<br/>

## 3. Atomic programming의 현실

1. PC의 메모리 접근 방법은 Atomic이 아닙니다.<br/>
2. 메모리에 쓴 순서대로 메모리가 수정되지 않습니다.<br/> 
    (정확히는 메모리에 쓴 순서대로 메모리의 내용이 관측되지 않습니다.)<br/>