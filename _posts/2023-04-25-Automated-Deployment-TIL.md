---
date: 2023-04-25 00:00:00
layout: post
title: Automated Deployment / TIL
subtitle: '배포 자동화 / 파이프라인'
description: 배포 자동화 / 파이프라인
image: https://res.cloudinary.com/duruq2coh/image/upload/v1681965389/gitio/devops_ulvdb1.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1681965389/gitio/devops_ulvdb1.png
category: DevOps
tags:
  - Pipeline
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **배포 자동화**  

파이프라인을 통해 배포를 자동화하기 위한 과정을 알아보자.

이전에 3티어 아키텍처를 만들어보며 알아두었던 EC2, S3, RDS를 활용할 건데, 이 세 가지의 기본적인 생성 방법은 [>> 여기 <<](https://seay0.github.io/3-Tier-Architecture-TIL/) 에서 참고하자.


### **1. S3, EC2 생성**
---

제일 먼저, GitHub에서 배포할 서버와 클라이언트 레포지토리를 포크한 후, 클론해서 로컬 환경에 소스 코드를 저장한다.

EC2 생성 후 인스턴스에 연결한 다음 npm 설치도 잊지 않고 하도록 하자. 

**>> node.js / npm / nvm 설치 명령어**
```bash
sudo apt-get update
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
nvm install --lts
# sleep 5
sudo apt-get install npm -y
```

S3 버킷을 생성하는데, 이전과 다르게 npm 모듈 설치와 빌드를 수동으로 하지 않고 자동으로 하게 yaml 파일을 작성하여 파이프라인에 올릴 것이다. 따라서 S3 버킷의 내부는 비워두고, 정적 웹 호스팅용으로 구성한다.

자동화를 위해 작성할 ```buildspec.yaml``` 파일을 최상위 디렉토리에 작성한다.

```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - cd client
      - npm install
  build:
    commands:
      - npm run build

artifacts:
  files:
    - '**/*'
  base-directory: client/build
```

이 yaml 파일은 사용자가 GitHub에 push할 때 자동으로 위 명령어들이 실행된다. 

<br>

### **2. 클라이언트 배포 파이프라인 구축**
---

S3를 이전에 했던 방법으로 똑같이 생성하였으면, 자동화를 위한 파이프 라인을 만들어줘야 한다.

AWS 콘솔의 CodePipeline 메뉴로 접속해 파이프라인 생성을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682423524/gitio/post/deploy/1_foi5v2.png)

파이프라인 이름을 지정하고 다음으로 넘어간다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682423524/gitio/post/deploy/2_qndycq.png)

소스 스테이지는 소스를 어디에서 받아올지 지정하는 부분이다.

우리는 GitHub에서 포크한 레포지토리에 들어있는 서버와 클라이언트 소스를 통해 배포를 진행할 예정이기 때문에, 소스 스테이지에서 소스 공급자를 GitHub(버전 2)를 선택하고 하단에 GitHub에 연결을 클릭한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682423524/gitio/post/deploy/3_ceq2tu.png)

연결 이름을 임의로 정하고 GitHub에 연결 버튼을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682423524/gitio/post/deploy/4_hgqjjc.png)

새 앱 설치를 누르고, 깃허브에 로그인 되어있는 본인의 계정을 선택하고 비밀번호를 입력한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682423524/gitio/post/deploy/6_yrdgdh.png)

설정 완료 후에 원래 화면으로 돌아오면 내 깃허브 유저 이름이 뜬다. 저걸 선택해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682423525/gitio/post/deploy/7_wpsyy4.png)

깃허브에 연결하고 나면 리포지토리 이름과 브랜치 이름이 뜨는데, 본인의 상황에 알맞게 선택해주고, 다음을 눌러준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682423525/gitio/post/deploy/8_h7ruyq.png)

다음으로 빌드 스테이지는 순서 1번에서 작성하였던 yaml 파일을 읽어 npm 모듈 설치와 빌드를 지정해둔 경로에서 자동으로 진행한다.

빌드 스테이지의 공급자를 CodeBuild로 지정하고, 프로젝트 생성 버튼을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682423525/gitio/post/deploy/10_jmprqs.png)

프로젝트 생성 버튼을 누르고 프로젝트 이름을 지정한 다음 밑으로 내려보면 환경 부분이 있는데, 운영 체제를 본인에 맞게 선택해준다. 

나의 경우 Ubuntu를 사용하기 때문에 우분투를 선택했다. 런타임은 Standard, 이미지는 5.0 버전을 선택해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682423525/gitio/post/deploy/11_gwakue.png)

다음은 배포 스테이지이다. 배포 스테이지를 통해 지정한 경로에 빌드된 빌드 폴더 내부의 내용을 S3 버킷에 자동으로 업로드/업데이트 해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682423525/gitio/post/deploy/12_g4t47w.png)

<br>

### **3. EC2 인스턴스에 태그와 역할 부여**
---

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682427067/gitio/post/deploy/14_oq6zxy.png)

EC2 메뉴로 가서 생성해둔 EC2를 선택한 후 우측 상단에 ```작업 → 인스턴스 설정 → 태그 관리``` 로 이동한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682427067/gitio/post/deploy/15_qu7ser.png)

태그는 사용자 또는 그룹에 액세스 권한을 부여하려는 인스턴스에 특정 태그를 추가하여 특정 태그가 있는 모든 인스턴스에 대한 액세스를 허용하는 IAM 정책을 생성한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682427067/gitio/post/deploy/16_qkywar.png)

다음으로 ``작업 → 보안 → IAM 역할 수정`` 으로 이동한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682427067/gitio/post/deploy/17_gjmulx.png)

새 IAM 역할을 생성한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682427067/gitio/post/deploy/18_zw1j11.png)

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682427067/gitio/post/deploy/19_ajzcta.png)

다음 화면에서 역할 만들기를 클릭하고, 신뢰할 수 있는 엔터티 유형에서 AWS 서비스와 사용 사례에서 EC2를 선택한 후 다음으로 이동한다. 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682427067/gitio/post/deploy/20_laz0zs.png)

역할을 위에 보이는 세 가지를 검색해서 선택한 후 다음으로 이동한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682427068/gitio/post/deploy/21_un7nsk.png)

역할 이름을 부여한 후 생성을 해줄건데, 생성 후 신뢰 관계를 수정해줄 것이다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682427068/gitio/post/deploy/22_n4cyd1.png)

신뢰 관계 수정은 생성한 역할을 클릭하여 신뢰 관계 탭으로 가서 신뢰 정책 편집을 눌러준다.

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Principal": {
				"Service": ["ec2.amazonaws.com",
				"codedeploy.ap-northeast-2.amazonaws.com"]
			},
			"Action": "sts:AssumeRole"
		}
	]
}
```

위와 같이 신뢰 관계 내용 중 "Service" 부분의 내용을 배열 형태로 바꾸어준 후, "codedeploy.ap-northeast-2.amazonaws.com"를 추가해주고 저장한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682427068/gitio/post/deploy/23_fghpnr.png)

신뢰 관계 편집이 끝나면 IAM 역할 수정으로 돌아서 방금 생성한 역할을 선택한다.

<br>

### **4. 인스턴스 보안그룹 수정**
---

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682427068/gitio/post/deploy/24_xhjeka.png)

인스턴스 보안 그룹에 "launch-wizard-1"로 들어가서 하단의 인바운드 규칙 편집을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682427068/gitio/post/deploy/25_ahh2ng.png)

HTTP(80), HTTPS(443)의 IPv4, IPv6의 인바운드를 모두 허용해준다.

<br>

### **5. 서버 배포 파이프라인 구축**
---

최상위 디렉토리에 ```appspec.yml``` 파일을 생성한다. 이 파일을 배포 자동화를 도와주는 CodeDeploy-Agent가 인식하는 파일이다.

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ubuntu/sprint-practice-deploy-for04

hooks:
  ApplicationStop:
    - location: scripts/stop.sh
      runas: root
  AfterInstall:
    - location: scripts/initialize.sh
      runas: root
  ApplicationStart:
    - location: scripts/start.sh
      runas: root
```

최상위 디렉토리에 ```scripts``` 디렉토리를 생성한 후, 다음 코드들을 추가해준다.

**>> initialize.sh**
```shell
#!/bin/bash
cd /home/ubuntu/sprint-practice-deploy-for04/server
npm install
npm install pm2@latest -g
sudo apt-get update
sudo apt-get install authbind
sudo touch /etc/authbind/byport/80
sudo chown ubuntu /etc/authbind/byport/80
sudo chmod 755 /etc/authbind/byport/80
```

**>> start.sh**
```shell
#!/bin/bash
cd /home/ubuntu/sprint-practice-deploy-for04/server

export DATABASE_USER=$(aws ssm get-parameters --region ap-northeast-2 --names DATABASE_USER --query Parameters[0].Value | sed 's/"//g')
export DATABASE_PASSWORD=$(aws ssm get-parameters --region ap-northeast-2 --names DATABASE_PASSWORD --query Parameters[0].Value | sed 's/"//g')
export DATABASE_PORT=$(aws ssm get-parameters --region ap-northeast-2 --names DATABASE_PORT --query Parameters[0].Value | sed 's/"//g')
export DATABASE_HOST=$(aws ssm get-parameters --region ap-northeast-2 --names DATABASE_HOST --query Parameters[0].Value | sed 's/"//g')

authbind --deep pm2 start app.js
```

**>> stop.sh**
```shell
#!/bin/bash
cd /home/ubuntu/sprint-practice-deploy-for04/server
pm2 stop app.js 2> /dev/null || true
pm2 delete app.js 2> /dev/null || true
```

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432961/gitio/post/deploy/26_frme9b.png)

위 쉘 파일들을 생성한 후, CodeDeploy의 애플리케이션 메뉴로 이동하여 애플리케이션 생성을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432961/gitio/post/deploy/27_fktw91.png)

애플리케이션 이름과 플랫폼을 EC2/온프레미스로 설정해주고 애플리케이션을 생성한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432962/gitio/post/deploy/28_ttvvi5.png)

생성한 애플리케이션을 클릭해서 하단에 배포 그룹 생성을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432961/gitio/post/deploy/29_lhqklu.png)

배포 그룹의 이름과 서비스 역할에 아까 생성한 IAM 역할을 선택한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432962/gitio/post/deploy/30_iw1cq9.png)

아래로 내려 환경 구성에 Amazon EC2 인스턴스를 선택하고 아까 생성한 태그를 선택한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432961/gitio/post/deploy/31_e0if3d.png)

마지막으로 로드 밸런서 체크를 해제하여 비활성 해주고 그룹을 생성한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432962/gitio/post/deploy/32_dv2jkb.png)

이제 서버 배포 파이프라인을 생성해보자. CodePipeline 메뉴로 이동하여 파이프라인 이름을 설정하고 다음으로 넘어간다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432962/gitio/post/deploy/33_r0jolw.png)

소스 스테이지에 아까 클라이언트 배포 파이프라인에서 했던 것과 똑같이 설정을 해준다. (체크한 부분 확인)

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432962/gitio/post/deploy/34_gqqvdb.png)

다음으로 뜨는 빌드 스테이지는 서버 배포이기 때문에 빌드가 필요 없다. 넘기자.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432962/gitio/post/deploy/35_cd1cir.png)

마지막 배포 스테이지에서 배포 공급자를 CodeDeploy를 선택하고 위에서 생성한 애플리케이션과 배포 그룹을 선택해준다. (나는 이미 했던 실습을 정리하는 과정이기 때문에 위에서 예시로 생성했던 이름들과 다를 수 있다. 참고 바람)

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432962/gitio/post/deploy/36_r63hpu.png)

파이프 라인 생성!

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432962/gitio/post/deploy/38_jkny3y.png)

다음으로 CodeBuild 메뉴로 이동하여 아까 생성했던 빌드 프로젝트를 선택한 후 편집에 환경을 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682432963/gitio/post/deploy/39_ux7n3x.png)

아래로 내려보면 추가 구성 부분이 있는데, 추가 구성을 열어 환경 변수를 지정해준다. 서버를 연결해줄 ec2의 DNS를 적어주고 저장한다. 
  
유의사항 : 주소의 앞에 'http://'를 적어주도록 하고, 주소의 끝에 '/' 가 붙어있지 않은지 확인하자.

<br>

### **6. 데이터베이스 연결**
---

RDS는 이전에 했던 블로그 내용을 따라 생성한다. 하나 바꿔야할 것은 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682434609/gitio/post/deploy/40_oefz1t.png)

사진에 EC2 컴퓨팅 리소스에 연결을 선택하여 생성되어 있는 EC2를 선택해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682434609/gitio/post/deploy/41_ogtmbv.png)

다음으로 Parameter Store 메뉴로 이동하여 파라미터 생성을 누른다. 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682434610/gitio/post/deploy/42_qpyu5s.png)

이름은 환경변수 이름으로 지정해주고, 값에 환경 변수에 맞은 내용을 넣어주면 된다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682434609/gitio/post/deploy/43_p8pvmo.png)

나는 다음과 같이 생성하였다.
* DATABASE_HOST= RDS 엔드포인트
* DATABASE_PASSWORD= 호스트 암호
* DATABASE_PORT= RDS 포트 번호
* DATABASE_USER= 호스트 이름

위 내용을 마치고 EC2 인스턴스 내부에 AWS CLI를 설치해준다.

```bash
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ unzip awscliv2.zip
$ sudo ./aws/install
```

위 명령을 실행하여 AWS CLI를 설치하고 나면 다음과 같이 AWS 버전을 확인할 수 있다.

```bash
$ aws --version
aws-cli/2.11.15 Python/3.11.3 Linux/5.15.0-1033-aws exe/x86_64.ubuntu.20 prompt/off
```

이 과정을 모두 끝내고 GitHub에 수정된 내용을 모두 push 해주면 자동으로 파이프라인에서 연결해둔 GitHub 레포지토리의 업데이트 된 소스를 받아와 Build, Deploy를 자동으로 진행한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682435042/gitio/post/deploy/44_ppjoqd.png)

빌드, 배포가 끝난 후 초록색 체크 표시가 뜨면 배포한 서버와 클라이언트를 실행해보고 결과를 확인해보자.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682435136/gitio/post/deploy/46_b4bfaa.png)

서버 돌아가구요~~~~~~~~

![](https://res.cloudinary.com/duruq2coh/image/upload/v1682435136/gitio/post/deploy/45_lcg8sr.png)

클라이언트에서 로그인과 데이터베이스 연결까지 성공 ~~~~~~~

<br>

### ***끝 입니다.***