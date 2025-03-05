---
title:  "[SCG 프로젝트] 클라이언트 간 충돌 확인하기"
excerpt: ""

categories: [Side Project, SearchingCharacterGame]
tags: [SideProject, C&#47;C&#43;&#43;]

toc: true
toc_sticky: true
 
date: 2024-11-09
last_modified_at: 2024-11-09
--- 

> 이 포스트는 개인 프로젝트의 기록입니다.  

이번 개발내용은 [지난 포스트](https://mgcllee.github.io/posts/SCG_Sync_multi_user/)에서 언급한 클라이언트 간 충돌을 간단하게 구현했습니다.  

두 객체간 충돌 확인 코드는 아래와 같습니다.  

```c#
static bool check_collision(int client_ticket)
{
    for (int i = 0; i < clients.Count; i++)
    {
        if (i == client_ticket) continue;
        lock (lockObject)
        {
            if (clients[i].is_impact(clients[client_ticket]))
            {
                Console.WriteLine($"Client ({client_ticket}) and ({i}) impact!!!");
                return true;
            }
        }
    }
    return false;
}
```

![충돌판정](/assets/img/side_project_img/scg_collision_datarace.png){: width="600", height ="600"}  

두 클라이언트가 동시에 충돌하였지만 lock 없이 연산을 진행할 경우,  
연산이 완료되는 시점에 따라 충돌 후 위치값이 **예상했던 값과 다를 수 있습니다.**   

여기서 **lock**은 매우 중요한 역할을 합니다.  
충돌 체크 중, 다른 스레드에 의해서 객체가 이동할 수 있기 때문에 lock을 사용해서  
**유효한 데이터**를 사용하도록 작성하였습니다.  

<br/>

## 진행 상황

{% include embed/youtube.html id='z1uu7XnezeY' %}

현재 모습은 위 영상처럼  
두 객체간 충돌이 판정되고 다른 방향으로 이동할 수 있도록 구현하였습니다.  

다음 개발에서는 플레이어가 조종 가능한 객체를 추가할 예정입니다.  

<br/>   

## 소스코드 확인하기

[Github 주소](https://github.com/Mgcllee/SearchingCharacterGame/tree/9acace5c5c56fcc0932de7c9ecbb838badd8206e)