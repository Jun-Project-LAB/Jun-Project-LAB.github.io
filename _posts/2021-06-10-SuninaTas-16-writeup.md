---
title: "SuninaTas 16th Write-Up"
tags: [Network, wargame]
categoriess: [Network]
---

# Index

1. [문제 분석](#문제-분석)
2. [Traffic 분석](#traffic-분석)
3. [Exploit](#exploit)

* * *

## 문제 분석

문제에 접속하면 다음과 같은 문구와 함께 압축파일 하나를 다운로드 받을 수 있습니다.

> Can you find the password for a member of SuNiNaTaS.com?

다운로드 받은 파일을 압축해제 하면 pcap 확장자를 가진 파일을 확인할 수 있고 위 문구와 종합하여 생각했을 때 dump 된 traffic을 분석하여 특정 계정의 password를 알아낼 필요가 있을 것으로 보입니다.

## Traffic 분석

Wireshark를 사용하여 해당 파일을 열어보면 생각보다 많은 패킷들이 dump 되어 있는 것을 확인할 수 있습니다. Traffic의 접근은 HTTP 패킷을 찾은 뒤, suninatas.com 도메인과 관련된 패킷을 찾는 순서로 진행하였습니다.

해당되는 패킷을 찾은 뒤 follow http stream을 눌러 확인한 결과 메인 페이지 접속부터 로그인 시도와 관련된 stream임을 확인할 수 있었습니다.

![Suninatas_main_dump](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/suninatas_16_main.png?raw=true)

Main page에 요청에 대한 응답으로 로그인과 관련된 javascript 함수를 확인할 수 있었는데 이를 통해 `mem_action.asp` 파일을 통해 로그인을 진행한다는 것을 알 수 있었습니다.

```javascript
function fnLogin(){
		var cont = document.login_page;

		if (cont.Hid.value == ""){
			alert("Input id!");
			cont.Hid.focus();
		}else if (cont.Hpw.value == ""){
			alert("Input password!");
			cont.Hpw.focus();
		}else{
			cont.action = "../member/mem_action.asp";
			cont.target = "ifr_mem";
			cont.submit();
		}
	}
```

따라서 HTTP stream 중 `mem_action.asp` 파일을 요청을 타겟으로 분석을 계속하였습니다.

## Exploit

위에서 얻은 결과들을 기반으로 Traffic을 분석한 결과 다음과 같은 id, password 조합들을 발견할 수 있었습니다.

- `Hid=suninatas&Hpw=suninatas`

- `Hid=blackkey&Hpw=blackkey`

- `Hid=ultrashark&Hpw=sharkpass01`

- `Hid=ultrashark&Hpw=%3Dsharkpass01`

- `Hid=ultrashark&Hpw=%3DSharkPass01`

한 번의 로그인 시도가 아닌 여러 번의 시도가 있었던 것을 확인할 수 있었고 마지막 시도 후 추가적인 시도가 없었음을 통해 로그인에 성공한 것이라고 판단하였습니다. 이후 password를 찾으라고 하였기에 발견한 password가 flag인줄 알았으나 아니었고, 해당 계정을 통해 실제 suninatas page에서 로그인을 시도하자 AUTH KEY를 획득할 수 있었습니다.
