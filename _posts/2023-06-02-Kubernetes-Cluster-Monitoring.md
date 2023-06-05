---
date: 2023-06-02 00:00:00
layout: post
title: Kubernetes Cluster Monitoring / TIL
subtitle: 'Kubernetes Cluster Monitoring'
description: Kubernetes Cluster Monitoring
image: https://res.cloudinary.com/duruq2coh/image/upload/v1685956060/gitio/Monitoring_sfnawq.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1685956060/gitio/Monitoring_sfnawq.png
category: Monitoring
tags:
  - Service Monitoring
  - Kubernetes
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **Kubenetes Cluster Monitoring**  
---

쿠버네티스의 경우 클러스터 안에 다수의 노드, 그리고 그 안에 파드를 비롯한 **다양한 워크로드**가 많게는 수백 개가 실행되는 형태로 구성되어 있다. 단일 노드의 경우 리눅스 명령어를 이용해 하드웨어의 상황을 파악하고, 각 프로세스 모니터링을 했다면, 쿠버네티스의 경우 각 노드는 전적으로 **컨트롤 플레인에 의해 관리**되므로 우리는 모니터링에 대해 **쿠버네티스 API**를 적극 이용해야 한다. 

클러스터 모니터링에서 노드가 사용하는 리소스는 ```kubectl top``` 이라는 명령어로 확인할 수 있다. 이 명령어는 노드와 파드가 각각 얼마만큼의 CPU/메모리 리소스를 사용하고 있는지 확인할 수 있다.

만약 백엔드 영역의 파드 하나에서 문제가 발생하면, 파드를 연결한 서비스 역시 제대로 작동하지 않을 것이다. 사용자는 UI에서 문제가 있다는 정도만 확인할 뿐, 실제로 어떤 곳에서 무슨 원인에 의해 발생한 지는 알기 어렵다. 

이 경우 각 파드에서 사용하는 리소스에 문제가 발생하면 **미리 경고**를 주거나, Liveness Probe를 통해 애플리케이션에서 발생하는 응답이 오류로 전달되는 경우 이를 **즉시 모니터링** 할 수 있으면 좋을 것이다. 

또한 노드가 물리적으로 멀리 떨어진 상황일 경우 네트워크에 대한 문제가 발생할 수 있다. 따라서 응답 대기 시간(Response Latency)에 대한 점검도 중요하다.

<br>

**클러스터 환경의 주요 이슈**
* **쿠버네티스 환경 그 자체** - 제어판의 주요 컴포넌트 상태가 비정상적인 경우
* **노드의 리소스 가용량 (CPU, 메모리 요청에 대한 비율)** - 노드의 가용량 리소스보다, 리소스 요청량이 커서 파드가 배포되지 않는 경우
* **노드의 리소스 사용량** - 노드 리소스가 부족하여 컨테이너에 크래시가 발생한 경우
* **워크로드 이슈** - 마운트한 파일 시스템의 용량이 부족하거나, 특정 컨테이너가 반복적으로 재시작하는 경우

<br>

> ## **Prometheus Monitoring**  
---

프로메테우스는 **오픈소스 모니터링/알림 시스템**으로, 노드, 프로메테우스 자체를 모니터링할 수 있다. 쿠버네티스를 지원,관리하는 CNCF에서 프로메테우스를 관리하고 있으며, 이 두 도구를 비롯해 시각화를 담당하는 Grafana와 함께 세 도구 조합은 정석적으로 널리 사용되고 있다.  

(Grafana는 프로메테우스의 구성요소는 아니지만, 프로메테우스에서 권장하는 시각화 도구이다.)

**프로메테우스 구성요소**
* 시계열(time series) 데이터를 저장한다.
* 서버는 다양한 exporter로부터 각 대상 메트릭을 pull하여 주기적으로 가져오는 모니터링 시스템이다.  
  **ex >>** 쿠버네티스 관련 메트릭을 가져오고 싶다면 쿠버네티스 exporter를, mongoDB 관련 메트릭을 가져오고 싶다면, mongoDB exporter를 사용하면 된다.
* Alter manager는 경고 및 알람을 담당한다.
* 사용자가 데이터를 질의할 수 있는 Web UI가 존재한다. 이 때 질의 언어로 PromQL(Prometheus Query Language)를 사용한다. 

<br>

**Kubernetes exporter**  

프로메테우스에서 쿠버네티스 관련 메트릭을 가져올 수 있는 쿠버네티스 exporter가 존재한다. 이 때 프로메테우스 exporter는 Kube API를 사용하여 대부분의 필요한 정보를 가져올 수 있다.

쿠버네티스에서 프로메테우스와 그라파나, 그리고 슬랙과 같은 메신저와의 연동은 보통 다음 그림과 같이 구성한다. 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1685928919/gitio/post/kubernetes/158340501-2cda5ff5-0dd0-4241-8ffe-353c94688c31_c6eowf.png)

쿠버네티스에서 프로메테우스와 그라파나를 연동하면, 클러스터 내의 애플리케이션 및 인프라의 상태를 모니터링하고, 이를 시각화하여 더 나은 인사이트를 얻을 수 있다. 

프로메테우스는 쿠버네티스 클러스터 내에서 실행중인 애플리케이션의 **지표를 수집하고 저장**하는 역할을 한다. 예를 들어, 애플리케이션의 CPU 사용량, 메모리 사용량, 요청 수 등과 같은 데이터를 수집한다.

수집한 데이터는 그라파나와 연동하여 *시각화*할 수 있다. 그라파나는 다양한 종류의 대시보드를 생성하고, **프로메테우스가 수집한 데이터를 그래프, 도표, 경고 등으로 시각화**하는데 사용되며, 이를 통해 운영자는 애플리케이션의 상태와 성능에 대한 통찰력을 얻을 수 있다. 또한 필요한 경우 **알림을 설정하여 잠재적인 문제에 대해 빠르게 대응**할 수 있다.

슬랙과 같은 메신저와 쿠버네티스를 연동하여 애플리케이션 또는 **인프라 관련 이벤트에 대한 알림**을 받을 수 있다. 예를 들어, 클러스터 내에서 오류가 발생하거나 장애 상황이 발생하면, 슬랙으로 알림을 전송하여 관련 팀 또는 운영자가 신속하게 대응할 수 있도록 도와준다. 이를 위해 쿠버네티스 이벤트를 모니터링하고, 이벤트 발생 시 슬랙 봇을 통해 알림을 보내는 등의 작업을 수행할 수 있다.

쿠버네티스, 프로메테우스, 그라파나, 슬랙과 같은 메신저를 연동함으로써, 쿠버네티스 환경에서 **애플리케이션의 상태를 모니터링**하고, **시각적인 형태로 표현**하며, **팀과의 협업을 강화**할 수 있다. 이를 통해 애플리케이션의 **신뢰성과 안정성을 향상**시키고, **장애 대응 시간을 단축**할 수 있다.