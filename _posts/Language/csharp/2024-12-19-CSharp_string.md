---
title:  "[C#] 문자열(string)과 내부 메서드"
excerpt: ""

categories: [Language, C#]
tag: [C#]

toc: true
toc_sticky: true
 
date: 2024-12-19
last_modified_at: 2024-12-19
---

> 개인 학습을 기록한 내용을 담고 있어 **추후 수정**될 수 있습입니다.  

<br/>

## 문자열  
---

문자열은 C# 에서 여러 문자를 이어붙인 자료형이라고 했습니다.  
그러나 단순히 이어붙인 것 뿐만 아니라 클래스처럼 string 내부에 여러 **메서드**를 갖고 있습니다.  

이번 포스트에서는 string의 사용과 내부 메서드들에 대해 알아보겠습니다.  

<br/>

## String 선언하기
---

string 의 선언은 C# 의 다른 기본 자료형과 동일한 방법을 사용합니다.  

```c#
public class Program
{
	public static void Main()
	{
		string str;
		str = "ABC";
		Console.WriteLine(str);
	}
}
```

string 은 선언하면서 초기화를 하지 않아도 컴파일이 가능합니다.  
그러나 위 코드에서 'str = "ABC"' 라는 코드가 없다면 그 다음 줄에서  
출력하는 str에 오류가 발생할 수 있습니다.  

따라서 가능하면 선언과 함께 초기화하는 것이 권장됩니다.  

<br/>

## string 내부 메서드
---

string 은 내부에 여러 메서드를 갖고 있으며 이를 활용해 문자열을 조작할 수 있습니다.  
대표적인 메서드로 **변형, 분할, 부분 문자열 찾기, 포맷** 등 다양한 메서드들이 존재합니다.  

<br/>

### 부분 문자열 찾기 - Contains
---

```c#
str = "Hello World";

if(str.Contains("Hello")) {
	Console.WriteLine(str);
}
else {
    Console.WriteLine("Nothing");
}
```

<br/>

contains 메서드는 contains 메서드를 호출한 문자열에서 전달받은 **부분 문자열이 존재하는지** 알려주는 메서드입니다.  
즉, 원본 문자열 내에 원하는 부분 문자열의 존재 여부만 파악하기 때문에 반환 값은 True, False 중 하나입니다.  

<br/>

### 부분 문자열 찾기 - IndexOf
---

```c#
public class Program
{
	public static void Main()
	{
		string str = "Hello World";
		Console.WriteLine(str.IndexOf("World")); // 6
	}
}
```

이 메서드는 전달 받은 문자열 혹은 문자가 몇 번째부터 존재하는지 인덱스 번호를 알려줍니다.  
모든 문자열의 첫 번째 인덱스는 0번부터 시작하므로 위의 결과는 6 이 됩니다.  

여기서 주의할 점은 0 번째 부터 순차적으로 검사하기 때문에 조건이 부합되는 첫 번째 인덱스만 출력됩니다.  

따라서 아래의 코드에서도 다른 인덱스가 아닌 6이 출력됩니다.  

```c#
public class Program
{
	public static void Main()
	{
		string str = "Hello World World World World";
		Console.WriteLine(str.IndexOf("World")); // 6
	}
}
```

<br/>

### 변형 - '+' 연산자
---

```c#
public class Program
{
	public static void Main()
	{
		string left_str = "Hello";
        string right_str = "World";
		Console.WriteLine(left_str + right_str); // Hello World
	}
}
```

문자열에서 덧셈 연산자는 두 문자열을 붙이는 작업을 수행합니다.  
위 코드에서는 left_str 의 오른쪽에 right_str 을 붙여 결과적으로 "Hello World" 라는 문자열이 출력됩니다.  

<br/>

### 변형 - ToLower
---

```c#
public class Program
{
	public static void Main()
	{
		string str = "HELLO WORLD";
		Console.WriteLine(str.ToLower()); // hello world
	}
}
```

이 메서드는 문자열 내부에 있는 대문자를 소문자로 변환시켜주는 역할을 합니다.  
이미 소문자인 경우, 그 문자에 대해서는 아무것도 하지 않고  
대문자에 대해서만 소문자로 변경시킵니다.  

<br/>

### 변형 - ToUpper
---

```c#
public class Program
{
	public static void Main()
	{
		string str = "hello world";
		Console.WriteLine(str.ToUpper()); // HELLO WORLD
	}
}
```

이 메서드는 위의 ToLower() 과 반대로 소문자를 대문자로 변경시켜줍니다.  

<br/>

### 변형 - Replace
---

```c#
public class Program
{
	public static void Main()
	{
		string str = "Hello World";
		Console.WriteLine(str.Replace("World", "Korea")); // Hello Korea
	}
}
```

이 메서드는 2개의 문자열(문자)를 넘겨줍니다.  
첫 번째 문자열은 원본 문자열에서 **교체될 대상**  
그리고 두 번째 문자열은 원본 문자열에 **새롭게 들어갈 문자열**입니다.  

<br/>

### 분할 - Split
---

```c#
public class Program
{
	public static void Main()
	{
		string str = "Hello Beautiful World Beautiful Korea";
		var str_arr = str.Split("Beautiful");
		foreach(string s in str_arr) {
			Console.WriteLine(s);
		}
	}
}
```

이 메서드는 주어진 문자열(문자)를 기준으로 문자열을 분할하는 작업을 수행합니다.  
그리고 분할한 결과를 **string[]** 배열로 반환합니다.  

위의 결과는 공백을 포함하여 아래 처럼 여러 줄에 나뉘어 출력됩니다.  

> Hello 
>  World 
>  Korea

<br/>

Split 메서드는 문자열만 받는 것이 아니라 다양한 형태의 자료형을 받을 수 있습니다.  

```c#
// 문자 넘기기
string str = "Hello World";
var str_arr = str.Split(' ');
foreach(string s in str_arr) {
	Console.WriteLine(s);
}

// 문자 배열 넘기기
string str = "number:1,2,3,4,5\t";
var str_arr = str.Split(new char[]{':', ',', '\t'});
foreach(string s in str_arr) {
	Console.WriteLine(s);
}


// 문자열 배열 넘기기
string str = "number:1,2,3,4,5\t";
var str_arr = str.Split(new string[]{":", ",", "\t"}, System.StringSplitOptions.RemoveEmptyEntries);
foreach(string s in str_arr) {
	Console.WriteLine(s);
}

// [2 번째 인수]
// System.StringSplitOptions.RemoveEmptyEntries : 리턴 값에 빈 문자열 포함하지 않음
// StringSplitOptions.None : 리턴 값에 빈 문자열이 포함됨  
```

<br/>

### 분할 - Substring
---

```c#
public class Program
{
	public static void Main()
	{
		string str = "Hello World";
		Console.WriteLine(str.Substring(7));    // orld
        Console.WriteLine(str.Substring(4, 3)); // o W
	}
}
```

Substring 메서드는 2 가지 사용 방법이 존재합니다.  
숫자를 1개만 넘겨줄 경우, 해당하는 인덱스부터 문자열의 끝(마지막 인덱스)까지를 반환하고  
숫자를 2개 넘겨줄 경우, 첫 번째 인자의 인덱스부터 두 번째 인자만큼의 개수를 반환합니다.  

<br/>

### 포맷 - string.Format
---

여러 변수와 함께 여러 문자열을 합쳐 출력하려면 아래와 같은 방법이 있습니다.  

```c#
int age = 20;
int curr_year = 2024;
string left_str = "This year is ";
string middle_str = " and I am ";
string right_str = "years old.";

Console.WriteLine(left_str + curr_year + middle_str + age + right_str);
// This year is 2024 and I am 20years old.
```

그러나 이와 같은 방법은 문자열이 복잡해지고 변수가 많아질수록 코드 가독성이 떨어지는 단점이 존재합니다.  

따라서 이를 해결할 수 있는 방법이 string.Format 이라는 함수입니다.  

```c#
int age = 20;
int curr_year = 2024;
string str = string.Format("This year is {0} and I am {1} years old.", curr_year, age);
Console.WriteLine(str);
```

이 함수는 문자열의 형식을 미리 작성하고 이후에 변수를 삽입하는 방법입니다.  
문자열 속 변수를 넣을 자리에는 괄호 '{_}' 를 사용해서 표시하고  
괄호 내부에는 몇 번째 인수를 넣을지 숫자로 표시합니다.  

<br/>
