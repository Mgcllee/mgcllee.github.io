---
title:  "[SideProject] 어째서 코드를 깨끗하게 만들어야할까"
excerpt: ""

categories: [Side Project, Fireboy Watergirl and Earthman]
tags: [SideProject, C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-09-25
last_modified_at: 2024-09-25
--- 

프로그램의 소스 코드는 한 번에 완벽하게 작성할 수 **없습니다.**  
팀 프로젝트에서는 한 팀원이 작성한 코드를 다른 팀원이 읽을 수 있고  
개인 프로젝트에서도 기능을 추가하거나 수정하다 보면 이전 코드를 읽을 때가 있습니다.  
여기서 이전 코드를 이해하는데 시간이 소모될 수 있고  
최악의 경우, 매번 같은 코드 이해하고자 처음부터 다시 읽어야 될 수 있습니다.  
심지어는 코드 작성자 조차 과거 자신이 작성한 코드를 이해하지 못한다면  
프로젝트 진행에 있어 심각한 문제를 초래할 수 있습니다.  
<br/>
<br/>

# 더러운 소스 코드
<br/>

깨끗한 코드가 "읽으면서 짐작했던 기능을 각 루틴이 그대로 수행하는 코드" 라면  
반대인 "읽으면서 짐작했던 기능과 **다르게** 수행하는 코드" 는 최악의 코드라고 할 수 있습니다.  
이 외에도 **일관성** 이 없으며, **가독성** 이 낮은 코드들도 최악의 코드라고 할 수 있습니다.  
<br/>
제가 작성한 최악의 코드 중 하나는 [main.cpp 코드가 700 줄](https://github.com/Mgcllee/Fireboy_Watergirl_and_Earthman/blob/a1a11df9377f31daf09633e1b7a54db3cfd0f314/FBWG_Server/main.cpp) 코드입니다.  
프로그램은 실행되지만 프로그래머의 역할 중 하나인 **유지 보수** 를 망각한 코드입니다.  
<br/>
700줄 짜리 코드의 문제점은 여러 가지가 존재하지만 가장 심각한 문제점은 다음과 같습니다.  

* 함수, 전역 변수, 지역 변수 등 모든 것에 일정한 **명명 규칙** 이 없다.  
* 객체 지향 언어(C++)를 사용했음에도 전역 함수와 전역 변수를 **빈번하게** 사용한다.  
* [특정 함수](https://github.com/Mgcllee/Fireboy_Watergirl_and_Earthman/blob/a1a11df9377f31daf09633e1b7a54db3cfd0f314/FBWG_Server/main.cpp#L212)에 역할이 너무 많다.  

대부분의 문제점은 결국 **깨끗하지 않은 코드** 가 원인입니다.  
따라서 이 문제점을 해결하고자 리팩토링을 진행하게 되었습니다.  
<br/>
<br/>

# 깨끗해지고 있는 코드
<br/>

```c++
#pragma once

#include "NetworkSettings.h"
#include "GameMaker.h"

int main() {
	NetworkSettings* network_settings = new NetworkSettings(ULONG(INADDR_ANY), USHORT(PORT_NUM));

	GameMaker* game_maker = new GameMaker();
	array<Client, 3>* clients = game_maker->get_clients();

	SOCKET* listen_socket = network_settings->get_listen_socket();
	ClientAccepter client_accepter(listen_socket);
	while (false == client_accepter.accept_all_client(clients)) {
		network_settings->reset_listen_socket();
	}

	game_maker->run_game();
	game_maker->cleanup_game();

	network_settings->close_listen_socket();
	return 0;
}
```

위 소스 코드는 리팩토링을 진행 중인 [main.cpp 코드](https://github.com/Mgcllee/Fireboy_Watergirl_and_Earthman/blob/5bad3a88d6310c6a9802cc99020f6bc45da6c79b/FWE_Server/main.cpp) 입니다.  
이 코드가 완벽하고 깨끗한 코드라 장담할 수 없지만 이전 코드와 비교한다면  
기능을 유지하면서 700 줄에서 23 줄로 줄이고 **가독성** 을 최대한 높이고자 하였습니다.  
<br/>

```c++
// 서버 소스 코드
class PacketSender {
public:
	PacketSender(array<Client, 3>* member);

	virtual void send_packet(Client& client, void* packet);
	virtual void sync_send_packet(void* packet);

	array<Client, 3>* clients;
};

class StageUpdatePacket : public PacketSender {
public:
	StageUpdatePacket(array<Client, 3>* member);

	void sync_send_packet(void* packet) override;
};

class ClientMovePacket : public PacketSender {
public:
	ClientMovePacket(array<Client, 3>* member);

	void sync_send_packet(void* next_position) override;
};
```

FWE(Fireboy Watergirl and Earthman) 프로젝트는 단일 서버와 3개의 클라이언트가  
소켓으로 통신해 즐길 수 있는 게임입니다.  
따라서 게임에 사용될 여러 패킷들이 존재하며 종류에 따라 수행하는 동작도 다릅니다.  
<br/>
모든 동작을 한 함수에서 수행할 경우 함수 외부에서 보았을 때, **동작을 예상하기 어렵습니다.**  
따라서 저는 클래스의 상속과 가상 함수로 **예상되는 코드** 를 작성하고자 하였습니다.  
<br/>
서버의 소스 코드 이므로 PacketSender 는 서버에서 패킷을 송신하는 클래스이고  
이 클래스의 자식 클래스들은 모두 서버에서 클라이언트로 송신하는 클래스들 입니다.  
코드에서 ClientMovePacket 클래스는 클라이언트(유저)가 움직였을 때 사용되는 클래스이며  
sync_send_packet 메서드는 모든 클라이언트와 동기화(sync) 하는 동작을 합니다.  
<br/>
<br/>

# 앞으로 해야할 일
<br/>

로버트 C.마틴의 책 CleanCode에서 [유지 보수는 결코 끝나지 않는다.](https://mgcllee.github.io/posts/CleanCode_02/) 라는 말처럼  
지금 작성한 코드가 나중에도 보기 좋다는 보장은 어디에도 없습니다.  
<br/>
앞으로도 꾸준하게 코드를 유지 보수하면서 깨끗한 코드에 근접하고자 합니다.  
