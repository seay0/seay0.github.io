---
date: 2023-04-11 07:26:03
layout: post
title: Docker? &#35;2 / TIL
subtitle: 'Docker CLI 실습'
description: Docker 실습
image: https://res.cloudinary.com/duruq2coh/image/upload/v1681175914/gitio/docker_dk06pg.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1681175914/gitio/docker_dk06pg.png
category: Docker
tags:
  - Docker
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **Docker ?**

Docker는 컨테이너 방식으로, 어떠한 실행 환경에 구애 받지 않고 애플리케이션을 실행할 수 있게 하는 소프트웨어이다.  

**컨테이너 방식**  
1. 의존성 충돌 문제를 해결해준다.
2. 개발과 배포 환경을 일치시킨다.
3. 수평 확장을 쉽게 해준다.
4. 각 서버에 새로운 내용을 배포하기 쉽게 만들어준다.  

<br>

### **1. 의존성 충돌 문제 해결**  
--- 

어떠한 애플리케이션은 반드시 이 어플을 실행하기 위해서 구축되어야 하는 환경이 존재하는데, 이것을 ***프로그램 A는 프로그램 B에 의존 관계를 가지고 있다***고 말한다.  
  
만일 A 프로그램과 B 프로그램이 C 프로그램의 서로 다른 버전을 요구할 때, ***도커 서버는 프로그램을 컨테이너 내에 구성***하기 때문에 어떠한 의존성도 공유하지 않고, 각자 고유의 의존성을 포함할 수 있게 된다.  

### **컨테이너가 격리하는 것 / 독립적으로 소유하는 자원**  
* **프로세스**  
  특정 컨테이너에서 작동하는 프로세스를 기본적으로 그 컨테이너 안에서만 액세스할 수 있고, 컨테이너 안에서 실행되는 프로세스는 다른 컨테이너의 프로세스에 영향을 줄 수 없다.
* **네트워크**  
  기본으로 컨테이너 하나에 하나의 IP주소가 할당되어 있다.
* **파일 시스템**  
  컨테이너 안에서 사용되는 파일 시스템은 *구획화되어 있기 때문에, 해당 컨테이너에서의 명령이나 파일 등의 액세스를 제한할 수 있다.

###### *구획화 : 섹션(파티션)을 나누는 행위

<br>

### **2. 개발과 배포 환경 일치** 
---

다양한 프로그램들은 각 프로그램마다 배포판에 따라 다양하고 다른 설치 과정이 진행된다. 이 다양한 설치 과정이 docker를 실행중이라면 쉽게 설치할 수 있다.  

또한 애플리케이션 구성 자체가 컨테이너화 되면 (Docker Compose라는 툴을 이용), YAML 파일 하나 + 명령어 하나로 모든 애플리케이션 실행 환경 구성을 완료할 수 있다.

```bash
docker-compose up
```
따라서 Docker는 개발을 컨테이너 위에서 진행할 경우, 모든 개발 팀이 동일한 환경 하에 개발을 진행할 수 있다. 실행 및 개발 환경의 일치는 서비스 배포 환경에서도 동일하게 적용될 수 있다.  

서버도 컨테이너에 담긴 애플리케이션을 실행하는 방식으로 서비스를 제공한다. 따라서 Amazon Web Service의 EC2 상에 도커를 설치하거나, 도커 컨테이너를 EC2 서버에서 실행할 수 있게 하는 서비스인 EC2를 이용해 보다 쉽게 애플리케이션을 배포할 수 있다.  

이전에 개발자들의 밈중 하나였던 "제 컴퓨터에서는 작동되는데요?" 라는 말은 컨테이너 환경에서는 더 이상 유효하지 않는다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681260850/gitio/post/0F015ojiD49EssQ_6ixaC-1634633110721_usrbzn.png)

<br>

### **3. 수평 확장 / 배포**  
---

글로벌 웹 서비스는 전 세계인들이 사용하여 그 트래픽이 어마어마한데, 서비스 제공자들은 이러한 트래픽 분산을 위해 **프록시 서버**를 운영하며, 여러 대의 동일한 서버 중 한 군데를 이용하도록 한다. (리버스 프록시: 로드 밸런서)  

이러한 서비스에 도커가 필요하다. 컨테이너의 기술의 가장 큰 장점은 **실행 환경의 일치**이기 때문에, 서버 증설에 컨테이너 기술이 활발하게 이용된다. 이는 동일한 애플리케이션 구성을 바탕으로 새로운 서버에 해당 애플리케이션을 컨테이너로 실행하고, 로드 밸런서에 이 서버를 추가하기만 하면 된다. (AWS는 서버 만들기와 삭제를 자동으로 해준다.)  

* **오케스트레이션 도구** : 설명한 기술을 응용하여, 새 버전의 애플리케이션을 여러 서버 중 몇 대에만 운영하여 테스트하게 해주는 도구 (ex. 쿠버네티스)

<br>

### **Docker의 핵심 키워드**  
---

* #### **컨테이너**  
  애플리케이션이 의존성, 네트워크 환경, 파일 시스템에 구애받지 않고, 도커라는 기술 위에 실행될 수 있도록 만든 애플리케이션 상자  

* #### **이미지**
  실행되는 모든 컨테이너는 이미지로부터 생성된다. 이미지는 애플리케이션 및 애플리케이션 구성을 함께 담아놓은 템플릿으로, 이를 이용해 즉시 컨테이너를 만들 수 있다. 이미지를 이용해 여러 개의 컨테이너를 생성하여 수평 확장이 가능하다.  

  이미지는 기본 이미지로부터 변경 사항을 (git 처럼) 추가 / 커밋 해서 또 다른 이미지를 만들 수도 있다.  

* #### **레지스트리**
  이미지는 레지스트리에 저장된다. 도커 CLI에서 이미지를 이용해 컨테이너를 생성할 때, 호스트 컴퓨터에 이미지가 존재하지 않으면, 기본 레지스트리로부터 다운로드 받는다. (ex. Docker hub, Amazon ECR)
  
<br>

### **이미지 구분**  
---

```
Registry_Account/Repository_Name:Tag
```

* **레지스트리 (Registry)**  
  도커 이미지를 관리하는 공간  
  특별히 다른 것을 지정하지 않으면, Docker Hub를 기본 레지스트리로 설정한다.  
  ex. Docker Hub, Private Docker Hub, 회사 내부용 레지스트리 등  
  
* **레포지토리 (Repository)**  
  레지스트리 내에 도커 이미지가 저장되는 공간  
  이미지 이름이 사용되기도 한다. (github repo와 비슷함)  

* **태그 (Tag)**
  해당 이미지를 설명하는 버전 정보를 주로 입력한다.
  특별히 다른 것을 지정하지 않으면, ```latest``` 태그를 붙인 이미지를 가져온다.

```
docker/whalesay:latest
```
위 문장을 해석해보면,  

'Docker Hub 레지스트리에서 ```docker```라는 유저가 등록한 ```whalesay``` 이미지 혹은 레포지토리에서 ```latest``` 태그를 가진 이미지를 가져온다.'  

라는 의미이다.