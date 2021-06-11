---
title: "Memory Leakage"
tags: [memory, pwnable theory]
categories: [System]
---

# Index

1. [Memory Leakage 취약점](#memory-leakage-취약점)
2. [인접 메모리 유출](#인접-메모리-유출)
	- [Case 1](#case-1)
3. [Reference](#reference)

* * *

## Memory Leakage 취약점

Memory Leakage는 프로그램 상에서 더 이상 필요하지 않은 메모리에 대한 잘못된 관리 혹은 BOF를 통한 허용되지 않은 인접 메모리 혹은 특정 주소의 값을 유출할 수 있는 취약점을 의미합니다.

C 언어의 경우 `malloc()`, `free()` 등의 명령을 이용하여 **heap 영역**에 메모리의 할당과 해제 등을 수행할 수 있는데, 이때 할당한 메모리에 대해 해제를 해주지 않을 경우 발생할 수 있는 대표적인 취약점입니다.

## 인접 메모리 유출

인접 메모리 유출은 문자열 출력 함수를 사용하여 변수에 값을 출력할 때 각각의 변수 사이에 Null Byte가 없이 이어져 있어 의도되지 않은 다른 변수의 값이 출력되는 것입니다.

C 언어를 기준으로 생각해보면 `%s` 지시자를 사용하여 문자열을 출력할 때 Null Byte를 만날 경우 출력을 종료하는데 이때 Overflow와 같은 이유로 종단의 Null Byte가 다른 값으로 덮어질 경우 기존의 종단점이 아닌 다른 종단점에서 출력을 멈추게 됩니다.

`scanf()`와 같은 함수는 문자열을 입력받을 때 마지막에 Null Byte를 추가하지만 `read()` 함수와 같이 마지막에 Null Byte를 추가하지 않는 입력 함수도 있습니다. 따라서 이러한 함수에서 값을 조작할 경우 Null Byte를 임의의 값으로 변조하는 것이 가능합니다.

다음 예시는 wargame 문제 중 신기하였던 구성이어서 정리해보았습니다.

### Case 1

- 변수의 구조

먼저 main을 확인하였을 때 아래 순으로 선언되었습니다.

```c
struct orign copied;
char flag[56];
int index;
```

구조체의 경우 아래와 같이 두 개의 변수가 선언되어 있었습니다.

```c
struct orign {
	char name[16];
	int age;
}
```

이후 flag 변수의 경우 `memset()` 함수를 통해 0으로 초기화를 해주었습니다.  처음 코드를 확인했을 때는 `memset()` 함수에 대해 제대로 이해하지 못했어서 이를 heap 영역이라고 판단하였으나 이후 이는 관련이 없음을 알 수 있었습니다.

문제를 확인했을 때 가장 의문이었던 점은 name을 입력받을 때는 `read()` 함수를 사용하는 반면에 age를 입력 받을 때는 `scanf()` 함수를 사용하였다는 점 입니다. Memory leak과 관련된 자료를 확인하였을 때 `scanf()` 함수의 경우 마지막에 Null Byte를 추가하기에 취약점으로 사용할 수 없다고 학습했기 때문입니다.

이유에 대해 찾아본 결과 `scanf()`를 통해 입력받는 형식이 int 형을 입력받기 때문이라고 생각되었습니다. `scanf()`의 man page를 확인해보면 '%s' 지시자에 대해 다음과 같은 설명이 있는 것을 확인할 수 있습니다.

> hold the input sequence and  the  terminating null byte ('\0'), which is added automatically.

이에 대한 정확한 검증을 해보고자 코드를 작성하였으나 아직 완벽한 검증은 성공하지 못했습니다.

Name의 경우 `read()` 함수로 입력받고 Age의 경우 Null Byte가 추가되지 않는 것을 확인할 수 있기에 Memory Leak이 가능함을 확인할 수 있었습니다.

다른 Write-Up을 보고 알게된 것 중 하나로는 Age에 Printable Character의 10진수 ASCII 값을 입력하여 '%s'로 출력할 때 연결되게끔 하는 것이었습니다.

* * *

## Reference

- <https://www.geeksforgeeks.org/memory-layout-of-c-program/>

- <https://www.tutorialspoint.com/c_standard_library/c_function_memset.htm>

- <https://9oat.tistory.com/4?category=61772>
