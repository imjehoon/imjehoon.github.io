---
layout: post
title: "Docker Image"
tags: [Docker, Kubernetes, DockerImage]
category: [server]
comments: true
---

### 도커 이미지 명령어 및 설명 

* 도커 허브에 ubuntu 라는 이미지가 어떤게 있는지 찾는 명령어
```shell
docker search ubuntu
```

* 도커 이미지 생성
```sh
docker commit \
-a "imjehoon" -m "my first commit" \
commit_test \
commit_test:first

# -a:커밋 작성자, -m: 메시지, commit_test:first(tag)
# docker commit [OPTION5] CONTAINER [REPO51TORY[8TAG]]
```

* 위에서 생성된 이미지로 새로운 이미지 생성 ( 근데 왜 얘는 docker images 로 하면 안나오지? 음...런만 하는거라 그런가?
```sh
docker run -i -t --name commit_test2 commit_test:first
```

* 도커 이미지에 대해 좀 더 상세한 내용을 알고 싶을 경우
  * 두개 명령어 실행시 Layers 부분을 보면 first 보다 second가 한줄 더 많은 것을 알 수 있다. ( 한번 더 수정 되었기 때문이다. )
```sh
docker inspect commit_test:first
docker inspect commit_test:second
```

* 도커 이미지에 대한 커밋 히스토리
```sh
docker history commit_test:second
```

* 도커 이미지 삭제
  * 명령어 실행시 오류 발생이유   
    컨테이너가 사용 중 인 이미지를 docker rmi 로강제로스쩌l하면이미지의이름이(none)으로변경 되며, 이러한 이미지들을 댕글링(dangling) 이미지라고 합니댜 댕글링 이미지는 docker images -f dangling=true 명령어를 사용해 별도로 확인할 수 있습니다.   
Untagged: commit_test:first -> 부여된 이름만 삭제 하겠다.
```sh
docker rmi commit_test:first
```

* 이미지 추출
  * -o 추출될 파일명
```sh
docker save -o ubuntu_14.04.tar ubuntu:14.04

# 저장된 이미지를 다시 로드
docker load -i ubuntu_14.04.tar 
```

* 도커 이미지 올리기
  * 이미지 이름의 접두어는 이미지가 저징되는 저징소 이름으로 설정합니다. 즉, 특정 이름의 저징소에 이미지를 올리려면 저징소 이름(시응자의 이름)을 이미지 앞에 접 두어로 추가해야 합니다.
  * docker tag 명령어를 시용하면 이미지의 이름을 추기할 수 있습니다. tag 명령어의 형식은 ‘dockel tag[기존의이미지이륌[새롭게생성될이미지이륌'으
```sh
# ubuntu로 commit_container1 을 생성함 오타
docker run -it --name comit_container1 ubuntu:14.04 

# my-image-name:0.0 으로 이미지 커밋
docker commit comit_container1 my-image-name:0.0 
docker tag my-image-name:0.0 imjehoon/my-image-name:0.0
docker push imjehoon/my-image-name:0.0

-- 팀 저장소 생성후 이미지 올리기
# my-image-name:0.0 에 jhlab/my-image-name:0.0 이라는 태그를 단다
docker tag my-image-name:0.0 jhlab/my-image-name:0.0  

# push하면 태그로 알아서 repository에 올라가네요 -> 이미지 저장소 자체를 private로 설정 안하면 모든 사람이 읽기 권한을 가짐.( 쓰기는 아닌듯)
docker push jhlab/my-image-name:0.0 
```

* 웹훅 추가
  *  저징소에 추가된 새로운 이미지를 각서버에 배포하는 애플리케이션을 작성할때 유용하게 활용할 수 있습니다.

* 사설 repository
  * 도커에서 공식적으로 제공하는 레지스트리 컨테이너
  * --restart 옵션은 컨테이너가 종료됐을때 재시작에 대한 정책
```sh
docker run -d --name myregistry \
> -p 5000:5000 \
> --restart=always \
> registry:2.6
curl localhost:5000/v2/

docker tag my-image-name:0.0 172.30.1.36:5000/my-image-name:0.0
docker push 172.30.1.36:5000/my-image-name:0.0
# error발생
The push refers to repository [172.30.1.36:5000/my-image-name]
Get https://172.30.1.36:5000/v2/: http: server gave HTTP response to HTTPS client
# 기본적으로 도커 데몬은 HTTPS를 시용하지 않은 레지스트리 컨테이너에 접근히지 못하도록 설정 아래처럼 docker 설정에서 자기 ip 등록해주면됨

docker push 172.30.1.36:5000/my-image-name:0.0
docker pull 172.30.1.36:5000/my-image-name:0.0

# 레지스트리 삭제 삭제 할 때 불륨도 함께 삭제하고 싶다면 docker rm 명령어에 --volumes옵션을 추가합니다.
docker rm --volumes myregistry

```
![스크린샷 2020-02-11 오후 10 19 17](https://user-images.githubusercontent.com/6602020/74240375-3104a600-4d1d-11ea-9898-5e4bdbfe3084.png)
