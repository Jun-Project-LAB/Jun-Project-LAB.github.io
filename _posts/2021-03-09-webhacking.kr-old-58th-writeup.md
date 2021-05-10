---
title: "Webhacking.kr challenge(old) 58th Write-Up"
tags: [javascript, wargame]
categories: [Web]
---

Webhacking.kr 58 Write-Up
-------------------------

## **문제 분석**

문제를 접속하면 페이지의 하단에 일종의 채팅 프로그램과 같이 입력창과 send 버튼이 있습니다. 처음 아무런 문자열이나 입력 후 send 버튼을 눌렀을 때 "command not found" 라는 문구가 출력되는 것을 보아 일종의 interactive shell이라고 판단하였고 기본적인 linux 명령어를 시도해보았습니다. 정상적으로 결과를 얻을 수 있던 명령어는 **"ls, help, id"**가 있었으며 help 명령어를 통해 flag라는 명령어가 사용 가능한 것을 알 수 있었습니다.

* * *

## **문제 해결 조건 탐색**

flag 명령어를 입력할 경우 아래와 같이 admin 계정만 사용할 수 있다고 출력되는 것을 확인할 수 있었습니다.

![flag_command](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/webhacking_kr_58_help.png?raw=true)

따라서 admin 계정의 권한을 얻을 필요가 있다고 판단되었고 page source를 확인한 결과 기본적으로 username이 guest로 되어 있는 것을 확인할 수 있었습니다.

![javascript](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/webhacking_kr_58_username.png?raw=true)

* * *

## **문제 해결**

처음에는 guest로 되어있는 username을 admin으로 수정 후 시도하였으나 flag를 얻는데 실패하였습니다.

따라서 통신의 중간 과정에서 이를 임의로 수정할 필요가 있다고 생각하였고 BurpSuite를 사용하여 문제 풀이를 시도하였습니다.

BurpSuite를 사용하여 flag 명령을 사용하였을 때의 패킷을 확인하던 중 아래와 같이 권한을 확인하는 것을 발견할 수 있었습니다.

![query](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/webhacking_kr_58_query.png?raw=true)

위 부분에서 guest 부분을 admin으로 수정 후 패킷을 전달할 경우 아래와 같은 응답이 돌아오는 것을 확인할 수 있었습니다.

![flag](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/webhacking_kr_58_flag.png?raw=true)
