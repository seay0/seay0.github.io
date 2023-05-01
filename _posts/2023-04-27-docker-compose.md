---
date: 2023-04-27 00:00:00
layout: post
title: Fastify & MongoDB, docker-compose
subtitle: 'Fastify & MongoDB docker-compose 동시 실행'
description: Fastify & MongoDB, docker-compose
image: https://res.cloudinary.com/duruq2coh/image/upload/v1681964503/gitio/docker_p2kvi1.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1681964503/gitio/docker_p2kvi1.png
category: Docker
tags:
  - Fastify WAS
  - MongoDB
  - Docker Compose
  - DevOps BootCamp
author: seay0
paginate: false
---
> ## **1. Fastify WAS 서버 생성 / 컨테이너 화**

---

먼저 fastify를 이용해 기본적인 WAS 서버 세팅을 해볼 것이다.
fastify가 깔려있지 않을 경우 다음 명령을 통해 fastify를 설치해준 후, 정상적으로 설치가 되었는지 확인해본다.

```bash
$ npm i --global fastify-cli
$ fastify
# 다음과 같은 명령이 출력되면 설치가 완료된 것이다.
Fastify command line interface available commands are:

...

Launch 'fastify help [command]' to learn more about each command.
```

설치 완료 후, 다음 명령을 통해 fastify 프로젝트를 새로 생성해준다.

```bash
$ fastify generate <project-name>
```

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682769638/gitio/post/team/2/2_m50n5f.png)

나는 <project-name>을 helloworld-was로 하였고, 폴더와 프로젝트가 정상적으로 생성되었다. ```npm install```을 통해 필요한 모듈을 설치해준다.

기존에는 127.0.0.1에서 fastify를 테스트하지만, 로컬호스트가 배포 시에는 노출이 되지 않는다. 따라서 fastify 주소를 0.0.0.0 으로 세팅해주어야 한다고 fastify 공식 문서에서 권장하고 있다.

이 때 0.0.0.0은 모든 IPv4로부터 수신이 가능하다는 뜻이다.

```json
// package.json
"scripts": {
  "test": "tap \"test/**/*.test.js\"",
  "start": "fastify start -l info app.js --address 0.0.0.0",
  "dev": "fastify start -w -l info -P app.js"
}
```

위와 같이 start 뒤에 ```--address 0.0.0.0``` 옵션을 붙여주어 ```npm run start```를 할 경우 로컬에서 0.0.0.0:3000 으로 볼 수 있게 세팅해준다.

이렇게 세팅을 한 후 서버를 실행시켜보자.

```bash
$ npm run start

> helloworld-was@1.0.0 start
> fastify start -l info app.js --address 0.0.0.0

{"level":30,"time":1682905582125,"pid":7346,"hostname":"seay0-fairycode","msg":"Server listening at http://0.0.0.0:3000"}
```

기존에는 127.0.0.1:3000 에서 열렸던 서버의 주소가 0.0.0.0:3000 로 바뀐 것을 볼 수 있다. curl을 통해 접속해보면 정상적으로 서버가 열린 것이 확인된다.

```bash
$ curl http://0.0.0.0:3000
{"root":true}
```

<br>

> ## **2. Dockerfile 작성**

---

이제 간단하게 구축된 node.js 애플리케이션인 fastify 서버를 컨테이너화 하기 위해 fastify 최상위 디렉토리에 Dockerfile과 .dockerignore를 만든다.

.dockerignore에는 용량이 큰 node_modules 같은 것을 컨테이너화 하지 않기 위해 컨테이너에 올리지 않을 파일들을 적어주면 된다.

```bash
# .dockerignore
node_modules
```

Docerfile은 Docker docs의 [>> 공식 문서 <<](https://docs.docker.com/language/nodejs/build-images/)를 참고하여 작성하였다.

```dockerfile
# Dockerfile

FROM node:16-alpine
# node 16 버전을 사용한다.

WORKDIR /app
# /app 디렉토리를 작업 디렉토리로 설정한다.

COPY package*.json ./
# package*.json 형식으로 생긴 파일을 WORKDIR 위치에 복사한다.

RUN npm install
# WORKDIR에서 npn install을 통해 node_modules를 설치해준다.

COPY . .
# WORKDIR에 있는 전체 파일을 가져와서 이미지에 소스 코드를 추가한다.

EXPOSE 3000
# 컨테이너가 3000 포트를 노출하도록 설정한다.

CMD ["npm", "run", "start"]
# 컨테이너가 시작되었을 때 실행되는 명령을 설정한다.
```

Dockerfile을 작성한 후 이미지 빌드를 해주면, 정상적으로 이미지 생성이 완료된다.

```bash
$ docker build -t fastify-was:1.0 .
```

근데 ,,, 오류가 났다. 

```bash
[+] Building 0.1s (3/3) FINISHED                                                                           
 => [internal] load build definition from Dockerfile                                                  0.0s
 => => transferring dockerfile: 32B                                                                   0.0s
 => [internal] load .dockerignore                                                                     0.0s
 => => transferring context: 34B                                                                      0.0s
 => ERROR [internal] load metadata for docker.io/library/node:16-alpine                               0.0s
------
 > [internal] load metadata for docker.io/library/node:16-alpine:
------
ERROR: failed to solve: rpc error: code = Unknown desc = failed to solve with frontend dockerfile.v0: failed to create LLB definition: failed to do request: Head "https://registry-1.docker.io/v2/library/node/manifests/16-alpine": dial tcp: lookup registry-1.docker.io: Temporary failure in name resolution
```

네트워크 연결이 갑자기 끊겼더라구요 ^^ 껐다 키니까 연결되고 빌드도 정상적으로 됨. 제발 억까 ㄴ

```bash
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
fastify-was   1.0       <secret^^>     27 seconds ago   187MB
```

이런 식으로 이미지를 빌드할 수 있는데, 나는 이제 docker-compose와 yaml 파일을 이용해 was 이미지와 mongo 이미지를 동시에 실행하는 과정을 진행해볼 것이다.

<br>

> ## **3. docker-compose.yml 작성**

---

먼저 Mongo 이미지를 가져온다.

```bash
$ docker pull mongo
```

이제 docker-compose.yml 파일을 작성해준다.

```yml
# docker-compose.yml
version: '3.1'

services:

  mongo:
    image: mongo:latest
    restart: always
    ports: 
      - '27017:27017'

  fastify: 
    image: fastify-was:1.0
    restart: always
    ports: 
      - '3000:3000'
    depends_on:
      - mongo
```

이렇게 작성한 후 ```docker-compose up```을 실행하면 was 서버와 mongo 서버가 각각 지정한 localhost:포트에서 열린다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911873/gitio/post/team/docker-push/1_lfjnqw.png)
![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911873/gitio/post/team/docker-push/2_oqcxkf.png)

mongo는 mongo compass에서도 확인할 수 있다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911873/gitio/post/team/docker-push/3_nixhvg.png)
![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911873/gitio/post/team/docker-push/4_wabjss.png)

<br>

> ## **4. GitHub Action을 이용한 이미지 build 자동화**

---

GitHub Action을 이용하기 위해 최상위 디렉토리에 ```.github/workflows``` 폴더를 생성한다.

workflows 폴더 안에 docker-push.yml 파일을 작성해준다.

```yaml
# docker-push.yml
name: Docker Build

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/fastify-was:1.1
```

yml 안에 적혀있는 ```${{ secrets.DOCKER_USERNAME }}```는 GitHub 세팅에서 설정해줄 수 있다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911873/gitio/post/team/docker-push/5_yqg9pe.png)

Github에서 생성한 레포지토리의 Setting으로 들어간다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911873/gitio/post/team/docker-push/6_imd4ue.png)

왼쪽 메뉴에 Secrets and variables의 Actions 탭을 선택한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911873/gitio/post/team/docker-push/7_gjxkxm.png)

New repository secret을 클릭한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911873/gitio/post/team/docker-push/8_mkyxom.png)

위 사진과 같이 DOCKER_USERNAME과 그에 해당하는 값을 배치해주고 저장한다. USERNAME에는 아이디가 아닌 도커 우측 상단에 있는 사용자 이름을 입력해주어야 한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911873/gitio/post/team/docker-push/9_pmb0gx.png)

DOCKER_PASSWORD도 같은 방식으로 생성해준다. PASSWORD는 Docker 계정 비밀번호를 적어주면 된다.

이제 설정을 마쳤으니 깃허브에 push를 해주면 Action 탭에서 액션이 돌아간다.

```bash
$ git add .
$ git commit -m 'docker-push'
$ git push
```

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911874/gitio/post/team/docker-push/10_hz3r5d.png)

방금 푸시한 커밋 이름으로 돌아가고 있는 것을 볼 수 있다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911873/gitio/post/team/docker-push/11_k03ylr.png)

이렇게 체크 표시가 뜨면 Docker Hub에 정상적으로 이미지가 푸시 된 것이다. 로그인 해서 확인해보자.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911874/gitio/post/team/docker-push/12_poepaw.png)

yml에서 지정한 이미지 이름으로 새로 생성된 것이 보인다. 자세히 볼까?

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682911874/gitio/post/team/docker-push/13_ct1d2j.png)

태그 탭으로 가보면 태그도 yml에서 지정한대로 1.1로 생성되어 있다.

나는 이 과정을 진행하며 기존에 helloworld-was로 만들었던 폴더 안의 내용물들을 모두 Github 레포지토리를 생성해서 clone한 뒤에 Github 폴더 안으로 옮겨 주었다.

```
Error: buildx failed with: ERROR: failed to solve: rpc error: code = Unknown desc = failed to solve with frontend dockerfile.v0: failed to read dockerfile: open /var/lib/docker/tmp/buildkit-mount1197326096/Dockerfile: no such file or directory
```

제대로 된 위치로 옮겨 주지 않아서 위와 같은 에러가 발생했다. 깃허브 폴더의 최상위 디렉토리에 .github/workflows 폴더가 존재하고, Dockerfile이 같은 위치에 있어야 Action이 정상적으로 돌아가기 때문이었다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682912703/gitio/post/team/docker-push/14_rsxocs.png)

폴더를 바꾸면서 폴더 이름이 바뀌었기 때문에 package.json 파일의 제일 위에 적혀있는 이름도 fastify-practice로 바꿔주었다 ... ㅋ hoxi 모르잖아요

```bash
$ docker pull seay0/fastify:1.1
```

깃헙 액션으로 이미지를 새로 푸시한 후 docker-compose up으로 테스트를 해보았는데, 그 전에 방금 Action으로 새로 푸시한 이미지를 받아와서 docker-compose.yml 파일에 image이름을 수정해주었다.

```yaml
fastify: 
  image: seay0/fastify-was:1.1
  restart: always
  ports: 
    - '3000:3000'
  depends_on:
    - mongo
```

위와 같이 이미지 이름을 바꾼 후 다시 docker-compose up을 해보았는데,

```bash
ERROR: for fastify-practice_mongo_1  Cannot start service mongo: driver failed programming external connectivity on endpoint fastify-practice_mongo_1 (0c47d94fb29cf79b7b3077a20f0d419ea8ebbe1f8d0bb5f0d7a8b5804a5cef83): Bind for 0.0.0.0:27017 failed: port is already allocated
```

포트가 이미 사용중이란다. 이전에 실행하던 컨테이너를 다시 지우고 해줘야겠다 ㅎㅎ

```bash
$ docker container ps -a
# 모든 컨테이너 조회
$ docker container stop <멈출 컨테이너 ID>
$ docker container rm <지울 컨테이너 ID>
```

지우고 다시 compose up을 해보면 정상적으로 서버가 작동된다.

<br>

> ## **5. Mongo 서버 환경 변수 설정**

---

기존에 설정했던 docker-compose.yml 파일을 수정해서 나의 MongoDB에 아무나 접속할 수 없게 설정해보자.

```yml
mongo:
  image: mongo:latest
  restart: always
  ports: 
    - '27017:27017'
  environment:
    MONGO_INITDB_ROOT_USERNAME: root
    MONGO_INITDB_ROOT_PASSWORD: example
```

실행 중이던 mongo 컨테이너를 없애고, mongo 쪽에 environment를 추가해서 다시 컨테이너를 만들어 줄 것이다.

코드를 수정한 후 ```docker-compose-up```을 해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682913728/gitio/post/team/docker-push/15_rs8upr.png)


그럼 기존에 포트만 있어도 접속이 가능하던 DB가 위와 같은 오류가 뜨면서 DB를 이용할 수 없다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682913728/gitio/post/team/docker-push/16_uvesgj.png)

방금 설정한 아이디와 비밀번호를 입력하고 다시 접속을 해보면,

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682913728/gitio/post/team/docker-push/17_mmgviz.png)

이렇게 정상적으로 이용이 가능해진다.

<br>

***오늘은 여기까지 끝 !!***