---
date: 2023-04-20 00:00:00
layout: post
title: GitHub Action / TIL
subtitle: 'Automate Build & Test'
description: Automate Build & Test
image: https://res.cloudinary.com/duruq2coh/image/upload/v1682394547/gitio/KakaoTalk_20230424_214818964_05_k3gbjz.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1682394547/gitio/KakaoTalk_20230424_214818964_05_k3gbjz.png
category: DevOps
tags:
  - Github Action
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **GitHub Action**  

깃허브에서 제공하는 CI 서비스로, 소스 코드가 변경될 때마다 지정된 작업을 자동으로 수행할 수 있다. (새로운 코드를 커밋할 때마다 코드 빌드, 테스트, 배포, 린트 등 작업 자동화)

**사용하는 이유**
* 자동화된 작업 수행
* 지속적 통합 및 배포
* 커뮤니티 기반 작업 공유

yaml 문법으로 작성된 워크플로우로 구성되어 있으며, 워크플로우는 하나 이상의 작업으로 구성된다. 작업은 수행해야 하는 쉘 스크립트 또는 도커 컨테이터로 정의된다. 작업은 트리거에 의해 실행되며, 이 트리거는 레포지토리의 이벤트와 일치하는 규칙을 정의한다. 

예를 들어 코드가 커밋되었을 때, 레포지토리에 새로운 풀리퀘가 생성될 때, 레포지토리에 새로운 이슈가 생성될 때 등이 트리거가 될 수 있다.


<br>

### **환경 변수**  
---
운영체제 혹은 소프트웨어가 각기 다른 컴퓨터 또는 사용자마다 별도로 가질 수 있는 고유한 정보를 담는 데 사용하는 변수

환경 변수는 새로운 배포마다, 새로운 기능들을 배포함으로써 설정값이 바뀐다. 따라서 코드의 버전을 관리하는 것처럼 설정값의 버전을 관리할 필요가 있다.

**설정을 환경변수를 통해 분리하는 이유**
* 환경 변수는 코드 변경 없이 배포 때마다 쉽게 변경할 수 있고, 설정 파일과 달리, 코드 push할 때 올라가지 않기 때문에 잘못해서 코드 저장소에 올라갈 가능성도 적다.
* 다른 설정 메커니즘과 달리 언어나 OS에 의존하지 않는다.


**환경 변수 설정 예**
* Linux 운영체제 : export 명령어
* 코드를 통해 환경 변수를 설정하는 법 : node.js => process.env
* 서비스 내에서 환경 변수 설정하는 법 : **GitHub Action**
