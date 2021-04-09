---
title: "Webhacking.kr challenge(old) 36th Write-Up"
tags: [wargame, system]
categories: [system]
---

Webhacking.kr 36 Write-Up
-------------------------

## **문제 분석**

가장 처음 문제에 접속하면 무언가 설명문을 확인할 수 있습니다. 내용은 다음과 같습니다.

> ```
While editing index.php file using vi editor in the current directory, a power outage caused the source code to disappear.
Please help me recover.
```

내용을 간단하게 해석하면 vi editor를 이용하여 index.php 파일을 작성 중 정전으로 인하여 source code가 사라졌다고 합니다. 해당 파일의 복구를 도와달라는 말이 있는 것을 보아 해결 조건과 관계가 있음을 추측할 수 있었습니다.

## **문제 해결 방법**

문제의 난이도 자체는 기존에 vim editor를 사용해 보았다면 아주 쉬운 난이도라고 생각되었습니다. 우선 대상 파일의 경우 위에서 언급된 index.php 파일이 될텐데 vim editor는 파일을 open하고 있는 동안 "**.\<filename\>.swp**" 라는 임시 파일을 생성하고 있습니다.

이 임시파일은 파일을 close 함과 동시에 자동으로 삭제됩니다. 그러나 정상적인 종료가 아닌 정전과 같은 비정상 종료가 발생할 경우 해당 임시파일은 삭제되지 않고 기존의 파일을 open 할 때 recovery option을 출력해주는데 이를 통해 복구가 가능합니다.

그러나 현 문제의 경우 해당 시스템에 접속하여 있는 것이 아니기에 우선 url에 임시파일의 형식을 입력해보자 swap 파일을 다운로드 받을 수 있었습니다. vim의 옵션을 찾아보니 -r 옵션을 추가하여 직접 swp 파일을 open하면 복구된 파일로 출력된다는 것을 확인할 수 있었고 flag를 확인할 수 있었습니다.

정작 flag의 내용이 nano editor가 있는 것이 아이러니한 점 같습니다.
