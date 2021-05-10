---
title: "Dreamhack Web hacking Lecture(Server-side)"
tags: ["basic theory", server-side, "dreamhack lecture"]
categories: ["Web"]
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

## **File Upload**

File Upload 기능의 경우 사용자의 사진, 문서와 같은 파일을 서버에 업로드하여 다른 사용자와 공유하기 위한 목적으로 사용됩니다.
그러나 이때 서버에 업로드 되는 파일에 대한 충분한 검증이 없을 경우 이를 악용한 취약점이 발생할 수 있습니다.

* * *

## **File Download**

File Download의 경우 File Upload와 반대되는 기능으로 파일을 다운로드 할 때 제대로 된 검증을 하지 않을 경우 Path Traversal을 이용하여 기존의 location이 아닌 상위 location의 파일*(eg. /etc/passwd)*을 download 할 수 있게되어 추가적인 공격을 위한 발판이 될 수 있습니다.

* * *

이를 예방하기 위해서는 download 받으려는 파일의 경로나 이름을 넘기지 않되, 반드시 필요하다면 /나 \\\\와 같은 문자를 적절하게 필터링 할 필요가 있습니다.

혹은 Database에 download 될 파일의 경로와 1대 1로 매칭되는 랜덤키를 생성하여 파일을 download 하기 이전에 Database에 존재하는 파일인지 구별하는 방법도 권장된다고 합니다.

* * *

## **Bussiness Logic Vulnerability**

**Bussiness Logic**이란 규칙에 따라 데이터를 생성/표시/저장/변경하는 로직, 알고리즘 등을 말합니다. 즉, 하나의 서비스를 구성하고 있는 각각의 세부 기능들이라고 이해할 수 있습니다.

이 취약점의 경우 서비스의 기능에서 적용되어야 할 로직이 없거나 잘못 설계된 경우 발생하게 됩니다. 주로 발생되는 원인은 권한 확인과 같이 서비스 동작에 영향을 줄 수 있는 부분인 것을 알 수 있었습니다.

Bussiness Logic과 관련된 취약점 형태는 다음과 같은 것들이 있습니다.

```
- Bussiness Logic Vulnerability
 : 정상적인 흐름에서 검증 과정의 부재 및 미흡으로 인해 이가 악용되는 취약점
- IDOR(Inscure Direct Object Reference)
 : 변조된 파라미터 값이 다른 사용자의 오브젝트 값을 참조할 때 발생되는 취약점
- Race Condition
 : Bussiness Logic의 순서가 잘못되거나, 한 오브젝트에 여러 요청이 동시에 처리되는 상황에서 발생하는 취약점
```

* * *

Bussiness Logic Vulnerability을 방어하기 위해서는 서비스의 로직을 확실하게 이해하고, 설계 및 개발 단계에서 예상되는 위협을 파악하고 이를 조치하는 것이 필요합니다.

### **IDOR**

**Insecure Direct Object Reference**는 객체 참조시 사용하는 객체 참조 키가 사용자에 의해 조작됐을 때 조작된 객체 참조 키를 통해 객체를 참조하고, 해당 객체 정보를 기반으로 로직이 수행되는 것을 의미합니다. 이 과정에서 사용자가 참조하고자 하는 객체에 대한 권한 검증이 올바르지 않을 경우 문제가 발생합니다.

이를 예방하기 위해서는 객체 참조 시 사용자의 권한을 검증하는 것이 중요합니다. 또한 로그인 등의 사용자 인증을 거친 후에는 사용자 식별을 위한 정보를 별도의 입력 데이터로 검증하는 것이 아닌 세션을 통해 서버 내에서 처리하는 것이 안전합니다.
그 외에도 객체를 참조하기 위해 사용하는 키를 단순 문자열이 아닌 무작위 문자 생성 등을 통해 이를 예측하지 못하도록 하는 것 또한 방법이 될 수 있습니다.

* * *

### **Race Condition**

**Race Condition**은 공유 자원 처리 과정에서 해당 자원에 대한 동시 다발적인 접근으로 인해 발생하는 취약점입니다. 주로 Database 또는 파일 시스템과 같이 웹 애플리켕션에서 공유하는 자원들에 대한 접근 과정에서 이를 참조하는 타이밍의 차이로 인해 취약점이 발생됩니다. 악용될 경우 검증 과정에서의 데이터와 수정 과정의 데이터 간의 차이로 인해 의도하지 않은 결과가 발생되게 됩니다.

이를 방지하기 위해서는 안전하게 보호되어야 하는 로직들에 대해 하나의 접근이 끝난 후 다음 접근을 처리하도록 쓰레드 락 등을 통해 동시 다발적인 접근을 방지하여야 합니다. 혹은 race condition을 발생시키기 위해서는 다량의 접근이 필요한 점을 예방하기 위한 CSRF token, capcha 등을 통해 이를 방지함으로써 보호할 수 있습니다.
