---
title: "Pwnable Theory"
tags: [pwnable, linux, "basic theory"]
categories: [system, "pwnable theory"]
---

Linux System backgound theory for Pwnable
------------------------------------------

Pwnable을 위해 필요한 linux system 명령어나 file system 동작 등 기본적인 부분에 대해 정리하였습니다.

## **Command**

### uniq

파일 내용 중 중복 내용에 대해 검사할 필요가 있었는데 중복이 있음에도 불구하고 중복이 없는 것으로 count 되었습니다. 관련하여 찾아본 결과 정렬이 되어있는 경우에 정상적으로 동작하였으며 이를 통해 순차적으로 중복에 대해 검사한다는 것을 유추할 수 있었습니다.

> 예시 코드

```unix
cat note.txt | sort | uniq -c
```

* * *
