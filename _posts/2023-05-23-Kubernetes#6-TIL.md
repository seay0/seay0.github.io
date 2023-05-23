---
date: 2023-05-23 00:00:00
layout: post
title: Kubernetes? &#35;6 / TIL
subtitle: 'helm'
description: helm
image: https://res.cloudinary.com/duruq2coh/image/upload/v1684374884/gitio/Kubernetes_ahpltn.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1684374884/gitio/Kubernetes_ahpltn.png
category: Kubernetes
tags:
  - Kubernetes
  - helm
  - Package manager
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **Helm** 
---

helm은 쿠버네티스 패키지 매니저로, 쿠버네티스 워크로드를 하나로 묶어서 패키지 형태로 만들고, 배포하고, 설치할 수 있는 도구이다. 하나의 애플리케이션 구성이 최소 하나 이상의 파드와 서비스로 구성되어 있음을 생각해 봤을 때, 별개의 워크로드를 하나하나 적용하기보다는, 한 번에 여러 개의 워크로드가 즉시 배포된다면 간편할 것이다.
* helm의 패키지 : **차트**
* helm의 패키지 저장 공간 : **저장소**
* 차트를 설치하여, 쿠버네티스 클러스터에 구동될 때, 차트의 인스턴스 : **릴리스**

<br>

**helm 설치**

Window에서 helm을 사용하기 위해서는 chocolatey를 설치하고 진행하면 편리하다.

cmd를 관리자 권한으로 실행한 후 다음 명령어를 입력한다.

```bash
> @powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin
```

다른 경로에 설치할 경우 ChocolateyInstall 환경 변수에 설치할 폴더를 지정하고 수동으로 폴더를 생성한다.

```bash
> set ChocolateyInstall=d:\devel\choco
```

choco 로 패키지를 설치하고 powershell 에서 바로 실행시 경로를 못 찾는다고 나올 때 한 번 실행해 준다.

이제 다음 명령어를 이용해 helm을 설치한다.

```bash
> choco install kubernetes-helm
```

명령어 실행 중 Yes No 선택 화면이 나오면 (Y)를 선택해주면 설치가 완료된다.

<br>

**Jenkins 설치**

```bash
> helm repo add jenkins https://charts.jenkins.io
"jenkins" has been added to your repositories
> helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jenkins" chart repository
Update Complete. ⎈Happy Helming!⎈
```

Get Repo Info : repository 정보를 업데이트한다.

```bash
> helm install jk jenkins/jenkins   
```

Install Chart : jk는 릴리스(인스턴스) 이름이다.  

```bash
Error: INSTALLATION FAILED: Kubernetes cluster unreachable: Get "https://127.0.0.1:65118/version": dial tcp 127.0.0.1:65118: connectex: No connection could be made because the target machine actively refused it.
```

도커와 minikube를 실행하지 않고 진행하면 아래와 같은 오류가 난다. 꼭 실행중인지 확인한 후 진행하자.

```bash
> helm install jk jenkins/jenkins  
...

NOTES:
1. Get your 'admin' user password by running:
  kubectl exec --namespace default -it svc/jk-jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
2. Get the Jenkins URL to visit by running these commands in the same shell:
  echo http://127.0.0.1:8080
  kubectl --namespace default port-forward svc/jk-jenkins 8080:8080

3. Login with the password from step 1 and the username: admin
4. Configure security realm and authorization strategy
5. Use Jenkins Configuration as Code by specifying configScripts in your values.yaml file, see documentation: http://127.0.0.1:8080/configuration-as-code and examples: https://github.com/jenkinsci/configuration-as-code-plugin/tree/master/demos

...
```

위 터미널 출력에 나와있는 NOTES 안내대로 암호를 획득하고, 포트 포워딩을 통해 서비스를 노출시켜, http://localhost:8080 으로 접속해보자.

**NOTES**  
1. ```kubectl exec --namespace default -it svc/jk-jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo``` 명령은 Jenkins의 'admin' 사용자. 'jk-jenkins' 서비스 컨테이너 내에서 명령을 실행하고 비밀번호를 출력한다.

2. 다음 명령은 Jenkins 인스턴스에 로컬로 액세스하기 위한 Jenkins URL을 얻는 데 사용된다.
* ```echo http://127.0.0.1:8080``` 명령은 단순히 http://127.0.0.1:8080 인 Jenkins URL을 인쇄한다.
* ```kubectl --namespace default port-forward svc/jk-jenkins 8080:8080``` 명령은 로컬 시스템의 localhost:8080 에 대한 요청이 다음에서 실행 중인 기본 네임스페이스의 'jk-jenkins' 서비스로 전달되도록 포트 전달을 설정한다.

3. 1단계에서 비밀번호를 얻은 후 웹 브라우저에서 Jenkins URL(http://127.0.0.1:8080)을 방문할 수 있다. 1단계에서 검색한 'admin' 사용자 이름과 비밀번호를 사용하여 로그인한다.

4. Jenkins에 로그인한 후, 필요에 따라 보안 영역과 권한 전략을 구성할 수 있다. 사용자 인증 및 액세스 제어를 설정하여 Jenkins 인스턴스를 보안하는 것이 포함된다.

5. 구성에는 Jenkins Configuration as Code (JCasC)를 사용하는 것이 좋다. 'values.yaml' 파일에서 'configScripts'를 지정하여 코드로 구성 파일을 정의할 수 있다. 