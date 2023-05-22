---
date: 2023-05-22 00:00:00
layout: post
title: Kubernetes? &#35;5 / TIL
subtitle: 'Network / Ingress'
description: Network / Ingress
image: https://res.cloudinary.com/duruq2coh/image/upload/v1684374884/gitio/Kubernetes_ahpltn.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1684374884/gitio/Kubernetes_ahpltn.png
category: Kubernetes
tags:
  - Kubernetes
  - Ingress
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **Kubernetes Network** 
---

**Ingress**  

Ingress는 클러스터 내의 서비스에 대한 외부 접근을 관리하는 API 게이트웨이이다. 일반적으로 HTTP를 관리하며 로드밸런서, SSL Termination, 가상 호스팅을 제공한다.

외부 IP 주소를 할당해 주는 서비스와 로드 밸런서를 생성하고 컨테이너로 트래픽을 보내는 방법을 이용하여 파드를 노출시킬 수 있다. 그렇다면 Ingress를 별도로 사용해야 하는 이유는 무엇일까?

Ingress 리소스는 **로드 밸런싱**과 더불어 **호스트 기반 라우팅**을 지원한다. (밑에 실습에서 만든 서비스는 로드 밸런서와 클러스터 IP로 바꿨기 때문에 Ingress가 로드 밸런서의 역할을 수행해야 한다.)

아주 단순한 애플리케이션도 서비스는 두 개 이상의 HTTP 요청을 가지는 것이 보통인데, 각각 Web Server와 WAS로 대표된다. 이러한 서비스의 접근을 별도의 포트로 구분하여 접속하게 할 수도 있지만, **하나의 호스트 상에서 라우팅으로 구분**하면 보다 유연한 서비스를 만들 수 있다.  
```
ex > 
Web Server는 '/'로, WAS는 '/api'로 라우팅할 수 있다. 

YAML 파일에서 spec.rules.host에 **별도의 호스트**를 지정하여 Web Server는 www.domain.click, WAS는 api.domain.click으로 설정하는 것도 가능하다.
```

실습으로 Ingress를 만들고 적용해보자.

1. 기존 서비스 타입을 LoadBalancer로 ClusterIP로 바꾸고 적용한다. ClusterIP는 클러스터 내에서만 접근이 가능하며 더 이상 EXTERNAL-IP는 사용할 수 없다.
2. Ingress 리소스를 다음과 같이 만들고 적용한다. ```kubectl apply -f ingress.yaml```

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  namespace: default
spec:
  rules:
  - host: localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

3. 생성된 Ingress는 ```kubectl get ingress```를 통해 조회할 수 있다.

```bash
> kubectl get ingress
NAME                              CLASS    HOSTS       ADDRESS   PORTS   AGE
ingress.networking.k8s.io/nginx   <none>   localhost             80      2m24s
# 여기에 Ingress가 생성되었다.
```

4. Ingress는 Ingress 리소스(정책 그 자체) 외에도 *Ingress 컨트롤러(정책을 실행시키는 도구)가 반드시 필요하다. minikube를 사용하는 경우, Ingress 컨트롤러는 애드온으로 별도의 설치가 필요하다. 

```bash
> minikube addons enable ingress
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.

...

🌟  The 'ingress' addon is enabled

> kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-fjkh8        0/1     Completed   0          61s
ingress-nginx-admission-patch-g7jzr         0/1     Completed   0          61s
ingress-nginx-controller-6cc5ccb977-7hjfj   1/1     Running     0          61s
# nginx-controller가 작동중인 것을 확인할 수 있다.
```

5. 설치가 완료되면, ```minikube tunnel```로 터널을 열고 http://localhost 에 접속해보면 다음과 같이 nginx 서버가 열린 것을 확인할 수 있다.


**Q .**  
애플리케이션에 HTTP 500과 같은 에러가 발생한 경우, 컨테이너를 다시 실행해야 한다. HTTP 에러가 발생했다는 것을 어떻게 알 수 있고, 어떻게 해야 쿠버네티스가 에러가 발생한 컨테이너를 자동으로 재시작하게 만들 수 있을까?

**A .**  

**HTTP 에러 발생 시기 식별**
* 컨테이너에서 실행중인 애플리케이션에서 HTTP 오류가 발생하는 시기를 알기 위해 Kubernetes 클러스터에 대한 모니터링 및 로깅을 구성할 수 있다. **모니터링 도구**(Prometheus, Grafana 등)를 이용해 HTTP 응답 코드를 비롯한 지표를 수집하여 애플리케이션의 상태와 성능을 추적할 수 있다.

* **로깅 솔루션**(Elasticsearch, Kibana, Grafana 등)을 설정하여 애플리케이션에서 생성된 로그를 캡처하고 분석할 수 있다. 로그를 모니터링하면 HTTP 오류를 검색하고 발생 시기를 식별할 수 있다.

<br>

**Liveness probes**  

Liveness probes는 주기적으로 컨테이너의 상태를 확인하고 프로브가 실패하면 자동으로 다시 시작한다. 애플리케이션의 특정 엔드포인트에 HTTP 요청을 하고 예상되는 응답 코드 범위를 지정하도록 Liveness probes를 구성할 수 있다.

다음은 Kubernetes 배포 매니페스트에서 Liveness probes를 구성하는 예이다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: your-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: your-app
  template:
    metadata:
      labels:
        app: your-app
    spec:
      containers:
        - name: your-app-container
          image: your-app-image
          ports:
            - containerPort: 80
          livenessProbe:
            httpGet:
              path: /health
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 10
```

Liveness probes는 컨테이너가 시작된 후 15초부터 시작하여 10초마다 컨테이너의 /health 엔드포인트에 대한 HTTP GET 요청을 만든다. 응답 코드가 성공 범위(오류의 경우 500)를 벗어나면 Kubernetes는 자동으로 컨테이너를 재시작한다. 

이렇게 Liveness probes를 구성하면 HTTP 오류가 발생한 컨테이너가 자동으로 재시작되어 애플리케이션의 가용성과 안정성을 유지하는 데 도움이 된다.

---
* Ingress Controller : 규칙을 이행하는 실질적인 애플리케이션 컨테이너이며, nginx 같은 애플리케이션이 예시이다. (nginx는 호스트 기반 라우팅 로드 밸런싱이다.)