---
title: "Windows Boot Process"
tags: [windows, "system theory"]
categories: [System]
---

# Index

1. [Phase 1](#phase-1)
2. [Phase 2](#phase-2)
3. [Phase 3](#phase-3)
4. [Phase 4](#phase-4)

## Preview

Windows System을 공부하며 booting process 부분에 대해 별도로 정리할 필요가 있다고 생각하여 작성하였습니다.

## Phase 1

- **Preboot**

Phase 1에서는 POST 즉, Power-on-self-test를 통하여 기본적인 system에 대한 검사를 실시합니다. 여기서 검사란 유효한 disk system이 있는지, 다음 phase로 넘어가기에 문제가 없는지가 있습니다.

Windows의 경우 사용하는 firmware가 BIOS냐 UEFI냐에 따라 POST 이후 부팅 관리자로 진입할 때의 차이가 있다고 합니다.

### BIOS의 경우

BIOS는 부팅 가능한 하드 디스크로부터 boot sector인 MBR을 확인합니다. 이 MBR은 MBR Boot code와 partition table을 가지고 있는데 MBR boot code를 실행하여 **system volume**을 찾아 메모리에 로드합니다. 이때 로드되는 system volume을 VBR(Volume Boot Record)라고 합니다.

VBR에도 VBR boot code가 존재하며 이는 자신의 partition에서 OS의 architecture에 해당하는 **boot manager program**을 찾아 실행합니다. 이는 "%SystemDrive%\bootmgr" 위치에 존재합니다.

* * *

### UEFI의 경우

EFI의 경우 BIOS와 다르게 boot code가 별도의 MBR이나 VBR에 존재하는 것이 아닌 firmware 내에 존재합니다. 따라서 별도의 MBR, VBR 접근 과정을 거치지 않고 "%SystemDrive%\EFI\Microsoft\Boot\Bootmgfw.efi"에 존재하는 실행 프로그램을 구동시킵니다.

## Phase 2

- **Boot Manager**

Firmware의 방식에 상관없이 최종적으로는 **Boot Manager(이하 bootmgr)**을 메모리에 로드하여 실행시킵니다. bootmgr은 BCD(Boot Configuration Data) 설정 데이터를 읽어와 시스템을 시작하는데 이는 레지스트리의 "HKLM/BCD00000000" 경로에 존재한다고 합니다.

최종적으로 Windows Boot Loader인 **winload.exe**를 로드 후 실행합니다. 이는 "%SystemDrive%\System32" 경로에 존재하며 NT Based Windows의 경우 **NTLDR**이라고 하는 NT Loader가 bootmgr과 winload.exe의 역할을 같이 수행하였습니다.

## Phase 3

- **OS Loader**

이 과정에서는 winload.exe가 필수적인 드라이버를 로드하여 windows kernel을 실행합니다. 다음 과정으로 진행하기 위한 충분한 로드가 끝나면 winload.exe와 같은 경로에 존재하는 **ntoskrnl.exe**를 실행합니다.

## Phase 4

- **NT OS Kernel**

부팅의 마지막 단계로 레지스트리 설정과 추가적으로 필요한 드라이버를 로드합니다. 최종적으로 윈도우 로그인 화면이 표시되기까지 각종 초기화나 설정에 대해 광범위하게 진행됩니다.
