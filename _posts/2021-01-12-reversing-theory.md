---
title: "Reverse Engineering Basic Theory"
tags: [reversing, "basic theory", assembly]
categories: [reversing, assembly]
---

## **Reverse Engineering**

해킹에서 reverse engineering(이하 *reversing*)의 경우 완성된 결과물을 재해석하여 그 동작 과정을 밝혀내고, 취약점을 찾아내기 위해 사용됩니다.

Reversing의 심화적인 내용에 대해 알기 이전에 이를 이해하기 위해 필요한 assembly에 관한 기초적인 지식에 대해 정리하였습니다.

* * *

## **GPR**

**General-Purpose Registers**는 특별히 용도를 정해두지 않고 다양하게 사용할 수 있는 register입니다. x64의 범용 register는 총 16개로 원칙적으로는 용도가 정해져 있지 않지만, 관행적으로 그 쓰임새가 정해져 있는 경우도 있습니다.

* * *

- **rax**

rax는 primary accumulator로 입/출력 및 대부분의 산술 명령에서 사용됩니다.

- **rbx**

rbx는 base register로 indexed addressing에 사용할 수 있습니다.

*addressing에 관한 내용은 아래 링크 참조*
<https://www.cs.helsinki.fi/u/kerola/tito/koksi_doc/memaddr.html>

- **rcx**

rcx는 count register로 반복문을 사용할 때 횟수를 세는 역할을 합니다. c의 경우 보통 ++ 연산자를 이용해 수를 더해가며 횟수를 세는 반면에 assembly는 차감하는 방식으로 count를 한다고 합니다.

- **rdx**

rdx는 data register로 이 register 또한 입/출력에 사용합니다. 큰 값을 포함하는 곱셈, 나눗셈 연산에서는 rax와 같이 사용되기도 합니다.

* * *

## **Other Register**

다음 레지스터들은 범용 목적이 아닌 특정 동작을 위해 사용되는 레지스터들입니다.

- **rsp**

rsp는 stack pointer로 stack의 가장 윗 주소를 가리킵니다.

- **rip**

rip는 instruction pointer로 다음에 실행될 명령어가 위치한 주소를 가리키고 있습니다. 이는 컴퓨터 기초 이론에 대해 배울 때 나오는 program counter와 동일한 역할을 합니다.

* * *

## **Instructions**

Instructions은 Assembly에서 동작을 제어하기 위해 사용하는 명령어입니다.

## **Data Movement**

먼저 값을 레지스터나 메모리 주소에 옮기는 명령어들입니다.

- mov

src의 값을 dst로 옮기는 명령어이며 다음과 같이 사용할 수 있습니다.

>`mov dst, src`

- lea

lea의 경우 Load Effective Address로 src의 주소를 dst에 저장합니다.

>`lea dst, src`

* * *

## **Arthmetic Oprations**

산술 연산과 관련된 레지스터로 FLAGS 레지스터의 CF(carryup flag), OF(overflow flag), ZF(zero flag)와 관련이 있습니다.


