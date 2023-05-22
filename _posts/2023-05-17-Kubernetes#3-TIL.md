---
date: 2023-05-17 00:00:00
layout: post
title: Kubernetes? &#35;3 / TIL
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



```bash
$ kubectl apply -f cozserver-deployment-v1.yaml --record
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/cozserver created
```

```bash
$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
cozserver                    1/1     Running   0          33m
cozserver-8467d957d9-6tn2k   1/1     Running   0          25s
cozserver-8467d957d9-7zq76   1/1     Running   0          25s
cozserver-8467d957d9-pp6jp   1/1     Running   0          25s
```

