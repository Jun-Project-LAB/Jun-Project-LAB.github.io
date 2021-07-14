---
title: "Background Theory in Pwnable - libc & glibc"
tags: ["pwnable theory"]
categories: ["System"]
---

# Index

1. [Purpose](#purpose)
2. [Episode 1](#episode-1)
3. [Reference](#reference)

* * *

## Purpose

**Background Theory in Pwnable** 게시글은 series 형태로 작성될 예정이며 pwnable을 공부하며 중요하게 짚고 넘어가지 않지만 나중에 쓸모있을 것 같은 내용을 짚고 넘어갈 예정입니다.

다양한 기법에서 바탕이 되는 토막 상식 등을 작성하는 만큼 이 게시글들이 모였을 때 하나의 큰 틀을 만들어낼 수 있도록 작성할 것입니다.

## Episode 1

여러 문제를 접하다보면 같은 프로그램을 대상으로 동일한 공격 기법을 적용해도 다른 결과를 얻은 때가 있었습니다. 이는 컴파일러, os의 version 등에 따라 linking 되는 library의 종류가 다르기 때문이라고 예상할 수 있었습니다.

Library에 대해 알아보던 중 가장 첫 번째 의문이 생긴 것은 libc와 glibc의 차이였습니다.

### libc

libc는 `strcpy()`와 같은 standard C function이나 `getpid()`와 같은 POSIX function(system call)이 구현되어 있는 Library 파일입니다. 물론 libc에 모든 함수가 구현되어 있는 것은 아니며 여러 library 파일에 걸쳐 구현되어 있습니다.

libc는 single library file로 `.so`와 `.a` 두 가지 version을 사용할 수 있습니다.

### glibc

glibc는 GNU libc의 약자로 libc 뿐만 아닌 libm, libthread와 같은 다른 core library 또한 같이 제공해주는 project의 일종입니다. libc와 다른 library가 아닌 glibc에서 참조하는 파일 중 하나가 libc임을 알 수 있었습니다.

glibc의 경우 Linux systems에서 보편적으로 사용되는 표준 라이브러리라고 합니다.

## Reference

- <https://stackoverflow.com/questions/11372872/what-is-the-role-of-libcglibc-in-our-linux-app>
