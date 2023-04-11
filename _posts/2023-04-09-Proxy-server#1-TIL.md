---
date: 2023-04-09 12:00:00
layout: post
title: Proxy server &#35;1 / TIL
subtitle: '프록시 서버란 ?'
description: 프록시 서버란 ?
image: https://res.cloudinary.com/duruq2coh/image/upload/v1681175914/gitio/aws_bbbsnj.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1681175914/gitio/aws_bbbsnj.png
category: DevOps Bootcamp
tags:
  - proxy server
  - DevOps BootCamp
author: seay0
comments: true
---

# **Proxy Server**


* 원 서버를 대리하여 통신하며 중계 역할을 한다.  
  * 캐시, 로드밸런서, 보안 등  
* **프록시 서버를 보는 입장** 
  * 클라이언트 입장 : 서버라고 인식
  * 서버 입장 : 클라이언트라고 인식
* 웹 서버에서는 클라이언트의 IP를 숨겨 프라이버시를 강화하는 데 사용할 수 있다.  
* 서버로부터 응답을 압축하여 네트워크 대역폭을 줄이고 성능을 향상 시킨다.  

<br>

![proxy](https://res.cloudinary.com/duruq2coh/image/upload/v1681095463/gitio/post/proxy/1_pnzeqp.png)

<br>

**포워드 프록시 (forward proxy)**  
* 일반적인 프록시로 클라이언트-서버 구조에서 **클라이언트 쪽**을 대리한다.  
* 클라이언트에서 서버로 리소스를 요청할 때 직접 요청하지 않고 **프록시 서버를 거쳐서 요청**한다.

**리버스 프록시 (reverse proxy)**  
* **애플리케이션 서버의 앞**에 위치하여 클라이언트가 서버에 요청할 때 리버스 프록시를 호출하고,
* 리버스 프록시가 origin 서버로부터 응답을 전달받아 다시 클라이언트에게 전송하는 역할을 한다.

<br>

## **로드밸런서 (Load Balancer)**

---

서비스를 제공할 수 있는 용량이 단일 서버로 충분하더라도, 서버를 한 대로 구성하면 장애가 발생했을 때 정상적인 서비스를 제공할 수 없다.

서비스 가용성을 높이기 위해 하나의 서비스는 보통 두 대 이상의 서버로 구성하는 데 각 서버 IP주소가 다르므로 사용자가 서비스를 호출할 때 어떤 IP로 서비스를 요청할 지 결정해야 한다.

여러 대의 서버를 사용하면 특정 서버에 장애가 발생했을 때 전체 사용자에게 영향을 미치진 않지만 부분적으로 서비스 장애가 발생하는데, **로드 밸런서**는 이러한 문제를 해결하기 위한 방안이다.

<br>

* 로드 밸런서는 사용자로부터 요청이 들어오면 받아서 사용자 별로 다수의 서버에 요청을 분산시킨다.
* 때문에 서버에 장애가 발생하더라도 다른 서버에서 서비스를 제공받을 수 있게 한다.  

<br>

**L4 로드 밸런서**  
* TCP, UDP 기반 / 일반적인 로드 밸런서 동작 방식  
* 기존 로드 밸런서의 부하 분산 기능 뿐 아니라, TCP 계층의 최적화 보안 기능을 함께 제공한다.  
* 최근에는 4계층과 7계층의 기능을 모두 지원한다.

**L7 로드 밸런서 (ADC, Application Delivery Controller)**  
* 프로토콜(HTTP, FTP, SMTP) 정보 기반 / 리버스 프록시 역할을 수행한다.  
* HTTP 헤더 정보나 URI 같은 정보를 기반으로 프로토콜을 이해한 후 부하를 분산한다.  
* 4계층에서 7계층까지 로드 밸런싱 기능을 제공하며, 장애 극복, 리다이렉션 기능도 함께 수행한다.

<br>

## **캐시 (Cache)**

---

캐시가 없을 경우 사용자는 데이터가 변경되지 않아도 계속 네트워크를 통해 데이터를 다운로드 받아야 한다. 따라서 용량이 클 수록 비용이 커지고 로딩 속도가 느려진다. 

* **캐시** : 데이터나 값을 미리 복사해놓은 임시 장소
* 계산이나 접근 시간 없이 더 빠른 속도로 데이터에 접근할 수 있다.
* 브라우저에 캐시를 저장할 땐 헤더에 cache-control 속성을 통해 캐시가 유효한 시간을 지정할 수 있다.
* 유효한 시간을 지정하면 응답 결과를 캐시에 저장하며 지정한 시간 동안 유효하게 된다.
* 요청이 또 다시 오면 캐시를 우선 조회하여 캐시가 존재하거나 지정한 시간이 지나지 않아 유효하다면 해당 캐시에서 데이터를 가져온다.
* 유효 시간이 지나게 되면 서버에 재요청하여 다시 네트워크 다운로드가 발생하고 기존 캐시를 지우고 새 캐시로 업데이트 하며 유효 시간이 초기화된다.

<br>

**프록시 캐시**

평소 우리는 해외 사이트에서 불편함 없이 빠르게 정보를 받아볼 수 있다. 이러한 것들이 클라이언트와 원 서버 사이에 프록시 캐시가 도입되어 있기 때문이다.

우리가 어떠한 자료를 찾을 때 여러 사람이 찾은 자료일수록 이미 캐시에 등록되어 있기 때문에 빠른 속도로 자료를 가져올 수 있다. 이는 캐시가 같은 국내에 있기 때문에 원서버에 접근하는 것보다 훨씬 빠르게 자료를 가져올 수 있는 것이다.

<br>

## **캐시 지시어 (directives)**
캐시에 관련된 헤더 & 조건부 요청 헤더

---

### **Cache-Control : 캐시 무효화**  
웹 브라우저가 민감한 정보까지 임의로 캐싱할 때 무효화하는 헤더  

<br>  

* **Cache-Control: s-maxage**  
  * 프록시 캐시에만 적용되는 max-age  
      
    
* **Cache-Control: no-cache**  
  * 데이터는 캐시해도 되지만, 항상 원 서버에 검증하고 사용 (이름에 주의)  
      
    
* **Cache-Control: no-store**  
  * 데이터에 민감한 정보가 있으므로 저장하면 안됨  
      
    
* **Cache-Control: private**  
  * 응답이 해당 사용자만을 위한 것, private 캐시에 저장해야 함(기본 값)  
  * 클라이언트에서 사용하고 저장하는 캐시  
      
    
* **Cache-Control: public**  
  * 응답이 public 캐시에 저장되어도 됨  
  * 프록시 서버의 캐시  
      
    
* **Cache-Control: must-revalidte**  
  * 캐시 만료 후 최초 조회 시 원 서버에 검증해야 함  
  * 원 서버 접근 실패 시 반드시 504(Gateway Timeout) 오류가 발생해야 함  
  * must-revalidate : 캐시 유효 시간이라면 캐시를 사용함  
      
    
* **Age: 60 (HTTP 헤더)**  
  * Origin 서버에서 응답 후 프록시 캐시 내에 머문 시간(초)  
      
    
* **Pragma: no-cache**  
  * HTTP 1.0 하위 호환  
      
    
* **Expires: Mon, 01 Jan 2020 00:00:00 GMT**  
  * 캐시 만료일을 정확한 날짜로 지정하며, HTTP 1.0부터 사용한다.  
  * max-age와 함께 사용하면 Expires는 무시된다.

<br>

**Cache-Control : no-cache, no-store, must-revalidate**  
**Pragma: no-cache**

**확실한 캐시 무효화 응답을 하고 싶을 때 위 캐시 지시어를 모두 넣어야 한다.**

<div id="disqus_thread"></div>
<script>
    /**
    *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
    *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables    */
    /*
    var disqus_config = function () {
    this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    */
    (function() { // DON'T EDIT BELOW THIS LINE
    var d = document, s = d.createElement('script');
    s.src = 'https://seay0-github-io.disqus.com/embed.js';
    s.setAttribute('data-timestamp', +new Date());
    (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
