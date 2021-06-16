---
title: "Heap Alignment"
tags: [memory, "pwnable theory"]
categories: [System]
---

# Index

1. [Heap Memory Offset](#heap-memory-offset)
2. [Heap Alignment](#heap-alignment)
3. [Chunk](#chunk)
4. [Reference](#refernce)

* * *

## Heap Memory Offset

Heap overflow를 학습하던 중 주소와 관련하여 의문을 가지게 되었습니다. 로컬 바이너리 파일을 분석하여 얻은 payload와 실제 flag를 획득할 수 있는 payload에 차이가 있었기 때문입니다.

로컬에서 분석하였을 때는 현재 heap의 data 영역의 시작에서 다음 heap의 데이터 영역까지 48 Byte 떨어져 있는 것으로 분석하였으나 실제로 flag를 획득하기 위해서는 40 Byte 떨어진 것으로 계산하여야 정상적으로 shell을 획득할 수 있었습니다.

해당 내용에 대한 질문의 답변으로는 다음과 같았습니다.

> Heap은 0x10으로 정렬된 부분에 할당되기 때문에 offset의 차이가 있을 수 있다.

해당 내용을 확실하게 알기 위해 관련된 자료를 추천받아 이해한 내용을 토대로 다시 정리해보았습니다.

## Heap Alignment

먼저, C99 states의 section 7.20.3 에서 아래와 같이 있음을 확인할 수 있었습니다.

> The pointer returned if the allocation succeeds is suitably aligned so that it may be assigned to a pointer to any type of object.

위 내용에 대해 "*정상적으로 할당에 성공하였을 경우 포인터에 대한 정렬의 과정을 거치고 이 과정 중 0x10 단위로 정렬된 부분에 할당된다.*" 라는 뜻으로 이해하였습니다. 해당 내용에 대한 검증을 위해 간단한 코드를 작성하여 출력 결과를 확인해보았습니다.

```c
#include <stdio.h>
#include <malloc.h>

int main()
{
        char *heap1 = (char *)malloc(13);
        char *heap2 = (char *)malloc(27);
        char *heap3 = (char *)malloc(34);

        printf("heap1 Address is: %p\n", heap1);
        printf("heap2 Address is: %p\n", heap2);
        printf("heap3 Address is: %p\n", heap3);

        return 0;
}
```

더 확실한 검증을 위해 heap의 할당 크기에 차이를 주었으며 실행 결과는 다음과 같았습니다.

```
heap1 Address is: 0x5604added2a0
heap2 Address is: 0x5604added2c0
heap3 Address is: 0x5604added2f0
```

## Chunk

추천받은 내용 중 chunk와 관련된 내용도 찾을 수 있었습니다. Chunk의 경우 user data 외에도 *prev\_size*, *size* 부분을 **Header** 라고 칭하는데 이는 64bit 기준으로 0x10 Byte의 크기를 가집니다.

따라서 chunk의 size는 `요청한 크기 + header의 크기`가 되며 malloc을 통해 할당을 할 경우 포인터 변수에 저장되는 값은 User data의 시작 주소를 의미한다고도 합니다.

또한 chunk는 0x10 Byte 단위로 할당되기 때문에 `malloc(0x10)` 을 수행하였더라도 chunk header와 user data의 크기를 합치면 최소 0x20 Byte의 크기를 가집니다.

*prev\_size* 영역의 경우 경우에 따라 이전 chunk의 data 영역으로 사용되는 경우도 있다고 하였습니다. 만약 `malloc(0x38)`을 수행하였을 경우 `malloc(0x30)`을 우선 실행 후 다음 chunk의 *prev\_size* 영역을 자신의 data 영역으로 활용합니다.

그러나 여기서 1 Byte가 더 추가된 `malloc(0x39)`의 경우 `malloc(0x40)`과 같이 동작하여 공간을 확보합니다.

* * *

## Reference

- <https://opentutorials.org/module/4290/27219>

- <https://stackoverflow.com/questions/5061392/aligned-memory-management>
