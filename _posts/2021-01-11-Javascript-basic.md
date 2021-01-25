---
title: "JavaScript Basic theory"
tags: [javascript, "basic theory"]
categories: ["programing theory", javascript]
---

## **Javascript**

javascript에서 사용되는 대표적인 data type의 경우 7가지를 예로 들 수 있습니다. 여기서 javascript와 다른 programing language의 차이점을 보면 c 혹은 python과 같은 언어의 경우 data type에 맞는 선언자가 있으나 javascript의 경우 그렇지 않음을 알 수 있습니다.

***물론 이는 변수를 선언하는 방법에 있어서 차이가 있는 것 때문이기도 합니다.***

* * *

## **Data Type**

```
- Number
 : 정수 그리고 흔히 int type에 해당된다.
- String
 : char 혹은 string을 모두 포함하는 포괄적인 개념의 문자/문자열
- Boolean
 : TRUE | FALSE
- Undefined
 : 변수의 값이 비어 있는, 아직 정의되지 않은 상태
- Null
 : Undefined와 같은 empty value
   Null의 경우 typeof를 통해 확인하였을 경우 object로 출력됨.
- Symbol
 : 최근에는 자주 사용되지 않음
- Bigint
 : 더 넓은 범위의 int type
```

* * *

## **변수 선언**

javascript에서 변수를 선언할 때 사용할 수 있는 keyword는 let, var, const가 있습니다. 앞서 data type을 설명할 때 얘기하였듯이 javascript는 dynamic type이기 때문에 data type을 적는 것이 아닌 위 keyword를 사용하게 됩니다.

- let & var

let과 var는 많은 비슷한 특징을 가지고 있습니다. 변수 선언 시 초기값을 지정하지 않아도 사용할 수 있으며 할당되어 있는 값을 변경할 수도 있습니다. let은 var의 기능을 개선하기 위해 추가되었는데 var의 경우 이미 선언된 변수를 재선언하여도 오류가 발생하지 않게 되어 복잡한 프로그램을 설계할 시에 단점이 될 수 있었습니다. 따라서 let은 이미 선언된 변수에 대해 재선언을 불가하도록 개선하였습니다.

다음으로 가장 특징적인 차이는 scope의 차이입니다. var는 function-scope, let과 const의 경우 block-scope입니다. 이는 쉽게 이해하면 global variables, local variables로 이해할 수 있습니다. 

- const

const의 경우 let과 마찬가지로 block-scope에 해당하는 변수 선언 방법이지만 변수를 선언할 때 **반드시 초기값이 있어야 하며**, **재할당, 재선언이 불가**하다는 특징이 있습니다. 즉, 상수의 개념으로 이해할 수 있습니다.

* * *

## **Template Literals**

일반적인 프로그래밍에서 문자 혹은 문자열을 표현하는 방법은 single quotes('') 혹은 double quotes("")가 있습니다. Javascript에서는 backtick(\`\`)을 사용하여 더 다양한 문자열을 표현하는 것 또한 가능합니다.

multi-line 출력을 할 때 backtick을 사용할 경우 line feed 문자를 사용할 필요 없이 그저 Enter를 입력하는 것만으로 표현이 가능합니다. 아래 두 가지 경우 모두 동일한 결과를 출력합니다.

```javascript
console.log("hi\nI'am Human!");

console.log(`hi
I'am Human!`);
```

또한 backtick을 사용하여 template literals를 이용할 수 있습니다. 쉽게 이해하자면 Python의 format() 함수와 비슷한 형식으로 사용이 가능합니다.

```javascript
const age = 20;
const name = "Yena";

const helloText = `Hi my name is ${name} and i'am ${age} years old~`

console.log(helloText);

/*
Hi my name is Yena and i'am 20 years old~
*/
```

* * *

## **Type Conversion & Coercion**

Javascript는 기본적으로 자동적으로 형변환을 지원합니다. 그러나 때에 따라서 자동적으로 변환되는 결과가 아닌 수동적으로 변경할 필요가 있을 수도 있는데 이때, Number() 함수와 String() 함수를 사용할 수 있습니다.

Number 함수의 경우 String을 Number 형으로 변환 시켜주며 String 함수의 경우 반대로 Number 형을 String 형으로 변환시켜줍니다. 이때 기존 변수에 할당되어 있는 값에는 영향을 끼치지 않습니다.

또한 Javascript는 + 문자를 사용하여 문자열을 이어붙여 출력할 수 있는데 이때 만약 String과 Number가 같이 있을 경우에는 무조건 Number를 String으로 변경하여 실행합니다. 그 외에 -, \*, /, <, > 와 같은 산술, 비교 연산자 등은 String을 Number로 변환하여 실행합니다.

* * *


