---
title: "Webhacking.kr challenge(old) 4th Write-Up"
tags: [wargame, hash]
categories: [Crypto]
---

Webhacking.kr 4 Write-Up
------------------------

## **문제 분석**

문제에 접속하면 아래 사진과 같이 40자의 문자열과 제출 칸이 있는 것을 확인할 수 있었습니다. 영어, 숫자가 혼합된 40자의 문자열은 hash 함수 중 sha-1의 특징이므로 hash와 관련한 것임을 알 수 있었습니다.

![Webhacking.kr_4_Main](https://raw.githubusercontent.com/Jun-Project-LAB/Jun-Project-LAB.github.io/main/_image/webhacking_kr_4_main.png)

* * *

## **소스코드 확인**

view-source를 통해 코드를 확인한 결과 다음과 같은 PHP 코드를 볼 수 있었습니다.

```php
<?php
  sleep(1); // anti brute force
  if((isset($_SESSION['chall4'])) && ($_POST['key'] == $_SESSION['chall4'])) solve(4);
  $hash = rand(10000000,99999999)."salt_for_you";
  $_SESSION['chall4'] = $hash;
  for($i=0;$i<500;$i++) $hash = sha1($hash);
?>
```

먼저 코드의 첫 줄에 sleep(1) 함수를 사용하여 brute force 공격을 방지하는 것을 확인할 수 있었습니다. 또한 페이지를 요청할 때마다 hash 값이 바뀌었기 때문에 페이지에 직접적으로 공격하는 것이 아닌 다른 방법을 사용할 필요가 있었습니다.

원본값의 경우 10000000부터 99999999 중 random으로 값을 생성한 후 "salt_for_you"라는 문자열의 salt를 덧붙인 것이 원본값으로 사용되었습니다.

그 후 원본값을 sha1 함수로 500번 반복하여 hashing 한 것을 확인할 수 있었습니다. hash의 경우 단방향 함수이기에 이론적으로 복호화가 불가능하지만 a라는 글자를 hashing 하였을 때 무조건 같은 값이 나오는 hash의 특징을 이용하여 rainbow table을 생성 후 이로부터 1:1 매칭을 통해 원본값을 구하는 것이 가능합니다.

* * *

## **Script 작성**

기본적으로 숫자의 범위가 넓을 뿐더러 500번의 hashing을 수작업으로 하는 것은 비효율적이기 때문에 python을 이용한 자동화 스크립트를 사용하였습니다. 많은 연산량을 필요로 하는 만큼 threading을 통하여 다중처리가 가능하도록 구성하였습니다.

```python
#!/usr/bin/python

from hashlib import sha1
import threading

ref = str(input("Enter Your Reference: "))
salt = "salt_for_you"
index = int(input("Enter Index Number: "))
s_num = 10000000
e_num = 99999999+1
rge = int((e_num-s_num)/index)

def execute(index, ref, salt, start, rge):
    for a in range(start+rge*(index-1), start+rge*index):
        hs = str(a)+salt
        result = hs
        for i in range(500):
            result = sha1(result.encode('utf-8')).hexdigest()
        with open("/tmp/result{}.txt".format(index), 'a') as file:
            file.write("hash is : {}, key is : {}\n".format(result, hs))
        if str(result) == str(ref):
            print("key : {}".format(hs))

if __name__ == '__main__':
    for i in range(1, index+1):
        calc = threading.Thread(target=execute, args=(i, ref, salt, s_num, rge))
        calc.start()
```

python script를 사용하여도 연산 도중에 로그인 세션이 만료될 경우 다시 로그인을 해야하며 이로인해 hash 값이 변하게 되었기 때문에 연산과 동시에 file에 저장하여 만약 값이 바뀌었을 경우에도 해당 file로부터 검색하여 시간을 단축하고자 하였습니다.

파일의 경우 정상 동작의 확인을 위해 thread 별로 다르게 저장하였으며 모든 파일로부터 hash값을 찾기 위한 방법으로는 bash script를 사용하였습니다.

```bash
for a in {1..100};do grep 3a8f2f1cfec5fae6ffb3dbf2a2f29166f136e5a4 /tmp/result$a.txt;done
```

일치하는 값을 찾은 후 해당 값을 입력 후 제출하면 문제를 해결할 수 있습니다.
