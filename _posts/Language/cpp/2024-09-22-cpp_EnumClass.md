---
title:  "[C++] enum 과 enum class"
excerpt: ""

categories: [Language, C&#47;C&#43;&#43;]
tags: [C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-09-22
last_modified_at: 2024-09-22
---

## Enum 이란?

C++ 는 정수형, 실수형, 문자형 등 여러 자료형을 지원하고 있지만  
사용자에 따라 C++ 에서 지원하지 않는 자료형이 필요할 수 있습니다.  
이를 위해서 C++ 에서는 enum 이라는 자료형을 지원합니다.  
<br/>

enum 은 enumerate(열거하다) 라는 단어의 약자로 여러 값들을 열거하고 정수형으로 사용할 수 있습니다.  
컴파일러는 enum 의 멤버들을 상수로 판단하고 컴파일을 진행합니다.  
<br/>

```c++
enum TYPE {
    IDLE,
    MOVE,
    ATTACK,
    JUMP
}
```

enum 타입은 위의 코드처럼 문자열로 각 멤버를 선언할 수 있고  
첫 번째 멤버에 값을 넣지 않는다면 0부터 1씩 차례대로 대입됩니다.  
> IDLE = 0, MOVE = 1, ATTACK = 2, JUMP = 3  

만약 IDLE 혹은 다른 멤버에 값을 대입하고 싶을 경우 아래 코드처럼 작성하면 됩니다.  

```c++
enum TYPE {
    IDLE = 10,
    MOVE,       // 대입값이 없으므로 11
    ATTACK,     // 대입값이 없으므로 12
    JUMP = 40
}
```

<br/>

## enum 의 특징

* enum 타입은 컴파일 타임에 정수형으로 변환  
* enum 타입에는 enum 값 이외 값 대입 불가  
* 관련된 값을 묶음으로서 코드 가독성 상승  
* 서로 다른 enum 이라도 **동일한 이름의 멤버가 있어선 안된다.**  
<br/>

enum 은 여러 특징이 존재하지만 동일한 이름의 멤버가 존재하지 못 한다는 것은  
제게 있어서 치명적인 단점이였습니다.  
  
예전에 진행한 [Fireboy watergirl and earthman 프로젝트](https://github.com/Mgcllee/Fireboy_Watergirl_and_Earthman/blob/a1a11df9377f31daf09633e1b7a54db3cfd0f314/FBWG_Server/protocol.h) 리팩토링 중 패킷의 타입을,  
#define 키워드로 모두 선언했지만 **코드 가독성** 이 저하되고 패킷 **확장에 불리** 하다고 판단하여  
enum 으로 패킷 타입을 묶고자 하였습니다.  
<br/>

```c++
enum PACKET_TYPE_S2C {
	Loading = 0,
	ChangeStage,
	SelectRole,
	ChangeRole,
    
    // ...

	Endout,
	PlayerOut
};

enum PACKET_TYPE_C2S {
	ChangRole = 0,
	SelectRole,
	Move,
	Endout
};
```

C2S(Client to server)타입과 S2C(Server to client)타입으로 분리하여 위의 코드처럼 수정하였습니다.  
  
그러나 문제는 **SelectRole, Endout** 과 같이 C2S, S2C 모두에 선언되어 있는 패킷 타입이 문제였습니다.  
enum 의 이름은 다르더라도 중복되는 이름은 허용되지 않기 때문에  
위와 같은 코드는 **컴파일이 불가능** 한 코드입니다.  
  
따라서 이 문제를 해결하고자 namespace 사용 등 여러 방법을 찾게 되었고  
**가독성과 확장성** 을 높일 수 있는 방법으로 C++11 에서 추가된 enum class 을 사용하기로 결정하였습니다.  
<br/>

## enum class 란?

enum class 는 C++11 에서 채택된 기술로 중복을 허용하지 않는 enum의 단점이 보완 되었습니다.  
enum class 는 기존 enum 키워드 뒤에 class 를 붙여 사용할 수 있고  
호출할 때는 기존 enum 과 동일하게 사용하면 됩니다.  
<br/>

## enum class 특징

```c++
enum class TYPE {
    IDLE,
    MOVE
};

if(TYPE::MOVE == 1) {   // 오류
    cout << "MOVE!";
} else {
    cout << "Can't Move...";
}
```

enum 은 암시적 형변환을 사용할 수 있었지만 enum class 에서는 **불가능** 합니다.  

```c++
enum class TYPE {
    IDLE,
    MOVE
};

if(static_cast<int>(TYPE::MOVE) == 1) {   // 변환 성공
    cout << "MOVE!";
} else {
    cout << "Can't Move...";
}
```

따라서 위 코드처럼 **명시적 변환** 을 사용해야 합니다.  
<br/>

<br/>

> 이 포스트는 개인적으로 공부하며 기록한 내용으로 이후 변경될 수 있습니다.  