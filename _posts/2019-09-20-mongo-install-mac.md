---
layout: post
title: "mongodb mac에 brew를 사용해서 설치 및 실행"
tags: [mongo, mongodb, mac, brew]
category: [db]
comments: true
---

### mongodb brew 설치하기.

2019.09.20 기준으로 brew install mongodb 실행할 Warning: homebrew/core is shallow clone 와 같은 에러가 발생한다
다행히 다른 방법으로 brew를 활용한 쉬운 설치 방법이 있다.
아래와 같이 실행하면 일단 mongodb는 설치된다.

```shell
  brew tap mongodb/brew
  brew install mongodb-community
```

### mongodb brew 실행하기.

기본적으로 설치한 후 mongod를 커맨드에서 실행시 /data/db폴더가 없다는 에러가 발생할것이다.
그럼 일단 자신이 원하는 폴더를 만들고 아래와 같이 --dbpath 옵션을 추가하여 실행하면 mongodb가 시작은된다.
```shell
  mkdir /usr/local/mongodb-data
  mongod --dbpath /usr/local/mongodb-data
```

이제 다른 커맨드를 열고 mongo 명령어를 실행하면 잘 접속 되지만, 서비스로 실행한게 아니라서 실행한 커맨드를 종료하면 몽고디비가 종료된다.
마지막으로 몽고 디비를 서비스로 등록하고 dbpath를 수정하는 방법이다.

```shell
 vi /usr/local/etc/mongod.conf

 systemLog:
  destination: file
  path: /usr/local/var/log/mongodb/mongo.log
  logAppend: true
storage:
  #dbPath: /usr/local/var/mongodb
  dbPath: /Users/imjehoon/data/db <-- 요 부분을 수정해주면 된다.
net:
  bindIp: 127.0.0.1

# 몽고 디비 시작 
brew services start mongodb-community
# 몽고 디비 종료
brew services stop mongodb-community
```

이제 서비스로 시작하면 커맨드를 닫아도 계속 실행중인 것을 확인 할 수 있다.
다음은 기본적인 데이터 다루기를 포스팅 할 예정이다.
