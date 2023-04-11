---
date: 2023-04-09 12:00:00
layout: post
title: Proxy server &#35;2 / TIL
subtitle: '프록시 서버 실습'
description: 프록시 서버 실습
image: https://res.cloudinary.com/duruq2coh/image/upload/v1681175914/gitio/aws_bbbsnj.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1681175914/gitio/aws_bbbsnj.png
category: DevOps Bootcamp
tags:
  - proxy server
  - DevOps BootCamp
author: seay0
---

# **Proxy Server**


내 컴퓨터를 origin 서버의 리버스 프록시 서버로 만드는 실습을 해보자 ! (캐싱 기능을 포함한 프록시 서버를 작성해야 한다.)  

먼저 프록시 서버를 만들기 위해서 /etc/nginx/nginx.conf 파일을 sudo 권한으로 다음과 같이 수정한다.

```nginx
server {
	listen 10026;
	server_name localhost;

	location / {
		proxy_pass http://3.39.193.36:8080;
        # nginx가 요청을 프록시 해야하는 업 스트림 서버의 URL이다.
		proxy_set_header myname "devops-04-OSeaYeon";
        # my name이라는 사용자 지정 헤더를 설정하고 " " 안의 값을 할당한다.
		proxy_set_header X-Real-IP $remote_addr;
        # X-Real-Ip 헤더를 요청하는 클라이언트의 IP 주소를 설정한다.
        # $remote_addr은 요청을 만드는 클라이언트 IP 주소를 포함하는 변수이다.
		proxy_set_header Host $host;
        # Host 헤더를 원래 요청의 Host 헤더 값으로 설정한다.
    }
}
```

수정 후 다음 명령어를 차례대로 실행해 본다.  

<br>

```bash
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
// 제대로 된 코드가 작성되지 않으면 failed가 뜬다.

$ sudo systemctl reload nginx
# 만약 nginx가 시작되지 않은 상태라면 'reload'가 아닌 'start'로 실행한다.

$ curl -I http://localhost:10026
```

origin 서버의 주소는 3.39.193.136:8080 이며, 프록시 서버(localhost:10026)로 요청을 보내면 아래와 같은 화면으로 서버가 작동된다. 출력되는 내용은 내가 프록시 서버에 요청을 세 번 보냈기 때문에 이름이 세 번 뜨는 것이다.

<br>

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681099162/gitio/post/proxy/2_ehcex0.png)

---

## **프록시 서버에 캐싱 기능 포함하기**  

이제 다시 nginx.conf 파일을 수정해서 캐싱 기능을 포함해보자.

```nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m inactive=5s;
# 업스트림 서버에서 캐시된 응답을 저장하는 데 사용할 Nginx의 캐시 경로를 /var/cache/nginx로 설정한다.
# 캐시 영역의 이름(key_zone)은 my_cache이며, 메모리는 10MB로 제한된다.
# inactive : 5초 동안 액세스 하지 않은 캐시 항목은 자동으로 제거된다.

proxy_cache_key "$scheme$request_method$host$request_uri";
# 캐시된 각 요청에 대한 캐시 키를 설정한다.
# 캐시 키는 요청 방법($request_method), 요청 스키마($scheme), 호스트 이름($host) 및 URI($request_uri)를 기반으로 한다.

server {
	listen 10026;
	server_name localhost;

		location / {
		proxy_pass http://3.39.193.136:8080;
		proxy_set_header myname "devops-04-OSeaYeon"; 
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header Host $host;

		proxy_cache my_cache;
        # 현재 위치 블록과 일치하는 요청에 대한 캐싱을 활성화한다.
        # 사용할 캐시 영역의 이름(my_cache)을 지정한다.

        proxy_cache_valid any 5s;
        # 응답 상태 코드에 관계 없이 각 응답의 캐싱 시간을 5초로 설정한다.

		add_header X-Cache-Status $upstream_cache_status;
        # 응답이 캐시에 제공되고 있는지 여부를 나타낸다.
        # 헤더의 값은 $upstream_cache_status 변수의 값으로 설정된다.

		if ($upstream_cache_status = "MISS") {
			add_header Cache-Control "public, max-age=5";
		}
        # 업스트림 서버에 캐시를 찾을 수 없는 상태면
        # 사용자의 브라우저 캐시가 이 응답을 최대 5초 동안 저장할 수 있음을 의미한다.
		if ($upstream_cache_status = "HIT") {
			add_header Cache-Control "public, max-age=5, must-revalidate";
		}
        # 업스트림 서버에서 응답이 캐시에서 제공되었음을 나타내며
        # 사용자의 브라우저 캐시는 최대 5초 동안 이 응답을 저장할 수 있지만,
        # 해당 시간 이후 캐시된 복사본을 사용하기 전에 서버에서 유효성 검사를 다시 해야한다.
		if ($upstream_cache_status = "EXPIRED") {
			add_header Cache-Control "no-cache";
		}
        # 업스트림 서버에서 응답이 캐시에서 발견되었지만 만료되었음을 나타내며
        # 사용자 브라우저 캐시가 이 응답의 캐시된 복사본을 사용할 수 없으며
        # 서버에 다시 요청해야 함을 의미한다.

        # 위의 if 문은 $upstream_cache_status 변수 값을 기반으로 서버 응답에 특정 Cache-Control 헤더를 추가하는 구문이다.
	}
}
```

코드를 위와 같이 작성한 뒤 nginx 테스트와 reload를 해준 후 curl로 요청을 보내게 되면
요청의 결과에 헤더 값이 다음과 같이 출력된다.

```bash
$ curl -I http://localhost:10026
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Mon, 10 Apr 2023 04:21:22 GMT
Content-Type: text/plain
Content-Length: 43
Connection: keep-alive
Last-Modified: Mon, 10 Apr 2023 04:21:14 GMT
ETag: "64338eba-2b"
X-Cache-Status: MISS
Accept-Ranges: bytes

$ curl -I http://localhost:10026
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Mon, 10 Apr 2023 04:21:26 GMT
Content-Type: text/plain
Content-Length: 43
Connection: keep-alive
Last-Modified: Mon, 10 Apr 2023 04:21:14 GMT
ETag: "64338eba-2b"
X-Cache-Status: HIT
Accept-Ranges: bytes

$ curl -I http://localhost:10026
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Mon, 10 Apr 2023 04:21:28 GMT
Content-Type: text/plain
Content-Length: 86
Connection: keep-alive
Last-Modified: Mon, 10 Apr 2023 04:21:22 GMT
ETag: "64338ec2-56"
X-Cache-Status: EXPIRED
Accept-Ranges: bytes
```

위와 같이 나오는 이유에 대해 더 자세히 알아보자.  
* 명령을 처음 실행하면 서버가 응답 헤더에 X-Cache-Status: MISS 와 함께 200 OK 응답을 반환한다. 이는 서버가 요청된 리소스의 캐시된 버전을 찾지 못했기 때문에 업스트림 서버에서 검색하여 클라이언트로 전달해야 함을 나타낸다.
* 명령을 두 번째로 실행하면 서버가 응답 헤더에 X-Cache-Status: HIT와 함께 200 OK 응답을 반환한다. 이는 서버가 요청된 리소스의 캐시된 버전을 찾아 업스트림 서버에서 검색할 필요 없이 클라이언트에 직접 제공했음을 나타낸다.
* 명령을 세 번째로 실행하면 응답 헤더에 X-Cache-Status: EXPIRED와 함께 200 OK 응답을 반환한다. 이는 요청된 리소스의 캐시된 버전이 만료되었음을 나타낸다.('proxy_cache_path' 지시문의 'inactive=5s'에 따른 결과)

<br>

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
