---
date: 2023-05-16 00:00:00
layout: post
title: Kubernetes? &#35;2 / TIL
subtitle: 'Pod / Deployment'
description: Pod / Deployment
image: https://res.cloudinary.com/duruq2coh/image/upload/v1684374884/gitio/Kubernetes_ahpltn.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1684374884/gitio/Kubernetes_ahpltn.png
category: Kubernetes
tags:
  - Kubernetes
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **Kubernetes Workload** 
---

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684455306/gitio/post/kubernetes/module_03_pods_tzlhq8.svg)

### **Pod**
---
파드는 그 자체로 하나의 논리적인 호스트이며, 쿠버네티스의 배포 가능한 가장 작은 컴퓨팅 유닛이다. 파드는 다음 요소들을 포함할 수 있고, 마치 도커 컨테이너처럼 파드 내에서 다음 요소들은 격리된다.

* 하나 이상의 애플리케이션 컨테이너
* IP 주소
* 볼륨과 같은 공유 스토리지

쿠버네티스에서는 "*워크로드 리소스"를 만들기 위해 보통 YAML 파일과 같은 리소스 정의 파일을 사용한다. 다음과 같은 형식으로 작성될 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

이 yaml 파일은 "nginx" 라는 쿠버네티스 포드를 설명하고 있으며, 포드는 Nginx 이미지 버전 1.14.2를 사용하여 단일 컨테이너를 실행하도록 구성된다. 컨테이너는 포드 내부에서 실행되는 Nginx 웹 서버에 대한 외부 액세스를 허용하는 포트 80을 노출한다.

위에서 만든 파드 정의 파일을 생성하고 배포하기 위해서는, 다음 명령을 사용한다.

```bash
$ kubectl apply -f sample-pod.yaml
```

apply 명령을 통해 ```pod/nginx created``` 라는 결과가 출력되었으면 다음 명령을 통해 상태를 확인해 볼 수 있다.

위 명령과 비슷하지만 다른 명령이 존재한다.

```bash
$ kubectl create -f sample-pod.yaml
```

create는 YAML 또는 JSON 파일에 지정된 구성을 기반으로 새 리소스를 만든다. 리소스가 이미 있으면 오류가 발생하고, 일반적으로 처음으로 새 리소스를 생성하는 데에 사용된다.

apply는 기존 리소스에 변경 사항을 적용하거나 리소스가 없는 경우 새 리소스를 만든다. 리소스가 이미 있는 경우 YAML 또는 JSON 파일에 지정된 변경 사항으로 업데이트하고, 없는 경우에는 생성한다. 일반적으로 새 리소스를 생성하고 기존 리소스를 업데이트 하는 데 사용된다.

```bash
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS        AGE
nginx                             1/1     Running   0               33s
```

또한 다음 명령을 이용해 파드에 대한 자세한 정보를 볼 수 있다.

```bash
$ kubectl describe pod/nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Fri, 19 May 2023 09:52:49 +0900
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               172.17.0.4

...

Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  100s  default-scheduler  Successfully assigned default/nginx to minikube
  Normal  Pulling    99s   kubelet            Pulling image "nginx:1.14.2"
  Normal  Pulled     92s   kubelet            Successfully pulled image "nginx:1.14.2" in 6.893196687s
  Normal  Created    90s   kubelet            Created container nginx
  Normal  Started    90s   kubelet            Started container nginx
```

이 외의 파드 관련 명령어이다.

```bash
$ kubectl describe pod <pod_name>
# 특정 포드에 대한 자세한 정보를 가져온다.

$ kubectl delete pod <pod_name>
# 특정 포드를 삭제하고, 쿠버네티스가 종료되며 클러스터에서 제거된다.

$ kubectl log <pod_name>
# 특정 포드에서 생성된 로그를 볼 수 있다.

$ kubectl exec <pod_name> -- <command>
# 실행중인 포드 내부에서 명령을 실행한다. 

$ kubectl port-forward <pod_name> <local_port>:<pod_port>
# 로컬 컴퓨터와 지정된 포드 간의 연결을 설정한다. 지정한 로컬 포트로 향하는 모든 트래픽은 해당 포드 포트로 전달되어 포드 내부에서 실행중인 서비스와 상호 작용 할 수 있다. 
```
<br>

### **Deployment**
---

평소 우리는 Deploy를 배포라고 번역했지만, 쿠버네티스에서는 서비스 노출의 의미가 아닌 파드의 교체/배치(placement)와 관련된 명세이다. 

Deployment는 **파드를 업데이트 하기 위한 선언적 명세**이다.
* 레플리카셋, 즉 복제본 구성을 이용해 파드를 원하는 개수만큼 실행시킬 수 있다.
* 제어판 Control Plane을 이용해 파드를 업데이트 할 수 있다.
* 파드를 롤백하는 것도 가능하다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684458570/gitio/post/kubernetes/2_pmcsdj.png)

파드는 일시적이고, 언제나 삭제될 수 있음을 감안하고 만들기 때문에 사용자가 개별 파드를 만들 일은 많지 않다. 파드가 실행되는 공간인 노드가 만일 실패하는 경우, 그 안에서 실행되는 파드 역시 사용할 수 없게 된다.

쿠버네티스의 핵심은 **컨테이너를 오케스트레이션** 하는 것으로, 파드 장애 시 자동 복구하거나, 복제하거나 하는 등의 일을 자동으로 처리하는 데에 있다. AWS로 따지면 ECS가 하는 일과 비슷하다. 

결론적으로, 파드는 Deployment, Stateful Set, Demonset을 이용해 관리하는 것이 바람직하다. 이 워크로드 리소스는 파드 템플릿을 항상 포함하고 있다.

<br>

**다양한 배포 전략**

* **재생성 (Recreate)** : 이전 버전을 삭제하고 새 버전 생성
* **블루/그린 배포** : 한 번에 이전 버전에서 새 버전으로 연결을 전환
* **롤링 배포** : 이전 버전을 scale down하고, 새 버전을 scale up 하는 방식으로 단계별로 교체, 롤 아웃(rollout)이라고 부른다.
* **카나리 배포** : 새 버전이 잘 작동한다고 판단되면, 이전 버전을 교체

Deployment는 파드의 복제본을 자동으로 업데이트하게 해주는 명세이므로, 쿠버네티스가 지원하는 배포 전략으로는 재생성(Recreate)과 롤링 배포(RollingUpdate) 방식을 선택할 수 있다.

**Deployment 예시**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  # 파드 템플릿 ~
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

* Nginx 웹 서버를 실행하기 위한 "nginx-deployment"라는 이름의 Deployment이다. 
* 'apiVersion' : 사용중인 쿠버네티스 API 버전을 지정한다.
* 'kind' : 생성되는 쿠버네티스 리소스의 유형을 정의한다.
* 'metadata' : 이름 및 레이블을 포함해 deployment에 대한 메타데이터 정보를 포함한다.
* 'label' : 레이블을 deployment에 할당한다. 이 경우 값이 "nginx"인 레이블 "app"이다.
* 'replicas' : nginx deployment의 복제본(인스턴스)이 3개 있어야 함을 지정한다.
* 'selector' : 이 deployment에 속하는 파드를 식별하는 데 사용되는 레이블 선택
* 'matchLabels' : 파드 일치에 사용되는 레이블을 지정한다. 이 경우 'app' 레이블이 'nginx'로 설정된 파드이다.


---
*워크로드 : 쿠버네티스에서는 “쿠버네티스 상에서 작동되는 애플리케이션”을 의미하며, 클라우드 분야에서는 “어떤 애플리케이션을 실행할 때 필요한 IT 리소스의 집합”이라는 의미로 통용된다.