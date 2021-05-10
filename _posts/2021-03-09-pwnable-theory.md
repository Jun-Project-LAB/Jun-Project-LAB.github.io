---
title: "Pwnable Theory"
tags: ["pwnable theory"]
categories: [System]
---

Linux System backgound theory for Pwnable
------------------------------------------

1. [Command](#command)
2. [Program](#program)

Pwnable을 위해 필요한 linux system 명령어나 file system 동작 등 기본적인 부분에 대해 정리하였습니다.

## **Command**

### uniq

파일 내용 중 중복 내용에 대해 검사할 필요가 있었는데 중복이 있음에도 불구하고 중복이 없는 것으로 count 되었습니다. 관련하여 찾아본 결과 정렬이 되어있는 경우에 정상적으로 동작하였으며 이를 통해 순차적으로 중복에 대해 검사한다는 것을 유추할 수 있었습니다.

> 예시 코드
```
cat note.txt | sort | uniq -c
```

* * *

### diff

두 개의 파일 중 변경된 부분에 대해 찾을 필요가 있었습니다. 물론 wargame 문제의 특성 상 상당히 많은 양의 내용이 저장되어 있었기에 수동으로 비교하기에는 많은 시간이 필요로 되었습니다. 문제 풀이 힌트에 diff 명령어가 있어서 해당 명령어를 찾아본 결과 두 개 파일 간의 차이점을 비교할 수 있었습니다.

사용법은 간단하게 "diff <file_1> <file_2>"로 사용할 수 있었습니다.

> 예시 코드
```
diff old_password new_password
```

> 출력 결과
```
42c42
< w0Yfolrc5bwjS4qw5mq1nnQi6mF03bii
---
>> kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd
```

위 출력 결과에서 < 로 시작한 부분이 file_1의 내용, > 이후로 있는 내용이 변경된 file_2의 내용이 됩니다. diff 명령어 외에도 다양한 비교 명령어에 관해서는 아래 링크를 참조하면 좋을 것 같습니다.

<http://www.incodom.kr/Linux/%EA%B8%B0%EB%B3%B8%EB%AA%85%EB%A0%B9%EC%96%B4/diff>

* * *

### git

git command를 주로 push용으로만 script를 작성해서 거의 쓸 일이 없다보니 새로 안 내용이 많았기에 간략하게 정리해보았습니다.

> commit log 확인
```
git log -> 일반적인 commit history 확인
git log -p -> commit history의 diff를 출력
git log -2 -> 최근 두 개의 결과 출력
```

> branch 확인
```
git branch -a
```

> branch 변경
```
git checkout <branch명>
```

## **Program**

### Vim

주로 파일 수정할 때만 사용하였는데 생각보다 다양한 기능이 있었습니다. 모든 command는 Shift + : 을 누른 상태를 가정하고 작성하였습니다.

- Bash shell 실행

```
set shell=/bin/bash
shell
```
