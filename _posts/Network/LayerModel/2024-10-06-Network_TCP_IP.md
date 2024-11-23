---
title:  "[네트워크] TCP/IP 는 무엇일까"
excerpt: ""

categories: [Network]
tags: [Network, socket, MTU, MSS]

toc: true
toc_sticky: true
math: true

date: 2024-10-06
last_modified_at: 2024-10-06
---

[저번 포스트](https://mgcllee.github.io/posts/OSI_7/)에서는 네트워크 통신을 위한 국제 통신 모델인 OSI 7 계층에 대해서 알아보았습니다.  
이번 포스트에서는 OSI 7 계층의 3, 4 계층에 대해 조금 더 자세히 알아보겠습니다.  

<br/>

## 통신규약 모음 (Internet Protocol Suite)

인터넷에서 컴퓨터들이 서로 정부를 주고받는 데 쓰이는 **통신규약(프로토콜) 모음** 을  
인터넷 프로토콜 슈트(Internet Protocol Suite) 라고 합니다.  
통신규약 모음 중 TCP와 IP가 가장 많이 쓰이기 때문에 TCP/IP 프로토콜 슈트라고도 합니다.  

<br/>

## TCP/IP 

![TCP/IP](/assets/img/Network/TcpIpPacket.png){: width="350", height="350"}  

IP 는 Internet Protocol 의 약자로, 인터넷에서 정보를 송수신할 때, 사용하는 통신규약을 의미합니다.  
IP 는 패킷이 정상적으로 전달되었는지 보증하지 않고, 패킷의 송수신 순서가 의도와 다를 수 있습니다.  
네트워크 계층에서 목적지를 찾는 기능(Routing)에 사용됩니다.  

TCP 는 IP 위에서 동작하는 프로토콜로, 데이터의 전달을 보증하고 패킷 순서를 보증합니다.  
전송 계층에서 사용자(컴퓨터) 간의 연결을 확인하고 데이터 전송 여부 등을 확인합니다.  
HTTP, FTP, SMTP 등 다양한 응용 프로그램 프로토콜이 TCP 를 기반으로 동작하기 때문에  
TCP 와 IP 를 묶어서 TCP/IP 라고 부릅니다.  

정리하면, TCP/IP 를 사용한다는 것은 IP 주소를 사용해서 데이터를 목적지로 전달하고  
TCP 를 사용해 유효한 데이터가 순서를 지켜 도착하도록 한다는 것 입니다.  

<br/>

## TCP

TCP 는 OSI 7 계층 중 4 계층(전송 계층)에서 사용합니다.  
IP 가 패킷들의 관계를 신경쓰지 않고 **오직 목적지만** 설정했다면  
TCP 는 양쪽 사용자 단말(Endpoint)이 준비되었는지, 송수신 패킷이 유효하게 완료되었는지,  
패킷 내용 중 누락된 것은 없는지 등 패킷에 대해 점검합니다.  

송수신한 패킷에서 무엇을 확인해야 하는지는 TCP Header 에 담겨있습니다.  
TCP Header 에는 SYN, ACK, FIN, Source Port, Destination Port, Sequence Number 등  
신뢰성 보장, 흐름 제어와 관련된 정보들이 담겨있습니다.  
그리고 IP Header, TCP Header 를 제외한 TCP 가 담을 수 있는 데이터의 크기를 세그먼트(Segment)라고 부릅니다.  

TCP 는 송수신을 위해 IP 뿐만 아니라  
목적지(컴퓨터)에 도착한 뒤 어떤 프로세스에 전달해야 하는지 저장하는 Port 번호도 저장합니다.  
TCP Header 에서 출발지 포트(Source Port)와 도착지 포트(Destination Port)에 저장하여 패킷 전달에 사용됩니다.  

<br/>

## 3-way-handshake

TCP 는 신뢰성 있는 통신을 위해서 패킷을 전송하기 전 송신자, 수신자 서로가  
네트워크 통신이 가능한지, 가능하다면 얼마나 받을 수 있는지 등을 확인합니다.  

여기서 사용되는 것이 TCP Header 의 **SYN, SYN/ACK, ACK** flag 입니다.  
1. 송신자가 수신자에게 SYN 를 송신해서 통신이 가능한지 확인합니다.  
2. 수신자가 SYN 를 받고 수신할 준비가 되었다면, SYN/ACK 를 보내 수신 준비 완료를 알립니다.  
3. 송신자가 SYN/ACK 를 받았다면, ACK 를 보내 송신 시작을 알립니다.  

![3-way-handshake](/assets/img/Network/ThreeWayHandshakeProccess.png){: width="350", height="350"}  

TCP 를 사용하는 모든 통신은 반드시 **3-way-handshake 로 시작합니다.**  

<br/>

> 개인 공부 기록용 블로그입니다. 틀린 부분이 있을 경우 댓글 혹은 메일로 지적해주시면 감사하겠습니다! 

## 참고

[[전송 제어 프로토콜], 위키 백과](https://ko.wikipedia.org/wiki/%EC%A0%84%EC%86%A1_%EC%A0%9C%EC%96%B4_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C)  
[[네트워크 프로그래밍 : TCP/IP 개론], 조인씨 블로그](https://www.joinc.co.kr/w/Site/Network_Programing/Documents/IntroTCPIP)  