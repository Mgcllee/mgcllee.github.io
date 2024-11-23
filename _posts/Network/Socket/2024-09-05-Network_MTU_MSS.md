---
title:  "[네트워크] 소켓 통신에서 버퍼 용량 결정하기"
excerpt: ""

categories: [Network, Network Socket]
tags: [Network, socket, MTU, MSS]

toc: true
toc_sticky: true
math: true
 
date: 2024-09-05
last_modified_at: 2024-09-05
---

통신을 위해 소켓을 만들고 서버에서 bind와 listen 을 실행하고 클라이언트는 connect 를 실행한 하면  
서버와 클라이언트는 간단한 패킷을 주고 받을 준비가 됬다고 할 수 있습니다.  
<br/>

[패킷(packet)](https://ko.wikipedia.org/wiki/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC_%ED%8C%A8%ED%82%B7)이란 정보 기술에서 패킷 방식의 컴퓨터 네트워크가 전달하는 데이터의 형식화된 블록입니다.  
즉, 컴퓨터가 네트워크에서 데이터를 주고 받을 때 정한 규칙이라고 할 수 있습니다.  
<br/>

패킷은 OSI 7계층을 지나면서 사용하는 버전이 IPv4, IPv6 중 무엇인지 저장하고  
TCP, UDP 와 같이 어떤 프로토콜을 사용하는지 그리고 오류 방지를 위한 체크섬 등 여러가지 정보를 저장합니다.  
<br/>

패킷을 사용한 소켓 통신 프로그램을 만들면서 가장 고민해온 것이 여러 헤더가 붙어 있는 패킷에서  
패킷 속 설계한 데이터가 나눠지지 않고 한 번에 전송하고 수신되는 **적절한 크기가 있을까** 라는 질문입니다.  
<br/>

이 궁금증을 해결하고자 여러 자료를 찾던 중 **최대 전송 단위(MTU)** 와 **최대 세그먼트 크기(MSS)** 를 발견하였고  
이를 이번 포스트에서 정리하고자 합니다.  
<br/>

## 최대 전송 단위

MTU(Maximum Transmission Unit)란 네트워크를 통해 전송할 수 있는 최대 패킷의 크기를 의미합니다.  
현재 컴퓨터에서 MTU 의 크기를 확인하는 방법은 다음 사진과 같습니다.  
<br/>

![MTU_cmd](/assets/img/Network/MTU_cmd.png){: width="1026" height="252" }  

> netsh interface ipv4 show subinterfaces  

<br/>

먼저 표의 가장 마자막 열인 "인터페이스" 는 사용되는 네트워크 장치를 의미합니다.  
"바이트 인", "바이트 아웃"은 현재 컴퓨터를 기준으로 수신, 송신된 패킷의 바이트 누적 크기를 의미합니다.  
마지막으로 가장 오른쪽에 있는 MTU 가 패킷의 최대 전송 단위를 의미합니다.  
<br/>

MTU 값들을 살펴보면 루프백 주소를 제외하면 전부 값이 **1500** 으로 동일합니다.  
1500 는 제조사에서 설정한 값이 아닌 국제 표준인 IEEE802.3 에서 MTU 의 값을 1500 byte 로 결정한 것입니다.  
<br/>

네트워크에 담을 수 있는 패킷의 최대 크기는 1500 byte 라는 것을 확인할 수 있었는데  
중요한 것은 1500 byte 안에 프로그램에서 설계한 패킷만 담겨있는 것이 아니라  
처음에 언급한 여러 헤더들도 함께 담겨있기 때문에 실제로는 1500 byte 를 전부 사용할 수 없습니다.  
따라서 실제 사용 가능한 크기인 **최대 세그먼트 크기** 에 대해서 알아보겠습니다.  
<br/>

## 최대 세크먼트 크기

MSS(Maximum Segment Size)란 MTU에서 Header 용량을 제외한 나머지 용량을 뜻합니다.  
<br/>

![MSS](/assets/img/Network/MSS_TCP_segment_packet_diagram.png){: width="800" height="500" }    

<br/>

MSS 크기는 반드시 MTU 보다 작으며 MTU 에서 IP 헤더와 TCP 헤더 크기를 빼면 알 수 있습니다.  
일반적으로 IP 헤더와 TCP 헤더의 크기는 20 byte 이기 때문에 MSS 는 1460 byte 라는 크기가 됩니다.  
여기서 대부분의 소켓 프로그램에서는 1460 byte 를 전부 사용하지 않고 1024 byte ( $$2^10 = 1024$$ )를 사용합니다.  
<br/>

## 패킷 크기 결정하기 

만약, 설계된 패킷이 1460 byte 보다 더 크다면 해당 패킷은 송신 혹은 수신이 불가능할 수 있습니다.  
반대로 패킷이 너무 작으면 패킷 크기에 비해서 네트워크 자원 소모가 많아 손해를 볼 수 있습니다.  
따라서 MSS 의 크기를 고려해 1024 byte 에 근접한 패킷 크기가 가장 적절합니다.  
패킷에 저장해야 하는 데이터가 1024 byte 보다 클 경우 해당 데이터를 여러 패킷으로 나누어 설계하거나  
C++ 에서 제공하는 "#pragma pack (push, 1)" 을 이용하여 패킷의 크기를 최소한으로 줄일 수 있습니다.  
<br/>

## 참고

[위키피디아 패킷 정의](https://ko.wikipedia.org/wiki/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC_%ED%8C%A8%ED%82%B7)  
[cloudflare MSS 사진](https://www.cloudflare.com/ko-kr/learning/network-layer/what-is-mss/)  
