---
title: "Dreamhack Web hacking Lecture(Server-side)"
tags: ["basic theory", "server-side basic", "sql injection", "os command", ssti, ssrf]
categories: ["web theory"]
---

## Server-Side Basic

Dreamhack의 Web hacking 강의 중 Server-side basic part 수강 내용을 바탕으로 정리하였습니다.

* * *

## **Injection**

**Injection**의 경우 주입이라는 의미를 가지고 있으며, 공격자가 사용자의 입력값에 변조된 값을 입력하였을 때 웹 애플리케이션의 처리 과정에서 
이를 문법적인 데이터 혹은 구조로 해석하여 발생되는 취약점을 의미합니다. Injection의 종류는 다음과 같은 것들이 있습니다.

```
- SQL Injection
 : 대표적인 Injection 기법 중 하나로 Database 관리에 사용하는 SQL Statement를 변조하여 요청에 영향을 주는 기법입니다.
- Command Injection
 : OS Command를 사용 시 사용자의 입력 데이터에 의해 실행되는 command로 인해 발생하는 취약점입니다.
- Server Side Template Injection(SSTI)
 : Template 변환 도중 사용자의 입력 데이터가 Template로 사용되어 발생되는 취약점입니다.
- Path Traversal
 : URL/File path를 사용 시 사용자의 입력 데이터에 의해 임의의 경로로 접속할 수 있는 취약점입니다.
- Server Side Request Forgery(SSRF)
 : 공격자가 서버에 변조된 요청을 할 수 있는 취약점입니다.
```

* * *

## **SQL Injection**

**Structed Query Language**의 약자로 RDBMS의 데이터를 관리하기 위해 사용되는 언어입니다. Database와 연계되는 많은 웹 애플리케이션이 SQL을 사용하여 상호작용 하게 되는데 이 때, 정상적인 입력값이 아닌 악의적인 SQL Query를 전송하여 Database의 내용을 추출하거나 변조, 삭제 등이 가능하게 됩니다.

 SQL Injection이 가장 많이 발생하는 기능은 로그인 기능이 해당됩니다. ID와 Password를 입력하면 이를 Database에 전송하여 확인하는 절차를 거치게 되는데 이 때 인증을 우회하거나 데이터 추출을 위해 추가적인 Query를 작성할 수도 있습니다.

* * *

SQL Injection 취약점을 예방하기 위해서는 사용자의 입력 데이터가 SQL Query로 해석되지 않도록 해야 합니다. 이를 예방하기 위해서는 문자열 구분자(', ")를 일반적인 문자열로 변환하는 것도 중요하지만 ORM과 같이 검증된 SQL Library를 사용하는 것도 해결책으로 선택할 수 있습니다.

ORM은 Object Relational Mapper의 약자로 SQL Query 작성을 돕기 위한 Library입니다. 이는 사용자의 입력값을 Library 단에서 스스로 escape하고 query에 mapping 시키기 때문에 보다 안전한 SQL query를 실행할 수 있습니다.

* * *

## **Command Injection**

웹 애플리케이션은 각자 OS command를 실행하기 위한 함수를 가지고 있습니다. 여기서 OS command란 linux의 shell 혹은 windows의 cmd, powershell에서 실행할 수 있는 command를 의미합니다.

웹 애플리케이션에서 OS command를 사용하는 경우는 구현하고자 하는 기능이 OS에 실행 파일이 존재할 경우 편리하기 때문입니다. 이 때 OS Command의 경우 shell을 이용하여 실행되는데 shell의 경우 한 줄에 여러 명령어를 사용하기 위해 다양한 특수문자가 있습니다.

- ` 문자
- $()
	- 위 두 문자의 경우 치환 기능을 가지고 있습니다. 
- &&
- \|\|
	- 위 두 문자는 명령어 연속 실행을 위한 문자이며 뒷 명령어의 경우 앞 명령어의 에러 유무에 영향을 받게 됩니다.
- ;
	- 위 문자는 명령어 구분자로 &&나 \|\|와 다르게 앞 명령어의 에러 유무와 관계없이 뒷 명령어도 실행합니다.
- \|
	- Pipe(\|)의 경우 앞 명령어의 결과가 뒷 명령어의 입력으로 사용됩니다.

* * *

Command Injection을 예방하기 위한 가장 좋은 방법은 웹 애플리케이션에서 OS Command를 사용하지 않는 것입니다. 필요한 기능이 Library로 구현되어 있으면 해당 Library를 사용하는 것이 권장되며 만약 꼭 필요할 경우 filtering을 통해 취약점을 예방하는 것이 좋습니다.

- 정규 표현식을 통한 Whitelist filtering
- meta character를 Single Quotes(')로 감싸기
	- **중요** 만약 Double Quotes(")를 사용할 경우 $ 혹은 `가 해석되기에 Single Quotes를 사용해야 함

* * *

## **SSTI**

**Server Side Template Injection**의 약자로 웹 애플리케이션에서 동적인 내용을 HTML로 출력할 때 미리 정의한 Template에 동적인 값을 넣어 출력하기도 하는데, 이 때 Template source에 사용자 입력이 가능할 경우 악의적인 입력을 통해 의도하지 않는 기능을 실행할 수 있는 취약점입니다.

SSTI의 경우 {{}} 혹은 ${}와 같은 문법을 통해 명령을 실행할 수 있습니다.

따라서 이를 예방하기 위해서는 사용자의 입력이 Template source에 삽입되지 않도록 하고 출력을 위해서는 Template context에 값을 넣어 출력하도록 해야 합니다.

* * *

## **Path Traversal**

Path Traversal의 경우 사용자의 입력 데이터가 적절한 필터링 없이 URL 혹은 File path에 직접적으로 사용될 경우 의도하지 않은 임의의 경로에 접근할 수 있는 취약점입니다.

/ 문자의 경우 기존의 URL 구조에서 알 수 있듯이 path identifier 즉, 구분자이며 ..의 경우 자신의 상위 디렉터리를 의미합니다. 또한 ?의 경우 GET 방식에서 Query를 위해 사용하는 문자인데 이 때 ? 문자의 뒤는 모두 query로 해석하기 때문에 응용할 경우 주석과 같은 역할로도 사용할 수 있어보입니다.

이를 예방하기 위해서는 사용자의 입력 데이터에 URL Encoding과 같은 encoding 기법을 통해 일반적인 문자로 인식되도록 하여 예방할 수 있습니다.

* * *

## **SSRF**

**Server Side Request Forgery**의 약자로 웹 애플리케이션에서 변조된 요청/의도하지 않은 서버로 요청을 보내는 취약점을 의미합니다.

SSRF의 경우 웹 애플리케이션에서 사용자가 입력한 URL에 요청을 보내는 기능에서 비롯되기도 하는데 이를 예방하기 위해 사용자가 입력한 URL의 host를 whitelist로 검증할 수 있습니다.

* * *


