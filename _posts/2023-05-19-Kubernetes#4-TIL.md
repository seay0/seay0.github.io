---
date: 2023-05-19 00:00:00
layout: post
title: Kubernetes? &#35;4 / TIL
subtitle: 'Volume / Stateful Set'
description: Volume / Stateful Set
image: https://res.cloudinary.com/duruq2coh/image/upload/v1684374884/gitio/Kubernetes_ahpltn.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1684374884/gitio/Kubernetes_ahpltn.png
category: Kubernetes
tags:
  - Kubernetes
  - Volume
  - Stateful Set
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **Kubernetes Stateful** 
---

파드는 일시적이며 언제나 삭제될 수 있다. 즉, 파드 그 자체는 Stateless하다. 이러한 파드의 교체와 배치를 담당하는 것이 Deployment이다. Deployment는 레플리카셋을 통해 파드를 scale out하며, 이 때 만들어지는 파드들은 상호 대체 가능하다. 

파드에 상태를 남겨야 하는 Stateful 애플리케이션으로 MySQL, mongoDB, redis와 같은 데이터베이스가 있다. 데이터베이스 애플리케이션이 담긴 파드가 사라질 때, 데이터가 함께 사라지도록 두면 안된다. 그렇기 때문에 쿠버네티스에도 영속적인 데이터를 저장하기 위해 Volume을 연결할 수 있다. 파드에 Volume을 연결하여 데이터를 영속적으로 저장할 수 있다.

하지만 파드 명세에 [PV(Persistence Volume)](https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes)를 정의해서 직접 연결하는 것은 스토리지와 애플리케이션 간에 긴밀한 결합을 도입하기 때문에 좋은 방법이 아니다. 

이 결합으로 인해 다음과 같은 제한과 단점이 발생할 수 있다.

* **이식성 부족** : 특정 스토리지 인프라에 묶이게 되어 이식성이 제한되고 응용 프로그램 및 관련 데이터를 다른 클러스터 또는 스토리지 솔루션으로 이동하기 어렵다.
* **제한된 유연성** : 애플리케이션과 독립적으로 스토리지 리소스를 확장하고 관리하는 기능이 제한될 수 있다. 예를 들어 PV 크기를 조정하거나 마이그레이션하려는 경우 연결된 파드를 중지하거나 다시 시작해야 애플리케이션이 중단될 수 있다.
* **복잡성 및 유지 관리** : 추가적인 복잡성과 유지 관리 오버헤드가 발생할 수 있다. (애플리케이션 수준의 프로비저닝, 크기 조정 및 백업과 같은 스토리지 관련 작업 처리)
* **보안 및 격리** : 스토리지 별 세부 정보와 자격 증명을 애플리케이션 계층에 노출할 수 있다. 애플리케이션이 낮은 수준의 스토리지 작업 및 설정에 액세스할 수 있으므로 잠재적으로 보안 및 격리가 손상될 수 있다.

이러한 의존도를 줄이기 위해 PVC(Persistence Volume Claim)을 이용해 PV와 연결한다. PVC는 파드가 볼륨의 세부 사항을 몰라도 볼륨을 사용할 수 있게 도와준다.
* PV는 실제로 데이터가 저장되는 공간이다.
* PVC는 기본 스토리지 인프라에서 애플리케이션을 분리하는 상위 수준의 추상화 계층 역할을 한다.
* PVC는 PV를 선택, 연결해 주는 요청 그 자체이다.

<br>

### **PVC 작동 방식**  
---

```PVC 생성 → PV에 바인딩 → PV 탑재 → 라이프사이클 관리```

1. Kubernetes에서 스토리지를 사용하기 위해 파드는 용량, 액세스 모드 및 스토리지 클래스와 같은 원하는 **스토리지 요구 사항**을 지정하여 **PVC 객체를 생성**한다. 
2. PVC가 생성되면 Kubernetes 컨트롤 플레인은 **PVC의 요구사항과 일치하는 적합한 PV**를 찾는다. 일치하는 PV가 사용 가능하고, 아직 다른 PVC에 바인딩 되지 않는 경우 PVC에 **동적으로 바인딩**된다.
3. PVC가 PV에 바인딩된 후 포드는 사양에서 **PVC를 참조**할 수 있다. 파드가 노드에 예약되면 PVC와 연결된 PV가 지정된 마운트 경로에서 **파드의 파일 시스템에 마운트**된다.
4. PVC는 **라이프사이클 관리 기능**을 제공한다. PVC 크기를 조정하여 더 많거나 적은 스토리지를 요청할 수 있으며 Kubernetes는 그에 따라 연결된 **PV를 동적으로 조정**한다. 더 이상 필요하지 않은 경우 PVC를 삭제할 수도 있으며 구성된 회수 정책에 따라 연결된 PV를 회수하거나 삭제할 수 있다.

PVC를 사용함으로써 애플리케이션은 **특정 스토리지 세부 정보에 구애받지 않고 이동**할 수 있고, 기본 스토리지 인프라와 독립적으로 만들 수 있다. PVC는 Kubernetes 클러스터 내에서 **스토리지 리소스를 관리하고 소비**하는 유연하고 확장 가능한 방법을 제공한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684732117/gitio/post/kubernetes/hlEhZmhAzKDKCuc5fQC4r-1650398862538_yuvpwa.png)

파드를 정의할 때 PVC를 통해 볼륨에 연결할 수 있다. 이러한 파드를 Deployment로 배포해도, PVC를 통해 연결되는 볼륨은 하나이기 때문에 결국 같은 스토리지를 사용하는 모양새가 된다.

파드마다 별도의 데이터를 저장할 수 있게 만들려면, **수평 확장되는 파드**에 서로 다른 PV가 연결되어야 한다. 파드가 PV를 동적으로 요청해서 그때그때 볼륨이 생성되게 하면 [스토리지 클래스 리소스](https://kubernetes.io/ko/docs/concepts/storage/dynamic-provisioning/)는 사용자가 요청할 때, 자동으로 PV를 프로비저닝 할 수 있게 돕는다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684732986/gitio/post/kubernetes/9mrE0wievDmSR9yeTbnF1-1650398888796_dihi4j.png)

<br>

### **StatefulSet**
---

애플리케이션 구성(파드)을 복제해도, 스토리지 클래스를 이용해 파드가 필요로 하는 볼륨을 자동으로 프로비저닝하여 연결한다. 따라서 첫 번째 파드를 primary(master), 두 번째 파드를 secondary 복제본처럼 구성하는 것도 가능하다.

Deployment와 StatefulSet 둘 다 동일한 컨테이너 스펙을 기반으로 둔 파드들을 관리한다는 점에서 같지만, StatefulSet은 **파드의 순서**와 **고유성**을 보장한다. StatefulSet이 생성한 파드는 **상호 대체할 수 없는 파드**이다.

**StatefulSet을 사용할 때 주의 사항**  
* 파드에 지정된 스토리지는 관리자에 의해 PVP(Persistent Volume Provisioning)를 기반으로하는 스토리지 클래스를 요청해서 프로비전하거나 사전에 프로비전에 되어야 한다.
* *Headless Service가 필요하다.

<br>

**Q .** 네트워크 트래픽에 따라 요청이 랜덤하게 분산되는 일반적 서비스에 비해, Headless Service는 특정 StatefulSet에 트래픽을 보낸다. 그렇게 해야하는 이유는 무엇일까?

* **A .** Headless Service는 StatefulSet의 **개별 파드에 직접 액세스하고 통신할 수 있는 방법**을 제공하여 안정적인 네트워크 ID를 유지하고 트래픽 분산에 대한 더 많은 제어를 제공한다. 이는 StatefulSet 내에서 **정렬된 통신, 결정론적 동작 또는 사용자 정의 로드 밸런싱이 필요한 애플리케이션**에 특히 유용하다.

---
*Headless Service : 로드 밸런싱을 수행하거나 클러스터 IP를 제공하지 않고 파드 집합을 노출하는 데 사용되는 특수한 유형의 서비스이다. 클러스터 IP가 할당되고 로드 밸런싱을 사용해 개별 파드와 직접 통신할 수 있다.