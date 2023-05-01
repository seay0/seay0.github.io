---
date: 2023-04-28 00:00:00
layout: post
title: Docker Image ECS Deployment
subtitle: 'Docker Image ECS Deployment'
description: Docker Image ECS Deployment
image: https://res.cloudinary.com/duruq2coh/image/upload/v1681175914/gitio/aws_bbbsnj.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1681175914/gitio/aws_bbbsnj.png
category: AWS
tags:
  - Docker
  - AWS
  - ECS
  - ECR
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **1. Docker Image Build / Pull / Push**  
---

```bash
$ docker build -t hello-was:1.0 .
```

이전 글에서 제작한 fastify-was 서버의 최상위 디렉토리 위치에서 이미지로 빌드한다. 제일 끝에 '.' 부분은 was 서버의 루트 경로에 대한 상대경로를 나타낸 것이다.

* ```-t``` : 가상 터미널을 할당하는 옵션으로 컨테이너 내부에서 터미널과 상호작용할 수 있다.


*참고로 나는 Mongo 서버가 열리지 않으면 was 서버도 열리지 않는 코드로 작성이 되었기 때문에, Mongo 서버부터 열어주어야 한다.*

```bash
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
hello-was    1.0       <비밀>         2 minutes ago   311MB
```

빌드 된 이미지의 목록을 확인해본다. 정상적으로 hello-was의 1.0 버전의 이미지가 생성된 것을 볼 수 있다. (Image ID는 비밀이다 ^^.)

```bash
$ docker container run --name hello-was -p 3000:3000 hello-was:1.0
```

docker container를 통해 빌드한 이미지를 3000포트에서 실행하고, http://localhost:3000 에서 정상적으로 돌아가는지 확인한다. 정상적으로 돌아가지 않는다면 was 서버 코드를 수정해서 로컬에서 돌아가는지 확인하고 다시 이미지 빌드를 하도록 하자.

```bash
$ docker pull mongo
```

Docker hub에서 mongo 이미지를 가져오고, 이미지가 정상적으로 생성되었는지 ```docker images```를 통해 확인한다.

<br>

> ## **2. Elastic Container Registry로 이미지 push**  
---

AWS의 ECR 서비스로 접속하여 새로운 리포지토리를 프라이빗으로 생성한다.
생성 후 ECR 리포지토리로 들어가보면 우측 상단에 '푸시 명령 보기'라는 메뉴가 존재한다. 그 메뉴를 클릭해보면, 다음과 같은 창을 확인할 수 있다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864710/gitio/post/team/2/11_apmtti.png)

위 명령들을 실행하기 위해서는 AWS CLI 설치 및 로그인이 필요하다. 다음 명령을 실행해서 AWS CLI를 설치한다.

```bash
# AWS CLI 설치 방법
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ unzip awscliv2.zip
$ sudo ./aws/install
```

설치가 완료되면, AWS IAM 서비스로 접속해서 등록해두었던 사용자로 들어간다. (등록해두었던게 없으면 새로 등록하자.) 들어가보면 중간 탭에 '보안 자격 증명'을 클릭하며 '액세스 키 만들기'를 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682865523/gitio/post/team/2/42_bf4al6.png)

CLI 를 선택하고 하단의 동의 버튼을 누른다. 다음 탭에서 태그를 원하는 이름으로 지정하고 다음으로 넘어가면, 액세스 키와 비밀 액세스 키가 생성된다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682865523/gitio/post/team/2/43_v571bo.png)

비밀 액세스 키는 이 창을 넘어가버리면 다시는 키를 볼 수 없기 때문에 본인이 기억할 수 있는 곳에 꼭 메모를 해두자. 

```bash
$ aws configure
AWS Access Key ID []: 비밀~
AWS Secret Access Key []: 비밀~
Default region name []: ap-northeast-2
Default output format []: json
```

key 발급이 완료되면, 위 명령을 실행해서 aws 액세스 키와 비밀 액세스 키 등을 입력하고 ECR 푸시 명령 제일 처음에 있던 AWS-CLI 사용 명령어를 그대로 복사해서 붙여 넣기 하면 정상적으로 로그인이 완료되었다는 로그가 출력된다. 


```bash
$ docker tag <Image-ID> <ECR-Repository-URI>/seay0-ecr:was-1.0
$ docker push <ECR-Repository-URI>/seay0-ecr:was-1.0

$ docker tag <mongo-Image_ID> <ECR-Repository-URI>/seay0-ecr:mongo
$ docker push <ECR-Repository-URI>/seay0-ecr:mongo
```

로그인에 성공하면 위 명령을 통해 #1에서 생성한 mongo와 was 이미지에 태그를 붙여서 ecr 주소로 push 한다. 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682866385/gitio/post/team/2/44_a7qwka.png)

ECR 리포지토리를 확인해보면, 두 이미지가 정상적으로 push 된 것을 볼 수 있다.

<br>

> ## **3. Task / service 생성**  
---

ECS를 통해 서비스를 배포할 때 작업 순서를 기억하자.

**1. 태스크 정의**  
**2. 클러스터 생성**  
**3. 서비스 생성**  


제일 먼저, AWS Elastic Container Service 서비스로 접속하여 새로운 태스크를 정의해준다. 내가 만든 fastify 서버는 mongo 서버가 돌아가야 작동하기 때문에, mongo 서비스를 먼저 생성해줄 것이다. 

mongo는 Network Load Balancer에서 돌아가기 때문에, 먼저  EC2 메뉴의 로드 밸런서 탭에서 NLB를 생성해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864711/gitio/post/team/2/24_lm0g2o.png)

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864711/gitio/post/team/2/25_hcetnl.png)

로드 밸런서 타입을 NLB를 선택하고, 이름을 지정해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864711/gitio/post/team/2/26_kmoeur.png)

네트워크 매핑에 가용 영역을 모두 선택한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864711/gitio/post/team/2/27_g8wffe.png)

로드밸런서를 생성하기 위해서는 타켓 그룹이 필요하기 때문에, 새로운 타겟그룹을 생성해준다. 타겟 타입은 IP address로 설정하고 포트를 TCP:27017로 지정한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864711/gitio/post/team/2/28_sjembz.png)

다음 탭의 타겟 그룹 생성을 바로 눌러준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864711/gitio/post/team/2/29_j4baad.png)

다시 로드밸런서 생성으로 돌아와서 리스너에 TCP:27017을 선택해주고, 방금 생성한 타겟 그룹을 선택하고 로그 밸런서 생성을 마친다.

이제 mongo-task를 생성한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864710/gitio/post/team/2/21_uhrz5z.png)

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864709/gitio/post/team/2/12_glxj75.png)

태스크와 컨테이너 이름은 원하는 대로 지정해주고, 이미지 URI는 위 사진의 우측에 이미지 URI 복사를 통해 가져올 수 있다. 
(리포지토리 URI에 이미지 태그를 :다음에 붙여주어도 된다.)

예시로 was:1.0 이미지 사진을 가져왔지만, mongo-task는 mongo 서버를 배포할 작업이기 때문에 mongo 이미지의 URI를 가져오자. 

컨테이너 포트는 mongo에서 사용하는 포트인 27017 포트로 매핑을 해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682870819/gitio/post/team/2/50_hl0u2o.png)

내가 생성한 mongo 서버에 아무나 접근할 수 없게 아이디 비밀번호 환경 변수를 설정해준다.

나머지 과정은 수정할 내용이 없으니 모두 건너 뛰고 생성을 마친다.

이제 mongo 서비스 배포를 할건데, 서비스를 배포하려면 클러스터가 필요하므로, 클러스터를 먼저 생성해준다. 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864710/gitio/post/team/2/15_pkq8gd.png)

was와 mongo 서비스 둘 다 같은 클러스터에서 배포할 예정이다. 클러스터를 생성할 때 서브넷에 RDS가 들어간 것들은 빼주고 생성해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864710/gitio/post/team/2/16_apfl5d.png)

클러스터 생성을 하고 나면 하단에 서비스 생성 버튼을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864711/gitio/post/team/2/22_hnpl66.png)

서비스의 컴퓨팅 옵션은 시작 유형으로 설정한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864710/gitio/post/team/2/23_e8j5gs.png)

서비스를 배포할 태스크 정의에 이전에 생성한 mongo-task를 선택하고, 태스크 수를 한 개로 지정한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864710/gitio/post/team/2/19_mffkh0.png)

클러스터에서 했던 것과 같이 서브넷에 RDS가 들어간 것들은 제외해주고, 보안 그룹을 기존에 있던 보안 그룹으로 지정한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682868444/gitio/post/team/2/45_xpurkm.png)

내 기존 보안 그룹의 인바운드 규칙이다,,, 안전빵으로 필요한 포트를 모두 열어둔 만능 보안 그룹이다 !!!!!!

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864711/gitio/post/team/2/30_nwpsl1.png)

로드 밸런서 유형을 NLB로 선택하고, 위에서 생성해두었던 로드 밸런서와 대상 그룹을 선택한다. 기존 대상 그룹 사용을 선택하면 리스너가 알아서 고정된다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864711/gitio/post/team/2/31_yiicuh.png)

서비스 생성 후 몇 분 정도 기다려보면, 배포 상태가 뜬다. 
배포 현재 상태가 완료로 뜨면 mongo compass를 통해 접속을 해보자.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682870937/gitio/post/team/2/51_o0fdck.png)

설정한 아이디 비밀번호 없이 DNS로 접속을 시도할 경우 다음과 같이 DB를 사용할 수 없다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682870937/gitio/post/team/2/52_nrmudf.png)

아이디 비밀번호를 입력하여 접속할 경우 정상적으로 작동 되는 것을 볼 수 있다.

이제 fastify-was도 같은 방법으로 서비스 배포를 해보자.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864709/gitio/post/team/2/13_uvfbf1.png)

was 태스크 정의도 위에서 했던 대로 이미지 URI를 가져와서 컨테이너 이미지와, 포트 매핑은 TCP:3000으로 지정해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864711/gitio/post/team/2/33_g8tl3l.png)

was에서는 mongo의 NLB 주소를 읽을 수 있는 환경 변수를 따로 설정해 주어야 한다. 이 때 값 유형은 ValueFrom으로 설정해주고, 값은 Secret Manager에서 가져온 보안 암호 URI로 설정해 줄 것이다. 

보안 암호의 ARN 형식은 다음과 같다.

```
arn:aws:secretsmanager:region:aws_account_id:secret:appauthexample-AbCdEf:<env>::
```
---
### **3-1. Secret Maneger**

다음은 Secret Maneger에서 환경 변수를 보안 암호로 바꾸는 방법이다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864712/gitio/post/team/2/34_mbwtxt.png)

was-task의 태스트 실행 역할로 들어간다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864712/gitio/post/team/2/35_y6mq42.png)

새로운 정책을 연결한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864712/gitio/post/team/2/36_nvsyq3.png)

검색으로 'secret'을 검색해보면 'SecretManagerReadWrite'라는 정책이 있다. 정책을 선택해서 추가해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864712/gitio/post/team/2/37_r0p6bw.png)

다음과 같이 정책이 추가된 것을 볼 수 있다. 이 정책이 추가되어야 Secret Manager를 통해 보안 암호를 설정하여 사용할 수 있다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682870291/gitio/post/team/2/47_qz0amn.png)

이제 Secret Manager로 접속하여 새 보안 암호 저장을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682870292/gitio/post/team/2/48_ggem03.png)

보안 암호 유형을 '다른 유형의 보안 암호'를 선택한 후, 키/값에 본인이 설정하고 싶은 환경 변수와 값을 넣어주고 생성한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682870292/gitio/post/team/2/49_uvktmg.png)

생성을 마치면 보안 암호 ARN이 뜨는데, 이 부분을 복사해서 was-task의 환경 변수에 형식에 맞춰 붙여넣어주면 된다.

---
이어서 was 서비스 생성을 해보자.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864710/gitio/post/team/2/17_vvgebq.png)

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864710/gitio/post/team/2/18_us9bos.png)


was 서비스도 컴퓨팅 옵션을 시작 유형으로 선택하고 생성한 task와 버전이 최신인지 확인한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864710/gitio/post/team/2/19_mffkh0.png)

네트워킹 부분에 서브넷도 이전 몽고 서비스에서 했던 것과 같이 RDS가 들어간 것은 다 지워주고, 보안 그룹을 기존에 만들어두었던 보안그룹으로 설정해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864710/gitio/post/team/2/20_ws2jww.png)

로드 밸런서는 http와 https로 배포할 것이기 때문에 ALB와 타겟 그룹을 새로 생성해준다. 리스너 포트는 80번으로 지정해주었다. (뒤에 3000 쓰기 귀찮음)

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864712/gitio/post/team/2/38_zosf7z.png)

서비스 생성을 마치고 기다려보면 was 서버도 정상적으로 배포가 되었다. 서비스에 연결해둔 ALB DNS 주소로 접속해보면

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864712/gitio/post/team/2/39_eywhn7.png)

다음과 같이 정상적으로 뜬당.

<br>

> ## **4. 구매한 도메인의 https로 was 서버 연결**  
---

이 부분은 간단히 Route53에서 구매해둔 도메인의 호스팅 영역에 레코드를 추가해주면 된다.

레코드 추가 전에 ALB에 HTTPS:443 리스너를 추가해주어야 한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682872022/gitio/post/team/2/53_jaced2.png)

was에 연결해둔 ALB로 들어가서 Add Listner를 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682872023/gitio/post/team/2/54_c9um2v.png)

리스너의 프로토콜을 HTTPS:443으로 설정해주고, 디테일 액션에 Forward를 선택한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682872023/gitio/post/team/2/55_zhz4pc.png)

타겟 그룹도 ALB에 연결해둔 타겟 그룹을 선택해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682872023/gitio/post/team/2/56_bj1rne.png)

마지막으로 리스너 세팅에 Certificate Manager에서 발급 받았던 인증서를 선택해주고 Add 버튼을 눌러준다. 이렇게 하면 https로도 도메인을 연결할 수 있다.

이제 호스팅 영역에 레코드를 추가해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864712/gitio/post/team/2/40_fnathz.png)

이름은 www로 해주었고, 별칭에 체크하여 was에 연결한 ALB를 선택하고 레코드를 생성해주었다. 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682864712/gitio/post/team/2/41_ahfnbm.png)

레코드를 생성한 후 ```https://www.<yourdomain>```으로 접속해보면 이것도 잘 뜬다!

만약 인증서가 오류가 났거나 재발급을 요청했는데 검증 대기중에 한참 머무른다면, 도메인을 새로 구매하는 것이 좋다 T-T,,, 

나도 인증서 재발급을 눌렀다가 1시간을 기다렸는데 검증 대기중이 끝도 없길래 도메인을 3$짜리 .click을 재 구매해서 인증서를 다시 발급 받았다... 

이렇게 fastify 서버 이미지와 mongo 이미지를 ECS로 배포하는 과정 설명을 마친다 ~!@~!@~@!~@