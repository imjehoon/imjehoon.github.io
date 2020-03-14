---
layout: post
title: "DockerFile이란"
tags: [Docker, Kubernetes, DockerFile]
category: [server]
comments: true
---

### Dockerfile
* 직접 컨테이너를 생성하고 이미지로 커밋해야 하는 번거로움을 덜 수 있을뿐더러 깃과 같은 개발 도구를 통해 애플리케이션의 빌드 및 배포 자동화 가능.
* Dockerfile은 empty 디렉터리에 저장하는게 좋아 -
* 한 줄이 하나의 명령어가 되고, 명령어(Instruction)를 명시한 뒤에 옵션을 추가하는 방식 (대소문자 상관없지만 일반적으로 대문자로 표기) 

### 명령어
- FROM: 생성할 이미지의 베이스가 될 이미지 반드시 한번 이상 입력, 사용하려는 이미지가 도커에 없다면 자동으로 pull
- MAINTAINER: 이미지를 생성한 개발자 정보 일반적으로 이메일 등, 도커 1.13.0버전 이후로 사용안하고 LABEL maintainer "imjehoon" 이런식으로 사용
- LABEL: 이미지에 메타데이터 "키:값" 형태 docker inspect 명령어로 이미지 정보 확인 가능
- RUN: 이미지를 만들기 위해 컨테이너 내부에서 명령 실행, 이미지 빌드시 별도의 입력을 받아야 하는 RUN이 있다면 build는 이를 오류로 간주하고 빌드 종료
  - RUN ["/bin/bash", "echo hello >> test2.html"] /bin/bash 셸을 이용해서 명령어를 실행 
  - RUN ["실행 가능 파일", "명령줄 인자1", "명령줄 인자2", ...]
- ADD: 파일을 이미지에 추가. 추가하는 파일은 Dockerfile이 위치한 디렉터리인 context에서 가져옴
  - ADD test.html /var/www/html test.html을 /var/www/html 디렉터리에 추가
  - ADD 명령어는 JSON 배열의 형태 ["추가할 파일 이름",..., "컨테이너에 추가될 위치"] 추가할 파일명은 여러개 지정 가능, 배열의 마지막 원소가 컨테이너에 추가될 위치
- ENV: 환경 변수를 지정 할 수 있다.
```sh
vi Dockerfile3

FROM ubuntu:14.04
ENV test /home
WORKDIR $test
RUN touch $test/mytouchfile

docker build -f Dockerfile3 -t myenv:0.0 ./

# 에러나는디요
docker run -it --name myenv:0.0 /bin/bash
docker: invalid reference format.

docker run -i -t myenv:0.0

# -e옵션으로 변수 오버라이드 가능
docker run -i -t --name env_test_override \
-e test=myvalue \
myenv:0.0
```
- VOLUME: 빌드된 이미지로 컨테이너를 생성했을때 호스트와 공유할 컨테이너 내부의 디렉터리 설정
  - ["home/dir", "home/dir2"] 처럼 여러개 사용 가능 home/dir home/dir2도 사용 가능
```sh
```
  
- WORKDIR: 명령어를 실행할 디렉터리, 배시 셸에서 cd 명령어를 입력하는 것과 같은 기능
  - WORKDIR /var WORKDIR www/html == WORKDIR /var/www/html
- EXPOSE: Dockerfile의 빌드로 생성된 이미지에서 노출할 포트를 설정, 반드시 이 포트가 호스트의 포트와 바인딩 된느 것은 아니며, 단지 컨테이너의 80번 포트를 사용하는걸 나타냄
- CMD: 컨테이너가 시작될 때마다 실행할 명령어, Dockerfile에서 한번만 사용 가능
  - json 배열 형태 ["실행 가능 파일", "명령줄 인자1", "명령줄 인자2", ...] 형태로도 사용 가능
  - ENTRYPOINT 명령줄 인자로도 사용 가능

### Dockerfile
```sh
vi Dockerfile

FROM ubuntu:14.04
MAINTAINER imjehoon
LABEL "purpose"="practice"
RUN apt-get update
RUN apt-get install apache2 -y
ADD test.html /var/www/html
WORKDIR /var/www/html
RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
EXPOSE 80
CMD apachectl -DFOREGROUND

# -t옵션은 생성될 이미지의 이름을 설정 ./ <- 현재 경로
docker build -t mybuild:0.0 ./   
docker images

# 생성된 이미지로 컨테이너를 실행 -P옵션은 이미지에 설정된 EXPOSE의 모든 포트를 호스트에 연결하도록 설정
docker run -d -P --name myserver mybuild:0.0 

# 실제 어떤 포트와 연결 되었는지
docker port myserver 

# 해당 라벨이 붙은 이미지만 출력
docker images --filter "label=purpose=practice"
```

### 이미지 빌드
1.빌드 컨텍스트를 읽음 - 컨텍스트는 하위 폴더도 포함되게 되므로 불필요한 빌드 파일이 포함되면 대 재앙, .dockerignore라는 파일을 작성하면 빌드시 제외 가능
  - ADD, RUN등의 명령어가 실행될때마다 새로운 컨테이너가 하나씩 생성되며 이를 이미지로 커밋 계속 레이어를 덮어 쓰는 구조.
  - Removing intermediate container bc37⊂d63ebe3 중간에 생성된 이미지를 삭제 한다는 로그!

### 캐시를 이용한 빌드
  - git 같이 내부 저장소 파일이 바뀔 경우 캐시를 안타게 하려면 --no-cache 옵션을 추가함
    - docker build --no-cache -t mybuild:0.0 .
  - 캐시 수동 지정 가능 eg) 도커허브 nginx 공식 저장소에서 nginx:latest 빌드시 일부만 변경된 경우
    - docker build --cache-from nginx -t my_extend_nginx:0.0 .
```sh
vi Dockerfile2 

FROM ubuntu:14.04
MAINTAINER imjehoon
LABEL "purpose"="practice"
RUN apt-get update

# -f or -file 옵션으로 Dockerfile 이름 지정 가능
docker build -f Dockerfile2 -t mycache:0.0 ./

Sending build context to Docker daemon  4.096kB
Step 1/4 : FROM ubuntu:14.04
 ---> 6e4f1fe62ff1
Step 2/4 : MAINTAINER imjehoon
 ---> Using cache
 ---> 050a156e211b
Step 3/4 : LABEL "purpose"="practice"
 ---> Using cache
 ---> 90b69867d112
Step 4/4 : RUN apt-get update
 ---> Using cache
 ---> 65b802f87e81
Successfully built 65b802f87e81
Successfully tagged mycache:0.0

```

### 멀티 스테이지 빌드 (Multi-state build)
  - 17.05이상 버전에서 이미지의 크기를 줄위이 위한 빌드
  - 하나의 Dockerfile 안에 여러개의 FROM 이미지를 정의 빌드 완료시 생성될 이미지 크기를 줄이는 역활

### Dockerfile 빌드 주의점
```sh
# 실제 100mb 크키 파일을 삭제해도 이전 레이어가 남아있기 때문에 크키가 우분투보다 100mb 크다!!
FROM ubuntu:14.04
RUN mkdir /test
RUN fallocate -l 100m /test/dummy
RUN rm /test/dummy

## and로 명령어를 묶어서 실행하면 위와 같은 케이스를 피할 수 있다.
ROM ubuntu:14.04
RUN mkdir /test && \
fallocate -l 100m /test/dummy && \
rm /test/dummy
```

