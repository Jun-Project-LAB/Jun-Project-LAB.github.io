---
title: "Suninatas 22 Write up"
categories: SQLi
tags: [blindSQLi, wargame, sunitas]
---

Suninatas 22 Write-Up
---------------------

# 문제 분석

문제에 접속하면 아래와 같은 로그인 창과 필터링 되는 단어들을 확인할 수 있었습니다.
필터링 내용들을 살펴보면 여태까지 봤던 문제 중 가장 많은 것을 볼 수 있었습니다.

![Suninatas_22_Main](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/suninatas_22_main.png)

문제의 풀이 조건에 대해 알 필요가 있으므로 먼저 개발자 도구를 통해 소스코드를 확인한 결과
 주석에 admin 계정으로 로그인하라는 조건이 있는 것을 찾을 수 있었습니다.

![Suninatas_22_source](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/suninatas_22_source.png)

따라서 이 문제의 flag는 admin 계정의 password 혹은 로그인 할 경우 얻을 수 있는 것으로 짐작할 수 있었습니다.

또한 테스트하기 위한 계정으로 guest/guest 계정이 있다고 하여 해당 계정을 이용하여 로그인 해보았습니다.
