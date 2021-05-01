---
title: "Reversing.kr Easy Keygen Write-Up"
tags: [reversing, wargame]
categories: [reversing]
---

# Index

1. [How it Works](#how-it-works)
2. [Correction Condition](#correction-condition)
3. [Endcard](#endcard)


* * *

## How it Works

문제 파일을 다운로드하면 분석 대상인 exe 파일과 함께 README 파일이 존재하는 것을 확인할 수 있습니다. README의 내용을 확인해보면 다음과 같습니다.

```
ReversingKr KeygenMe

Find the Name when the Serial is 5B134977135E7D13
```

특정 Serial key를 제공해주고 이에 해당되는 "Name" 항목을 찾으라는 것을 알 수 있습니다. Name 요소에 대해 focus를 두고 프로그램을 실행해 보았습니다.

프로그램의 동작은 먼저 Name을 입력받고 그 다음 Serial을 입력받아 조건에 만족할 경우 Correction을 출력하고 그렇지 않을 경우 Wrong을 출력하는 것으로 확인되었습니다.

아마 Name과 Serial을 비교하는 부분이 있을 것으로 예상되어 IDA를 이용하여 해당 부분을 찾아보았습니다.

## Correction Condition

처음 분석을 시도할 때는 top-down으로 분석을 시도하였으나 엉뚱한 곳에서 시간을 많이 소비하게 되어 정답 조건으로부터 역순으로 비교 조건을 찾아나갔습니다. 문제 풀이에 직접적으로 도움이 된 코드는 다음과 같습니다.

```
.text:004010E2                 lea     esi, [esp+13Ch+var_C8]
.text:004010E6                 lea     eax, [esp+13Ch+var_12C]
.text:004010EA
.text:004010EA loc_4010EA:                             ; CODE XREF: sub_401000+108↓j
.text:004010EA                 mov     dl, [eax]
.text:004010EC                 mov     cl, dl
.text:004010EE                 cmp     dl, [esi]
.text:004010F0                 jnz     short loc_40110E
```

먼저 Stack의 `esp+13Ch+var_C8` 위치에는 입력받은 Name을 어떠한 형태로 재가공한 값이 저장되어 있었습니다. 그 밑에 줄에서 확인할 수 있는 `esp+13Ch+var_12C`에는 입력받은 Serial 값이 그대로 저장되어 있었습니다.

해당 값을 각각 esi, eax register에 저장 후 이 두 문자열을 비교하는 부분이 `cmp dl, [esi]` 내용이었습니다. 즉, 가공된 Name의 값과 Serial의 값이 일치해야 한다는 것인데 'a' 라는 문자를 자릿수를 늘려가며 넣어본 결과 아래와 같이 변경됨을 알 수 있었습니다.

```
첫 번째 문자 : +10
두 번째 문자 : -20
세 번째 문자 : -30
네 번째 문자 : +10
다섯 번째 문자 : -20
여섯 번째 문자 : -10
일곱 번째 문자 : +10
여덟 번째 문자 : -20
```

여기서 덧셈, 뺄셈되는 수는 16진수 기준이며 8자리임을 알 수 있던 근거로는 'a'라는 문자를 입력하였을 때 이 문자의 Hex 값인 61이 각각 한 자리씩 저장되는 것을 확인할 수 있었습니다. Serial의 문자열이 총 16자이므로 이를 2로 나누면 총 8개의 문자라는 것을 알 수 있었습니다.

위에서 확인한 조건과 하나씩 수작업으로 비교하며 얻어낸 정답은 `K3yg3nm3` 였습니다.

## Endcard

뭔가 문제를 너무 어거지로 푼 느낌이 강해서 정확한 풀이법을 찾아보았습니다. 찾아보기 이전에 입력받은 값을 치환하는 부분이 있지 않을까 하고 찾아보았는데 아래와 같은 code가 그 역할을 해주는 것을 확인할 수 있었습니다.

```
.text:00401077 cmp     esi, 3
.text:0040107A jl      short loc_40107E
.text:0040107C xor     esi, esi
.text:0040107E
.text:0040107E loc_40107E:                             ; CODE XREF: sub_401000+7A↑j
.text:0040107E movsx   ecx, [esp+esi+13Ch+var_130]
.text:00401083 movsx   edx, [esp+ebp+13Ch+var_12C]
.text:00401088 xor     ecx, edx
.text:0040108A lea     eax, [esp+13Ch+var_C8]
.text:0040108E push    ecx
.text:0040108F push    eax
.text:00401090 lea     ecx, [esp+144h+var_C8]
.text:00401094 push    offset aS02x                    ; "%s%02X"
.text:00401099 push    ecx
.text:0040109A call    sub_401150
.text:0040109F add     esp, 10h
.text:004010A2 inc     ebp
.text:004010A3 lea     edi, [esp+13Ch+var_12C]
.text:004010A7 or      ecx, 0FFFFFFFFh
.text:004010AA xor     eax, eax
.text:004010AC inc     esi
.text:004010AD repne scasb
.text:004010AF not     ecx
.text:004010B1 dec     ecx
.text:004010B2 cmp     ebp, ecx
.text:004010B4 jl      short loc_401077
```

Code를 확인해보면 반복문으로 구성되어 있는데 처음 반복문을 시작할 때 esi의 값이 3 이상일 경우 esi의 값을 0으로 초기화 후 진행해주는 것을 확인할 수 있었습니다.

ecx에 옮기는 값인 `esp+esi+13Ch+var_130` 에는 순서대로 10h, 20h, 30h의 값이 존재하고 있었습니다. 그 다음으로 edx에 옮기는 값인 `esp+ebp+13Ch+var_12C` 위치에는 입력받은 Name 값이 존재하고 있었습니다.

그 다음 code를 살펴보면 ecx와 edx를 xor 연산을 수행한 후 ecx에 저장하는 것을 확인할 수 있었습니다. 그 뒤로 4개의 인자를 push 후 call을 통해서 어떤 함수를 호출하였는데 memory의 변화를 통해 추측컨데 xor의 연산 결과를 어떤 변수에 저장하는 과정으로 예상되었습니다.

함수가 종료된 후 ebp를 1 증가시키는데 가장 처음 ebp의 값은 0에서부터 시작하는 것을 확인할 수 있었습니다. ecx를 0FFFFFFFFh와 or 연산을 해주는데 결과적으로 ecx를 0FFFFFFFFh로 초기화한 것과 같은 결과가 됩니다.

다음으로 eax를 0으로 초기화하고 esi의 값을 1 증가 시킵니다. 그 후 `repne scasb` 명령을 실행하는데 eax의 값은 0이고 비교 대상인 edi의 값은 입력받은 Name의 값인데 이 둘을 비교하려는 목적보다는 초기화한 ecx를 1씩 감소하기 위한 목적으로 사용된 것 같았습니다.

반복이 끝난 후 not을 통해 ecx를 변환하면 9가 되고 `dec ecx` 까지 실행하면 ecx에 8이라는 값이 남게 됩니다. '8' 이란 값의 의미를 생각해보면 Name의 정답에 해당하는 글자의 자릿수임을 알 수 있습니다. 이후 ebp와 ecx를 비교하여 ebp가 ecx보다 작은 동안 위 과정을 반복합니다.
