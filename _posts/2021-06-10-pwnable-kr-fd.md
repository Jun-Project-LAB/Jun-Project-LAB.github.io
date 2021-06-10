---
title: "Pwnable.kr fd Write-Up"
tags: [wargame]
categories: [System]
---

# Index

1. [문제 분석](#문제-분석)
2. [프로그램 분석](#프로그램-분석)
3. [Exploit](#exploit)

* * *

## 문제 분석

문제에 접속하면 setuid가 설정된 fd 프로그램과 소스코드, flag 파일을 확인할 수 있었습니다.

```
fd@pwnable:~$ ls -l
total 16
-r-sr-x--- 1 fd_pwn fd   7322 Jun 11  2014 fd
-rw-r--r-- 1 root   root  418 Jun 11  2014 fd.c
-r--r----- 1 fd_pwn root   50 Jun 11  2014 flag
```

우선 프로그램의 동작을 간단히 확인해보면 다음과 같습니다.

> 어떠한 인자도 없이 단독 실행할 경우

```
fd@pwnable:~$ ./fd
pass argv[1] a number
```

argv[1]의 인자는 number라는 문구가 출력되는 것을 확인할 수 있습니다. 따라서 임의의 숫자를 전달하여 실행해보았습니다.

* * *

> argv[1] 인자에 숫자를 지정하여 실행할 경우

```
fd@pwnable:~$ ./fd 1
learn about Linux file IO
```

Linux의 file I/O에 대해 공부하라는 문구가 출력됩니다.

위의 결과들을 통해서 file descriptor와 관련된 문제임을 예상할 수 있습니다.


## 프로그램 분석

프로그램 외적으로 확인할 수 있는 부분은 끝났다고 생각하여 gdb를 사용하여 분석을 시도하였습니다. 추가적으로 알아낸 내용들은 다음과 같습니다.

1. argv[1]의 값이 1보다 클 경우(2 이상일 경우) "learn about Linux file IO" 출력 부분으로 점프 후 종료됨.
2. 특정 조건을 만족할 때, read를 통해 값을 입력받는데 이때 이 값이 `LETMEWIN` 일 경우 `/bin/cat flag` 명령을 수행함.

즉, 특정 조건이 무엇인지에 대해 찾아내 LETMEWIN 이라는 문자열을 입력할 경우 flag를 얻을 수 있다는 것을 알 수 있었습니다.

## Exploit

Exploit의 경우 자력으로 성공하진 못했습니다. 대략적으로 file descriptor의 역할은 알고 있다고 생각하였으나 코드를 분석하는데 있어서 놓쳤던 부분이 몇 가지 있었습니다. 풀이 영상을 통해 새로 알게된 사실에 대해 추가해보았습니다.

> read 함수와 FD의 관계

read 함수를 통해 사용자로부터 값을 입력받을 때 왜 fd를 입력하는 곳에 0을 지정하는지 제대로 이해하지 못하고 있었습니다. fd에서 0은 결국 stdin을 의미하는 것이었고 파일로부터 입력받는 것이 아닌 직접 콘솔을 사용하여 입력받기 위함인 것을 알 수 있었습니다.
