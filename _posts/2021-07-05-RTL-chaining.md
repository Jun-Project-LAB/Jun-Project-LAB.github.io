---
title: "RTL Chaining"
tags: ["pwnable theory"]
categories: ["System"]
---

# Index

1. [](#)

* * *

## Overview

RTL Chaining은 기존에 단일 동작만을 수행할 수 있던 RTL 공격에서 다중 동작을 수행할 수 있도록 발전된 형태입니다.

기본적인 개념을 공유하기 때문에 해당 기법을 통해 우회할 수 있는 보호 기법, 공격 조건들은 매우 유사합니다.

## Key Points

RTL chaining에서 가장 중요한 핵심은 gadget입니다. 동일한 코드로 작성된 프로그램이어도 architecture(32bit / 64bit)에 따라 필요한 gadget에 차이가 있습니다.

Exploit payload에 있어서 중요한 차이는 RTL에서는 dummy로 사용되던 부분이 RTL Chaining에서는 다음 동작을 위한 RET 주소로 사용된다는 점입니다.

이를 그림으로 정리하면 다음과 같이 형태가 바뀌었음을 확인할 수 있습니다.

[그림]

## Function Calling Convention

RTL chaining 공격을 성공하기 위해서는 적절한 gadget을 사용하는 것이 중요합니다.

상황에 맞는 gadget을 사용하기 위해서는 공격 중 `stack의 변화`와 `함수 인자값 전달 방식`에 대해 이해할 필요가 있습니다.

### Stack의 변화

프로그램에서 함수를 호출할 때, push와 pop을 반복하며 stack에 변화가 생깁니다.

호출되는 함수가 인자값을 사용하지 않더라도 공통적인 prologue와 epilogue 과정을 거치며 stack 내의 변화가 발생합니다.

- 정상적인 경우

정상적인 함수 호출의 경우 `push ebp; mov ebp, esp`  동작을 수행하는 prologue와 `leave; ret` 동작을 수행하는 epilogue를 통해 특별한 오류 없이 정상적으로 호출을 종료합니다.

- RTL Chaining의 경우

RTL 공격은 함수의 mapping 된 주소를 통해 직접 접근하기 때문에 정상적인 prologue와 epilogue를 가지지 않습니다. 이를 이해하기 위해 stack의 변화를 통한 공격의 흐름을 살펴보았습니다.

    - Before Epilouge

main 함수에서 read 함수를 호출했다는 가정 하에 stack을 재구성하였습니다.

|Stack|Caption|Register|
|:---:|:-----:|:------:|
|ret|Return Address|-|
|sfp|-|<-ebp|
|First Argument|-|-|
|Second Argument|-|-|
|Bottom of Stack|-|\<-esp|

    - After Epilogue

Epilogue가 시작되기 이전(leave가 실행되기 이전)에는 esp가 현재 사용중인 stack의 영역 중 최하단을 가리키고 있습니다.. `leave` 명령은 세부적으로 `mov esp, ebp; pop ebp` 과정을 진행합니다.

먼저 `mov esp, ebp`를 실행하여 esp가 sfp를 가리키게 됩니다. 이후 `pop ebp`를 실행하게 됨으로써 sfp에 있는 값(caller의 sfp)를 ebp register로 이동합니다.

위 과정을 수행한 뒤 stack의 상태는 아래와 같습니다.

|Stack|Caption|Register|
|:--:|:------:|:------:|
|ret|Return Address|\<-esp|
|sfp|Caller's ebp|-|

    - Before Prologue

`ret` 명령의 경우 내부적으로 `pop eip; jmp eip` 명령을 수행하게 됩니다. 이때 RTL 공격에 의해 변조된 함수로 이동하게 됩니다.

일반적으로 함수의 prologue를 설명할 때 포함하지는 않으나 call을 통해 함수를 호출할 경우 ret 주소를 stack에 저장하는 `push eip` 과정을 수행합니다.

그러나 변조된 ret을 통해 접근할 때는 이러한 과정을 거치지 않으며 현재 stack의 상태는 아래와 같이 됩니다. `system()` 함수를 사용해 `/bin/sh`를 호출하는 것을 예시로 작성하였습니다.

|Stack|Caption|Register|
|:---:|:-----:|:------:|
|arg1|`/bin/sh`|-|
|Addr|ret after `system()`|\<-esp|
|ret|`system()` Address|-|

ebp의 경우 변조된 함수가 호출되기 이전의 caller의 ebp 값을 가지고 있습니다. 

    - After Prologue

이후 함수의 prologue를 수행하면 다음과 같이 stack이 변화합니다.

|Stack|Caption|Register|
|:---:|:-----:|:------:|
|arg1|`/bin/sh`|-|
|Addr|ret after `system()`|-|
|sfp|-|\<-ebp,esp|

기존에 system 함수를 호출하기 위해 변조되었던 ret 주소 부분이 현재 system 함수에서 사용되는 sfp로 변경된 것을 확인할 수 있었습니다.

이후 함수를 죵료할 때 진행하게 되는 epilogue 과정을 예상해보면 dummy의 부분이 왜 ret 주소가 되는지 알 수 있습니다.

첫 번째 argument가 dummy 이후에 들어가는 이유 또한 정상적인 system 함수 호출 과정에서 사용할 인자를 push 한 뒤 call을 진행하기 때문에 정상 함수 호출과 일치하는 형식임을 확인할 수 있었습니다.

* * *

### 함수 인자값 전달 방식

call을 사용하여 함수를 호출할 때 x86과 x64에서의 동작 방식에 조금의 차이가 있습니다. 먼저 두 architecture의 공통점은 다음과 같습니다.

1. 인자를 전달할 때 오른쪽에서 왼쪽의 순서로 저장
2. 함수의 반환값은 eax에 정리

이외에도 공통점이 있을 수 있지만 공격에 있어 핵심적인 공통점은 위 두 가지를 꼽을 수 있습니다. x86과 x64의 가장 큰 차이는 인자를 전달할 때 `어떠한 공간을 쓰는가?` 입니다.

x86에서는 여태까지 배워왔듯이 모든 인자를 `stack`을 사용하여 전달합니다. 그러나 x64는 6개까지의 인자는 사전에 정의된 register를 사용하여 전달합니다. 사용되는 register는 다음과 같습니다.

|Register|Usage|
|:------:|-----------|
|rdi|1st argument|
|rsi|2nd argument|
|rdx|3rd argument & 2nd return register|
|rcx|4th integer argument|
|r8|5th argument|
|r9|6th argument|

만약 이 이상의 argument를 전달할 필요가 있을 때는 7번째 인자부터 stack에 push 후 사용합니다.

## Gadget

RTL과 RTL chaining, x86과 x64의 calling convention의 차이를 확실히 알게 되었다면 사용하게 되는 gadget 또한 다를 것을 예상할 수 있습니다.

만약 RTL chaining payload를 작성할 때 두 개의 함수를 직접적으로 연결할 경우 제대로 된 argument 전달이 불가합니다.

따라서 이를 맞춰주기 위해 gadget을 사용하여 esp를 조정 후, 해당 함수를 호출하여 의도한 동작이 가능하게끔 합니다.

x86의 경우 대표적으로 `pop; pop; ret`과 같이 `pop`과 `ret`으로 이루어진 gadget을 사용합니다. 경우에 따라 `add esp, 8; pop; ret`과 같은 gadget을 사용하는 것도 가능합니다.

여기서, pop을 할 때 어느 register에 저장하느냐는 공격의 성공 여부에 관여하지 않습니다.

그러나 x64의 경우 첫 번째 인자가 rdi register에 저장되기에 `pop rdi; pop rsi; ret`과 같이 호출 규약에 부합하는 gadget을 사용하여야 합니다.
