---
title: "Memory Corruption"
tags: [memory, pwnable, "basic theory", "dreamhack lecture"]
categories: [system]
---

Index
-----

1. [Buffer Overflow](#buffer-overflow)
2. [Out Of Boundary](#out-of-boundary)
3. [Off By One](#off-by-one)
4. [Format String Bug](#format-string-bug)
5. [Double Free & Use-After-Free](#double-free-&-use-after-free)



## Buffer Overflow

> What is Buffer Overflow?
```
Buffer Overflow(이하 BOF) 취약점은 버퍼가 허용할 수 있는 양의 데이터보다 많은 값이 저장되어 버퍼가 넘치는 취약점입니다.
```

BOF는 발생하는 위치에 따라 Stack Buffer Overflow, Heap Overflow로 나눌 수 있습니다. 이는 인접한 메모리를 오염시키는 취약점이기 때문에 어떤 메모리를 대상으로 하느냐에 따라 공격 방법이 달라지기 때문입니다.

Stack BOF와 Heap BOF의 차이점이 Stack 영역인지 Heap 영역인지에 따른 차이인만큼 Stack과 Heap의 차이에 대해서도 간략하게 정리하였습니다.

- Stack

Stack은 Function에 의해서 만들어진 임시 변수를 저장하는 공간입니다. Stack에서는 runtime 동안 변수가 선언, 저장과 초기화가 이루어지며 임시 저장 메모리입니다.

이러한 특성으로부터 알 수 있듯이 작업이 끝나면 Stack 내의 변수들이 자동으로 삭제됩니다. 이 Section에는 주로 method, 지역 변수, 참조 변수가 저장됩니다.

* * *

- Heap

Heap 영역은 프로그래밍 언어에서 전역 변수를 저장하는 공간입니다. 기본적으로 모든 전역 변수들은 heap memory 공간에 저장되며 동적 메모리 할당을 지원합니다.

Stack과 다르게 Heap은 자동으로 관리되지 않기 때문에 사용한 메모리 공간을 해제하지 않을 경우 "메모리 누수"가 발생합니다.



## Out Of Boundary

> What is Out of Boundary?
```
Out of Boundary(이하 OOB)는 Buffer의 범위를 벗어나는 인덱스에 접근할 때 발생하는 취약점입니다.
```

Index를 통해 접근할 때 제대로 된 경계 검사를 하지 않을 경우 범위를 벗어난 임의의 주소에 접근할 수 있게 됩니다. 이를 통해 memory 값을 임의로 조작할 수 있습니다.



## Off by One

> What is Off by One?
```
Off by One이란 경계 검사에서 "하나"의 오차가 있을 때 발생하는 취약점입니다.
여기서 하나의 오차란 buffer의 경계 계산 혹은 반복문의 횟수 계산 시 < 대신 <=을 사용하거나 0부터 시작하는 index를 고려하지 못해서 발생하는 오차를 뜻합니다.
```



## Format String Bug

> What is Format String Bug?
```
Format String Bug(이하 FSB)는 printf나 sprintf와 같이 FS를 사용하는 함수에서 프로그래머가 지정한 문자열이 아닌 사용자의 입력이 FS로 전달될 때 발생하는 취약점입니다.
```

이에 대해 쉽게 예를 들면 사용자의 문자열을 입력받는 곳에서 %x, %d와 같은 FS를 입력할 경우 이후의 인자가 전달되지 않기 때문에 쓰레기 값을 출력하게 된다고 합니다.

최신 컴파일러에서는 이러한 취약점을 막기 위하여 노력하고 있으나 그럼에도 발견될 경우 프로그램에 큰 영향을 줄 수 있는 취약점입니다.

- 표준 C Libray에서 FS를 사용하는 대표적인 함수들
	- printf
	- sprintf/snprintf
	- fprintf
	- vprintf/vfprintf
	- vsprintf/vsnprintf



## Double Free & Use-After-Free


