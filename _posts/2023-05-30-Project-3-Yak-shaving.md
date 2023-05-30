---
date: 2023-05-30 00:00:00
layout: post
title: 3rd Project memoir & Yak shaving / TIL
subtitle: 'memoir & Yak shaving'
description: memoir & Yak shaving
image: https://res.cloudinary.com/duruq2coh/image/upload/v1684374884/gitio/Kubernetes_ahpltn.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1684374884/gitio/Kubernetes_ahpltn.png
category: Project
tags:
  - Yak shaving
  - DevOps BootCamp
author: seay0
paginate: false
---

### **시나리오**
---

<도넛-스테이츠>는 온라인으로 도넛을 판매합니다.
웹사이트를 통해서 주문 버튼을 누르는 것으로 구매(Sales API)가 가능합니다.
창고에 재고가 있다면 재고가 감소하고 구매가 완료됩니다.
유튜브스타 hoyong.LEE가 도넛-스테이츠의 도넛이 맛있다고 영상을 올렸습니다.
그를 따르는 데브옵스 수강생들이 몰려듭니다. 주문이 급등합니다.
창고에 재고가 없기 때문에 구매가 불가능한 경우가 발생합니다.
창고의 도넛의 재고가 다 떨어지면 제조 공장에 알려서 다시 창고를 채우는 시스템을 구축해야 합니다.
제조 공장인 <팩토리-스테이츠>에 주문을 요청(Leagcy Factory API)할 수 있습니다.
주문이 요청되면 일정 시간이 지난 후 창고에 재고가 증가합니다.

<br>

### **상황 / 요구 사항**  
---

비효율적인 레거시 시스템 때문에 고객의 불만사항이 접수되고 있습니다.
제품별 재고부족 요청이 빈번하게 발생되고 있지만 전달 과정에서 지연과 누락 등 문제 상황이 발생하고 있습니다.
안정적으로 요청이 전달될 수 있도록 시스템을 개선해야 합니다.
비정상적으로 처리된 요청의 경우 운영팀에 상황을 알려야 합니다.

**요구사항 1 : 재고부족으로 인한 구매실패에 대한 조치**  
1. Sales API를 통해 요청을 받은 서버가 데이터베이스에서 재고 상황을 확인합니다.
2. 재고가 있다면 감소시키고 응답으로 판매완료 내용을 전달합니다.
3. 재고가 없는 경우 공장에 주문을 진행합니다
4. 재고가 없다는 내용을 담은 메세지 페이로드와 함께 SNS topic이 생성됩니다.
5. 메시지가 SQS로 구현된 stock_queue에 수신됩니다.

**요구사항 2 : 메시지 누락 상황에 대한 조치**
1. 빈번한 요청으로 메시지 누락이 발생합니다.
2. SQS에서 처리완료되지 않은 메시지들을 체계적으로 관리할 dead_letter_queue를 생성해야합니다.
3. stock_queue와 dead_letter_queue를 연결합니다.

**요구사항 3 : Legacy 시스템(Factory → Warehouse) 성능문제에 대한 조치**
1. 안정적으로 이벤트가 전달될 수 있는 시스템을 구축해야 합니다.
2. stock_queue 의 메시지를 소비하는 stock_lambda에 의해 Factory API가 호출됩니다.
3. Factory의 생산 완료 메시지를 수신한 stock_inc_lambda가 DB에 상품 재고를 증가시킵니다.

---
**구성 요소**

1. Sales API (Sales API를 통해 백엔드에 요청)
2. Factory API (재고가 부족할 경우 Factory API를 통해 재고 확보 요청)
3. 프론트엔드(웹사이트) : cURL / postman / k6 등을 통한 API 호출로만 구현  
4. 백엔드(서버) : 구매 시 창고에서 재고 확인 후 재고 감소 로직 구현
5. 데이터베이스(창고) : RDS에 mysql db 구성  
요청에 따른 재고 상태 변경

<br>

### **Project 시작**
---

**프로젝트 시작 전 처음 그렸던 다이어그램**

![](https://res.cloudinary.com/duruq2coh/image/upload/v1685411605/gitio/post/team/3/diagram1_guqvyr.png)


**요구사항 구현을 마친 후 그린 다이어그램**

![](https://res.cloudinary.com/duruq2coh/image/upload/v1685411606/gitio/post/team/3/diagram2_fdsi1x.png)

사용자가 API Gateway를 통해 구매 요청을 보내면 Sales api Lambda는 RDS에 재고 확인 후 SQS를 통해 상황에 맞는 메시지를 생성하고 처리한다. 

SQS는 SNS를 구독하여 SNS Topic으로 메시지를 전달할 수 있도록 두 서비스 간에 연결을 설정한다. 

**SNS - SQS 작동 방식**
* 메시지는 버퍼 또는 메시지 보관 영역 역할을 하는 SQS 대기열로 전송된다.
* SNS Topic은 메시지가 구독된 끝점에 게시되고 전달되는 논리적 액세스 지점이다.
* SNS Topic에 대한 구독자로 SQS 대기열을 구성할 수 있다. 즉, 메시지가 SNS 주제에 게시될 때마다 자동으로 구독한 SQS 대기열로 전달된다.
* 메시지가 SNS Topic에 게시되면 SNS는 구독한 SQS 대기열로 전달을 트리거한다. 메시지는 SQS 대기열에 저장되고 소비자 애플리케이션에서 검색할 수 있게 된다. 

처리 받지 못한 메시지는 DLQ로 이동하여 보관된다. DLQ로 이동한 메시지의 오류를 분석하여 개선하거나 수정 조치를 취할 수 있다. 

SQS는 Stock Lambda에 재고 부족 시 메시지를 보낸다. Stock Lambda는 payload에 필요한 정보와 채울 재고의 수 등을 담아 Factory API에 재고 생산을 요청한다.

요청을 받은 Factory API는 물품 생산 후 Increase Lambda에 재고를 추가하여 RDS의 재고 데이터를 증가시킨다.

<br>

### **Issue**
---

* SNS에서 SQS를 구독해서 메시지가 저장되지 않았었다. 반대로 SQS에서 SNS를 구독해서 메시지가 정상적으로 대기열에 추가되고 저장되었다.

* stock-increase-lambda에서 조건을 실패 시로 하여 SQS에 메시지가 저장되지 않았는데, 성공 시로 수정하여 문제가 해결되었다.

* RDS 연결 과정에서 기존에 환경변수로 설정되었던 것들의 이름이 RDS USERNAME과 겹쳐서 연결이 정상적으로 되지 않았었다. database.js 코드와 Lambda내의 환경 변수 이름을 다르게 설정(ex> DB_USERNAME)하여 문제가 해결되었다.

* Lambda에 트리거로 SQS 연결 시 권한 문제로 연결이 되지 않아서 Lambda의 권한으로 SQSFullAccess 권한을 부여하여 정상적으로 연결할 수 있었다. 

* Factory API에 payload를 전송하여 서버에 전송한 내용이 표시되는 것은 확인했지만, 생산이 끝났음에도 불구하고 재고가 추가되지 않았다. 

  원인은 increase_lambda에 incremental 변수 설정이었다. 기존엔 req.body.stock으로 설정되어 있었는데, stock-lambda에서 설정한 변수는 MessageAttributeProductCnt 이기 때문에 req.body.MessageAttributeProductCnt로 설정해주어야 했다. 
  
  변수 설정을 바꾼 뒤 재고 증가가 되었다.

---
**번외 억까 Issue**

```bash
serverless : 'serverless' 용어가 cmdlet, 함수, 스크립트 파일 또는 실행할 수 있는 프로그램 이름으로 인식되지 않습니다. 이름이 정확한지 확 
인하고 경로가 포함된 경우 경로가 올바른지 검증한 다음 다시 시도하십시오.
위치 줄:1 문자:1
+ serverless
+ ~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (serverless:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
```

serverless 명령어가 잘 되다가 mysql을 설치하니까 갑자기 명령어가 먹통이었다. mysql 명령어도 안되고... 환경 변수 문제인 것 같았다.

```
C:\path\to\serverless
C:\Program Files\MySQL\MySQL Server\bin
```

위 환경 변수를 추가해주고 다시 실행해봤는데 역시 먹통이었다. git bash에서 해봐도 안되고, ```npm install -g serverless``` 로 재설치를 해봐도 계속 command not found였다 ㅠㅠ

npm이 문제인가 싶어서 npm 환경 변수를 추가해주었는데 해결이 되었다 ....

```
%USERPROFILE%\AppData\Roaming\npm
```

그치만 여전히 code 터미널에선 되지 않고 파워쉘을 관리자 권한으로 실행했을 때 된다. (vscode도 관리자 권한으로 실행 했음 ㅜㅜ) 

<br>

### **후기**
---

프로젝트 1과 비교하면 비교적 2와 3은 할만 했던 것 같다. 하지만 개인적으로 이번 프로젝트의 로직을 완벽히 이해하고 있다고 생각했는데, 다이어그램을 그려보면서 생각이 바뀌었다. 어디를 어디에 연결 해야하고, 어떤 것이 무슨 역할을 하는지 제대로 이해하고 있지 못한 부분이 많았다. 다이어그램으로 표현하고 시나리오를 정리하면서 보니 내가 생각했던 것과 다르고, 틀린 부분도 많았다. 앞으로도 무언가를 할 때 요구사항과 시나리오를 따로 정리해서 표현하는 작업을 무조건 해야겠다고 생각했다.