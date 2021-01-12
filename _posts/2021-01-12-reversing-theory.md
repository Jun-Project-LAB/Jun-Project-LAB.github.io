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


### Unary Instructions

- inc, dec

dst의 값을 1 증가시키거나 감소시키며 c 언어로 생각하였을 때 전치 연산자와 같습니다.

>`inc dst`
>`dec dst`

- neg

dst에 들어있는 값의 부호를 바꿉니다. (2의 보수)

>`neg dst`

- not

dst에 들어있는 값의 비트를 반전해줍니다.

>`not dst`

* * *

### Binary Instructions

- add, sub

각각 덧셈, 뺄셈 연산을 수행합니다.

>`add dst, src`
>`sub dst, src`

- imul

dst에 들어있는 값에 src를 곱합니다.

>`imul dst, src`

- and, or, xor

dst에 들어있는 값과 src 간에 해당되는 논리 연산 후 결과를 dst에 저장합니다.

>`and dst, src`
>`or dst, src`
>`xor dst, src`

* * *

### Shift Instructions

- shl, shr

dst의 값을 n 만큼 왼쪽이나 오른쪽으로 shift 합니다. 이때의 shift는 logical shift이기 때문에 빈 bit에는 0이 채워집니다.

>`shl dst, n`
>`shr dst, n`

- sal, sar

shl, shr과 동일하게 shift 기능을 가지고 있으나 부호가 보존됩니다.

>`sar dst, n`
>`sal dst, n`

* * *

### Conditional Operations

다음 명령어들은 분기문이나 조건문과 같이 코드의 실행 흐름을 제어하는 것과 밀접한 관련이 있습니다. 특히 실행 흐름을 정할 때, flags 레지스터의 각종 플래그와 밀접한 관련을 가지고 동작합니다.

- test

test의 경우 and와 마찬가지로 논리 연산을 하지만 그 결과값을 피연산자에 저장하지 않는다는 특징을 가지고 있습니다. 대신 이 연산 결과가 flags register에 영향을 미치는데 결과가 음수일 경우 SF가 1이 되고, 결과가 0일 경우 ZF를 1로 만듭니다.

>`test dst, src`

- cmp

cmp 또한 sub과 마찬가지로 뺄셈을 진행하지만 그 결과값이 피연산자에 저장하는 것이 아닌 flags register에 영향을 미칩니다. 만약 dst = src일 경우 ZF=1, CF=0이 되고, dst < src일 때에는 ZF=0, CF=1, 반대로 dst > src일 경우 ZF=0, CF=0이 됩니다.

>`cmp dst, src`

- jmp(jump) & jcc(jump if condition is met)

jmp와 jcc 모두 피연산자가 가리키는 곳으로 점프하는 동작은 같지만 jmp의 경우 무조건 점프하고 jcc의 경우 조건에 따라 수행 여부가 달라집니다.

jcc가 사용하는 조건은 flags register의 flag와 관련이 있기 때문에 명령을 수행하기 전에 어떤 산술 연산을 하거나 test, cmp와 같은 연산을 수행한 결과를 바탕으로 명령의 동작 여부를 결정합니다.

jcc에 대해 자세히 설명하기 이전에 jmp의 경우 아래와 같이 간단하게 사용할 수 있습니다. location에는 이동할 주소가 입력됩니다.

>`jmp location`

jcc는 앞서 본 명령어들과 다르게 명령어의 이름이 아닌 여러 명령어를 묶어서 부르는 표현입니다. jcc에는 많은 종류의 명령어들이 있는데 아래 링크에서 자세히 확인할 수 있습니다.

<https://www.felixcloutier.com/x86/jcc>

* * *

### Stack Operations

다음으로는 stack과 관련된 명령어입니다. 프로그래밍을 하는 것에 있어서 지역 변수를 사용하는 것은 거의 필수로 자리잡았는데 이때 지역 변수들은 stack에 저장됩니다. stack은 register가 아닌 memory에 준비되는데 새로운 함수가 시작될 때 스택이 준비되는 것을 Function Prologue, 함수가 종료될 때 스택이 종료되는 것을 Function Epliogue라고 합니다. 이 과정은 스택의 가장 윗부분을 가리키는 rsp register와 밀접한 관련이 있습니다.

stack에 새로운 데이터를 추가할수록 stack은 점점 길어지는데 이름에서 알 수 잇듯이 가장 최근에 들어온 data를 이전 data 위에 쌓아가는 방식으로 길어지게 됩니다. 이때 rsp는 stack의 가장 위쪽을 가리키므로 마지막으로 데이터가 추가된 위치를 저장하는 register입니다.

물론 Assembly의 특성 상 architecture에 따라 stack의 동작이 다르게 나타날 수 있는데 새로운 data가 추가될 때 더 높은 메모리 주소에 저장되는 경우도 있고, 이와 반대로 더 낮은 메모리 주소에 저장되는 경우도 있습니다. Intel x86-64 architecture의 경우 후자의 방식에 해당됩니다.

- push & pop

push와 pop은 stack에 데이터를 추가하거나 뺄 때 사용합니다. 명령어를 사용할 경우 다음과 같이 사용할 수 있습니다.

>`push rdi`
>`pop rdi`

push와 pop의 과정을 자세하게 분석하면 다음과 같이 볼 수 있습니다.

> push
```
sub rsp, 8
mov [rsp], rdi
```

> pop
```
mov rdi, [rsp]
add rsp, 8
```

* * *

## **Procedure Call instructions**

마지막으로 함수를 호출하는 명령어와 종료하는 명령어입니다.

- call

함수를 실행할 때 call 명령어를 사용합니다. call의 경우 피연산자로 실행할 함수의 주소를 받으며 함수 종료 후 돌아올 주소 즉, return address를 stack에 push 후 호출된 함수의 주소로 jmp하는 것과 같은 원리로 동작합니다.

>`call location`

- ret

호출된 함수가 마지막으로 사용하는 명령어로 함수를 종료한 뒤 return address로 복귀합니다. ret은 별도의 피연산자가 필요하지 않습니다.

>`ret`


