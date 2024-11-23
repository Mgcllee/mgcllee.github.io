---
title:  "[네트워크] Overlapped I/O"
excerpt: ""

categories: [Network]
tags: [Network, socket]

toc: true
toc_sticky: true
math: true

date: 2024-11-21
last_modified_at: 2024-11-21
---

> 이 포스트는 ["게임 서버 프로그래밍 교과서"](https://product.kyobobook.co.kr/detail/S000001792817?LINK=NVB&NaPm=ct%3Dm3lamecg%7Cci%3De1bf25f4bf80022ba0f2750e1fc4a02f3a415449%7Ctr%3Dboksl1%7Csn%3D5342564%7Chk%3D8e4035492f1c600c6594d4fb77b82e721ba004ec)를 참고하여 작성된 포스트입니다.  

<br/>

이전 포스트에서 [논블로킹(Non-blocking) 소켓](https://mgcllee.github.io/posts/NonBlocking_Socket/)에 대해 설명했습니다.  
논블로킹 소켓의 장단점을 정리하면 아래와 같습니다.  

| 장점                                                        | 단점                                                                    |
| ----------------------------------------------------------- | ----------------------------------------------------------------------- |
| 스레드 블로킹이 없으므로<br/>중도 취소 같은 통제 가능       | 'would block'을 반환받을 경우<br/>**재시도 호출 낭비** 발생             |
| 스레드의 개수가 적거나 1개여도<br/>여러 소켓 사용 가능      | I/O함수를 호출할 때 전달하는 데이터 블록에 대한<br/>**복사 연산 발생**  |
| 스레드의 개수가 적기 때문에<br/>연산량, 호출 스택 낭비 적음 | connect()함수는 재호출이 필요없지만<br/>send(), receive()는 재호출 필요 |

<br>

논블로킹 소켓의 단점 중 첫 번째인 재시도 호출 낭비는 TCP 소켓의 송수신, UDP의 수신에서는 문제가 없지만  
UDP를 이용한 송신에서는 문제가 있습니다.  

예를들어, 송신할 데이터가 4byte인데 송신 버퍼에 남은 자리가 1byte인 경우  
TCP와 달리 **UDP는 일부만 송신할 수 없으므로** 'would block'이 발생합니다.  
이 과정은 송신 버퍼에 4byte가 남을 때 까지 반복되기 때문에 결국 CPU 낭비로 이어집니다.  

<br/>

논블로킹 소켓은 전달 받은 데이터를  
**사용자의 프로세스 영역에서 운영체제 커널 내 소켓 버퍼로 복사**하는 연산을 수행합니다.  
이러한 복사연산이 자주 발생한다면, 고성능 서버에서는 이러한 연산 자체가 부담이 될 수 있습니다.  

<br/>

## Overlapped I/O

위 문제점들을 해결하는 방법이 Overlapped 혹은 비동기 I/O 입니다.  
이 방법은 재시도 호출 낭비와 복사 연산 부하 문제를 해결해 주는 방법으로  
논블로킹 소켓과 진행 과정이 다릅니다.  

### 논블로킹 소켓 진행 과정

1. 소켓이 I/O 가능인 것이 있을 때까지 대기
2. 소켓에 대한 논블로킹 액세스
3. 'would block'이 발생하면 넘어가고, 그렇지 않으면 실행 결과를 반환

### Overlapped I/O 진행 과정

1. 소켓에 Overlapped 액세스 **걸기**
2. Overlapped 액세스 성공 여부 확인 후 결과값을 받아서 나머지 처리

Overlapped I/O의 진행 과정을 의사 코드로 표현하면 아래와 같습니다.  

```c++

// '진행 중인 상태 현황'을 보관하는 구조체
OVERLAPPED overlappedSendStatus;

// Overlapped 전용 함수 호출
// 전송 성공: length에 전송한 데이터의 길이 저장
// 전송 실패: I/O Pending(I/O 완료까지 대기) 상태를 저장
(result, length) = s.OverlappedSend(data, overlappedSendStatus);

if(length > 0) {
    // 데이터 전송 성공
} 
else if(result == WSA_IO_PENDING) {
    while(true) {
        (result, length) = GetOverlappedResult(s, overlappedSendStatus);
        
        // Overlapped 전용 함수를 호출하여 상태를 다시 확인
        if(length > 0) {
            // 데이터 전송 성공
        } else {
            // 아직 I/O Pending 상태
        }
    }
}
```

<br/>

## 송수신 버퍼의 크기가 0 일때, Overlapped I/O

소켓은 내부에 송수신 버퍼를 갖고 있습니다.  
이 송수신 버퍼의 크기를 임의로 설정할 수 있는데,  
이 크기를 0 으로 설정한다면 Overlapped I/O 는 다르게 동작합니다.  

Overlapped I/O 전용 함수를 호출하면 운영체제는 송수신할 데이터가 저장되어 있는  
**메모리 블록 자체를 버퍼로 사용**해버립니다.  
즉, 사용중인 메모리에 송수신한 데이터의 결과를 바로 저장할 수 있습니다.  
<br/>

## Overlapped I/O 의 주의사항

재시도 호출 낭비와 복사 연산을 줄일 수 있다는 장점이 있지만  
Overlapped I/O를 사용할 때는 주의해야 할 점도 있습니다.  
바로 Overlapped I/O 객체에 대한 **백그라운드 접근**으로 인한 주의사항입니다.  

Overlapped I/O 함수는 즉시 반환되지만 운영체제에서는 I/O 실행이 별도로 동시간대에 진행되는 상태입니다.  
즉, 운영체제는 함수의 인자로 넘겼던 데이터를 백그라운드에서 접근하고 있는 것입니다.  

따라서 Overlapped I/O 함수가 비동기로 진행 중인 일이 완료되기 전까지는  
함수에 전달한 인자들을 수정하거나 삭제해서는 안됩니다.  

마지막으로 Overlapped I/O는 운영체제가 윈도우일 때만 동작하는 함수이기 때문에  
리눅스나 다른 운영체제에서는 동작하지 않습니다.  