---
title:  "채팅 프로그램을 만들자 - 프로젝트 기본 설정하기"
excerpt: ""

categories: [Side Project, ChattingProgram]
tags: [SideProject, C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2025-03-10
last_modified_at: 2025-03-10
---

> 포스트 **'채팅 프로그램을 만들자' 시리즈**에서는 프로젝트가  
> 어떤 의도로 설계 되었는지부터 서버 프로그램에서 IOCP를 1개가 아닌 여러 개를 사용하도록 변경한 이유까지  
> 프로젝트를 진행하면서 있었던 일들을 다룰 예정입니다.  

<br/>

최근 네트워크 프로그래밍의 기본인 **네트워크 소켓**을 연구하기 위해 진행하고 있는 프로젝트가 있습니다.

이 프로젝트는 다수의 클라이언트 역할을 수행하는 **더미 클라이언트** 프로그램과  
더미 클라이언트의 접속을 받아 요청을 처리하는 **서버 프로그램**으로 구성되어 있습니다.  

프로젝트의 구조는 아래 그림과 같습니다.  

![첫도안](/assets/img/side_project_img/ChattingProgram/ChattingProgram_01.png){: width="550", height="550"}  

더미 클라이언트라는 프로그램 안에서 여러 네트워크 소켓 인스턴스가 생성되고 각 소켓은 1개의 클라이언트 역할을 수행합니다.  
여기서 클라이언트의 역할이란, 클라이언트가 서버로 채팅을 전송하는 것처럼  
서버에게 무엇을 전송하거나 원하는 작업을 요청할 수 있는 것을 의미합니다.  

<br/>

## 확장성 있는 더미 클라이언트 만들기
---

더미 클라이언트를 제작하면서 가장 신경 쓴 부분은 **확장성**입니다.  

이 프로젝트의 목적은 **서버 프로그램의 스트레스 테스트**로 다수의 클라이언트에서 요청한 작업을  
서버가 얼마나 잘 처리할 수 있는지 연습하기 위함에 있습니다.  

<br/>

```c++
void Client::communicate_server() {
	if (m_socket == NULL) {
		connect_to_server(SERVER_ADDR, PORT_NUM);
		return;
	}
	else if (id.empty()) {
		login_server();
		return;
	}
  
  // 클라이언트에서 서버로 채팅 전송
	C2S_SEND_CHAT_PACK send_pack;
  send_pack.size = sizeof(C2S_SEND_CHAT_PACK);
  send_pack.type = C2S_PACKET_TYPE::SEND_CHAT_PACK;
  wcscpy_s(send_pack.sentences, chat_sentences[target].c_str()); 
  send_packet(send_pack);

  S2C_RECV_CHAT_PACK recv_pack;
  recv_packet(recv_pack);
  wprintf(L"%s\n", recv_pack.sentences);
	return;
}
```

<br/>

위의 코드는 더미 클라이언트에서 작성된 첫 코드입니다.  
서버와 채팅만 주고 받기 위한 코드로 기능적으로는 아무런 문제가 없었습니다.  

그러나 더미 클라이언트가 **일단 채팅만 송수신하는 프로그램**으로 제작될 경우, 이후 개발 낭비가 될 수 있는 상황이 올 수 있었습니다.
데이터베이스 요청, 최적/최단경로 계산 요청 등 채팅 송수신 보다 **더 복잡한 요청**을 테스트 하고 싶을 때  
communicate_server() 라는 메서드에 내용이 길어지면서 점차 이해하기 어려워지고  
결국, 더미 클라이언트 자체를 새롭게 만들어야 할 수도 있습니다.  

따라서 저는 낭비될 수 있는 개발 시간을 절약하고자  
스트레스 테스트를 위한 **새로운 작업을 최소한의 코드 변경으로 추가**할 수 있도록 노력했습니다.  
위에서 예시를 든 코드의 경우 다음과 같은 코드로 수정중에 있습니다.  

<br/>

```c++
void Client::communicate_server(int key) {
	if (m_socket == NULL) {
		connect_to_server(SERVER_ADDR, PORT_NUM, key);
		return;
	}
	else if (id.empty()) {
		login_server();
		return;
	}
	
	int job_type = distr_job(eng);
	switch (job_type) {
	case JOB_TYPE::SEND_CHAT: {
		send_chatting();
		break;
	}
  
  /*
    다른 작업을 쉽게 추가할 수 있음
  */

	case JOB_TYPE::USER_LOGOUT: {
		request_logout();
		break;
	}
	}
}
```

<br/>

메서드만 아니라 C2S_SEND_CHAT_PACK, S2C_RECV_CHAT_PACK 구조체의 설계도  
상속이라는 OOP 특성을 활용해서 만들었습니다.  

상속을 사용하기 전 2개의 구조체는 아래와 같은 내용을 포함하고 있었습니다.  

<br/>

```c++
struct C2S_SEND_CHAT_PACK {
  short size;
  short type;
	wchar_t sentences[MAX_BUF_SIZE];
};

struct S2C_RECV_CHAT_PACK {
  short size;
  short type;	
	wchar_t sentences[MAX_BUF_SIZE];
};
```

<br/>

이전에 언급했듯 이 프로젝트는 서버의 스트레스 테스트가 목적입니다.  
때문에 채팅 외 여러 구조체들이 추가될 수 있었고 새롭게 구조체를 작성할 때 마다  
size와 type이라는 반복되는 변수가 들어갔습니다.  

두 변수는 네트워크 통신에 사용되는 구조체라면 반드시 선언될 것이므로  
이를 BASIC_PACK 이라는 **상위(부모) 구조체를 만들어 이를 상속받고록 설계**하였습니다.  

<br/>

```c++
struct BASIC_PACK {
	short size;
	short type;
};

struct C2S_SEND_CHAT_PACK : BASIC_PACK {
	wchar_t sentences[MAX_BUF_SIZE];
};

struct S2C_RECV_CHAT_PACK : BASIC_PACK {
	wchar_t sentences[MAX_BUF_SIZE];
};
```

<br/>

BASIC_PACK 구조체를 상속받은 덕분에 코드의 길이를 줄이고  
코드의 내용이 더 쉽게 읽힐 수 있다는 장점을 얻을 수 있었습니다.  

<br/>

## 서버 프로그래밍 시작 전 결정해야 하는 것들
---

컴퓨터의 입출력 시스템을 사용하는 대표적 프로그램 중 하나가 바로 서버 프로그램입니다.  
네트워크 통신은 결국 **네트워크 인터페이스라는 하드웨어 장치**를 통해  
로컬 컴퓨터가 아닌 외부의 컴퓨터와 통신하기 때문입니다.  

프로그램이 외부 컴퓨터와 통신하려면 네트워크 인터페이스를 직접 사용하는 것이 아니라  
운영체제에게 요청을 해야 사용할 수 있습니다.  
컴퓨터의 자원(하드웨어)을 **효율적으로 관리하는 것이 운영체제의 역할**이기 때문입니다.  

따라서 서버 프로그램을 설계하는 단계에서 결정해야 할 것이  
**어떤 운영체제에서 서버 프로그램을 운용할 것인지** 입니다.  

예를들어, CLI에 익숙한 개발자는 Linux Ubuntu server 22.04 운영체제를 사용해서  
리눅스의 EPoll()이라는 I/O 기술을 사용할 수 있으며  
GUI에 익숙한 개발자라면 Windows 11을 사용해 윈도우의 IOCP라는 I/O 기술을 사용할 수 있습니다.  

<br/>

![Linux server](/assets/img/OS/UbuntuServer.png){: width="600", height="600"}  
<center>[Oracle Virtual Box에서 실행한 Linux Ubuntu server 22.04]</center>

<br/>

![Linux Desktop](/assets/img/OS/UbuntuDesktop.png){: width="600", height="600"}  
<center>[Ubuntu 공식 사이트에서 제공하는 Ubuntu Desktop 화면]</center>

<br/>

![Windows11 Home Edition](/assets/img/OS/Windows11Home.png){: width="600", height="600"}  
<center>[현재 사용중인 Windows 11 Home Edition 화면]</center>

<br/>

저는 Windows 11을 개발 환경에서 사용하고 있기 때문에  
윈도우 운영체제에서 지원하는 WinSock과 IOCP를 사용했습니다.  

> 여기서는 자세히 설명하지 않지만 **컨테이너**라는 기술을 사용해서  
> Windows에서 Linux 프로그램을 실행하거나 Linux에서 Windows 프로그램을 실행할 수 있습니다.  
> 좀 더 자세히 알고 싶으신 분은 Docker라는 기술에 대해 추천드립니다.  

<br/>

서버를 운용할 운영체제가 결정 되었다면 다음은 **개발 언어**를 선택해야 합니다.  
프로그래밍 언어는 매우 다양한 언어가 존재합니다.  

언어들은 각자 **고유한 특징이 있거나 특정 상황에 특화된 언어**들이 존재합니다.  
범용성이 큰 C++, C#을 사용하거나 Node.js, Django처럼 특정 프레임워크를 사용할 수 있습니다.  

프로젝트를 진행하기 전 언어 선택에 있어 제가 고려했던 점은 아래와 같은 것들이 있었습니다.  

<br/>

1. 서버에서 대량의 소켓을 다루기 위한 실행 속도가 빠른 **컴파일 언어**  
2. 클라이언트들의 **요청을 빠르게 처리하기 위한 비동기 혹은 동시성**이 있는 언어  

<br/>

프로그래밍 언어는 인터프리터 언어와 컴파일 언어로 나눌 수 있습니다.  
인터프리터 언어는 작성된 코드를 실시간 번역처럼 실행할 때 마다 코드를 읽는 것이지만  
컴파일 언어는 작성된 모든 코드를 컴파일러가 기계어로 번역하고 이를 실행하는 방식입니다.  
모든 코드가 이미 기계어로 되어있는 컴파일 언어의 프로그램은  
실시간으로 번역하는 인터프리터 언어보다 빠르기 때문에 C++를 선택하게 되었습니다.  

<br/>

![비동기와 동시성](/assets/img/side_project_img/ChattingProgram/비동기와동시성.png){: width="600", height="600"}  
<center>[순차성, 동시성, 병렬성의 형태]</center>

<br/>

비동기 혹은 동시성이 필요했던 이유는 다수의 클라이언트가 보낸 요청을 빠르게 처리하기 위함에 있었습니다.  
예를들어, 서버에 1만개 이상의 클라이언트가 접속한 상태에서 수신한 요청들을  
동기적으로 1개의 쓰레드가 실행한다면 나중에 요청한 클라이언트는 자신보다 먼저 요청한 클라이언트들의 요청이  
서버에서 처리될 때까지 대기해야 하기 때문에 처리가 느려진다는 단점이 있습니다.  
따라서 클라이언트의 요청을 빠르게 처리할 수 있도록 비동기 혹은 동시성을 사용할 수 있는 C++를 선택하게 되었습니다.  

<br/>

이번 포스트에서는 채팅 프로젝트에서 사용될 더미 클라이언트에 대해서 간략히 살펴보고  
서버 프로그램을 만들기 전 결정해야할 것들에 대해서 알아보았습니다.  

다음 포스트에서는 서버 프로그램의 구조에 대해서 자세히 설명해 보겠습니다.  

<br/>

### 출처

출처: "순차성, 동시성, 병렬성의 형태 이미지", [블로그 BlaCk_Log](https://black7375.tistory.com/90)
