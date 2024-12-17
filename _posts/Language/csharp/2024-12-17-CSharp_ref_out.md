---
title:  "[C#] 함수와 ref, out 키워드"
excerpt: ""

categories: [Language, C#]
tag: [C#]

toc: true
toc_sticky: true
 
date: 2024-12-17
last_modified_at: 2024-12-17
---

> 개인 학습을 기록한 내용을 담고 있어 **추후 수정**될 수 있습입니다.  

<br/>

## 함수  
---

C 언어 계열에서는 함수를 호출하면서 함수가 사용할 데이터를 넘겨줄 때, 매개변수를 사용합니다.  
이 매개변수를 넘겨주는 방법은 크게 **Call by Value**, **Call By Reference** 2가지 있습니다.  
C, C++, C# 모두 두 가지 방법을 사용하고 있지만 사용하는 방법은 각자 다릅니다.  

이번 포스트에서는 C# 에서 함수에 매개변수를 전달할 때 Call by Value 방법으로 전달하는 방법과  
Call by Reference 방법으로 전달하는 방법에 대해 알아보겠습니다.  

<br/>

## Call By Value
---

이 방법은 C 언어 계열에서 **기본적(Default)**으로 적용되는 매개변수 전달 방법입니다.  

```c#
public class Program
{
	public static void Main()
	{
		int number = 1;
		add_number(number);
		Console.WriteLine(number); // 1
	}
	
	static void add_number(int number) {
		number = number + 1;	
	}
}
```

<br/>

이 코드에서 add_number() 함수는 매개변수로 받은 number 를 1 만큼 증가시키는 작업을 수행합니다.  
그리고 다시 Main 함수로 돌아와 number 를 출력하면 add_number() 함수에서 수행한 작업 결과가 아닌  
작업하기 전 내용인 값 1 이 출력됩니다.  

이러한 이유는 add_number() 함수가 number 를 전달받을 때, **Call by Value**로 전달받았기 때문입니다.  

함수가 Call by Value 로 데이터를 전달 받으면, 함수에서는  
**main 함수에 있는 number가 아닌 add_number 함수의 고유한 number를 생성**하고 그 자리에  
**전달받은 값만 가져와 고유 number에 초기화**합니다.  
즉, main 함수의 number 변수와 add_number 함수의 number 변수는  
전혀 다른 메모리 공간(주소)에 존재하는 데이터라는 의미입니다.  

그러나 함수의 이름은 add_number 로 전달 받은 number 를 1 증가시키는 작업을 수행하고 싶습니다.  
여기서 필요한 것이 Call by Reference 라는 기술입니다.  

<br/>

## Call by Reference
---

<br/>

### ref 키워드
---

Call by Reference 는 함수에게 원본 데이터의 접근방법을 알려주는 방법입니다.  

```c#
public class Program
{
	public static void Main()
	{
		int number = 1;
		add_number(ref number);
		Console.WriteLine(number); // 2
	}
	
	static void add_number(ref int number) {
		number = number + 1;	
	}
}
```

<br/>

call by value 방법을 사용한 코드와 차이점은 **ref** 라는 키워드의 존재 유무입니다.  

ref 는 Reference 의 앞 3글자를 가져와 사용하는 키워드로  
Reference 라는 의미가 '참고(참조)'인 것 처럼 원본 데이터를 참고한다는 의미입니다.  

call by value 에서는 main 함수의 number 변수와 add_number 함수의 number 변수가  
서로 다른 메모리 공간을 갖고 각자 값을 갖는 방법이었다면  
call by reference 는 main 함수의 number 변수를 add_number 함수의 number 변수가  
**동일한 메모리 공간을 공유함으로서 동일한 데이터에 접근**할 수 있습니다.  

<br/>

## C++, C#의 레퍼런스와 C 의 포인터
---

> 이 설명은 C++ 포인터와 차이점을 설명하기 위한 내용으로  
> 포인터에 대한 개념이 없으실 경우, 넘겨도 괜찮습니다.  

C# 이 ref 키워드를 사용해 원본 데이터를 참조한다면  
C++ 에서는 & 키워드를 사용해 원본 데이터를 참조할 수 있습니다.  
두 키워드는 함수에서 추가적인 메모리를 사용하지 않고 원본 데이터를 **직접 참조**합니다.  

그러나 C 의 포인터는 포인터 변수를 위한 메모리 공간을 할당하고  
변수에 저장된 메모리 주소를 사용해 다른 메모리 공간에 접근합니다.  

3가지 기술 모두 원본 데이터에 접근이 가능하다는 특징이 있지만  
위 설명처럼 메모리 할당 여부에 따라 차이점이 존재합니다.  

| **특징**               | **C 언어: 포인터**                 | **C++: 레퍼런스**                | **C#: ref**                      |
|-------------------------|-----------------------------------|----------------------------------|----------------------------------|
| **메모리 할당**         | 별도의 메모리 공간(주소 저장용)    | 별도 메모리 할당 없음            | 별도 메모리 할당 없음            |
| **사용 방식**           | 변수의 주소를 저장하고 간접 참조   | 원본 변수를 직접 참조            | 원본 변수를 직접 참조            |
| **표현 방식**           | `int* ptr = &value;`              | `int& refValue = value;`         | `ref int refValue = ref value;`  |
| **메모리 접근**         | `*ptr`로 간접 접근 (dereference)  | 변수명만 사용해서 접근           | 변수명만 사용해서 접근           |
| **초기화 필요 여부**     | 초기화 없이 선언 가능 (`int* p;`) | 반드시 초기화 필요               | 반드시 초기화 필요               |
| **재할당 가능 여부**     | 가능 (`ptr = &otherValue;`)       | 불가능 (초기 참조 대상 고정)      | 불가능 (초기 참조 대상 고정)      |
| **Null 가능 여부**       | `NULL` 또는 유효하지 않은 주소 가능 | `nullptr` 또는 레퍼런스는 불가능  | 유효한 변수만 참조 가능           |
| **주소 연산**           | 가능 (`ptr++`, `ptr + 1`)         | 불가능                          | 불가능                          |
| **간접 참조 오버헤드**   | 존재 (포인터 주소를 따라감)       | 거의 없음                       | 거의 없음                       |
| **안전성**              | 낮음 (널 포인터, 잘못된 주소 접근) | 높음 (항상 유효한 변수 참조)      | 높음 (항상 유효한 변수 참조)      |
| **활용 예시**           | 동적 메모리 할당, 배열, 함수 포인터 | 함수 호출 시 매개변수 참조 전달   | 함수 호출 시 매개변수 참조 전달   |

<br/>


### out 키워드
---

out 키워드는 ref 키워드와 매우 유사합니다.  
단, ref 키워드와 다르게 호출된 함수 내부에서 **쓰기(Write)**를 강제 한다는 특징이 있습니다.  

ref 키워드를 사용한 변수는 해당 함수 내부에서 전혀 사용하지 않고 함수를 종료해도 문제가 없습니다.  
즉, 함수 내부에서 오직 읽기만 실행할 수 있기 때문에  
함수에서 ref 키워드를 사용하려면 전달하기 전, 반드시 변수를 초기화를 해야 합니다.  

반면, out 키워드는 첫 설명처럼 쓰기를 강제한다는 특징으로 **변수가 초기화된다는 보장**이 있어  
함수에 전달하기 전, 초기화를 하지 않아도 사용이 가능합니다.  

![C# error](/assets/img/CSharp/out_keyword_error.png){: width="650", height="650"}  

<br/>

out 키워드를 사용했음에도 함수 내부에서 write 작업이 없다면 컴파일 오류가 발생할 수 있습니다.  

<br/>

