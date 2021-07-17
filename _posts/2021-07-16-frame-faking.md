---
title: "Frame Faking"
tags: ["pwnable theory"]
categories: ["System"]
---

# Index

1. [What is Frame faking?](#what-is-frame-faking)
2. [Sequence of Attack](#sequence-of-attack)
3. [Reference](#reference)

* * *

## What is Frame faking?

Stack은 runtime 동안 여러 목적을 위해 공간을 할당하였다가 삭제하는 과정을 거칩니다. 그 과정에서 Stack의 base pointer를 저장하는 stack frame pointer 또한 저장과 삭제를 반복합니다.

Base pointer는 architecture에 따라 ebp, rbp 등으로 불리우며 **Frame faking** 공격을 이해하기 위해서는 먼저 함수의 epilogue를 이해할 필요가 있습니다. 이와 관련된 내용은 [[게시글](https://jun-project-lab.github.io/system/Function-Calling/#epilogue)]에서 정리하였습니다.

Epilogue 과정 동안 ebp와 esp 값의 변화가 생기는데 이때 ebp 값을 조작함으로써 esp의 값 또한 조작이 가능해지고 최종적으로 프로그램의 흐름을 원하는 곳으로 변경할 수 있게 됩니다.

이같은 공격 방식은 대상 프로그램에서 ret을 변조할 수 있는 정도의 overflow가 발생하지만 다른 library나 함수 주소로 변조하는 것이 불가능할 때 사용할 수 있다고 합니다.

## Sequence of Attack

공격에서 중요한 핵심은 `leave` instruction 이었습니다. `leave` instruction은 `mov esp, ebp; pop ebp`의 동작을 수행하는데 아래와 같은 stack 상태에서 ret 주소를 `leave` instruction으로 변조하면 최종적으로 esp를 변조하는 것이 가능합니다.

|Address|Value|Register|
|:-----:|-----|--------|
|ffffa00c|0xdeadbeef1|-|
|ffffa010|[system function address]|\<-esp|
|ffffa014|[exit function address]|-|
|ffffa018|["/bin/sh" address]|-|
|ffffa01c|0xffffa00c|\<-sfp(ebp)|
|ffffa020|[leave address]|\<-ret|

시작하기에 앞서 esp의 경우 임의의 주소에 지정하였습니다.

먼저 `leave`를 수행하면 현재의 esp를 ebp 주소로 옮기고 sfp 주소에 있는 값을 ebp로 pop 합니다. 일반적인 경우 다음의 ret 과정에서 현재 함수를 종료 후 실행하게 될 명령을 pop 후 jmp를 수행합니다.

그러나 변조된 payload의 경우 아래 상태에서 leave 명령을 다시 한번 수행합니다.

|Address|Value|Register|
|:-----:|-----|--------|
|ffffa00c|0xdeadbeef|\<-ebp|
|ffffa010|[system function address]|-|
|ffffa014|[exit function address]|-|
|ffffa018|["/bin/sh" address]|-|
|ffffa01c|0xffffa00c|\<-sfp|
|ffffa020|[leave address]|\<-ret, esp|

앞서 설명하였듯이 `leave`의 경우 내부적으로 esp에 ebp 주소를 복사 후 esp가 가리키고 있는 주소의 값을 pop을 수행하여 ebp에 할당하게 됩니다.

`leave` 명령을 수행한 뒤의 stack과 register의 상태는 아래와 같이 변경됩니다.

|Address|Value|Register|
|:-----:|-----|--------|
|ffffa00c|0xdeadbeef|-|
|ffffa010|[system function address]|\<-esp|
|ffffa014|[exit function address]|-|
|ffffa018|["/bin/sh" address]|-|
|ffffa01c|0xffffa00c|\<-sfp|
|ffffa020|[leave address]|\<-ret|

`leave`를 수행 후 `ret`을 다시 수행한다는 뜻이 이해하는데 약간 시간이 걸렸는데, `leave` 명령 이후에 `ret` 명령이 따라오기 때문에 ret을 실행하게 되는 것이었습니다.

간단하게 예시를 보자면 다음과 같습니다.

```
   0x08048571 <+86>:  leave
   0x08048572 <+87>:  ret
```

ret 위치를 leave 명령으로 변조한다는 것은 `0x08048571` 위치로 변조하는 것과 같습니다. 따라서 다음 실행하게 될 명령이 `0x08048572`가 됩니다.

ret 명령을 수행하기 이전에 esp가 system function address를 가리키고 있었기에 Return to Libc와 같은 효과를 나타냅니다.

## Reference

- <https://www.lazenca.net/pages/viewpage.action?pageId=12189944>
