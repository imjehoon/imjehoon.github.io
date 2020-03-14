---
layout: post
title: "Docker Compose란?"
tags: [Docker, Kubernetes, DockerCompose]
category: [server]
comments: true
---
### Docker compose란?
- 컨테이너를 이용한 서비스의 개발과 CI를 위해 여러개의 컨테이너를 하나의 프로젝트로서 다룰 수 있는 작업 환경을 제공
- 여러개의 컨테이너 옵션과 환경을 정의한 파일을 읽어 컨테이너를 순차적으로 생성
- 설정 파일은 run 명령어의 옵션을 그대로 사용, 컨테의너 의존성, 네트워크 볼륨 등 정의 가능
- 스웜 모드와 비슷하게 컨테이너 수를 유동적 조절 가능, 컨테이너 서비스의 디스커버리도 자동

### 설치
- https://docs.docker.com/compose/install
```sh
curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose 
chmod +x /usr/local/bin/docker-compose

# Note: If the command docker-compose fails after installation, check your path. You can also create a symbolic link to /usr/bin or any other directory in your path.
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

### 기본 docker-compose.yml 파일
- 실행 중 도커 컨테이너 모두 종료 
```sh 
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```
- docker-compose.yml 파일
```yaml
version: '3.0' # yaml파일 버전 1.2, 2.1, 3.0등이 있음 도커 컴포즈 1.8은 2, 1.10은 3.0 가능하면 최신버전 사용, 릴리즈 페이지에서 확인 가능
services: # 생성될 컨테이너들을 묶어놓는 단위
  web:  # 생성될 서비스의 이름
    image: alicek106/composetest:web
    ports:
      - "80:80"
    links:
      - mysql:db
    command: apachectl -DFOREGROUND
  mysql: # 생성될 서비스의 이름
    image: alicek106/composetest:mysql
    command: mysqld

## 실행 명령어
docker-compose up

## 하나의 서비스에는 여러개의 컨테이너가 존재할 수 있으므로 차례대로 증가하는 컨테이너의 번호를 붙여 서비스 내의 컨테이너 구별.
docker-compose scale mysql=2 
docker-compose ps

## 프로젝트 삭제
docker-compose down

## -p 옵션 사용시 프로젝트 이름을 명시 가능
docker-compose -p myproject up -d
docker-compose -p myproject ps
docker-compose -p myproject down
```

### Docker compose 활용 
- 기본적으로 현재 디렉터리 또는 상위 디렉터리에서 docker-compose.yml이라는 이름의 yaml파일을 찾아 컨테이너 생성
  - -f옵션을 사용하면 파일 위치 지정 가능 ``` docker-compose \ -f /home/xxx/my_compose_file.yml \ up -d ``` -p옵션과 함께 사용 가능  
- YAML 파일 작성
  - 크게 버전, 서비스, 볼륨, 네트워크 4가지를 정의하는 항목으로 구성되며 가장 많이 사용하는 것은 서비스 정의이며, 볼륨 정의와 네트워크 정의는 서비스로 생성도니 컨테이너에 선택적으로 사용
  1. 버전 정의 - 최신버전 사용하는게 좋음   
  ```yaml
  version: '3.0'
  ```
  2. 서비스 정의 - 도커 컴포즈로 생성할 컨테이너 옵션 정의. 각 서비스는 컨테이너로 구현되며, 하나의 프로젝트로서 도커 컴포즈에 의해 관리
    - https://docs.docker.com/compose/compose-file/
  ```yaml
  services:
    my_container_1:
      image: alicek106/composetest:mysql #컨테이너 생성할때 쓰일 이미지 이름 설정. 포맷은 docker run 과 같음 이미지 없으면 저장소에서 자동으로 내려받음.
      links: # docker run --link와 같으며 다른 서비스에 서비스명으로 접근 하도록 설정
        - my_container_2
        - redis
      environment: # docker run --evn -e 옵션과 동일. 서비스의 컨테이너 내부에서 사용할 환경변수 지정하며, 딕셔너리나 배열 형태 가능
        - MYSQL_ROOT_PASSWORD=mypassword
        - MYSQL_DATABASE_NAME=mydb
      command: apachectl -DFOREGROUND # 컨테이너가 실행될때 수행할 명령어 docker run 명령어의 마지막에 붙는 커맨드와 동일, Dockerfile RUN과 같은 형태 사용 가능 [ apachectl, -DFOREGROUND]
      depends_no: # 특정 컨테이너에 대한 의존 관계를 나타냄, 이항목에 명시된 컨테이너가 먼저 생성되고 실행
        - mysql # my_container_1보다 mysql이 먼저 생성 및 실행, links와 차이점은 서비스 이름만으로 접근 할 수 있다는 점이 다름   
      entrypoint: # https://docs.docker.com/compose/compose-file/#entrypoint 초기화 스크립트 등을 지정할 수 있다.
      ports: # docker run -p 와 같으며 서비스의 컨테이너를 개방할 포트 설정. ( 단일 호스트 환경에서 80:80과 같이 호스트의 특정 포트를 컨테이너에 연결하면 scale 명령어로 컨테이너 증가 불가)
        - "8080" # https://docs.docker.com/compose/compose-file/#ports
        - "8081-8005"
        - "80:80"
      build: ./composetest # 정의된 도커파일에서 이미지를 빌드해 서비스의 컨테이너를 생성하도록 설정 ./composetest디렉터리에 저장된 도커파일로 이미지빌드 
      extends: # 다른 compose 파일 상속가능
        file: #상속받을 파일
        service: #파일안에 상속 받을 서비스
    my_container_2:
      image: ...
  ```
```sh
# 특정 서비스의 컨테이너만 생성하되 의존성 없는 컨테이너 생성하려면
docker-compose up --no-deps web
```

### 네트워크 정의
- driver: 기본적으로 브리지 타입의 네트워크를 생성. yaml파일에서 driver항목을 정의해 다른 네트워크를 사용하도록 설정 가능 ( overlay타입 네트워크는 스웜모드나 주키퍼를 사용하는 환경이어야만 생성 가능 )
```yaml
version: '3.0'
services:
  myservice:
    image: nginx
    networks:
      - mynetwork
networks:
  mynetwork:
    driver: overlay
    driver_opts:
      subnet: "255.255.255.0"
      IPAdress: "10.0.0.2"
```

- ipam: IPAM(IP Address Manager) subnet, ip 범위 등을 설정 할 수 있다. driver항목은 IPAM을 지원하는 드라이버 이름 입력
```yaml
version: '3.0'
services:
  myservice:
    image: nginx
    networks:
      - mynetwork
networks:
  ipam:
    driver: mydriver
    config:
      subnet: 172.20.0.0/16
      ip_range: 172.20.5.0/24
      getway: 172.20.5.1  
```

- external: yaml 파일을 통해 프로젝트를 생성할 때마다 네트워크를 생성하는 것이 아닌, 기존의 네트워크를 사용하도록 설정

### 볼륨 정의
- driver: 볼륨 생성할때 사용할 드라이버 설정
- external: 외부 기존 볼륨을 사용하도록 설정 가능

### 파일 검증
```sh
docker-compose config
``` 

***

### 도커 스웜 모드와 함께 사용하기
- stack: yaml파일에서 생성된 컨테이너의 묶음으로서 yaml 파일로 스택을 생성하면 정의된 서비스가 스웜 모드의 클러스에서 일괄적으로 생성됨.  
즉 yaml파일에 정의된 서비스가 스웜 모드의 서비스로 변환된 것. docker-compose가 아니라 docker stack으로 제어 가능  
links나 depends_on과 같이 의존성 정의하는 항목 사용 불가.  
도커 컴포즈의 네트워크와 다르게 스택의 네트워크는 기본적으로 오버레이 네트워크 속성

```sh
# --config-file or -c로 파일 지정
docker stack deploy -c docker-compose.yml mystack

# stack 확인
docker stack ls
docker stack ps mystack

docker service ls
docker stack services mystack

# scale up
docker service scale mystack_web=2
# delete
docker stack rm mystack
```
 


