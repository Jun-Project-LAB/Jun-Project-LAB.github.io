---
title: "Dreamhack Web hacking Lecture(Client-Side)"
tags: ["client-side basic", sop, cors, xss]
categories: ["dreamhack", "web course"]
---

## Client-Side Basic

Dreamhack의 web hacking 강의 중 Client-side basic 강의를 보며 이해한 내용을 기반으로 작성하였습니다.

* * *

## **SOP**

**Same Origin Policy**의 약자로 Javascript 또는 HTML tag를 사용하여 외부 리소스를 불러오는 기능을 이용, 기존에 정의되지 않은 악의적인 행위를 할 수 있는데 이를 방지하기 위한 정책으로 만들어졌습니다.

이 정책은 서로 다른 Origin의 문서 또는 스크립트 간의 상호 작용을 제한함으로서 목적을 달성합니다. 여기서 Origin은 URL의 scheme, host, port를 통해 정의됩니다. 즉, 다른 origin이라고 함은 다른 URL을 의미합니다.

SOP의 경우 대체로 다른 Origin으로 정보를 송신하는 것은 허용되나 다른 Origin으로부터 정보를 수신받는 것은 허용되지 않는다고 합니다. 이는 결국 CSRF 공격에 악용될 수 있다는 단점이 있지만 정보의 송신을 금지할 경우 hyperlink를 사용할 수 없기에 정상적인 구현이 힘들게 됩니다.

* * *

## **CORS**

**Cross Origin Resource Sharing**의 약자로 추가 HTTP Header를 사용하여 한 Origin에서 사용 중인 웹 애플리케이션이 다른 Origin의 선택 자원에 접근할 수 있는 권한을 부여하도록 브라우저에 알려주는 체제입니다.

이는 SOP의 영향으로 인해 서로 다른 Origin과 Resource를 공유할 수 없는 것을 해결하기 위함으로 개발 또는 운영 목적으로 이를 필요로 하는 상황이 있을 수도 있기에 SOP가 적용된 상태에서도 리소스를 공유할 수 있는 방법이 CORS라고 합니다.

CORS를 구성하는 방법의 경우 **postMessage**, **JSONP**, **CORS Header 사용** 세 가지가 있습니다.

```
- postMessage
메시지를 주고받기 위한 이벤트 핸들러를 이용하여 리소스를 공유

- JSONP
스크립트 태그를 통해 다른 Origin의 Resource를 요청하고, 응답 데이터를 현재 Origin의 Callback 함수에서 다루는 방식으로 리소스를 공유

- CORS Header 사용
다른 Origin이 허용하는 설정 등을 HTTP Header를 통해 확인한 후 허용하는 요청을 보내 리소스를 공유하는 방식
```

* * *

## **XSS**

**Cross Site Scripting**의 약자로 서버의 응답에 공격자가 삽입한 악성 스크립트가 포함되어 사용자의 웹 브라우저에서 해당 스크립트가 실행되는 취약점을 의미합니다.

주로 사용자의 세션 혹은 쿠키를 탈취하여 권한을 획득하거나 페이지를 변조하는 등의 공격을 수행할 수 있으며 성공적으로 수행하기 위해서는 아래 두 가지 조건을 만족해야 합니다.

```
- 입력 데이터에 대한 충분한 검증 과정이 없을 것
 : 악성 스크립트 삽입을 위해서는 입력 데이터에 대한 검증의 허점이 필요합니다.

- 서버의 응답 데이터가 웹 브라우저 내 페이지에 출력 시 충분한 검증 과정이 없을 것
 : 이 또한 위와 마찬가지로 악성 스크립트가 웹 브라우저 렌더링에 성공하기 위해서는 검증의 허점이 필요합니다.
```

xss는 웹 서비스에 있어서 빈번하게 발생하는 취약점인 만큼 그에 대응하기 위한 다양한 기술 또한 있습니다. 아래는 그 기술들의 예시 중 하나입니다.

- Server-side Mitigations
- HTTPOnly flag 사용
- Content Security Policy 사용
- X-XSS-Protection
