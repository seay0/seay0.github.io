---
date: 2023-05-12 00:00:00
layout: post
title: Terraform? / TIL
subtitle: 'Terraform? / 실습'
description: Terraform? / 실습
image: https://res.cloudinary.com/duruq2coh/image/upload/v1683876865/gitio/dev-jeans_5_uqzbot.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1683876865/gitio/dev-jeans_5_uqzbot.png
category: IaC
tags:
  - Terraform
  - Intrastructure as Code
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **Terraform** 
---

IaC 도구를 사용하면 GUI를 통하지 않고 구성 파일을 사용하여 를 안전하고 일관되며 반복 가능한 방식으로 빌드, 변경 및 관리할 수 있게 해준다.

Terraform은 IaC 도구이며, 수동 구성이 아닌 코드를 통해 컴퓨터 시스템과 리소스를 생성하고 관리할 수 있다. 이 도구를 사용하면 사람이 읽기 쉬운 선언적 구성 파일에서 리소스와 인프라를 정의할 수 있으며 인프라의 라이프사이클을 관리할 수 있다.

Terraform은 구조화되고 재현 가능한 방식으로 인프라를 정의하고 배포하는 방법과 **인프라 관리를 위한 강력한 도구**가 되는 몇 가지 주요 기능을 제공한다.

1. **선언적 구문**을 지원하여 사용자가 원하는 인프라 상태를 정의하고 테라폼이 해당 상태를 달성하는 데 필요한 작업을 처리하도록 한다.
2. 다양한 공급자를 지원하여 **다양한 클라우드 플랫폼 및 서비스에서 리소스를 관리**할 수 있다.
3. 인프라 코드의 **모듈화 및 재사용을 허용**하여 복잡한 시스템을 보다 쉽게 유지 관리하고 확장할 수 있다.
4. 종속성 관리와 변경 사항을 적용하기 전에 계획하고 미리 볼 수 있는 기능을 제공하여 제어되고 **예측 가능한 배포**를 보장한다.

<br>

---

### **필요성**

* 최신 인프라 관리와 관련하여 시스템이 더욱 복잡해지고 동적으로 변하면서 수동 구성 및 관리는 시간이 많이 걸리고 오류가 발생하기 쉬우며 확장하기 어려워진다. 
* 테라폼은 인프라 프로비저닝 및 관리에 대한 일관되고 자동화된 접근 방식을 제공하여 이러한 문제를 해결한다. 
* 조직이 인프라를 효율적으로 배포 및 업데이트하고, 환경 전체에서 일관성을 유지하고, 인프라 변경에 대해 효과적으로 협업할 수 있도록 지원한다.

<br>

### **장점**

* 인프라를 코드 방식으로 촉진하여 공동 작업, 버전 제어 및 반복 가능성을 개선한다. 
* 인프라를 더 빠르고 안정적으로 프로비저닝할 수 있으므로 수동 구성에 필요한 시간과 노력이 줄어든다. 
* 테라폼의 변경 계획 및 미리보기 기능은 잠재적인 문제를 식별하고 비용이 많이 드는 실수를 방지하는 데 도움이 된다.

<br>


### **Q. Terraform의 선언적 방식으로 작성된 코드는 항상 인프라의 최신 상태를 의미한다. Terraform은 어떤 방식으로 인프라를 최신 상태로 유지할 수 있는 걸까?**
---

**A.**  

Terraform은 인프라를 **최신 상태로 유지**하며 **변경 사항을 추적하고 관리**하므로 구성 파일에 정의된 원하는 상태를 달성하는 데 필요한 리소스를 쉽게 생성, 업데이트 또는 삭제할 수 있다. 이 **선언적 접근 방식**은 인프라가 항상 원하는 구성과 일치하도록 하여 구성 드리프트의 위험을 줄이고 시간이 지남에 따라 인프라를 보다 쉽게 ​​유지 관리할 수 있도록 한다.

* **선언적 구성**  
Terraform을 사용하면 선언적 구성 파일을 사용하여 인프라의 원하는 상태를 정의한다. 이러한 구성 파일은 원하는 리소스, 해당 속성, 관계 및 종속성을 설명한다.

* **리소스 종속성 그래프**  
Terraform은 구성 파일을 분석하고 정의된 리소스 관계를 기반으로 종속성 그래프를 작성한다. 그래프는 원하는 상태에 도달하기 위해 리소스를 생성, 업데이트 또는 삭제해야 하는 순서를 나타낸다.

* **계획 및 적용**  
'terraform plan'을 실행하면 Terraform이 인프라의 현재 상태를 구성 파일에 정의된 원하는 상태와 비교한다. 원하는 상태에 도달하기 위해 필요한 변경 사항을 결정하고 취할 조치를 강조하는 계획을 제시한다.

* **멱등적 실행**  
Terraform은 멱등적 방식 즉,인프라를 원하는 상태로 만드는 데 필요한 변경만 수행한다. 리소스가 이미 원하는 상태인 경우 Terraform은 리소스를 변경하지 않는다.

* **버전 제어 통합**  
Terraform 구성 파일은 Git과 같은 버전 제어 시스템에 저장할 수 있다. 이를 통해 팀은 협업하고, 변경 사항을 검토하고, 기록을 추적하고, 필요에 따라 이전 버전의 인프라 구성으로 롤백할 수 있다.

* **지속적인 배포**  
Terraform은 지속적 통합/지속적인 배포(CI/CD) 파이프라인과 잘 통합된다. CI/CD 프로세스의 일부로 Terraform을 실행하면 구성 파일이나 인프라 코드가 업데이트될 때마다 인프라 변경 사항을 자동으로 적용할 수 있다.

<br>

### **Terraform 기본 설정**
---

Terraform을 구성하기 앞서 기본 설정부터 하도록 하자.

1. Terraform AWS 공급자를 인증하기 위한 AWS Configure 
2. AWS CLI 설치
3. Terraform CLI 설치

```bash
# Terraform CLI Install

$ sudo apt-get update
```

```bash
$ sudo apt-get install -y gnupg software-properties-common
```

```bash
$ wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

```bash
$ gpg --no-default-keyring \
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
--fingerprint
```
```bash
$ echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list
```

```bash
$ sudo apt-get update
```

```bash
$ sudo apt-get install terraform
```

Terraform CLI 설치가 완료되면 다음 명령을 이용해 사용 가능한 옵션에 대해 자세히 알아볼 수 있다.

```bash
$ terraform -help plan
```

다음 명령을 이용하여 Terraform CLI 도구에 대한 명령 자동 완성을 사용할 수 있다.

```bash
$ touch ~/.bashrc
$ terraform -install-autocomplete
```

<br>

### **Terraform 명령어**
---

```bash
$ terraform plan
```

* terraform plan 명령은 Terraform에 대한 실행 계획을 만드는 데 사용되며, 현재 Terraform 구성을 분석하고 배포된 리소스 또는 인프라와 비교한다.
* 리소스 생성, 수정, 소멸 등 변경 사항을 적용할 때 Terraform이 수행할 작업을 보여준다.
* 'terraform plan'을 실행하면 변경 사항을 실제로 적용하지 않고도 Terraform이 수행할 변경 사항을 검토할 수 있다. 
* 인프라를 수정하기 전에 잠재적인 문제나 의도하지 않은 변경을 식별하는 데 도움이 된다.

<br>

```bash
$ terraform init
```

* terraform init 명령은 Terraform 작업 디렉터리를 초기화하는 데 사용된다.
* 일반적으로 새 Terraform 프로젝트 또는 새 Terraform 구성으로 작업할 때 실행하는 첫 번째 명령이다.
* 백엔드를 초기화하고 필요한 공급자 플러그인을 다운로드하며 Terraform이 인프라를 관리할 수 있는 환경을 설정한다.
* main.tf 파일을 포함한 구성 파일을 읽고 원격 소스에서 필요한 공급자 종속성을 가져온다.
* 다른 Terraform 명령을 사용하기 전에 프로젝트가 올바르게 설정되고 사용할 준비가 되었는지 확인하는 중요한 단계이다.

<br>

```bash
$ terraform apply
```

* 'terraform apply' 명령은 Terraform 구성에 정의된 변경 사항을 대상 인프라에 적용하는 데 사용된다.
* terraform plan에서 생성된 실행 계획을 검토한 후 terraform apply를 실행하여 계획된 변경 사항을 실행할 수 있다.
* 인프라를 수정하기 전에 확인하라는 메시지가 표시된다.
확인이 되면 'terraform apply'는 필요에 따라 리소스를 생성, 수정 또는 삭제하여 실제 인프라를 Terraform 구성에 정의된 원하는 상태에 맞춘다.
* Terraform 구성 파일 및 공급자 플러그인을 기반으로 인프라를 프로비저닝하고 구성한다.
* 변경 사항이 적용될 때, 관리되는 리소스의 수와 복잡한 정도에 따라 시간이 오래 걸릴 수도 있다.

<br>

```bash
$ terraform destroy
```

* Terraform은 구성 파일(예: main.tf)을 읽고 이전에 Terraform을 사용하여 생성 또는 수정된 리소스를 식별한다.
* Terraform은 기본 클라우드 공급자(예: AWS, Azure, GCP)와 통신하여 정의된 구성과 일치하는 리소스를 삭제한다.
* Terraform 구성에 지정된 리소스가 클라우드 공급자의 인프라에서 제거된다. 여기에는 가상 머신, 네트워크, 스토리지 및 Terraform에서 관리하는 기타 리소스가 포함될 수 있다.
* "terraform destroy"는 강력한 명령이며 데이터 또는 구성이 손실될 수 있고, 리소스를 영구적으로 삭제한다는 점에 유의해야 한다. 
*  "terraform plan"에 의해 생성된 실행 계획을 검토하고 "terraform destroy"를 진행하기 전에 자원의 파괴 여부를 확인하는 것이 좋다.

<br>

### **Terraform 구성**
---

**1. Terraform**
```tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}

provider "aws" {
  region = "ap-northeast-2"
}

# Define variables
variable "server_port" {
  description = "Port for the web server"
  default     = 80
}
```
terraform 행은 필요한 공급자와 사용할 Terraform 버전을 구성하는 데 사용된다. required_providers 행은  "aws" 공급자가 필요하고, 공급자의 소스가 "hashicorp/aws"이며 버전이 4.16이어야 함을 지정한다. 즉, 테라폼은 지정된 공급자를 사용하여 인프라의 AWS 리소스와 상호작용 한다.

required_version 행은 이 구성을 실행하는 데 필요한 최소 버전이 1.2.0 이상이어야 함을 지정한다.

provicers 행은 특정 공급자("aws")를 구성한다. 테라폼이 AWS 리소스와 상호 작용할 때 "ap-northeast-2" 리전을 기본값으로 사용한다.

variable 행은 테라폼에서 변수를 정의하는 데 사용된다. "server_port"라는 변수로 웹 서버의 포트 번호를 정의한다. description은 변수의 용도 설명이고, default는 변수의 기본 값이다. 즉 server_port라는 변수에 기본값으로 80포트를 넣어준 것이다.

<br>

**2. EC2**
```tf
# Create EC2
resource "aws_instance" "app_server" {
  ami           = "ami-04cebc8d6c4f297a3"
  instance_type = "t2.micro"

  user_data = <<-EOF
    #!/bin/bash
    echo "Hello, World" > index.html
    nohup busybox httpd -f -p ${var.server_port} &
    EOF

  key_name = "devops04-seay0"
  tags = {
    Name = "full_stack_application"
  }
}
```
resource는 Terraform에서 리소스를 정의하는 데에 사용된다.

이 행은 EC2를 구성하는 행이며, 리소스 블럭 이름은 "aws_instance"이고 로컬 이름은 "app_server"이다. 이 이름은 테라폼 구성 내에서 이 리소스를 참조하거나 상호 작용하는 데 사용할 수 있다.

* **ami** : 인스턴스에 사용할 Amazon Machine Image(AMI) ID를 지정한다.
* **instance_type** : 인스턴스 유형을 지정한다.
* **user_data** : 인스턴스가 시작될 때 실행할 초기화 스크립트 또는 명령을 제공할 수 있다.  

  => bash 스크립트는 heredoc(EOF 사이에 포함됨)로 제공된다. 이 스크립트는 "Hello, World" 텍스트가 있는 "index.html" 파일을 만들고 "busybox httpd" 명령을 사용해 웹 서버를 시작한다. 

  ```${var.server_port}``` 값은 앞에서 정의한 "server_port" 변수의 값을 동적으로 주입하는 데에 사용이 된다.

* **key_name** : 인스턴스에 액세스하는 데 사용할 SSH 키 쌍의 이름을 지정한다.
* **tags** : 메타데이터 태그를 인스턴스에 연결할 수 있다. "Name" 키와 "full_stack_application" 값이 있는 단일 태그를 지정한다.


<br>

**3. VPC**
```tf
# Create VPC
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "my_vpc"
  }
}
```
"aws_vpc"라는 이름의 리소스를 정의하고 있으며, 로컬 이름은 "my_vpc"이다. 

* **cidr_block** : VPC의 IP 주소 범위를 정의한다. 이 경우 IP 주소 범위는 10.0.0.0 ~ 10.0.255.255 이다.

<br>

**4. Security Group**
```tf
# Create Public Security Group
resource "aws_security_group" "public_sg" {
  vpc_id      = aws_vpc.my_vpc.id
  name        = "public_sg"
  description = "Security group for public web server"

  tags = {
    Name = "public_sg"
  }
}

# Create Private Security Group
resource "aws_security_group" "private_sg" {
  vpc_id      = aws_vpc.my_vpc.id
  name        = "private_sg"
  description = "Security group for private web server"

  tags = {
    Name = "private_sg"
  }
}
```
두 개(Public, Private)의 AWS 보안 그룹을 생성하기 위한 리소스 블록이다.

* **vpc_id** : 이 보안 그룹이 속한 VPC의 ID를 지정한다. 이전에 생성된 VPC 리소스의 id 속성을 참조하여 이 보안 그룹이 해당 VPC와 연결됨을 나타낸다.
* **name** : 보안 그룹의 이름을 지정한다.
* **description** : 보안 그룹에 대한 설명을 제공한다.

<br>

**5. Subnets**
```tf
# Create Public Subnets
resource "aws_subnet" "public_subnet" {
  vpc_id = aws_vpc.my_vpc.id
  cidr_block = "10.0.0.0/24"
  availability_zone = "ap-northeast-2c"
  tags = {
    Name = "public_subnet"
  }
}

# Create Private Subnets
resource "aws_subnet" "private_subnet" {
  vpc_id = aws_vpc.my_vpc.id
  cidr_block = "10.0.1.0/24"
  availability_zone = "ap-northeast-2c"
  tags = {
    Name = "private_subnet"
  }                
}
```
AWS VPC 내에 두 개(Public, Private)의 서브넷을 생성하기 위한 리소스 블록이다.

* **cidr_block** : 10.0.0.0 ~ 10.0.0.255 범위의 IP 주소를 가질 수 있다.
* **avaliability_zone** : 서브넷이 생성될 가용 영역을 지정한다.

<br>

**6. IGW / Route Table**
```tf
# Create Internet Gateway
resource "aws_internet_gateway" "my_igw" {
  vpc_id = aws_vpc.my_vpc.id

  tags = {
    Name = "my_igw"
  }
}

# Create Public Route Table
resource "aws_route_table" "public_route_table" {
  vpc_id = aws_vpc.my_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.my_igw.id
  }
  tags = {
    Name = "public_route_table"
  }
}

# Create Private Route Table
resource "aws_route_table" "private_route_table" {
  vpc_id = aws_vpc.my_vpc.id

  tags = {
    Name = "private_route_table"
  }
}

# Connect Public Route Table
resource "aws_route_table_association" "publicRTAssociation" {
  subnet_id = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_route_table.id
}

# Connect Private Route Table
resource "aws_route_table_association" "privateRTAssociation" {
  subnet_id = aws_subnet.private_subnet.id
  route_table_id = aws_route_table.private_route_table.id
}
```
AWS Internet Gateway와 라우팅 테이블을 생성하고 이를 AWS VPC의 서브넷과 연결하기 위한 리소스 블록이다.

* "**aws_route_table"** : 모든 인터넷 바운드 트래픽('0.0.0.0/0')을 지정된 IGW로 보내는 단일 경로가 있는 공용 경로 라우팅 테이블을 생성한다. 
* **"aws_route_table_association"** : VPC 서브넷과 연결을 설정하여 서브넷이 라우팅 결정에 라우팅 테이블을 사용하도록 한다.
* 생성과 연결이 함께 작동하여 해당 서브넷 내의 리소스가 지정된 IGW를 통해 인터넷과 통신할 수 있도록 한다.

<br>

* **gateway_id** : 이 경로의 대상으로 사용될 IGW의 ID를 지정한다.
* **subnet_id** : 라우팅 테이블과 연결될 서브넷의 ID를 지정한다.
* **route_table_id** : 서브넷이 연결될 라우팅 테이블의 ID를 지정한다.

<br>

**7. Security Group Rule**
```tf
# Create Public SG Rule
resource "aws_security_group_rule" "publicSGRulesHTTPingress" {
  type = "ingress"
  from_port = 80
  to_port = 80
  protocol = "TCP"
  cidr_blocks = [ "0.0.0.0/0" ]
  security_group_id = aws_security_group.public_sg.id
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group_rule" "publicSGRulesHTTPegress" {
  type = "egress"
  from_port = 80
  to_port = 80
  protocol = "TCP"
  cidr_blocks = [ "0.0.0.0/0" ]
  security_group_id = aws_security_group.public_sg.id
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group_rule" "publicSGRulesHTTPSingress" {
  type = "ingress"
  from_port = 443
  to_port = 443
  protocol = "TCP"
  cidr_blocks = [ "0.0.0.0/0" ]
  security_group_id = aws_security_group.public_sg.id
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group_rule" "publicSGRulesHTTPSegress" {
  type = "egress"
  from_port = 443
  to_port = 443
  protocol = "TCP"
  cidr_blocks = [ "0.0.0.0/0" ]
  security_group_id = aws_security_group.public_sg.id
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group_rule" "publicSGRulesSSHingress" {
  type = "ingress"
  from_port = 22
  to_port = 22
  protocol = "TCP"
  cidr_blocks = [ "0.0.0.0/32" ]
  security_group_id = aws_security_group.public_sg.id
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group_rule" "publicSGRulesSSHegress" {
  type = "egress"
  from_port = 22
  to_port = 22
  protocol = "TCP"
  cidr_blocks = [ "0.0.0.0/32" ]
  security_group_id = aws_security_group.public_sg.id
  lifecycle {
    create_before_destroy = true
  }
}

# Create Private SG rules
resource "aws_security_group_rule" "privateSGRulesRDSingress" {
  type = "ingress"
  from_port = 3306
  to_port = 3306
  protocol = "TCP"
  security_group_id = aws_security_group.private_sg.id
  source_security_group_id = aws_security_group.public_sg.id
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group_rule" "privateSGRulesRDSegress" {
  type = "egress"
  from_port = 3306
  to_port = 3306
  protocol = "TCP"
  security_group_id = aws_security_group.private_sg.id
  source_security_group_id = aws_security_group.public_sg.id
  lifecycle {
    create_before_destroy = true
  }
}
```
보안 그룹의 지정된 포트에 대해 수신 트래픽을 허용하는 규칙을 생성하기 위한 리소스 블록이다.

* **type** : 이 규칙이 수신 트래픽을 의미하는지 송신 트래픽을 의미하는지 나타낸다.
* **from_port / to_port** : 이 규칙이 어떤 포트에서의 트래픽을 허용하는지 나타낸다.
* **security_group_id** : 이 규칙이 적용될 보안 그룹의 ID를 지정한다. 
* **lifecycle** : 이 리소스에 대한 수명 주기 구성을 지정하는 데 사용된다. 
* **create_before_destroy** : 모두 true로 설정되어 테라폼이 기존 보안 그룹 규칙을 삭제하기 전에 새 보안 그룹 규칙을 생성함을 의미한다. 이렇게 하면 규칙을 업데이트할 때 원활한 전환이 보장된다.

