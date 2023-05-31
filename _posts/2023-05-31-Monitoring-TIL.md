---
date: 2023-05-31 00:00:00
layout: post
title: Service Monitoring / TIL
subtitle: 'Service Monitoring'
description: Service Monitoring
image: https://res.cloudinary.com/duruq2coh/image/upload/v1681175914/gitio/aws_bbbsnj.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1681175914/gitio/aws_bbbsnj.png
category: AWS
tags:
  - Service Monitoring
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **Service Monitoring**  
---

CI/CD의 마지막 파이프라인인 Stage는 운영은 서비스에 생길 수 있는 현황을 파악하고 문제를 모니터링하는 과정으로 대표된다. 그렇다면 모니터링은 어떤 지표를 수집하고, 어떤 *메트릭을 기준으로 삼아야 할까?

**모니터링의 목표**
- 시간을 기준으로 측정되는 주요 메트릭을 최소화하여 고가용성을 달성
- 사용량을 추적하여, 배포에 앞서 세운 가설을 검증하고 개선 (애자일에서는 '[검증된 학습(Validated learning)](https://www.boldare.com/blog/lean-startup-validated-learning/)을 적용한다'라고 한다.)

**구글의 모니터링의 목표**
- 장기적인 트렌드 분석  
  - DB가 얼마만큼 용량을 차지하며, 얼마나 빨리 용량이 증가하는가?
  - DAU(일간 활성 사용자수)는 얼마나 빨리 증가하는가?  
- 시간의 경과 및 실험 그룹 간의 비교
  - 어떤 DB를 썼을 때 쿼리가 빠른가?
  - 캐시용 노드를 추가했을 때, 캐시 적중률이 얼마나 향상되는가?
  - 지난주보다 사이트가 얼마나 느려졌는가?
- 경고
  - 인프라의 어떤 부분이 고장 났는가? 혹은 고장날 수 있는가?

<br>

**모니터링의 종류**

-  | **블랙 박스 모니터링** | **화이트 박스 모니터링**
:--: | :--: | :--:
**관찰자 시점** |밖에서 바라봄 | 안에서 바라봄
**특징** |  인프라 수준, 쿠버네티스 컴포넌트 자체를 모니터링 | 측정 기준에 따른 모니터링
'-' | 애플리케이션이 왜 오류를 내는 지 알 수 없음 | 현상이 발생한 근거를 알 수 있는 모니터링 방식
**예** | CPU/메모리/스토리 등 | HTTP 요청, 500 에러의 발생 횟수, 레이턴시 등

어떠한 서비스가 제대로 작동되는 지 확인하려면, 서비스 또는 시스템과 관련한 모든 변수들을 모니터링해야 한다. 모니터링을 할 때에는 단계를 구분해서 계층적으로 할 필요가 있다.

논리적인 리소스의 집합이 하나의 상위 계층을 만든다. 컨테이너 오케스트레이션 툴이나, AWS의 서비스가 제공하는 계층을 이해하면, 어떤 것을 모니터링해야 하는지 보다 쉽게 파악이 가능하다. 파드나, 컨테이너 안에 포함된 애플리케이션의 메트릭은 별도로 다룬다.

**계층에 따른 모니터링 구분**

* 쿠버네티스 : 노드 > 클러스터 컴포넌트 > 파드
* ECS : 클러스터 > 서비스 > 태스크
* EC2 : 인스턴스에 대한 메트릭만 볼 수 있다.
* Lambda : 함수에 대한 메트릭만 볼 수 있다.

<br>

### **메트릭**
---
**시간에 따라 측정한 결과값**으로, **비즈니스 개념을 나타내는 수치 측정**을 의미하기도 한다. 

예를 들어, 시간당 CPU 사용률, 연간 순매출과 같이 시간이라는 차원이 함께 적용되어야 한다. 시간이 아닌 다른 차원(서비스 별 매출)을 기준으로 삼을 수도 있다.

* 주요 메트릭은, 단일 노드일 경우 리눅스를 통해 측정할 수 있다.
* 클러스터 형태, 즉 여러 대의 노드로 구성되어 있는 경우, AWS 콘솔(CloudWatch 등)을 통해 이미 제공되고 있는 경우가 많다.

**MS(Azure 서비스)에서 보는 메트릭**
- 캐시 사용률
- CPU, Memory
- 인스턴스의 개수
- 연결 유지

**Proxy 서버의 메트릭**  

애플리케이션 서버(AWS)의 앞단에 캐시 서버 혹은 인증 서버, 로드 밸런서와 같은 **Proxy 서버**가 존재한다면, 이는 애플리케이션 서버와는 **별도로 모니터링**해야 한다. 애플리케이션 서버가 각 노드의 컴퓨팅 자원을 모니터링 하는 데에 중점을 두었다면, Proxy 서버, 그 중에서도 HTTP 라우팅을 다루고 있는 서버는 **요청 그 자체와 연관된 메트릭을 위주로 모니터링**해야 한다.

**HTTP 요청/응답 관련 모니터링** 대상은 쿠버네티스의 경우 **인그레스**, AWS 생태계에서는 **Application Load Balancer**를 중점으로 보아야 한다.

<br>

### **메트릭 한눈에 보기**

-- | 컴퓨팅 유닛 관련 메트릭	| 요청/응답 관련 메트릭	|스케일링 관련 메트릭
:--: | :--: | :--: | :--:
k8s	| CPU 사용량 (utilization), 메모리 사용량, 네트워크 in/out, 디스크 사용량 (노드 및 파드 별) | etcd latency, ingress, 요청 개수, 요청 latency, 에러율	 | 디플로이먼트 상황
ECS	| CPU 사용량, 메모리 사용량, 네트워크 in/out (클러스터 및 서비스 별) | 해당 사항 없음 (ALB와 사용하여 분석해야 함) |	서비스 개수, (원하는/실행 중인/보류 중인) 작업 개수, 컨테이너 인스턴스 개수 
EC2	| CPU 사용량, 네트워크 in/out, 네트워크 패킷 in/out, 디스크 읽기/쓰기 (바이트), 작업 개수, CPU 크레딧 사용량, 밸런스	| 상태 검사 실패 횟수 | 해당 사항 없음
Lambda	| 해당 사항 없음 | 호출 개수, 실행 시간, 에러 개수 및 성공률, throttles, async delivery failures, IteratorAge | 동시 실행 횟수
ALB	| 해당 사항 없음 | 대상 응답 시간, 요청 개수, 응답 코드 개수 (2xx, 4xx, 5xx), 대상 연결 오류, 거부된 연결 개수 합계, 대상 TLS 협상 오류, 클라이언트 TLS 협상 오류, 활성 연결 개수, 새 연결 개수, 처리된 바이트, 사용된 Load Balancer 용량 단위 | 해당 사항 없음

---
**Application Load Balancer**

![](https://res.cloudinary.com/duruq2coh/image/upload/v1685499850/gitio/post/aws/2_ki15x1.png)

**ECS**

- **클러스터**

![](https://res.cloudinary.com/duruq2coh/image/upload/v1685499849/gitio/post/aws/3_lgo9j4.png)

- **서비스**

![](https://res.cloudinary.com/duruq2coh/image/upload/v1685499849/gitio/post/aws/4_ciqst4.png)


- **태스크**

![](https://res.cloudinary.com/duruq2coh/image/upload/v1685499984/gitio/post/aws/7_bnf2as.png)


**EC2** 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1685499849/gitio/post/aws/5_awnqut.png)

**Lambda**

![](https://res.cloudinary.com/duruq2coh/image/upload/v1685499849/gitio/post/aws/6_nkgbnu.png)

<BR>

### **사이트 신뢰성 엔지니어링 (SRE) 메트릭**  
---
CPU 및 메모리, 사용량 등을 파악하는 것 외에도 네트워크 요청에 따른 응답 상태, 요청의 횟수나 시간 등도 중요한 지표가 될 수 있는데, 이를 통해 어떤 서비스가 온전히 사용자에게 전달될 수 있도록 가용성을 극대화하는 기술/문화를 특별히 "사이트 신뢰성 엔지니어링(SRE)"라고 부른다.