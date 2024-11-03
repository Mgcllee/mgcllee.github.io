---
title:  "[C#] 프로세스 클래스"
excerpt: "Diagnostics를 이용한 외부 프로그램"

categories: [Language, C#]
tag: [C#]

toc: true
toc_sticky: true
 
date: 2024-06-27
last_modified_at: 2024-06-27
---

C#에서 프로세스를 관리하기 위해서는 System.Diagnostics를 사용해  

작성한 코드의 외부에 존재하는 **로컬 프로그램**을 실행시킬 수 있습니다.  

# 실행방법

Process 클래스의 메서드인 Start를 사용하는 방법이 있습니다.  

<br/>

```c#
Process.Start("경로 + 실행 프로그램 이름.exe");
```

<br/>

이때, Start 메서드 내부에는 실행프로그램의 확장자인 **.exe**가 포함되어야 합니다.  

그리고 현재 C# 프로세스와 동일한 위치가 아닐 경우, 실행파일의 위치도 함께 제공해야 합니다.  

<br/>

```c#
Process ps = new Process();
ps.StartInfo.FileName = "경로 + 실행 프로그램 이름.exe";
ps.StartInfo.CreateNoWindow = true;
ps.Start();
```

<br/>

위의 코드처럼 객체만 생성하고 실행은 이후에 할 수 있습니다.  

# 참고자료
[MS Learn dotnet8.0 Diagnostics](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process?view=net-8.0)