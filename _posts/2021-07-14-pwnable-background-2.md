---
title: "Background Theory in Pwnable - Pointer"
tags: ["pwnable theory"]
categories: ["System"]
---

# Index

1. [Episode 2](#episode-2)

* * *

## Episode 2

여러 문제를 접할수록 생각보다 pointer를 사용하는 문제가 많았습니다. Assembly 만으로 확인할 때에는 크게 문제가 되지 않았지만 막상 C 언어로 작성된 것을 보니 매우 복잡하여 episode 2는 이에 대한 내용을 정리하기로 하였습니다.

C 언어에서 pointer를 사용하는데 있어서 가장 중요한 단어는 `*` 문자입니다. 이를 통해 pointer 변수의 선언, 값 할당 등에 사용할 수 있습니다. 몇 가지 예시를 들면 다음과 같습니다.

```c
int* numPtr;    //int type pointer인 numPtr을 생성
char *chPtr;    //char type pointer인 chPtr 생성
```

위 간단한 예시를 통해 확인할 수 있듯이 '**자료형의 뒤**' 혹은 '**변수명의 앞'에 `*` 문자를 추가하여 pointer 변수 선언이 가능합니다.

다른 자료형과 마찬가지로 casting을 위해서는 아래와 같은 형태를 사용할 수 있습니다.

```c
(int *)
(char *)
(long int *)
등등..
```

Pointer type의 출력을 위해서는 `%p` directive를 사용하면 됩니다.

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
        int *ptr = malloc(sizeof(int));

        printf("Pointer: %p\n", ptr);

        return 0;
}

root@3218f793af99:~/Code# ./tmp
Pointer: 0x5636e298c260
```

Pointer 변수에 일반 변수의 주소를 저장하기 위해서는 다음과 같이 지정해야 합니다.

```c
numPtr = &a;
```

반대로 pointer 변수에 일반 변수와 같이 값을 할당하기 위해서는 "**역참조**"를 사용하여야 합니다.

```c
*numPtr = 123;
```

이 글을 작성하게 된 계기이자 이해하는데 많은 시간이 걸린 것은 아래 내용이었습니다.

```c
*(long int *)*ptr = *(long int *)(ptr+1);
```

위 내용을 assembly code로 보면 다음과 같습니다.

```
   0x00000000000011ba <+85>:	mov    rax,QWORD PTR [rbp-0x10]
   0x00000000000011be <+89>:	mov    rax,QWORD PTR [rax]
   0x00000000000011c1 <+92>:	mov    rdx,rax
   0x00000000000011c4 <+95>:	mov    rax,QWORD PTR [rbp-0x10]
   0x00000000000011c8 <+99>:	mov    rax,QWORD PTR [rax+0x8]
   0x00000000000011cc <+103>:	mov    QWORD PTR [rdx],rax
```

먼저 사전에 알아둘 정보 몇 가지가 있습니다.

- ptr 변수의 경우 long int pointer type으로 선언됨.
- ptr 변수에 1024 size의 long int pointer 메모리를 할당(malloc)
- 이후 ptr 변수에 read 함수로 입력받음

괄호를 기준으로 앞쪽 부분에 대해 해석하면 다음과 같습니다.

1. ptr의 역참조 즉, ptr 변수에 저장된 값을 사용
2. (long int *)에 의해 long int형의 pointer로 casting
3. casting한 *ptr 값을 다시 역참조

이를 종합하면 ptr 값에 저장된 값을 pointer type으로 casting하여 해당 값을 주소로 하는 공간을 사용하겠다는 의미입니다.

괄호를 기준으로 뒷쪽 부분은 다음과 같이 해석할 수 있습니다.

1. ptr+1의 값을 사용. But, long int pointer type이기 때문에 +1은 주소 값을 기준으로 +8과 같은 결과
2. 해당 값을 long int pointer로 casting
3. 마찬가지로 casting한 값을 역참조

최종적으로 ptr의 주소로부터 8 만큼 떨어진 공간의 값을 long int pointer로 casting 후 이 주소를 역참조 하는 값을 왼쪽에 할당하겠다는 의미입니다.
