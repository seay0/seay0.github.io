---
date: 2023-06-05 00:00:00
layout: post
title: Service Level / TIL
subtitle: 'Service Level'
description: Service Level
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

> ## **Service Level 관련 용어**  
---

서비스를 운영하는 데 있어서, 사용자에게 필요한 적정 수준을 정의하고 제공하기 위해, 서비스 제공자와 사용자는 서로 서비스 수준 협약(Service Level Arguments, SLA)을 맺는다. SRE 엔지니어는 척도를 통한 목표를 이해하고, 실제로 목표를 세우는 방법을 알아야 한다. 

**서비스 수준 척도 (SLI, Service Level Indicator)**  

서비스 수준을 판단할 수 있는 몇 가지를 정량적으로 측정한 값이다.
* 응답 속도 : 요청에 대한 응답이 리턴되기까지의 시간
* 에러율 : 전체 요청 수 대비
* 처리량 (throughput) : 초당 처리할 수 있는 요청 수
* 가용성 : 서비스가 사용 가능한 상태로 존재하는 시간의 비율
* 내구성 : 데이터 저장이 중요한 목적인 서비스의 경우 특히 중요

**서비스 수준 목표 (SLO, Service Level Objectives)**  

SLI에 의해 측정된 서비스 수준의 목푯 값 또는 일정 범위의 값을 의미한다.
```
SLI ≤ 목표치
최소값 ≤ SLI ≤ 최댓값
```

**서비스 수준 협약 (SLA, Service Level Agreementes)**  

SLO를 달성하지 못하면 어떻게 되는지를 적어놓은 약속이다. SRE가 직접 관여하지는 않는다.

<br>

### **지표 / 주요 목표 설정**
---
서비스나 시스템에 있어 중요한 지표를 판단하는 근거가 필요하다. 적절한 SLI의 선정은 시스템의 분류에 따라 달라질 수 있다.

* **사용자가 직접 대면하는 시스템** : 보통 프론트엔드에 해당하며 가용성, 응답 시간, 처리량이 중요하다.
* **저장소 시스템** : 응답 시간, 가용성, 내구성이 중요하다.
* **빅데이터 시스템** : 데이터 파이프라인이 이에 해당하며, 처리량, 엔드포인트 간 응답 시간이 중요하다.

<br>

**척도 수집**은 측정 원본을 합산하거나, 평균을 내는 방법이 있겠지만, 대부분의 경우 분포가 중요하다. 일부 요청이 빠르게 처리되어도, 나머지 요청이 균일하게 느리다면 실제로 서비스는 느린 것으로 간주할 수 있다. 평균은 이러한 흐름의 변화를 감지하기 어렵다.

SLO를 설정할 때, 주요 SLI의 정의를 표준화시키면 편리하다.
* 집계 간격 : 1분
* 집계 범위 : 하나의 클러스터에서 수행되는 모든 태스크
* 측정 빈도 : 매 10분
* 집계에 포함할 요청 : 전체 HTTP GET 요청

또한 다음과 같은 성능에 중점을 둔 SLO 목표를 설정할 수 있다.
* GET 호출의 90%는 1ms 이내에 수행되어야 한다.
* GET 호출의 99%는 10ms 이내에 수행되어야 한다.
* GET 호출의 99.9%는 100ms 이내에 수행되어야 한다.

<br>

### **Q1.** 
모니터링 시스템에는 메트릭 수집을 위한 두 가지 방식의 메커니즘이 존재한다. 바로 Pull 방식과 Push 방식이다. 프로메테우스는 어떤 방식의 메커니즘을 사용할까?

**A.**  
프로메테우스는 미리 설정된 간격으로 타켓 시스템에 HTTP 요청을 보내어 메트릭을 가져온다. 이런 방식으로 메트릭을 주기적으로 **Pull**하여 수집한다. 또한 이를 통해 확장성과 유연성 있는 메트릭 수집 시스템을 구축할 수 있다.

---
### **Q2.**  
Pull 방식과 Push 방식은 어떻게 다르며, 장단점은 무엇인지, 또한 해당 방식을 사용하는 모니터링 도구는 어떤 것들이 있을까?

**A.**  

**Pull 방식 (대표적 도구 : 프로메테우스)**  

메트릭을 수집하기 위해 수집 서버가 주기적으로 타겟 시스템에 요청을 보내어 메트릭을 가져온다. 수집 서버는 데이터를 원하는 빈도로 수집할 수 있으며, 타겟 시스템은 단순히 메트릭을 노출하는 역할을 한다. 

- 장점 : 메트릭 수집 주체와 수집 대상 시스템 간의 의존성이 낮으며, 수집 주기와 타이밍을 유연하게 조절할 수 있다. 또한 프로메테우스와 같은 도구를 통해 확장서과 유연성 있는 메트릭 수집 시스템을 구축할 수 있다.
- 단점 : 수집 서버에서 주기적으로 메트릭을 가져오기 때문에 지연이 발생할 수 있고, 메트릭을 노출하는 엔드포인트를 수집 대상 시스템에서 구현해야 한다.

**Push 방식 (대표적 도구 : 텔레그라프)**  

메트릭을 수집 대상 시스템에서 주도적으로 밀어넣는 방식으로, 수집 대상 시스템은 메트릭을 수집하여 메트릭 수집 서버로 전송(Push)한다. 수집 개상 시스템에서 메트릭 수집 및 전송 로직을 구현해야 하며, 수집 주기를 조절하기 어렵다. 

- 장점 : 실시간 메트릭 데이터를 전송할 수 있고, 수집 대상 시스템은 자체적으로 메트릭 수집 및 전송 로직을 구현할 수 있다.
- 단점 : 수집 대상 시스템에서 메트릭 수집 및 전송 로직을 구현해야 하고, 수집 주기를 동적으로 조절하기 어렵다.

---
### **Q3.**  
어떤 조직의 SLO가 다음과 같다. "GET 호출의 99%는 10ms 이내에 수행되어야 한다" 그렇다면, 이러한 SLO를 달성하려면 어떤 메트릭을 수집하고 어떻게 계산해야 할까? (척도는 표준화된 범용 지표를 사용한다.)

**A.**  

**1. 응답 시간 메트릭**
* GET 호출의 응답 시간을 측정하는 메트릭을 수집한다.
* 일반적으로 응답 시간은 밀리초 단위로 측정되며, 서비스 지연 시간을 확인할 수 있다.

**2. 분위수 계산**
* 99%의 응답 시간 분위수를 계산한다.
* 이를 통해 전체 응답 시간 데이터에서 상위 1%의 값, 즉 가장 높은 응답 시간을 확인할 수 있다.

**3. SLO와 비교**
* 계산된 99% 분위수 값을 SLO의 10ms와 비교한다.
* 만약 99% 분위수 값이 10ms 이내에 있다면, SLO를 달성한 것으로 간주할 수 있다.

표준화된 범용 지표를 사용하여 위의 메트릭을 측정할 수 있다. 일반적으로 프로메테우스와 같은 모니터링 도구를 사용하여 응답 시간 메트릭을 수집하고, 분위수 계산과 SLO 비교를 위해 쿼리와 알림을 설정할 수 있다. 프로메테우스는 수집된 데이터를 쿼리할 수 있는 PromQL(Query Language for Prometheus)을 제공하며, 알림 규칙을 설정하여 SLO를 모니터링하고 경고를 생성할 수 있다.