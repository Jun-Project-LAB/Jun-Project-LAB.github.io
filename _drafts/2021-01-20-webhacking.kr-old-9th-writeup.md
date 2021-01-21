---
title: "Webhacking challenge(old) 9th Write-Up"
tags: [blindsqli, wargame]
categories: [sqli]
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


