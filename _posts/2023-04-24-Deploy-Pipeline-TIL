---
date: 2023-04-24 00:00:00
layout: post
title: Deployment Pipeline / TIL
subtitle: '배포 자동화 / 파이프라인'
description: 배포 자동화 / 파이프라인
image: https://res.cloudinary.com/duruq2coh/image/upload/v1681965389/gitio/devops_ulvdb1.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1681965389/gitio/devops_ulvdb1.png
category: DevOps
tags:
  - Pipeline
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **배포 자동화**  

배포 자동화란, 한 번의 클릭 혹은 명령어 입력을 통해 전체 배포 과정을 자동으로 진행하는 것이다.

배포 자동화는 수동적이고 반복적인 배포 과정을 자동화함으로써 시간이 절약된다.

또한 전체 배포 과정을 매번 일관돠게 진행하는 구조를 설계하기 때문에, 휴먼 에러(Human Error)를 방지할 수 있다. 휴먼 에러란, 사람이 수동적으로 배포 과정을 진행하는 중에 생기는 실수를 말한다. 그 전에 했던 배포 과정과 비교하여 특정 과정을 생략하거나 다르게 진행하여 오류가 발생하는 것이 휴먼 에러의 예이다. 

<br>

### **배포 자동화 파이프라인**
---
파이프라인이란, 소스 코드의 관리부터 실제 서비스로의 배포 과정을 연결하는 구조를 뜻한다. 파이프라인은 전체 배포 과정을 여러 단계로 분리하고, 각 단계는 파이프라인 안에서 순차적으로 실행되며, 단계마다 주어진 작업을 수행한다. 

1. **Source 단계** : 원격 저장소에 관리되고 있는 소스 코드에 변경 사항이 일어날 경우, 이를 감지하고 다음 단계로 전달하는 작업을 수행한다. (버전 컨트롤 도구를 이용한 소스코드 관리 및 전달)
2. **Build 단계** : Source 단계에서 전달받은 코드를 컴파일, 빌드, 테스트하여 가공한다. Build 단계를 거쳐 생성된 결과물을 다음 단계로 전달하는 작업을 수행한다.
3. **Deploy 단계** : Build 단계로부터 전달받은 결과물을 실제 서비스에 반영하는 작업을 수행한다.

<br>

### **AWS 개발자 도구**
---
개발자 도구 섹션에서 제공하는 서비스를 활용해 배포 자동화 파이프라인을 구축할 수 있다.

**개발자 도구 제공 서비스 목록**
* **CodeCommit**  
Github와 유사한 서비스를 제공하는 버전 관리 도구이며, Source 단계를 구성할 때 CodeCommit 서비스를 이용한다. 보안에 대한 강점이 있지만, 과금 가능성을 고려해야 한다.
* **CodeBuild**  
Build 단계에서 이용하며, 유닛 테스트, 컴파일, 빌드와 같은 필수적으로 실행되어야 할 작업을 명령어를 통해 실행할 수 있다.
* **CodeDeploy**  
실행되고 있는 서버 애플리케이션에 실시간으로 변경사항을 전달하고 반영할 수 있다. 
* **CodePipeline**  
각 단계를 연결하는 파이프라인을 구축할 때 이용한다.
* **Cloud9**
* **CloudShell**
* **X-Ray**
* **AWS FIS**
* **CodeArtifact**
* **CodeStar**

<br>