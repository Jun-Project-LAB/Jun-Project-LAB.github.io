---
title: "Function Prologue & Epilogue"
tags: [function, "pwnable theory"]
categories: [System]
---

Frame Pointer Overflow(with. Off by one)
----------------------------------------

# Index

1. [함수 호출 과정](#함수-호출-과정)
	- [Prologue](#prologue)
	- [Epilogue](#epilogue)
2. [Reference](#reference)

> 본 게시글은 x64 Architecture를 기준으로 작성하였습니다.

## 함수 호출 과정

FPO 기법에 대해 공부하던 중 함수 호출과 관련한 이해가 필요하여 별도의 문서로 작성해보았습니다. 함수의 호출과 관련된 명령으로는 `call`, `leave`, `ret`을 생각해볼 수 있습니다. 함수가 호출되고 처음 실행되는 과정을 Prologue, 종료될 때 실행되는 과정을 Epilogue 라고 합니다. `call`의 경우 prologue에 해당하지는 않지만 stack 변화 이해를 위하여 같이 설명하였습니다.  먼저 함수가 호출될 때부터 Prolog 과정 동안 Stack에는 다음과 같은 변화가 일어납니다.

### 함수 호출

CALL 명령에 의해 함수가 호출되면 gdb와 같은 도구로 확인하였을 때는 바로 prologue 단계로 넘어간 것처럼 확인됩니다. 그러나 호출 전과 후의 rsp(혹은 esp) 값을 확인해보면 감소해 있는 것을 확인할 수 있습니다. 값이 감소되는 원인으로는 `push`를 예상할 수 있는데 stack은 "LIFO" 구조를 가지고 있기 때문에 가장 최근에 넣은 값이 낮은 주소에 위치하게 됩니다. 반면에 인터넷을 통해 자료를 찾아보면, SFP보다 높은 주소에 RET 값이 존재하는 것을 확인할 수 있습니다.

![RET와 SFP 이미지](https://t1.daumcdn.net/cfile/tistory/222966395443F64C0D)

이를 통해서 CALL 과정 중 RET 주소와 관련된 작업이 있다는 것을 유추할 수 있습니다. RET 주소는 함수의 호출이 종료된 후 진행할 다음 명령의 위치를 가리킵니다. 이는 곧 호출 전 caller의 rip 주소와 같은 의미이며 따라서 `push rip` 후 `jmp`를 통해 callee로 이동한다는 것을 알 수 있습니다.

call을 완료한 후 prologue를 시작하기 이전에 stack은 다음과 같이 구성되어 있습니다.

|주소|값|설명|
|:--:|:-:|--|
|높은 주소|`RET`|caller의 다음 rip|
|||||
|낮은 주소|||

### Prologue

call을 완료하면 prologue를 진행하게 됩니다. Prologue는 아래와 같이 두 개의 명령을 실행합니다.

```
push	rbp
mov	rbp,rsp
```

먼저 `push rbp`를 통해서 caller의 rbp를 stack에 저장합니다. 이후 현재 rsp를 rbp 레지스터로 복사하여 callee의 rbp를 새로 설정합니다. 이때 stack에 저장한 rbp가 SFP이며 SFP 값은 RET의 시작 위치를 저장하고 있습니다. 일반적으로 정상적인 프로그래의 경우prologue가 종료된 stack의 구조는 다음과 같습니다.

|주소|값|설명|
|:--:|:-:|---|
|rsp+8|RET|caller의 다음 rip|
|rsp|SFP|caller의 rbp|
|낮은 주소|||

> gcc를 통해 컴파일 후 gdb로 분석해보면 prologue 이전에 endbr64 라는 명령이 실행되는 것을 확인할 수 있습니다. 이는 `End Branch 64 bit` 라는 의미로 프로그램 내의 jump 혹은 indirect call의 유효한 주소를 표시하는데 사용하는 새로운 명령어 입니다. 이를 지원하지 않는 legacy machine에서는 `NOP`과 같은 동작을 수행합니다. [자세한 정보](https://stackoverflow.com/questions/56905811/what-does-endbr64-instruction-actually-do)

### Epilogue

callee의 procedure를 마친 후 epilogue를 진행하여 함수의 정리를 진행하게 됩니다. Epilogue는 아래 두 개의 명령을 통해 진행됩니다.

```
leave
ret
```

두 개의 명령 모두 별도의 인자 지정 없이 진행되지만 각각 내부 코드가 다음과 같이 수행됩니다.

> **leave**

```
mov	rsp,rbp
pop	rbp
```

leave 명령의 경우 callee의 rbp 값을 rsp에 복사 후 `pop rbp`를 통해 SFP 값을 rbp로 이동합니다. leave 명령을 수행한 뒤에 rsp의 값은 `rbp-0x8`을 가리키고 있으며 stack의 상태는 다음과 같습니다.

|주소|값|설명|
|:--:|:-:|---|
|rbp|RET|caller의 다음 rip|
|rsp|||
|낮은 주소||

> **ret**

```
pop	rip
jmp	rip
```

마지막으로 ret 명령을 실행하여 callee를 종료 후 caller로 복귀합니다. `pop rip`를 실행할 경우 RET 값을 rip로 이동 후 `jmp rip` 명령을 통해 caller로 복귀합니다.

* * *

## Reference

- <https://d4m0n.tistory.com/76>

- <https://aistories.tistory.com/12>

- <https://stackoverflow.com/questions/56905811/what-does-endbr64-instruction-actually-do>
