---
title:  "[C#] async 및 await를 사용한 비동기 프로그래밍"
excerpt: ""

categories: [Language, C#]
tag: [C#, async, await]

toc: true
toc_sticky: true
 
date: 2024-08-04
last_modified_at: 2024-08-04
---

## async 한정자

이번 포스트에서는 [비동기와 병렬](https://mgcllee.github.io/posts/Async_Parallel/)에서 설명한 비동기를 **C#에서 직접 사용**해 보겠습니다.  

C# 에서는 비동기 프로그래밍을 위해 async 한정자와 await 키워드가 존재합니다.  

C# 컴파일러가 async 한정자를 만나면 반환을 기다리지 않고 다음 다음 코드를 실행할 수 있도록 컴파일합니다.  

async 한정자를 사용하는 메서드는 반드시 반환 형식이 **Task, Task<TResult>, void** 형식이어야 한다는 제약 조건이 있습니다.  

Task, Task<TResult> 형식은 작업 완료까지 기다리는 메서드인 경우 사용하고  

void 형식은 실행하고 반환이 필요 없는 메서드에서 사용하면 됩니다.  

제가 작성한 [반환이 필요 없는 메서드의 예시](https://github.com/Mgcllee/RHTF/blob/234aa2e74fb2092397195033cce84a276055a685/RHTF_Server/RHTFMainServer/MainServer.cs#L66)로는 현재 진행중인 개인 프로젝트에서 C#을 이용한 비동기 함수가 있습니다.  

```c#
async void worker_thread()
{
    while(true)
    {
        // 비동기로 실행할 코드 내용
    }
}
```

이 함수의 경우, 서버의 시작부터 종료까지 **반복해서 클라이언트의 요청을 처리**하는 메서드이므로  

메서드의 반환 형식이 void로 되어있습니다.  

<br/>

## await 키워드

비동기 실행을 위해서는 async 한정자 외에도 **await 키워드**가 필요합니다.  

만약 async 한정자를 사용하는 메서드 내부에 await 키워드가 없는 경우  

해당 메서드는 호출자에게 제어권을 넘겨주지 않으므로 동기적으로 실행되게 됩니다.  

> 단, void 반환 형식에서는 await 키워드가 없어도 비동기로 실행됩니다.  

<br/>

```c#
private static async Task<Toast> ToastBreadAsync(int slices)
{
    // ...

    return new Toast(); // Task<TResult> 반환 형식
}

static async Task<Toast> MakeToastWithButterAndJamAsync(int number)
{
    var toast = await ToastBreadAsync(number);
    ApplyButter(toast);
    ApplyJam(toast);

    return toast;       // Task<TResult> 반환 형식
}

static async Task Main(string[] args)
{
    // ...
    
    var toastTask = MakeToastWithButterAndJamAsync(2);
    var toast = await toastTask;        // 실행 결과를 받습니다.
    Console.WriteLine("toast is ready");
    // ...
    
    Console.WriteLine("Breakfast is ready!");
}
```

<br/>

## 참고 및 코드 출처
[MS C# 공식 문서](https://learn.microsoft.com/ko-kr/dotnet/csharp/asynchronous-programming/)  

