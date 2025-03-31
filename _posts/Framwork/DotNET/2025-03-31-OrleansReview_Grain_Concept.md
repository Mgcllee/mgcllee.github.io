---
title:  "[.NET Orleans] Grain의 개념"
excerpt: ""

categories: [.NET, Orleans]
tags: [.NET, Orleans, C#]

toc: true
toc_sticky: true
 
date: 2025-03-31
last_modified_at: 2025-03-31
---

## Orleans에서 Grain이란
---

MS Learn에서 소개하는 Orleans 속 Grain의 설명은 아래와 같습니다.  

* 모든 Orleans 애플리케이션의 기본 구성 요소는 Grain입니다.  
* 액터 모델 측면에서 Grain은 가상 액터입니다.  
* Grain은 사용자가 정의한 키값(고유값), 행동, 상태를 포함하는 엔티티입니다.  

<br/>

Grain은 Orleans의 중요한 요소 중 하나로 프로그래밍 모델 중 하나인 액터 모델 속 액터와 비슷한 역할을 수행합니다.  
Orleans에서는 액터 모델이 아닌 **가상 액터 모델**이라는 프로그래밍 모델을 사용하고 있습니다.  
액터 모델과 가상 액터 모델은 **독립적인 액터**가 상태를 갖고 다른 액터와 **메시지를 주고 받으며 상호작용**한다는 공통점이 있지만  
액터의 생명 주기에서 차이점이 있습니다.  

액터 모델의 경우, 프로그래머가 명시적으로 생성하고 삭제하며 액터의 생명 주기를 직접 관리하지만  
Orleans의 경우, Orleans 프레임워크가 직접 액터(Grain)의 생명 주기를 관리하기 때문에  
프로그래머가 직접 관리할 필요 없습니다.  
Orleans 프레임워크에서 액터(Grain)이 필요하다고 판단될 경우, **액터(Grain)을 활성화**시키고  
더 이상 사용되지 않으면 **액터(Grain)을 비활성화**시킵니다.  

가상 액터 모델의 경우, Orleans에서 직접 Grain을 관리하기 때문에 각 Grain을 구분할 정보가 필요합니다.  
여기서 사용되는 것이 **키값(고유값)**입니다.  

Grain은 설계 단계에서 어떠한 종류의 키값(고유값)을 소유할 것인지 표현해야 합니다.  
MS Learn에서 소개하는 Identity의 종류는 아래와 같은 것이 있습니다.  

* IGrainWithGuidKey: Grain에서 Guid 값을 키값으로 사용하기 위한 인터페이스  
* IGrainWithIntegerKey: Grain에서 Int64 값을 키값으로 사용하기 위한 인터페이스  
* IGrainWithStringKey: Grain에서 string 값을 키값으로 사용하기 위한 인터페이스  
* IGrainWithGuidCompoundKey: Grain에서 복합 키를 키값으로 사용하기 위한 인터페이스  
* IGrainWithIntegerCompoundKey: Grain에서 복합 키를 키값으로 사용하기 위한 인터페이스  

<br/>

각 인터페이스는 이름에서 확인할 수 있듯이 64bit 정수, 문자열, GUID 값 등을 사용해서 고유한 키값을 소유할 수 있도록 합니다.  
때문에 Grain은 반드시 위 5가지 인터페이스 중 한 가지의 인터페이스를 상속받아야 합니다.  

위 5가지를 키값 인터페이스라고 부른다면  
키값 인터페이스는 Grain 클래스에 직접 상속하는 것이 아니라 Grain 인터페이스에서 상속 받고  
Grain 인터페이스를 Grain 클래스에서 상속 받아 사용합니다.  

코드로 표현한다면 아래와 같습니다.  

```c#
// Grain 클래스에서 사용할 인터페이스
// 키값으로 String 사용
public interface IChatClientGrain : IGrainWithStringKey
{
    Task SendMessageAsync(string message);
    Task ReceiveMessageAsync(string message);
}

// 위에서 선언한 IChatClientGrain과 함께 Grain 클래스를 상속받음
public class ChatClientGrain : Grain, IChatClientGrain
{
    public override Task OnActivateAsync(CancellationToken cancellationToken) {
        // 이름처럼 Grain이 _활성화_될 때, 함께 실행되는 메서드
    }


    // SendMessageAsync, ReceiveMessageAsync 메서드를 구현

    
    public override Task OnDeactivateAsync(CancellationToken cancellationToken) {
        // 이름처럼 Grain이 _비활성화_될 때, 함께 실행되는 메서드
    }
}
```

<br/>

> 개인 학습을 기록하기 위한 글로 .NET Orleans를 배우면서 작성한 내용입니다.
> 오류나 틀린 부분이 있을 경우 언제든지 댓글 혹은 메일로 지적해주시면 감사하겠습니다!
