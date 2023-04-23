---
date: 2023-04-19 00:00:00
layout: post
title: 3 Tier Architecture / TIL
subtitle: '3 Tier Architecture'
description: 3 Tier Architecture 실습
image: https://res.cloudinary.com/duruq2coh/image/upload/v1681175914/gitio/aws_bbbsnj.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1681175914/gitio/aws_bbbsnj.png
category: AWS
tags:
  - AWS
  - 3 Tier Architecture
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **3 Tier Architecture**  

형광색으로 체크해둔 부분을 확인하며 차근차근 구현해보자!


### **1. 도메인 구매 / 인증서 발급**
---
Route53 에서 원하는 도메인을 구매한다.

AWS Certificate Manager(ACM)에서 인증서를 요청한다. 이번 실습에서 Front(client)는 버지니아 북부 리전(us-east-1)에, back(server)는 서울(ap-northeast-2)에 발급받을 것이다. 

front를 us-east-1에 요청하는 이유는, CDN 서비스인 Cloudfront 사용 리전이 그쪽에 위치해있기 때문이다.

인증서는 https 접속을 하기 위해 필요하다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907899/gitio/post/aws/18_qfix1l.png)

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907899/gitio/post/aws/19_xgmeuz.png)

위와 같이 두 리전에 *.<도메인> 형식으로 도메인 이름을 지정해서 인증서를 요청한다.

요청 후 인증서 목록으로 가서 인증서를 클릭해보면,

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907899/gitio/post/aws/20_v95y7i.png)

다음과 같은 탭이 있는데, Rout 53에서 레코드 생성 버튼을 눌러 CNAME 레코드를 생성해준다.

CNAME 레코드를 생성하는 이유는 보안 이슈 때문이다. 만약 타인이 나의 도메인으로 인증서를 요청하면 그 도메인이 본인의 도메인이 맞는지 확인하기 위해 레코드를 통해 검증하는 것이다.

만일 Route 53에 해당 도메인이 존재한다면 호스팅 영역이 자동으로 생성되며 NS(이름 서버) 레코드와 SOA(권한 시작) 레코드가 자동으로 생성된다. (CNAME 포함)

<br>

### **2. EC2 인스턴스 생성**
---
AWS의 EC2 탭으로 가서 새 인스턴스를 생성한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681898975/gitio/post/aws/1_kpnb0b.png)

EC2 인스턴스의 이름을 정하고 OS와 버전을 선택한다.


![](https://res.cloudinary.com/duruq2coh/image/upload/v1681898975/gitio/post/aws/2_fhpfuy.png)

인스턴스 유형을 프리티어 사용이 가능한 't2.micro'로 선택하고, 키 페어를 선택해준다. 키 페어가 없다면 새 키 페어 생성으로 새로운 키를 생성한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681898975/gitio/post/aws/3_q672pm.png)


키 이름을 정해주고 생성해주면 자동으로 브라우저에서 키를 다운로드하는데, 이렇게 생성된 키들은 따로 폴더를 마련해서 저장해두는 것이 좋다. 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681898975/gitio/post/aws/4_y8nrip.png)

네트워크 설정에 체크한 부분을 모두 선택해준다. 보안 그룹을 생성하는 과정이다. 리전에 관계 없이 ssh, http, https에서 들어오는 트래픽을 모두 받아들인다는 의미이다.  
트래픽 | Port
:--: | :--:
SSH 트래픽 | 22 포트  
HTTP 트래픽 | 80 포트  
HTTPS 트래픽 | 443 포트   

ps. 나중에 RDS를 사용할 때 추가할 포트도 있다. (3306 포트)

위 사항을 모두 확인해보고 인스턴스 생성을 마친다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681898975/gitio/post/aws/5_gclmax.png)

EC2 인스턴스가 생성된 것을 볼 수 있다. 인스턴스를 선택하고 연결을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681898975/gitio/post/aws/6_xivfuj.png)

연결을 눌러서 SSH 클라이언트 탭을 클릭해보면 터미널에서 인스턴스 내부로 접속하는 방법이 명시되어 있다. 똑같이 따라해보자.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681898976/gitio/post/aws/7_zawj2b.png)

key를 따로 저장해둔 폴더로 이동하여 키에 권한을 부여한 후 접속 명령어를 입력한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681898976/gitio/post/aws/8_ittpsl.png)

무슨 기다란 문구가 쫙 뜨면서 터미널 좌측에 사용자 이름이 ubuntu@ip 주소로 바뀐 것을 볼 수 있다. 정상적으로 인스턴스 내부에 접속하게 된 것이다.

접속을 하고 나면 서버 파일이 있는 디렉토리에서 ```sudo npm start```를 하면 서버가 구동되는데, 인스턴스 내에서는 npm 명령이 설치되어 있지 않기 때문에 설치를 해주어야 한다. 

나의 경우 서버 파일이 깃허브에서 가져와야 했기 때문에 깃허브 설정과 ssh키 등록과 토큰 발급까지 새로 해주었다.

```bash
$ sudo apt-get update
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
nvm install --lts
# sleep 5
sudo apt-get install npm -y
```

위 명령을 입력하면 npm과 nvm 설치/설정이 알아서 이루어진다. 설치를 끝내고 npm install을 해주면 버전이 맞지 않다는 오류가 날 수도 있다. 그 오류가 나면 서버와 맞는 버전으로 바꿔주어야 한다. 나는 16.13.0 버전이 필요하기 때문에 다음 명령을 입력해 버전을 바꿔주었다.

```bash
nvm install 16.13.0
```

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681901855/gitio/post/aws/9_ibkmro.png)

바꿔주고 나서 ```sudo npm start```를 하면 서버가 80번 포트에서 구동된다.

<br>

### **3. 로드 밸런서 생성**

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907899/gitio/post/aws/23_tmzlqb.png)

EC2의 로드밸런서 메뉴로 접속해 Create load balancer를 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907899/gitio/post/aws/24_fbmg3f.png)

우리는 HTTP와 HTTPS 배포를 할 예정이기 때문에 Application Load Balancer를 Create 해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907899/gitio/post/aws/25_s0ma4q.png)

로드밸런서 이름과 스키마()를 Internet-facing으로 선택해준다. 두 옵션의 차이는 인터넷에 연결해야 하는지, 내부에 연결해야 하는지 차이이다. 

* Internet-facing : 인터넷에서 웹 서버 또는 기타 공개적으로 사용 가능한 서비스와 같이 인터넷에서도 액세스할 수 있는 백엔드 리소스로 트래픽을 분산하도록 설계되었다. 일반적으로 공개적으로 액세스할 수 있는 IP 주소가 있으며, 들어오는 요청의 진입점 역할을 한다.
* Internal : 데이터 센터의 서버 간 또는 가상 사설 클라우드와 같은 사설 네트워크 내에서 트래픽을 분산하고, 백엔드 리소스의 가용성과 확장성을 개선하며 공용 인터넷에서 숨겨진 상태로 유지하는 데 사용된다. 인터넷에서 액세스할 수 없으며, 일반적으로 개인 IP 주소가 있다. 

즉, 주요 차이는 접근성과 처리하는 트래픽 유형이다.  Internet-facing 스키마는 인터넷에서 공개적으로 액세스 가능한 백엔드 리소스로 트래픽을 분배하는 반면 Internal 스키마는 사설 네트워크 내에서 트래픽을 분배한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907899/gitio/post/aws/26_hx1fuo.png)

로드밸런서 선택 메뉴 중 네트워크 매핑 부분의 Mappings에는 가용 영역(AZ)이 적혀있는데, 이 부분은 만일 사용중이던 AZ에 장애가 발생하면 대체로 사용할 영역을 지정해두는 것이다. 장애 발생 시 서버가 정상적으로 돌아가야하기 때문에, 가용 영역은 두 가지 이상만 선택할 수 있게 되어 있다. 많이 체크하면 장애가 발생하여 서비스를 제대로 이용하지 못 할 확률이 낮아지지만 그만큼 비용이 추가된다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907899/gitio/post/aws/27_qv8mo0.png)

보안 그룹도 EC2 생성 할 때 만들어 두었던 launch-wizard-1을 선택한다. 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907900/gitio/post/aws/28_mcooys.png)

다음으로 리스너를 HTTP인 80번 포트와 HTTPS인 443번 포트에 만들어주는데, 그 전에 타겟 그룹 생성이 필요하다.

* **타겟 그룹 생성이 필요한 이유**  
타겟 그룹을 통해 애플리케이션 또는 서비스에 대한 트래픽 라우팅을 관리하고, 가용성 및 확장성을 개선하고, 성능 및 사용자 경험을 최적화하는 데 도움이 되는 고급 로드 밸런싱 기능을 활성화하는 유연하고 강력한 방법을 제공한다.  
* [**>> 관련 레퍼런스**](https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/application/load-balancer-target-groups.html)

Create target group을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907900/gitio/post/aws/29_x5xju8.png)

타겟 그룹의 타입은 EC2 인스턴스에서 실행할 것이기 때문에 Instances를 선택해주고, 이름을 지정해준 후 다음을 누른다. 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907900/gitio/post/aws/33_hg0acx.png)

다음 화면에서 생성하고자 하는 타겟 그룹을 선택 한 후 ```Include as pending below```를 누르면 하단의 Review targets로 내려가게 되는데, 그 상태로 우측 하단의 ```Create target group```을 누르면 타겟 그룹이 생성된다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907900/gitio/post/aws/30_aoxgxd.png)

그럼 다음과 같이 타겟 그룹에 생성한 타겟 그룹이 표시되는데, 생성한 타겟 그룹을 선택하면 된다. (나는 이미 만들어 둔 로드밸런서에 해당 타겟 그룹을 등록해 두었기 때문에 회색으로 등록할 수 없다고 뜨는 것)

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907900/gitio/post/aws/31_xc4gms.png)

똑같이 443번 포트로 리스너를 만들어서 타겟 그룹을 등록해준다.
443번 포트 리스너를 만들게 되면, Secure Listener 설정에 SSL/TSL 인증서 등록하는 부분이 필수 입력 사항으로 열리게된다. 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907900/gitio/post/aws/32_aw9ffj.png)

이 부분이 필수로 필요한 이유는, 인증서가 클라이언트와 로드 밸런서 간의 통신을 보호하는 데에 사용한다. 따라서 클라이언트가 로드 밸런서에 연결할 때 액세스하려는 웹 사이트 또는 서비스의 도메인 이름과 일치하는 유효한 인증서로 보안 연결을 설정해야 한다.

이렇게 로드 밸런서 생성을 마친다.

<br>

### **4. CloudFront 생성**
---
![](https://res.cloudinary.com/duruq2coh/image/upload/v1681916789/gitio/post/aws/34_avweb7.png)

AWS의 CloudFront에 접속하여 배포 생성을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681916789/gitio/post/aws/35_zu402j.png)

원본 도메인에 s3에서 생성한 도메인을 누르면, 경고 표시같은 것이 뜬다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681916789/gitio/post/aws/36_uu0k28.png)

웹 사이트 엔드 포인트 사용을 눌러준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681916789/gitio/post/aws/37_ldxhjk.png)

내려보면 뷰어 프로토콜 정책에 Redirect HTTP to HTTPS을 체크해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681916789/gitio/post/aws/38_imo76h.png)

설정에서 대체 도메인 이름의 항목과 사용자 정의 인증서를 선택해주어야 한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681916789/gitio/post/aws/39_rwxdny.png)

다음과 대체 도메인(CNAME)과 발급 받았던 인증서를 추가한 후 생성한다.

<br>

### **5. Route 53 호스팅 영역에 레코드 추가**
---
![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907899/gitio/post/aws/22_szryy5.png)

이제 앞서 인증서에서 cname을 만들 당시 생성되었던 Route 53 호스팅 영역에 들어가면, 이미 생성되어 있는 레코드 NS, SOA, CNAME을 볼 수 있는데, 이 외에 추가한 로드 밸런서와 CloudFront 레코드를 추가해주어야 한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681917277/gitio/post/aws/40_oqafrp.png)

로드 밸런서를 연결하는 백엔드 레코드 이름은 ```api.```을 붙여준다. 그리고 별칭 버튼을 on으로 바꾸어 라우팅 대상을 ALB로 변경, 리전을 서울로 바꾸면 이전에 생성하였던 로드 밸런서가 나오는데, 그 로드 밸런서를 선택해준다. 

다른 레코드 추가 버튼을 눌러 CloudFront 레코드를 하나 더 생성해주자.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681917277/gitio/post/aws/41_f54hvs.png)

위 방법과 같이 레코드 이름은 ```www.```, 별칭 버튼을 on으로 바꾸고, 별칭의 CloudFront를 선택하면 이전에 생성해두었던 것이 나오는데, 그 CloudFront를 선택해준다.

레코드 생성 버튼을 누르면 정상적으로 백과 프론트의 호스팅 영역 레코드가 생성된다. 

<br>

### **6. S3 버킷 생성**
---
AWS의 S3 탭에 버킷 생성을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681903335/gitio/post/aws/10_o2hefc.png)

버킷 이름은 client. 에 구매한 도메인의 주소를 적어 이름을 설정한다. 리전은 EC2를 생성했던 서울에 생성해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681903335/gitio/post/aws/11_tl9ziv.png)

버킷의 퍼블릭 액세스 차단을 풀어 어디서든 접속할 수 있게 해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681907898/gitio/post/aws/17_eh71o8.png)

위 내용 수정을 마치고 생성을 해준 후, S3의 속성 탭에 들어가서 밑으로 내려보면 정적 웹 사이트 호스팅이라는 파트가 있다. 정적 웹 사이트 호스팅이 비활성화 되어 있으면, 편집을 눌러 활성화로 바꾸어 준다. 

체크해둔 버킷 웹 사이트 엔드포인트 부분을 아래 설명하는 .env 파일의 s3 주소 부분에 붙여 넣으면 된다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681905528/gitio/post/aws/16_z2jibg.png)

설정할 때 다른 것은 바꿀 것이 없고, 인덱스 문서와 오류 문서의 이름을 s3 버킷에 업로드 한 파일 이름인 index.html 로 바꾸어 준다.

**Front client의 .env 파일**

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681904933/gitio/post/aws/15_gfvcto.png)

* REACT_APP_API_URL : https 서버 도메인 주소
* S3_ADDRESS : S3 주소
* HTTPS_ADDRESS : https 서버 도메인 주소
* ELB_DNS_ADDRESS : Elastic Load Balancer 주소

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681903335/gitio/post/aws/12_jattis.png)


client 폴더에서 .env 파일을 작성 후 npm run build를 해주면 client 폴더에 build 폴더가 생긴다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681903334/gitio/post/aws/13_x8uzof.png)

build 폴더 안의 파일들을 S3 버킷에 업로드 해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1681903335/gitio/post/aws/14_v3v0bz.png)

업로드는 폴더 채로 하면 안되고, build 폴더 안에 들어있는 폴더와 파일을 각각 따로 넣어주어야 한다.

<br>

위와 같은 설정을 마치고 호스팅 영역에서 등록해둔 레코드 주소로 접속해보면 정상적으로 서버와 클라이언트가 구동되는 것을 확인할 수 있다. 

나의 경우엔 ```https://api.seay0.link``` 에서는 백엔드인 서버가 서울 리전에서 로드 밸런서를 통해 돌아가게 하였고, ```https://www.seay0.link``` 에서는 프론트엔드인 클라이언트가 버지니아 북부 리전에서 클라우트 프론트를 통해 돌아가게 하였다.