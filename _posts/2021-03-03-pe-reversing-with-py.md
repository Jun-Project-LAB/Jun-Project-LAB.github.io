---
title: "Reversing PE file by Python"
tags: [PE]
categories: [Reversing]
---

Reversing with Python
---------------------

Reversing wargame 문제를 해결할 때 PE 파일 내에 존재하는 String Table을 사용할 필요성이 있는 문제가 있었습니다. 특정 패턴이 있는 경우 해당 패턴을 찾아내서 반복문을 통하여 table을 생성하면 되지만, 이 table의 경우 패턴을 찾을 수 없었고 이를 자동화할 방법을 찾지 못하였습니다.

따라서 우선 해당 table을 참조하며 수동으로 문제를 해결하였으나 이후에도 이런 상황이 있을 수 있기에 이 문제에 대한 다양한 풀이 스크립트를 찾아보았습니다. 여러 방법 중에서 가장 객관적으로 이해할 수 있는 스크립트를 찾았고 동작 과정과 사소하게 발생했던 오류와 관련하여도 정리하여 보았습니다.

## **Background theory**

Reversing에 사용하는 프로그램은 IDA를 기준으로 작성하였습니다. 이 방법을 사용하기 위해 꼭 IDA를 사용해야 하는 것은 아니며 추출하고자 하는 데이터의 file offset을 확인할 수 있다면 상관없습니다.

우선 IDA를 통하여 찾고자 하는 data의 시작점을 확인할 필요가 있습니다. 아래 사진을 참조하면 data section의 00007FF6270A3020부터 시작하며 첫 값은 c가 할당되어 있는 것을 확인할 수 있습니다.

![IDA_View_1](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/IDA_View_1.png?raw=true)

시작 값에 커서를 올려둔 뒤 창의 하단을 확인해보면 아래와 같이 Status를 확인할 수 있는데 이때, 맨 앞의 값이 File Offset에 해당됩니다. 

![IDA_View_bot](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/IDA_View_bot.png?raw=true)

이를 HxD를 통해서 해당 File Offset에 찾아가보면 일치하는 것을 확인할 수 있습니다.

![HxD](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/HxD.png?raw=true)

* * *

## **Python Code**

Data의 file offset을 확인하였으면 Python의 read 함수를 사용하여 data를 불러올 수 있습니다. 처음 reference로 잡은 스크립트가 r 옵션을 통해 파일을 읽어왔는데 이로 인해 decoding과 관련된 오류가 발생하였습니다. 이를 해결하기 위해서 rb 옵션 즉, byte mode로 open하여 해결할 수 있었습니다.

 Data를 추출하는 과정은 file offset을 16진수 표기법으로 표현한 slicing을 통해 가능하였습니다. 위의 내용을 예시로 들면 시작 위치는 0x2400이 되며 끝 위치는 (마지막 주소 - 시작 주소)가 됩니다. 시작 위치를 start라는 변수에 저장했다고 가정할 경우 아래와 같은 식으로 표현할 수 있습니다.

```python
string = data[start:start+n]
```

위 예시에서 n의 경우 마지막 주소와 시작 주소의 차가 됩니다.
