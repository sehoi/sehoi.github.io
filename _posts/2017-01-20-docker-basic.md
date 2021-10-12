---
layout: single
classes: wide
title: "docker 설치 및 기초 명령어"
date: 2017-01-06
description: "docker 설치 방법 및 기본적인 명령어들에 대해 정리해보았습니다."
categories:
  - etc
tags: [docker, docker container, docker image, dockerfile, docker centos 6]
comments: true
share: true
---

docker 설치 방법 및 기본적인 명령어들에 대해 정리해보았습니다.

## docker 홈페이지
- [https://docs.docker.com/](https://docs.docker.com/)
<br /><br />

## docker 설치 및 데몬 시작
<br />

#### 설치
- 일반 설치
  - [https://docs.docker.com/engine/getstarted/step_one/#/step-2-install-docker](https://docs.docker.com/engine/getstarted/step_one/#/step-2-install-docker)

- centos 6.x 버전 서버에 설치하는 경우([참고](https://www.liquidweb.com/kb/how-to-install-docker-on-centos-6/))
  - `$ sudo yum install docker-io`
<br />

#### 설정 변경
> - [참고](https://forums.docker.com/t/how-do-i-change-the-docker-image-installation-directory/1169/14)

- 설정 파일
  - `/etc/sysconfig/docker`
- 기본 디렉토리 변경 설정
  - `other_args="-g /home/docker"`
- tmp 디렉토리 변경 설정
  - `DOCKER_TMPDIR=/home/docker/tmp`
- proxy 설정
  - `export http_proxy="http://proxy.server.co.kr:port"`
  - `export https_proxy="https://proxy.server.co.kr:port"`
<br />

#### docker 데몬 시작
- `$ sudo /etc/init.d/docker start`
<br /><br />

## 기본적인 명령어
<br />

#### 이미지 관련
- 이미지 추가하기
  - [https://hub.docker.com/](https://hub.docker.com/)
  - `$ sudo docker pull IMAGE`
- 다운로드 받은 이미지 추가하기
  - `cat 이미지파일.tar.xz | sudo docker import - IMAGE:TAG`
  - `$ cat centos-\*.tar.xz | sudo docker import - centos:base`
- 추가된 이미지 보기
  - `$ sudo docker images -a`
- 이미지로 컨테이너 생성
  - `$ sudo docker run -t -i IMAGE:TAG`
- 이미지 삭제
  - `$ sudo docker rmi IMAGE:TAG`
<br />

#### 컨테이너 관련
> - [참고](https://gist.github.com/nacyot/8366310)

- 컨테이너 생성
  - `$ sudo docker run -it 컨테이너ID`
  - `$ sudo docker run -d 컨테이너ID`
- 컨테이너 정지/실행/재실행
  - `$ sudo docker stop|start|restart 컨테이너ID`
- 실행중인 컨테이너에 접속
  - `$ sudo docker attach 컨테이너ID`
- 실행중인 컨테이너에 새로운 TTY로 접속
  - `$ sudo docker exec -it 컨테이너ID /bin/bash`
- 접속중인 컨테이너에서 빠져나오기
  - `ctrl+p -> ctrl+q`
- 컨테이너 중지
  - `$ sudo docker kill 컨테이너ID`
- 컨테이너 보기
  - `$ sudo docker ps -a -s`
- 컨테이너 커밋하기(이미지 추가)
  - `$ sudo docker commit -m "커밋 메세지" -a "유저명" 컨테이너ID IMAGE:TAG`
- 마운트
  - `$ sudo docker run -v 로컬경로:컨테이너경로 -ti 이미지ID`
- 데몬 실행 & 포트 바인딩
  - `$ sudo docker run -d -p 로컬포트:컨테이너포트 이미지ID`
- 컨테이너 로그 확인
  - `$ sudo docker logs -f 컨테이너ID`
- 컨테이너 프로세스 확인
  - `$ sudo docker top 컨테이너ID`
<br />

#### Dockerfile 관련
- Dockerfile 생성
  - [http://www.joinc.co.kr/w/man/12/docker/Guide/dockerimages](http://www.joinc.co.kr/w/man/12/docker/Guide/dockerimages)
- Dockerfile로부터 컨테이너 실행
  - `$ sudo docker build -t IMAGE:TAG Dockerfile경로`
