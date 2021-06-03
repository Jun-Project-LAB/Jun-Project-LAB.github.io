---
title: "Memory Mitigation"
tags: ["pwnable theory", memory]
categories: System
---

메모리 보호기법
---------------

Pwnable 문제를 진행하다보면 다양한 메모리 보호기법이 적용되어 있는 것을 알 수 있습니다. 메모리 보호기법에 따라 접근 방식을 변경해야 할 필요가 있기에 이에 대한 자료를 찾아 정리하는 기회를 가졌습니다.

RELRO, Canary, NX, PIE에 대해서 알아보았으며 각각의 의미와 해당 설정 여부를 알 수 있는 방법들에 대해 찾아보았습니다.

## RELRO

RELRO는 "**RELocation Read-Only**"의 약자로 특정 바이너리 섹션을 읽기 전용으로 변경합니다. RELRO의 모드에는 `Partial RELRO`, `Full RELRO` 두 가지 모드가 있습니다.

### Partial VS Full

- Partial RELRO

GCC에서의 기본 설정이며 거의 모든 접할 수 있는 바이너리의 경우 최소한 partial RELRO 설정을 가지고 있습니다. 공격자의 관점에서 메모리 상에서 GOT가 BSS 보다 먼저 나오도록 하는 차이만 있습니다.

- Full RELRO

Full RELRO의 경우 GOT를 read-only로 설정하여 "GOT Overwrite" 공격에 사용할 수 없도록 변경합니다. 이 옵션이 기본 설정이 아닌 이유는 프로그램이 시작될 때 모든 symbol에 대해 해석하는 과정을 거치는데 만약 규모가 큰 프로그램의 경우 이 과정에서 많은 부하가 발생할 수 있기 때문이라고 합니다.

두 모드의 차이의 자세한 설명에 대해서는 [링크](http://www.lazenca.net/display/TEC/04.RELRO#id-04.RELRO-Explanation)를 참조할 수 있을 것 같으나 해당 링크에 대한 모든 내용을 이해하기에는 조금 시간이 필요할 것 같습니다.

### Difference

먼저 모드와 관계없이 RELRO 기법이 적용될 경우 Program Header에 `GUN_RELRO` 영역이 추가됩니다. 이는 `readelf -l <프로그램명>`을 통해서 확인할 수 있습니다.

> Full RELRO가 적용된 프로그램

```
Program Headers:
  Type           Offset             VirtAddr           PhysAddr
	         FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000002d8 0x00000000000002d8  R      0x8
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000698 0x0000000000000698  R      0x1000
  LOAD           0x0000000000001000 0x0000000000001000 0x0000000000001000
                 0x00000000000002e5 0x00000000000002e5  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x0000000000000158 0x0000000000000158  R      0x1000
  LOAD           0x0000000000002da8 0x0000000000003da8 0x0000000000003da8
                 0x0000000000000268 0x0000000000000270  RW     0x1000
  DYNAMIC        0x0000000000002db8 0x0000000000003db8 0x0000000000003db8
                 0x00000000000001f0 0x00000000000001f0  RW     0x8
  NOTE           0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000020 0x0000000000000020  R      0x8
  NOTE           0x0000000000000358 0x0000000000000358 0x0000000000000358
                 0x0000000000000044 0x0000000000000044  R      0x4
  GNU_PROPERTY   0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000020 0x0000000000000020  R      0x8
  GNU_EH_FRAME   0x0000000000002008 0x0000000000002008 0x0000000000002008
                 0x0000000000000044 0x0000000000000044  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000002da8 0x0000000000003da8 0x0000000000003da8
                 0x0000000000000258 0x0000000000000258  R      0x1
```

> Partial RELRO가 적용된 프로그램

```
Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  PHDR           0x000034 0x08048034 0x08048034 0x00160 0x00160 R   0x4
  INTERP         0x000194 0x08048194 0x08048194 0x00013 0x00013 R   0x1
      [Requesting program interpreter: /lib/ld-linux.so.2]
  LOAD           0x000000 0x08048000 0x08048000 0x0040c 0x0040c R   0x1000
  LOAD           0x001000 0x08049000 0x08049000 0x00368 0x00368 R E 0x1000
  LOAD           0x002000 0x0804a000 0x0804a000 0x001f8 0x001f8 R   0x1000
  LOAD           0x002f04 0x0804bf04 0x0804bf04 0x00134 0x00138 RW  0x1000
  DYNAMIC        0x002f0c 0x0804bf0c 0x0804bf0c 0x000e8 0x000e8 RW  0x4
  NOTE           0x0001a8 0x080481a8 0x080481a8 0x00044 0x00044 R   0x4
  GNU_EH_FRAME   0x00201c 0x0804a01c 0x0804a01c 0x0005c 0x0005c R   0x4
  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RW  0x10
  GNU_RELRO      0x002f04 0x0804bf04 0x0804bf04 0x000fc 0x000fc R   0x1
```

그러나 이는 공통적인 변경점으로 두 개의 모드를 구분하기 위해선 추가적인 확인이 필요하였습니다. 찾아본 결과, Partial RELRO와 Full RELRO에서 Section 영역에 일부 차이가 있음을 확인할 수 있었습니다. 그중 Full RELRO에서는 `FLAGS_1`이라는 Dynamic Section이 추가된다는 것을 이용하여 두 모드의 차이점을 구분할 수 있을 것이라 판단하였습니다. Dynamic Section의 경우 `readelf -d <프로그램명>`을 통해서 확인할 수 있었습니다.

> Full RELRO가 적용된 프로그램

```
Dynamic section at offset 0x2db8 contains 27 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x000000000000000c (INIT)               0x1000
 0x000000000000000d (FINI)               0x12d8
 0x0000000000000019 (INIT_ARRAY)         0x3da8
 0x000000000000001b (INIT_ARRAYSZ)       8 (bytes)
 0x000000000000001a (FINI_ARRAY)         0x3db0
 0x000000000000001c (FINI_ARRAYSZ)       8 (bytes)
 0x000000006ffffef5 (GNU_HASH)           0x3a0
 0x0000000000000005 (STRTAB)             0x4a0
 0x0000000000000006 (SYMTAB)             0x3c8
 0x000000000000000a (STRSZ)              167 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000015 (DEBUG)              0x0
 0x0000000000000003 (PLTGOT)             0x3fa8
 0x0000000000000002 (PLTRELSZ)           72 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x650
 0x0000000000000007 (RELA)               0x590
 0x0000000000000008 (RELASZ)             192 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000000000001e (FLAGS)              BIND_NOW
 0x000000006ffffffb (FLAGS_1)            Flags: NOW PIE
 0x000000006ffffffe (VERNEED)            0x560
 0x000000006fffffff (VERNEEDNUM)         1
 0x000000006ffffff0 (VERSYM)             0x548
 0x000000006ffffff9 (RELACOUNT)          3
 0x0000000000000000 (NULL)               0x0
```

> Partial RELRO가 적용된 프로그램

```
Dynamic section at offset 0x2f0c contains 24 entries:
  Tag        Type                         Name/Value
 0x00000001 (NEEDED)                     Shared library: [libc.so.6]
 0x0000000c (INIT)                       0x8049000
 0x0000000d (FINI)                       0x8049354
 0x00000019 (INIT_ARRAY)                 0x804bf04
 0x0000001b (INIT_ARRAYSZ)               4 (bytes)
 0x0000001a (FINI_ARRAY)                 0x804bf08
 0x0000001c (FINI_ARRAYSZ)               4 (bytes)
 0x6ffffef5 (GNU_HASH)                   0x80481ec
 0x00000005 (STRTAB)                     0x80482ec
 0x00000006 (SYMTAB)                     0x804820c
 0x0000000a (STRSZ)                      132 (bytes)
 0x0000000b (SYMENT)                     16 (bytes)
 0x00000015 (DEBUG)                      0x0
 0x00000003 (PLTGOT)                     0x804c000
 0x00000002 (PLTRELSZ)                   72 (bytes)
 0x00000014 (PLTREL)                     REL
 0x00000017 (JMPREL)                     0x80483c4
 0x00000011 (REL)                        0x80483ac
 0x00000012 (RELSZ)                      24 (bytes)
 0x00000013 (RELENT)                     8 (bytes)
 0x6ffffffe (VERNEED)                    0x804838c
 0x6fffffff (VERNEEDNUM)                 1
 0x6ffffff0 (VERSYM)                     0x8048370
 0x00000000 (NULL)                       0x0
```

grep 명령을 추가하여 더 명확한 확인이 가능하였습니다.

> Full RELRO

```
persian@Mint:~/C%$ readelf -d memory | grep FLAGS_1
 0x000000006ffffffb (FLAGS_1)            Flags: NOW PIE
persian@Mint:~/C%$
```

> Partial RELRO

```
root@0a9d68dcd731:~/pwn# readelf -d ex002 | grep FLAGS_1
root@0a9d68dcd731:~/pwn#
```

* * *

## Canary Word

Canaries 혹은 Canary word 기법은 buffer와 control data 사이에 특정값을 추가하여 buffer overflow를 감시하기 위한 방법입니다. 만약 BOF를 통해 RET 주소를 변조하고자 시도할 경우 정해진 buffer의 경계를 초과하여 임의의 값을 입력하게 되는데 이때 buffer와 RET 사이에 위치한 canary 값이 변경될 경우 overflow가 발생하였음을 감지할 수 있습니다.

Canaries에서 사용하는 type의 경우 **terminator**, **random**, **random XOR** 총 세 개의 type이 있습니다. StackGuard의 경우 세 가지 type 모두 지원하며 ProPolice의 경우 terminator와 random type을 지원합니다.

### Terminator VS Random VS Random XOR

- Terminator canaries

Terminator canaries는 대부분의 string operations을 기반으로 한 buffer overflow 공격에서 특정 문자열 종결자를 사용한다는 것에 기반하고 있습니다. 이때 canary는 `Null`, `CR`, `LF`, `FF`과 같은 문자들로 구성됩니다. 공격자는 RET 주소를 변경하기 위해서 이를 명시하기 이전에 NULL 문자를 삽입할 필요가 있는데, strcpy와 같이 null 위치까지만 복사하는 함수에 대한 overflow 예방이 가능합니다. 그러나 이 기법은 canary를 알려진 값으로 겹쳐 쓰고 정보를 틀린 값들로 제어하여 검사 코드를 우회할 가능성도 있습니다.

- Random canaries

Random canaries는 일반적으로 Entropy-gathering daemon으로부터 공격자들이 값을 예측할 수 없는 랜덤한 값을 생성합니다. Exploit을 위해 canary 값을 읽으려는 것은 가능하거나 타당치 않으며 해당 값을 반드시 읽어야 할 필요가 있는 buffer overflow protection code와 같은 코드만 허용됩니다. Random canary의 경우 보편적으로 프로그램이 초기화할 때 생성되어 전역 변수에 저장됩니다. 이 변수는 unmapped page에 할당되기 때문에 어떠한 트릭이나 버그를 사용하여 RAM을 읽으려고 시도할 경우 segmentation fault가 발생하거나 프로그램이 종료됩니다. 그러나 만약 공격자가 canary가 어디에 위치해 있는지 알고 있다면 stack으로부터 이 값을 읽는 것이 가능합니다.

- Random XOR Canaries

Random XOR Canaries는 control data의 전체 혹은 일부가 XOR-scramble 되어 생성됩니다. 이 방법의 경우 canary 혹은 control data가 망가질 경우 canary 값이 잘못됩니다. Random XOR canaries의 경우 Random canaries보다 stack으로부터 canary를 읽는 것이 조금 더 복잡한 것 외에 같은 취약점을 가지고 있습니다. 기존의 canary를 재생성하여 보호를 변조하기 위해서는 canary, algorithm, control data가 필요합니다. 이외에도 포인터를 제어 데이터의 조각을 가리키게 하는 공격 유형에 대해서도 보호 기능이 있다고 하는데 이 부분은 더 알아볼 필요가 있을 것 같습니다.

### Difference

Canary 기법이 적용된 파일의 경우 Symbol table에 `__stack_chk_fail`가 존재할 경우 설정되어 있는 것을 알 수 있습니다. 이를 확인하기 위해서는 `readelf -s <프로그램명>`을 통해서 확인할 수 있습니다. Symbol table을 출력하게 될 경우 매우 많은 양의 내용이 출력될 수 있기에 grep을 통해 더 쉽게 확인할 수 있습니다.

> Canary OK

```
persian@Mint:~/C%$ readelf -s memory | grep __stack_chk_fail
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __stack_chk_fail@GLIBC_2.4 (3)
    52: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __stack_chk_fail@@GLIBC_2
```

> Canary NO

```
root@0a9d68dcd731:~/test# readelf -s fsb | grep __stack_chk_fail
root@0a9d68dcd731:~/test#
```

* * *

## NX bit

NX bit의 경우 Stack과 Data 영역에 쓰기 권한과 실행 권한을 동시에 부여하지 않음으로써 shellcode 삽입에 성공하여도 이를 실행하지 못하도록 하는 기법입니다. 주로 rw 권한만을 부여하고 실행 권한을 제거합니다.

NX bit의 적용 여부 판단은 segment header 정보에서 'GUN_STACK'의 flag 값이 `RW`로 설정되어 있을 경우 활성화 되어 있다고 판단할 수 있습니다.

> NX OK

```
root@0a9d68dcd731:~/test# readelf -w -l fsb | grep GNU_STACK
  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RW  0x10
```

> NX NO

```
root@0a9d68dcd731:~/test# readelf -l nxno | grep GNU_STACK
  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RWE 0x10
```

* * *

## PIE

PIE는 **Position-independent executable**의 약자로 위치 독립 코드로 이루어진 실행 가능한 바이너리를 의미합니다. 위치가 독립되어 있다는 것은 바이너리 영역을 랜덤화하여도 프로그램 실행에 문제가 없다는 것을 의미하기도 합니다. 모든 영역을 randomizing 하여 데이터 영역의 주소 또한 바뀌지만 offset은 바뀌지 않는다.

### Difference

PIE의 적용 여부는 2단계에 걸쳐서 확인합니다. 먼저 ELF 파일의 타입을 확인하여 'EXEC' 일 경우 'No PIE'로 간주하고 'DYN' 일 경우 'PIE' 조건의 1단계를 만족한 것입니다. DYN type의 파일의 경우 다시 두 종류로 나뉘게 되는데 dynamic section에 'DEBUG' 엔트리가 있을 경우 'PIE Enabled'로 판단하고 해당 엔트리가 없을 경우 'DSO(Dynamic Shared Object)'로 판단합니다. 이와같이 2단계에 걸쳐서 판단하는 이유는 'DEBUG' 엔트리가 동적 링커가 제공하는 debug 구조체의 포인터로 실행 가능한 파일에만 존재하기 때문입니다.

#### First

Type의 경우 `readelf -h <프로그램명> | grep Type`을 통해서 확인할 수 있습니다.

> PIE Enabled

```
persian@Mint:~/C%$ readelf -h memory | grep Type
  Type:                              DYN (Shared object file)
```

> No PIE

```
root@0a9d68dcd731:~/test# readelf -h fsb | grep Type
  Type:                              EXEC (Executable file)
```

#### Second

DEBUG 엔트리의 경우 `readelf -d <프로그램명> | grep DEBUG`를 통해서 확인할 수 있습니다. No PIE로 설정된 프로그램 또한 Type은 EXEC이지만 실행파일이기에 DEBUG 엔트리가 존재하는 것을 확인할 수 있었습니다.

> PIE Enabled

```
persian@Mint:~/C%$ readelf -d memory | grep DEBUG
 0x0000000000000015 (DEBUG)              0x0
```

> No PIE

```
root@0a9d68dcd731:~/test# readelf -d fsb | grep DEBUG
 0x00000015 (DEBUG)                      0x0
```

* * *

## Reference

- <https://ctf101.org/binary-exploitation/relocation-read-only/>

- <https://zeromini0.tistory.com/167>

- <http://www.lazenca.net/display/TEC/04.RELRO#id-04.RELRO-Explanation>

- <https://en.wikipedia.org/wiki/Buffer_overflow_protection#Canaries>

- <https://security.stackexchange.com/questions/68918/buffer-overflow-terminator-canaries>

- <https://koharinn.tistory.com/49>

- <https://dreamhack.io/learn/2#25>

- <https://kangsecu.tistory.com/138>

- <https://en.wikipedia.org/wiki/Position-independent_code>

- <http://www.lazenca.net/display/TEC/06.PIE>

- <https://bpsecblog.wordpress.com/2016/06/28/memory_protect_linux_5/>
