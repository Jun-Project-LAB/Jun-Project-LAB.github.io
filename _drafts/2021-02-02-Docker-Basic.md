---
title: "Docker Basic Usage"
tags: ["docker", "container"]
categories: ["container"]
---

How to use Docker
-----------------

기존의 Virtual Machine 환경이 아닌 Container 환경을 위해 Docker를 사용해보기로 하였습니다. 기본적인 설치의 경우 아래 링크를 참고하여 진행하였으며 설치 과정 중 별도의 오류는 없었습니다.

<https://www.44bits.io/ko/post/easy-deploy-with-docker>

기본적으로 docker를 관리하는 명령어는 docker [command] 형태로 사용되는 것을 확인할 수 있었습니다.

먼저 실행하고자 하는 container를 설치하는 것은 docker pull \<name of container\>:[version]을 통해서 다운로드 받을 수 있었습니다. version의 경우 별도로 지정하지 않을 경우 latest version을 설치하며 image의 경우 docker hub에서 검색할 수 있습니다.

<https://hub.docker.com/>

설치된 이미지는 docker images 명령어를 통해 목록을 확인할 수 있습니다.

[!docker images](https://raw.githubusercontent.com/Jun-Project-LAB/Jun-Project-LAB.github.io/main/_image/docker-images.png)

댜운로드 받은 container의 실행은 docker run 명령어를 통해 진행할 수 있습니다. 명령어의 목록 중 docker start도 있어 이에 대한 차이점을 정리한 게시글을 찾을 수 있었습니다.

<https://linuxhandbook.com/docker-run-vs-start-vs-create/>


