---
title:  "[Algorithm] 이분 탐색"
excerpt: "빠르게 탐색하기"

categories:
  - Algorithm
tags:
  - [Algorithm]

toc: true
toc_sticky: true
 
date: 2024-06-14
last_modified_at: 2024-06-14
---

**이분 탐색 혹은 이진 탐색** 이라고 불리는 이 탐색 방법은  
정렬된 데이터가 배열에 저장되어 있을 때, 가장 높은 성능을 보여줍니다.  

먼저 탐색의 시작, 중간, 종료 인덱스를 각각 저장하는 변수를 생성합니다.  
그리고 탐색 값까지를 start, mid, end, target 이라는 변수에 저장한다고 하면  
배열은 다음 그림과 같이 그려집니다.

<br/>

![ex_01](/assets/img/binary_searching_img01.png)

<br/>

위의 그림처럼 정렬된 배열과 변수가 준비되었으면 탐색을 시작할 준비는 끝났습니다.  
탐색은 다음과 같은 순서로 진행됩니다.
(단, 아래 설명은 배열이 오름차순 정렬이 되어있는 상태라는 전제입니다.)

    1. mid 에 (start + end) / 2 값을 대입합니다.
    2. mid 값과 target 값을 비교합니다.
        a. target이 같으면 즉시 탐색을 종료합니다.
        b. target이 작으면 start 부터 mid - 1 까지 탐색
        c. target이 크면 mid + 1 부터 end 까지 탐색
    
<br/>

위의 그림에 맞춰 배열에 1부터 10까지 원소가 저장되어 있고  
이 배열에 8 이 존재하는지 확인해보겠습니다.

![ex_02](/assets/img/ex01.png)

<br/>

먼저 start와 end의 중간값인 5 ((10 + 1) / 2) 위치에 mid 를 표시합니다.  
그리고 mid와 target을 비교해 target이 크므로 start 위치값을 mid + 1 위치로 조정해줍니다.  

<br/>

![ex_03](/assets/img/ex02.png)

<br/>

그리고 다시 새로운 mid 값을 구해서 target과 비교합니다.  
새롭게 구한 mid 가 target 과 동일하므로 탐색을 종료합니다.  

이제 위의 순서도를 의사 코드로 표현하면 다음과 같습니다.

```c++

binary_search(start, end, target)
{
    if(start > end)
        return 0;

    mid = (start + end) / 2
    if(mid == target)
    {
        return mid;
    }
    else if(target < mid)
    {
        return binary_search(start, mid - 1, target);
    }
    else if(target > mid)
    {
        return binary_search(mid + 1, end, target);
    }
}

```

위의 코드에서 범위 값이 mid - 1, mid + 1 로 전달하는 이유는  
mid 를 비교하였을 때, 초과되거나 미만인 값이 target이므로 줄여서 보내는 것 입니다.  
또한 탐색 범위가 역전되는 현상을 방지하고자 함수의 실행과 동시에 start와 end를 비교해야 합니다.  

<br/>

이제 C++코드로 직접 구현해 보겠습니다.  
이분 탐색은 재귀문 혹은 반복문 모두 구현이 가능합니다.  
그러나 재귀문의 경우 **공간 복잡도가 높아질 수 있다는 단점** 이 있기 때문에  
가능하면 반복문으로 해결하는 것이 좋습니다.  

```c++

int target;

int binary_search_recursive(int start, int end)
{
    if(start > end)
        return 0;

    int mid = (start + end) / 2;

    if(mid == target)
    {
        return mid;
    }
    else if(target < mid)
    {
        return binary_search(start, mid - 1);
    }
    else if(mid < target)
    {
        return binary_search(mid + 1, end);
    }    
}

int binary_search_loop(int start, int end)
{
    int mid;
    while(start < end)
    {
        mid = (start + end) / 2;
        if(mid == target)
        {
            return mid;
        }
        else if(target < mid)
        {
            end = mid - 1;
        }
        else if(mid < target)
        {
            start = mid + 1;
        }
    }

    return -1;  // 범위 내 값이 없는 경우
}

```
