---
title: "Webhacking.kr challenge(old) 12th Write-Up"
tags: [wargame, javascript]
categories: [Web]
---

Webhacking.kr 12 Write-Up
-------------------------

## **문제 분석**

문제를 클릭하면 검은색 배경에 "javascript challenge" 라고 되어있는 것을 확인할 수 있습니다. 따라서 코드를 확인하기 위해 F12를 눌러 Script 부분을 확인한 결과 다음과 같았습니다.

```
ﾟωﾟﾉ = /｀ｍ´）ﾉ ~┻━┻   / /*´∇｀*/ ['_'];
o = (ﾟｰﾟ) = _ = 3;
c = (ﾟΘﾟ) = (ﾟｰﾟ) - (ﾟｰﾟ);
(ﾟДﾟ) = (ﾟΘﾟ) = (o ^ _ ^ o) / (o ^ _ ^ o);
(ﾟДﾟ) = {
    ﾟΘﾟ: '_',
    ﾟωﾟﾉ: ((ﾟωﾟﾉ == 3) + '_')[ﾟΘﾟ],
    ﾟｰﾟﾉ: (ﾟωﾟﾉ + '_')[o ^ _ ^ o - (ﾟΘﾟ)],
    ﾟДﾟﾉ: ((ﾟｰﾟ == 3) + '_')[ﾟｰﾟ]
};
(ﾟДﾟ)[ﾟΘﾟ] = ((ﾟωﾟﾉ == 3) + '_')[c ^ _ ^ o];
(ﾟДﾟ)['c'] = ((ﾟДﾟ) + '_')[(ﾟｰﾟ) + (ﾟｰﾟ) - (ﾟΘﾟ)];
(ﾟДﾟ)['o'] = ((ﾟДﾟ) + '_')[ﾟΘﾟ];
(ﾟoﾟ) = (ﾟДﾟ)['c'] + (ﾟДﾟ)['o'] + (ﾟωﾟﾉ + '_')[ﾟΘﾟ] + ((ﾟωﾟﾉ == 3) + '_')[ﾟｰﾟ] + ((ﾟДﾟ) + '_')[(ﾟｰﾟ) + (ﾟｰﾟ)] + ((ﾟｰﾟ == 3) + '_')[ﾟΘﾟ] + ((ﾟｰﾟ == 3) + '_')[(ﾟｰﾟ) - (ﾟΘﾟ)] + (ﾟДﾟ)['c'] + ((ﾟДﾟ) + '_')[(ﾟｰﾟ) + (ﾟｰﾟ)] + (ﾟДﾟ)['o'] + ((ﾟｰﾟ == 3) + '_')[ﾟΘﾟ];
(ﾟДﾟ)['_'] = (o ^ _ ^ o)[ﾟoﾟ][ﾟoﾟ];
(ﾟεﾟ) = ((ﾟｰﾟ == 3) + '_')[ﾟΘﾟ] + (ﾟДﾟ).ﾟДﾟﾉ + ((ﾟДﾟ) + '_')[(ﾟｰﾟ) + (ﾟｰﾟ)] + ((ﾟｰﾟ == 3) + '_')[o ^ _ ^ o - ﾟΘﾟ] + ((ﾟｰﾟ == 3) + '_')[ﾟΘﾟ] + (ﾟωﾟﾉ + '_')[ﾟΘﾟ];
(ﾟｰﾟ) += (ﾟΘﾟ);
(ﾟДﾟ)[ﾟεﾟ] = '\\';
(ﾟДﾟ).ﾟΘﾟﾉ = (ﾟДﾟ + ﾟｰﾟ)[o ^ _ ^ o - (ﾟΘﾟ)];
(oﾟｰﾟo) = (ﾟωﾟﾉ + '_')[c ^ _ ^ o];
(ﾟДﾟ)[ﾟoﾟ] = '\"';
~~이하 생략~~
```

## **문제 해결 방법**

위 코드를 보면 알 수 있듯이 난독화 되어있는 코드였는데 해당 방식을 찾아본 결과 aaencode 라는 방식으로 되어있는 것을 알 수 있었습니다.

decoding한 결과 다음 코드를 얻을 수 있었습니다.

```javascript
var enco = '';
var enco2 = 126;
var enco3 = 33;
var ck = document.URL.substr(document.URL.indexOf('='));
for (i = 1; i < 122; i++) {
    enco = enco + String.fromCharCode(i, 0);
}

function enco_(x) {
    return enco.charCodeAt(x);
}
if (ck == "=" + String.fromCharCode(enco_(240)) + String.fromCharCode(enco_(220)) + String.fromCharCode(enco_(232)) + String.fromCharCode(enco_(192)) + String.fromCharCode(enco_(226)) + String.fromCharCode(enco_(200)) + String.fromCharCode(enco_(204)) + String.fromCharCode(enco_(222 - 2)) + String.fromCharCode(enco_(198)) + "~~~~~~" + String.fromCharCode(enco2) + String.fromCharCode(enco3)) {
    location.href = "./" + ck.replace("=", "") + ".php";
}
```

코드를 간략하게 해석해보면 ck라는 값에 if 문에서 일치하는 값이 들어올 경우 문제가 해결되는 것임을 파악할 수 있었습니다. 일정한 규칙을 가지고 ASCII값을 사용하여 문자로 변환하는 것으로 보였는데 하나하나 계산하기 귀찮기에 if문에서 원하는 조건을 console로 출력한 결과 다음과 같은 값임을 알 수 있었습니다.

**youaregod~~~~~~~!**

GET 방식으로 ck=youaregod~~~~~~~!와 같이 입력할 경우 문제를 해결할 수 있었습니다.
