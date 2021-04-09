---
title: "Webhacking.kr challenge(old) 42th Write-Up"
tags: [wargame, javascript]
categories: [javascript]
---

Webhacking.kr 42 Write-Up
-------------------------

## **문제 분석**

문제에 접속하면 no, subject, file의 목차를 가지고 있는 표를 확인할 수 있었습니다.

![Webhacking.kr 42th Main](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/webhacking_kr_42_main.png?raw=true)

두 개의 다운로드 링크 중 subject가 test로 되어있는 항목의 경우 정상적으로 다운로드가 가능하였으나 flag.docx 링크의 경우 alert 문구가 출력되며 다운로드가 막히는 것을 확인할 수 있었습니다.

## **문제 해결**

문제의 해결을 위해서 처음에는 wget을 사용하여 file을 직접 다운로드하려고 시도했으나 링크 자체가 alert을 출력하는 javascript로 되어있는 것을 확인할 수 있었습니다. 반면에 test.txt file의 링크는 형식을 보았을 때 base64로 encoding 되어 있었는데, 이를 decoding 해보니 test.txt 라는 문자열이 출력되었습니다. 따라서 요청 형식에 맞춰 flag.docx를 base64로 encoding 후에 아래와 같이 기존 javascript 명령어가 있던 부분을 바꿔주었습니다.

```
"?down=ZmxhZy5kb2N4="
```

내용 적용 후 download를 클릭하여 flag.docx 파일을 다운로드 받을 수 있었고, 해당 파일을 열자 flag를 확인할 수 있었습니다.
