---
title: "Linux Boot Process"
tags: [linux, "system theory"]
categories: [System]
---

# Index

1. [Phase 1](#phase-1)
2. [Phase 2](#phase-2)
3. [Phase 3](#phase-3)
4. [Phase 4](#phase-4)

## Preview

Linux 또한 시스템에 대한 이해도를 높이고자 booting process에 대해 정리해 보았습니다.

## Phase 1

Linux 또한 전원을 인가하였을 때 POST 과정을 통한 사전 점검을 진행한 후 이상이 없을 경우 BIOS로 진입합니다. 
BIOS의 역할은 Windows와 크게 차이가 없는데 저장 장치로부터 부팅을 위한 MBR을 찾고 boot loader program이 정상적으로 감지되었을 때 이를 memory에 로드 후 실행합니다.

MBR은 부팅 가능한 디스크의 첫 번째 sector에 위치해 있으며 타입에 따라 /dev/hda 혹은 /dev/sda로 명명되어 있습니다. 이는 GRUB 로드 및 실행의 역할을 합니다. 연식이 있는 시스템의 경우 Boot loader로 GRUB가 아닌 LILO 또한 사용 되었다고 합니다.

## Phase 2

GRUB의 경우 kernel을 로드하기 전 단일 OS가 아닌 다중 OS 환경일 경우 부팅할 OS를 선택할 수 있는 간단한 메뉴를 제공합니다. 기본적으로는 일정 시간이 지난 후 가장 최근에 부팅한 OS로 진행되지만 설정에 따라 자동으로 시작되지 않고 OS 선택 후 Enter를 눌러야만 실행되도록 할 수도 있습니다.

## Phase 3

GRUB를 통해 OS를 선택한 뒤에는 kernel image를 로드합니다. 이때 압축되어 있는 kernel image를 해제하기 위해 swapper라는 프로그램을 통해 로드하게 됩니다. 이후 grub 설정에 정의되어 있는 root file system을 mount 합니다. 그 후 init process를 실행합니다.

## Phase 4

Init Process가 실행되면 runlevel program을 실행합니다. 최근의 배포판의 경우 이러한 역할을 포함한 더 확장된 기능을 가진 systemd로 대체되었습니다.

최종적으로 Runlevel에 해당하는 시스템 초기화를 진행합니다.
