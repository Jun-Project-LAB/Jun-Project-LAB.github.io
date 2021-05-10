---
title: "About SQL Injection"
tags: ["web theory", database]
categories: [Web]
---

About SQL Injection
-------------------

SQL Injection은 database에 query하기 위한 언어인 SQL에 취약점을 이용하여 악의적인 목적으로 조작하는 모든 공격을 의미합니다. SQL Injection에는 다양한 기법들이 존재하고 있으며, 
이 기법을 방어하기 위한 기술과 치열하게 공방전을 펼치며 발전하고 있습니다.

## **다양한 공격 기법**

- **Logic**

가장 기본적인 논리 연산을 이용한 공격 방법입니다. 주로 SQL에서는 and와 or이 사용되는데 and는 모든 조건이 만족되어야 참이 되는 반면 or는 하나만 참이 되어도 결과가 참이 되기 때문에 임의적으로 결과를 변경할 수 있습니다.

대표적으로 계정 인증 과정에서 id와 pw를 and로 묶어 두 조건이 모두 참이 되었을 때 로그인을 성공하도록 지정하였다면 해당 query문에 or 1 혹은 or 1=1과 같이 추가할 경우 id와 pw가 일치하지 않아도 로그인에 성공하였다고 반환하게 됩니다.

* * *

- **Union**

Union의 경우 다수의 SELECT 구문의 결과를 결합시키는 행위를 수행하는데 이를 사용하여 다른 테이블에 접근하거나 원하는 쿼리 결과 데이터를 생성, 어플리케이션에서 처리되는 데이터를 조작할 수 있습니다.

Union을 사용하기 위해서는 특정 조건이 만족되어야 하는데 첫 번째로 **이전 SELECT 구문과 Union SELECT 구문의 결과 column의 수가 같아야** 합니다. 또한 DBMS의 종류에 따라 이전 column과 Union SELECT Column의 타입이 같아야 하는 제품도 있습니다.

* * *

- **Subquery**

Subquery는 하나의 query 내에 또 다른 query를 사용하는 것으로 괄호를 통해 이를 선언할 수 있습니다.

`SELECT 'ABC', (SELECT 'DEF');`

Subquery의 경우 SELECT 구문만 사용할 수 있기 때문에 INSERT, DELETE와 같은 Query는 사용할 수 없습니다.

이를 이용하여 Main query가 접근하는 테이블이 아닌 다른 테이블에 접근하거나 기존의 Main Query가 INSERT, DELETE와 같이 SELECT 문이 아닌 구문에서도 SQL Injection을 통해 테이블의 데이터에 접근하는 것이 가능하게 됩니다.

* * *

- **Error Based**

Error Based SQL Injection의 경우 의도적으로 SQL문에 error를 발생시켜 정보를 획득하는 방법을 의미합니다.

Error Based SQL Injection을 성공적으로 활용하기 위해서는 Syntax Error와 같이 쿼리가 실행되기 전에 검증 가능한 에러가 아닌, Runtime Error가 필요합니다.

아래 쿼리의 경우는 MySQL에서 많이 사용되는 공격 형태 중 하나입니다.

```sql
SELECT extractvalue(1,concat(0x3a,version()));
```

출력 결과는 다음과 같습니다.

`ERROR 1105 (HY000): XPATH syntax error: ':5.7.29-0ubuntu0.16.04.1-log'`

위 출력 결과를 보면 알 수 있듯이 실행중인 서버의 OS가 확인이 가능합니다.

* * *

- **Blind**

Blind SQL Injection의 경우 데이터베이스 조회 후 결과를 직접적으로 확인할 수 없는 경우 사용할 수 있는 공격입니다. 이 기법의 원리는 DB 내의 결과와 사용자의 입력값을 비교하며 특정 조건을 만족할 경우 특별한 응답을 발생시켜 참/거짓을 구분합니다.

Blind SQL Injection을 성공적으로 수행하기 위해서는 아래 두 가지 조건을 만족할 핊요가 있습니다.

- 데이터 비교를 통해 참/거짓 구분
- 참/거짓 결과에 따른 특별한 응답

* * *

- **Time Based**

Time Based Blind SQL Injection은 시간 지연을 이용하여 참/거짓을 판단합니다. 주로 DBMS에서 기본으로 제공하는 sleep 같은 함수를 사용하거나, 무거운 연산과정을 발생시켜 쿼리 처리 시간을 지연 시키는 heavy query 등이 존재합니다.

	- MySQL

> sleep()

MySQL에서 시간지연을 발생 시킬 수 있는 함수는 sleep 함수와 benchmark 함수가 있습니다. sleep 함수의 경우 sleep(duration)을 통해서 duration에서 지정한 시간 동안 query를 지연 시킬 수 있으며 이 때 단위는 초 단위로 계산됩니다. sleep 함수의 반환값은 정상적으로 동작하였을 경우에는 0을, interrupt 된 경우에는 1을 반환한다고 합니다.

> benchmark()

benchmark 함수의 경우 benchmark(count, expr)과 같이 사용하며 expr에 정의된 식을 count 만큼 실행하여 시간지연을 발생시킵니다. 반환값은 항상 0을 가집니다.

> heavy query

MySQL은 기본적으로 information_schema DB에 tables라는 system table을 제공하는데 이 table에는 많은 수의 데이터가 포함되어 있습니다. 따라서 이 table을 사용하여 heavy query를 시도할 수 있습니다.

`mysql> SELECT (SELECT count(*) FROM information_schema.tables A, information_schema.tables B, information_schema.tables C) as heavy;`
*여기서 각각의 table 뒤에 A, B와 같이 입력해주는 이유는 기본적으로 같은 table을 조회하려고 할 경우 중복 오류가 발생하여 별칭을 지정하는 것으로 보임*

* * *

	- MSSQL

> waitfor

MSSQL의 경우 MySQL의 sleep 함수와 비슷한 역할을 하는 함수로 wailfor 구문이 있습니다. waitfor 구문의 경우 옵션에 따라 다르게 사용될 수 있으나 기본적인 내용에 대해서만 알아보았습니다.

`WAITFOR DELAY '00:00:05'`

WAITFOR 뒤에 DELAY 옵션을 사용할 경우 뒤에서 지정된 시간이 지난 후 query가 실행됩니다. 시간의 형식은 h:m:s 형식으로 지정됩니다.

* * *

`WAITFOR TIME '05:09:00'`

WAITFOR 뒤에 TIME 옵션을 사용할 경우 지정된 시간에 query가 실행됩니다. 시간의 형식은 DELAY와 동일하며 위 예시의 경우 오전 5시 9분에 쿼리가 실행됩니다.

> heavy query

MSSQL은 MySQL에서 사용되었던 table과 다르게 columns table을 사용하여 heavy query를 실행할 수 있습니다. DB명은 MySQL과 일치하기에 information_schema.columns와 같이 사용하면 됩니다.

	- SQLite

> heavy query

SQLite의 경우 RANDOMBLOB 함수를 사용할 경우 난수를 생성하게 되는데 이 때 변환 과정과 함수를 거치며 시간지연이 발생한다는 점을 이용하여 공격에 활용할 수 있습니다.

`LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB([SLEEPTIME]00000000/2))))`

* * *

- **Error Based Blind**

Error Based Blind SQL Injection은 임의적으로 에러를 발생시켜 참/거짓을 판단하는 공격 방법입니다. 앞서 본 Error Based의 경우 Error가 발생했을 경우 data가 출력될 필요가 있었지만, Error Based Blind 공격 시에는 발생 여부만 판단하면 되기 때문에 다른 Runtime Error도 사용할 수 있습니다. 간단한 예시로는 MySQL에서 처리할 수 있는 Double type의 최대 값을 넘겨 에러를 발생시키는 방법이 있습니다.

```
mysql> select if(1=1, 9e307*2,0);
ERROR 1690 (22003): DOUBLE value is out of range in '(9e307 * 2)'

mysql> select if(1=0, 9e307*2,0);
+--------------------+
| if(1=0, 9e307*2,0) |
+--------------------+
|                  0 |
+--------------------+
1 row in set (0.00 sec)
```

첫 번째의 경우 결과가 참이기 때문에 if문에 의해 연산이 실행되었고 에러가 발생하였으며 두 번째의 경우 거짓이기 때문에 연산이 실행되지 않은 것을 알 수 있습니다.

이와 같이 에러가 발생했을 때 HTTP Response Status Number 혹은 Application의 Response 차이를 통해 에러 여부를 확인하고 참/거짓 여부를 판단할 수 있다고 합니다.

* * *

- **Short-Circuit evaluation**

Logic 연산의 원리를 이용한 방법으로 A AND B 식의 경우 A가 거짓일 경우 결과는 거짓이 되기 때문에 B 연산을 수행하지 않는 것을 의미합니다. 이는 OR 또한 마찬가지로 A OR B에서 A가 참일 경우
 결과는 어차피 참이기 때문에 B를 수행하지 않게 됩니다.

* * *

## **Blind SQL Injection Tip**

Blind SQL Injection은 참/거짓을 통해 값을 추출하게 되는데 이때 보편적으로 ASCII에서 출력 가능한 문자의 범위를 대상으로 Brute-Force Attack을 수행하게 됩니다. 기본적으로 32 ~ 126까지 94개의 문자에 대해서 수행하는데 최악의 경우 94번을 모두 수행해야 한다는 의미가 됩니다. 따라서 비교적 시간을 단축할 수 있는 방법들이 있다고 합니다.

- **Binary Search(이진 검색 알고리즘)**

Binary search의 경우 **정렬된 리스트**에서 특정한 값을 찾는 알고리즘으로 대상 범위의 절반에 해당하는 숫자를 골라 찾고자 하는 값과 비교하여 클 경우 절반을 시작점, 절반보다 큰 숫자를 범위로 하여 다시 검색하고, 만약 작을 경우 절반을 범위의 끝점으로 하고 그 이하의 숫자를 범위로 하여 반복 계산하는 방법입니다. 

Binary Search를 사용하기 위해서는 문자열을 ord 함수와 같이 ASCII 값으로 바꿔주는 함수 등을 이용하여 비교할 필요가 있습니다.

- **Bit 연산**

ASCII는 7개의 비트(이진수)로 구성되어 있기 때문에 해당 원리를 이용하면 7번의 요청을 통해 한 바이트를 획득할 수 있습니다. 비트는 이진수로 구성되어 있기 때문에 1에 대해 요청을 하였을 때 결과가 거짓이 return 되면 해당 값은 0이고 만약 참이 return 될 경우 1인 것을 알 수 있습니다.

MySQL의 경우 bin 함수를 사용하여 int형을 bit 형태로 변경할 수 있습니다.

* * *

## **Exploition Technique**

SQL Injection을 수행할 때 사용되고 있는 DBMS의 종류를 파악한 뒤 공격하는 것이 더 효율적입니다. 따라서 DBMS의 종류를 파악할 수 있는 단서들에 대해 정리해보았습니다.

- **결과가 출력될 때**

결과가 출력될 경우 가장 기본적으로 DBMS 별로 다른 환경변수 혹은 함수를 사용한다는 것을 통해서 알 수 있습니다. 이에 사용되는 대표적인 함수가 version 함수입니다.

MySQL의 경우 version()과 @@version 모두 사용 가능하지만 PostgreSQL이나 MSSQL의 경우 각각 version()과 @@version만 동작합니다.

- **결과가 출력되지 않으나 에러가 출력될 때**

만약 에러만 출력될 경우에는 해당 DBMS에서 지원하지 않는 문법 혹은 존재하지 않는 system table을 조회하였을 때 에러의 발생 여부를 통해 확인할 수 있습니다. 

- **결과가 출력되지 않을 때**

	- **True/False를 통해 확인 가능할 경우**

Blind 방식을 사용하여 함수, 조건문에 대한 결과값을 확인하여 구분할 수 있습니다.

	- **출력이 존재하지 않는 경우**

위의 방식도 사용이 불가할 경우 Time Based 방식을 사용하는데 이 때, sleep 함수의 형태에 따라 구별할 수 있습니다. 예를 들어 MySQL은 sleep() 형식으로 사용하지만 PostgreSQL은 pg_sleep()로 사용하기에 실행 여부를 통해 확인할 수 있습니다.

* * *

## **System Tables**

DBMS마다 Database의 정보를 담고 있는 고유한 System table이 있습니다. 이 System table에는 DB 설정, 계정 정보 외에도 Database/Table/Column 정보, 현재 실행되고 있는 query 정보 등 다양한 정보들을 담고 있습니다.

따라서 System table에 담긴 정보를 알아낼 수 있다면 더 효율적으로 공격할 수 있습니다.

* * *

## **WAF Bypass**

WAF란 Web Application Firewall로 웹 해킹 시 주로 사용되는 키워드에 대해 방어하는 시스템이 존재하는 경우가 있습니다. 이럴 경우 SQL Injection 취약점이 발생하더라도 공격 Query가 제대로 실행되지 못하는 경우가 발생할 수 있습니다.

따라서 이러한 필터링을 우회하기 위해 비슷한 역할을 하는 다른 keyword로 우회하거나 다른 쿼리를 사용하는 방법이 존재합니다.

- **대소문자**

특정 keyword를 필터링할 때 대소문자를 구별할 경우 이를 혼합하여 사용함으로써 우회할 수 있는 방법이 있습니다. 예를 들어 substring을 대소문자 구별하여 필터링하면 SUBSTRING, SubStRIng과 같이 우회가 가능합니다.

- **Replace**

keyword를 필터링할 때 해당 keyword를 다른 단어로 바꿔버리는 replace를 통한 필터링을 할 경우도 있는데 이러한 경우도 다음과 같이 쉽게 우회할 수 있습니다. 만약 substring을 필터링할 때 `subsubstringstring`과 같이 작성할 경우 가운데 substring이 필터링 되어도 최종적으로 sub과 string이 합쳐져 substring이 됩니다.

- **그 외 keyword filter bypass 방법**

	- **concat**

concat의 경우 문자열을 합칠 수 있는 함수입니다. 사용법의 경우 concat(a, b)와 같이 하면 a와 b 문자열을 이어붙입니다. MySQL, MSSQL, PostgreSQL 등에서 사용할 수 있습니다.

MSSQL의 경우 concat 함수 대신 + 기호를 사용할 수도 있으며 SQLite의 경우 || 기호를 사용하여도 가능합니다.

	- **reverse**

reverse 함수는 입력된 문자열의 순서를 거꾸로 재배치합니다. MySQL, MSSQL 등에서 사용할 수 있습니다.

	- **진법 변환 표기**

진법 표기를 통해 문자열을 입력하는 것 또한 가능합니다. bin의 경우 0b~~, oct의 경우 x'~~', hex의 경우 0x~~와 같이 표기할 수 있습니다.

	- **문자 변환 함수**

직접 진법 변환을 하는 것이 아닌 함수를 사용하여 변환하는 것 또한 가능합니다. ASCII 값을 문자열로 변환할 때 MySQL, MSSQL, SQLite 등은 char() 함수를 사용하며 PostgreSQL 등은 chr() 함수를 사용합니다. 반대의 경우 MySQL 등은 ord() 함수를, MSSQL은 ASCII() 함수를 사용하여 변환할 수 있습니다.

	- **WhiteSpace**

공백을 우회하는 방법에는 여러 방법이 있지만 대표적으로 개행 문자(\\n)를 이용한 방법과 주석(/\*\*/)을 이용한 방법이 있습니다.

	- **MySQL Comment Execute**

앞서 /\*\*/의 경우 주석을 위해 사용할 수 있는 것을 알 수 있었는데 MySQL의 경우 /\*!command\*/와 같이 사용할 경우 WAF에서는 주석으로 인식하지만 DBMS에서는 command가 정상적으로 동작한다고 합니다.

	- **Select bypass**

Select가 필터링 된 경우 union values()를 통해 select와 같은 기능을 할 수 있습니다.


