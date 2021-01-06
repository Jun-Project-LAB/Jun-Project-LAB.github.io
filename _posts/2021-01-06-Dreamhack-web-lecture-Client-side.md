---
title: "Dreamhack Web hacking Lecture(Client-Side)"
tags: ["client-side basic"]
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
