---
title: "B반_최준영_4월 1주차"
tags: ["supply chain"]
categories: ["Security News"]
---

2021/04/02 Security News-4
--------------------------

# Index

1. [Subject](#subject)
2. [Related News](#related-news)
3. [Opinion](#opinion)

* * *


## Subject

최근들어 공급망 공격과 관련된 기사를 많이 접할 수 있었습니다. 가장 최근 기사로 큰 피해는 발생하지 않았지만 공식 깃 서버에 백도어를 심어둔 것이 발견된 것이 있었습니다. [해당기사](https://www.boannews.com/media/view.asp?idx=96026)

위 기사처럼 공식 배포 서버 뿐만 아니라 다양한 프로그램을 개발할 때 더 편리하게 개발하기 위해 공개되어 있는 api를 사용하거나 혹은 프레임워크를 사용할 때, 제대로 된 검증을 하지 않을 경우 발생하는 문제가 점점 증가하는 추세를 보이고 있습니다.

## Related News

클라우드에 대한 관심이 많아지면서 Docker와 같은 컨테이너에 악성코드를 심어 배포하는 사례 또한 기사화 되었습니다. [해당기사](https://www.bleepingcomputer.com/news/security/docker-hub-images-downloaded-20m-times-come-with-cryptominers/)

파이썬 또한 사용하다보면 다양한 패키지를 설치하여 사용하는데 이러한 패키지들을 대상으로 한 공격 또한 발견되었습니다. 비록 실제 공격 코드는 포함되지 않은 패키지들이었으나 실제 공격으로 이루어졌을 경우 큰 피해가 발생할 수도 있었습니다. [해당기사](https://www.boannews.com/media/view.asp?idx=95883&kind=5)

## Opinion

프로그램 개발을 하거나 docker를 사용하다보면 제대로 된 검증을 하지 않고 사용하게 되는 경우가 발생하는데 개인 프로젝트를 진행할 때에 있어서는 큰 피해가 아닐 수도 있으나 만약 기업의 프로젝트였을 경우 큰 피해로도 이어질 수 있다는 경각심을 알 필요가 있을 것 같습니다.
