---
date: 2023-05-17 00:00:00
layout: post
title: Kubernetes? / TIL
subtitle: 'Kubernetes?'
description: Kubernetes?
image: https://res.cloudinary.com/duruq2coh/image/upload/v1684374884/gitio/Kubernetes_ahpltn.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1684374884/gitio/Kubernetes_ahpltn.png
category: Kubernetes
tags:
  - Kubernetes
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **Kubernetes** 
---

오픈 소스로 만들어진 **컨테이너 *오케스트레이션 도구**로, 컨테이너화된 애플리케이션을 자동으로 배포, 스케일링하는 등의 관리 기능을 제공한다. 쿠버네티스는 각기 다른 환경(온프레미스 서버, VM, 클라우드)에 대응이 가능하다.

아키텍처의 트렌드가 모놀리식에서 마이크로서비스로 바뀌고, 이로 인해 컨테이너 개수가 증가하고, 여기에 확장성을 고려해 스케일링까지 더할 경우 수십, 수백개의 컨테이너가 생길 수 있다. 컨테이너 오케스트레이션 도구는 **수많은 컨테이너를 관리하고자 할 때, 보다 더 잘 관리하기 위한 툴**이다. 

쿠버네티스를 이용해 온프레미스 상에서 (사설) **클라우드 인프라를 구성**하고, 저렴한 클라우드 서비스의 일부분을 도입하여 **하이브리드 형태로 구성**하기도 하며, 필요에 따라 쿠버네티스로 **구성한 인프라를 통째로 AWS 등에 * 마이그레이션** 한다. (이식성) 주요 퍼블릭 클라우드 공급자들은 관리형 쿠버네티스를 지원한다. 예) AWS EKS

쿠버네티스는 사용하지 말아야 할 경우가 존재한다.
* 여러 Tier(단계)로 나뉘지 않은 모놀리식 아키텍처에는 적합하지 않다. 모놀리식 아키텍처는 먼저 마이크로서비스 분해 전략을 세우는 것이 우선이다.
* 적은 양의 컨테이너를 다룰 때에 적합하지 않다. (docker-compose로 충분히 관리할 수 있다.)
* 비교적 단순한 아키텍처에 스케일링이 필요한 경우, 서버리스 서비스를 사용하면 다소 저렴한 가격에 관리형 서비스를 이용할 수 있으며, AWS 내에서도 오토 스케일링과 같은 관리형 서비스가 배포 환경에서 제공된다.

[**>> 쿠버네티스 공식 문서 <<**](https://kubernetes.io/ko/docs/concepts/overview/)

<br>

### **Kubernetes 작동 원리**
---
![](https://res.cloudinary.com/duruq2coh/image/upload/v1684377383/gitio/post/kubernetes/1_banu3s.png)

위 사진은 쿠버네티스 아키텍처의 모습이다.

* **Cluster** : 최소 하나 이상의 제어판(Control Plane) 컴포넌트와, 이것과 연결된 몇 개의 워커 노드로 구성되어 있다. 
* **Control Plane** : 관리를 위해 필요한 프로세스들이 있으며, 클러스터가 잘 작동할 수 있게 돕는다.
* **Worker Node** : 한 개 이상의 컨테이너가 자리잡고 있으며, 애플리케이션이 실행되고 있는 곳이다. 워커노드는 kubelet이라는 프로세스가 돌아가고 있고, 다른 노드와 서로 통신하거나 컨테이너를 실행하는 등의 태스크를 실행할 수 있게 한다.
* **Pod** : 컨테이너 그룹과 컨테이너가 사용하는 볼륨, 컨테이너의 작동 정보
![](https://res.cloudinary.com/duruq2coh/image/upload/v1684380388/gitio/post/kubernetes/2_bklblk.png)

<br>

### **제어판 컴포넌트 프로세스**  
---
![](https://res.cloudinary.com/duruq2coh/image/upload/v1684381078/gitio/post/kubernetes/3_ihhxd8.png)

**Kube-APIServer**  

모든 클러스터 관리의 입구로서, 명령을 내릴 수 있는 관문이다. 실제로 쿠버네티스에서 제공되는 UI나 CLI 등에서 클러스터 관리를 위해 뭔가 명령을 내리면 API가 호출된다. 직접 호출도 가능하다.

**Kube-Controller-Manager**  

클러스터에서 무슨 일이 발생하는지를 추적하는 역할을 한다. 컨테이너가 죽거나 재시작되었을 경우, 컨트롤러 매니저는 이를 알 수 있다.

**Kube-Scheduler**

서버(노드) 리소스를 바탕으로 컨테이너(pod)가 노드에 배치되게 만드는 역할을 담당한다. 새로 생성된 컨테이너를 찾아 노드에 할당한다.

**Key-Value 저장소 ETCD**

클러스터 관리에 필요한 모든 데이터를 저장하는 공간이다. 인프라를 원하는 상태로 만들기 위해서는, 정상 상태에 대한 snapshot 및 관리에 필요한 메타데이터가 어딘가에 저장되어야 하는데, ETCD가 이를 담당한다.

---
*마이그레이션 : 최소한의 중단과 중단 시간을 보장하면서 한 노드 또는 클러스터에서 다른 노드 또는 클러스터로 워크로드(애플리케이션)를 이동하는 프로세스를 의미한다. 여기에는 워크로드와 관련된 상태 및 리소스를 새 대상으로 전송하는 작업이 포함된다.
