---
title: "Dreamhack Web hacking Lecture(Client-Side)"
tags: ["client-side basic"]
categories: ["dreamhack", "web course"]
---

## Client-Side Basic

Dreamhack의 web hacking 강의 중 Client-side basic 강의를 보며 이해한 내용을 기반으로 작성하였습니다.

* * *

## **SOP**

- SOP란 

**Same Origin Policy**의 약자로 Javascript 또는 HTML tag를 사용하여 외부 리소스를 불러오는 기능을 이용, 기존에 정의되지 않은 악의적인 행위를 할 수 있는데 이를 방지하기 위한 정책으로 만들어졌습니다.

이 정책은 서로 다른 Origin의 문서 또는 스크립트 간의 상호 작용을 제한함으로서 목적을 달성합니다. 여기서 Origin은 URL의 scheme, host, port를 통해 정의됩니다. 즉, 다른 origin이라고 함은 다른 URL을 의미합니다.

SOP의 경우 대체로 다른 Origin으로 정보를 송신하는 것은 허용되나 다른 Origin으로부터 정보를 수신받는 것은 허용되지 않는다고 합니다. 이는 결국 CSRF 공격에 악용될 수 있다는 단점이 있지만 정보의 송신을 금지할 경우 hyperlink를 사용할 수 없기에 정상적인 구현이 힘들게 됩니다.

* * *

## **CORS**


