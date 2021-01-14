---
layout: post
title: "mongo docker 설치 및 mongo compass 접속"
tags: [mongo, mongodb, mac]
category: [db]
comments: true
---

# 몽고디비 docker 실행 방법
1. docker run 실행
```sh
docker run -d --name mongo-db -v /Users/user/mongo/db:/data/db -p 27017:27017 mongo:4.2
```
2. 몽고디비 터미널 접속
```sh
docker exec -it mongo-db-t3 /bin/bash
-- 접속 후 
root@ac523620dea1: mongo
```

3. compass 다운로드
https://www.mongodb.com/products/compass

4. 설치 후 신규 접속 생성 CONNECT 버튼 누르면 로컬 몽고 디비 접속 완료! 끝! 

