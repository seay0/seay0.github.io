---
date: 2023-05-10 00:00:00
layout: post
title: Serverless Application &#35;2 / TIL
subtitle: 'Serverless Application / Photobook'
description: Serverless Application / Photobook
image: https://res.cloudinary.com/duruq2coh/image/upload/v1683161283/gitio/msa_b5yogy.png
optimized_image: https://res.cloudinary.com/duruq2coh/image/upload/v1683161283/gitio/msa_b5yogy.png
category: Micro Services
tags:
  - Serverless
  - Lambda
  - Micro Services
  - DevOps BootCamp
author: seay0
paginate: false
---

> ## **Serverless Application** 
---

Lambda와 s3 버킷을 이용한 서버리스 사진첩을 만들어보았다.

```bash
$ sam init
```
sam init 명령어를 통해 기존 템플릿을 사용해보자.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067360/gitio/post/serverless/1_aiutgd.png)

AWS Quick Start Templates의 1번을 선택해주고,

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067360/gitio/post/serverless/2_rcdcwz.png)

템플릿 리스트 5번의 'Standalone function'의 runtime은 'nodejs14.x' 버전을 선택한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067360/gitio/post/serverless/3_chexnb.png)

CloudWatch라는 애플리케이션에서 모니터링을 할 것인지 여부를 'y'로 선택해주고, 프로젝트 이름만 원하는 대로 적은 후 생성해주면 내가 만든 sam application 폴더 하나가 생성된다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067360/gitio/post/serverless/4_zx724f.png)

```bash
$ cd photobook
$ code .
```

cd로 'photobook'으로 들어가서 code .으로 vscode에서 볼 수 있게 폴더를 열어보자.

```js
exports.helloFromLambdaHandler = async (event) => {
    console.log(event)
    const message = 'Hello from Lambda!';

    console.info(`${message}`);
    
    return message;
}
```

photobook/src/handlers/hello-from-lambda.js의 기본 코드에 다음과 같이 콘솔에서 event 로그를 볼 수 있게 코드를 추가한 후, 빌드, 배포를 해준다.

```bash
$ sam build
$ sam deploy --guided

Configuring SAM deploy
======================

        Looking for config file [samconfig.toml] :  Found
        Reading default arguments  :  Success

        Setting default arguments for 'sam deploy'
        =========================================
        Stack Name [photobook]:  
        AWS Region [ap-northeast-2]: 
        #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
        Confirm changes before deploy [Y/n]: n
        #SAM needs permission to be able to create roles to connect to the resources in your template
        Allow SAM CLI IAM role creation [Y/n]: 
        #Preserves the state of previously provisioned resources when an operation fails
        Disable rollback [y/N]: 
        Save arguments to configuration file [Y/n]: 
        SAM configuration file [samconfig.toml]: 
        SAM configuration environment [default]: 

...

Successfully created/updated stack - photobook in ap-northeast-2
```

'sam deploy --guided'를 실행하면 선택 항목들이 출력되는데, 위에서 선택한 그대로 선택 후 기다려보면 Successfully가 뜨면서 배포 성공을 알려준다. 


![](https://res.cloudinary.com/duruq2coh/image/upload/v1684068437/gitio/post/serverless/18_pavv97.png)

이제 Lambda의 함수로 들어가보면, 방금 배포한 함수가 photobook이라는 이름으로 생성되어 있는 것을 볼 수 있다.

이제 사진들을 업로드할 s3 버킷을 생성해보자. 

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684068847/gitio/post/serverless/20_k3tkby.png)

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684068847/gitio/post/serverless/19_o5taaa.png)

버킷을 생성할 때 위 사항을 체크하고 생성해주도록 한다.

이제 다시 람다로 돌아가서 방금 생성한 트리거를 s3 버킷으로 연결해준다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067361/gitio/post/serverless/5_fuwl6q.png)

트리거를 s3로 선택한 후 방금 생성한 버킷을 선택하고 접미사를 '.jpeg'로 적은 뒤 체크를 하고 추가를 누른다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067361/gitio/post/serverless/6_c1ju2x.png)

연결을 마친 후 s3 버킷에 .jpeg 확장자로 된 사진을 업로드해보자.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067361/gitio/post/serverless/7_pcauji.png)

사진을 업로드 한 후 로그가 잘 찍히는지 확인하기 위해 Lambda의 모니터링 탭에 CloudWatch Logs 보기를 클릭해 이동한다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067361/gitio/post/serverless/8_gd1lye.png)

이동하면 위와 같은 화면이 뜨는데, 로그 스트림에 사진 생성 시 찍힌 로그가 존재한다. 클릭해보면, 다음과 같이 로그를 확인할 수 있다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067361/gitio/post/serverless/10_mltsev.png)

이 내용은 이전에 hello-from-lambda.js에서 console.log로 적어주었던 event의 로그가 찍힌 것이다. Records 객체가 배열 형태로 s3 객체를 담고 있다.

```js
exports.helloFromLambdaHandler = async (event, context) => {
    console.log(event.Records[0].s3)
    const message = 'Hello from Lambda!';

    console.info(`${message}`);
    
    return message;
}
```

그렇다면 console.log에 위 처럼 코드를 추가해서 Records의 0번째 인덱스에서 s3라는 객체를 콘솔에 출력하도록 작성한 후 빌드, 배포(sam build, sam deploy)를 해보자.

배포를 마친 후 위에 했던 방법과 똑같이 s3 버킷에 사진을 다시 업로드하여 로그를 확인해본다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067361/gitio/post/serverless/12_ddhe7x.png)

위와 같이 로그가 찍혔다. 로그에서 s3 버킷의 이름과 object에 무엇이 업로드 되었는지를 나타내고 있다. 

```js
exports.helloFromLambdaHandler = async (event, context) => {
    const {bucket,object} = event.Records[0].s3
    console.log({bucket,object})
    const message = 'Hello from Lambda!';

    console.info(`${message}`);
    
    return message;
}
```

우리가 필요한 정보는 bucket과 object이기 때문에 그것만 따로 빼주는 코드를 작성해보았다.
위와 같이 작성하면 다음과 같이 bucket과 object만 로그로 출력된다.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067361/gitio/post/serverless/13_abwvmk.png)

이제 사진의 사이즈를 변경해서 저장할 s3 버킷을 새로 하나 생성해주자.

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067362/gitio/post/serverless/14_bwfgh3.png)

이름을 지정하고 밑에 버킷 선택에 이전에 만들었던 버킷을 선택하면 같은 옵션의 버킷을 만들 수 있다.

이제 본격적으로 사진의 사이즈를 변경해서 새로운 버킷에 저장하는 코드를 작성해보자.

```js
const AWS = require('aws-sdk')
const s3 = new AWS.S3()
const sharp = require('sharp')

exports.helloFromLambdaHandler = async (event, context) => {
    const {bucket,object} = event.Records[0].s3
    console.log({bucket,object})
    const s3Object = await s3.getObject({
        Bucket: bucket.name,
        Key: object.key
      }).promise()
      
      const data = await sharp(s3Object.Body)
          .resize(200)
          .jpeg({ mozjpeg: true })
          .toBuffer()
      
      const result = await s3.putObject({
        Bucket: 'serverless-resize-photobook', 
        Key: object.key,
        ContentType: 'image/jpeg',
        Body: data,
        ACL: 'public-read'
      }).promise()

    const message = 'Hello from Lambda!';

    console.info(`${message}`);
    
    return message;
}

```

**const s3Object =**
* 사진을 업로드하는 기존 s3 버킷의 정보와 사진의 정보를 가져온다.
* Promise가 해결되면 await 키워드를 사용하여 결과를 s3Object 변수에 할당된다. 이제 s3Object 변수는 지정된 버킷 및 키에서 검색된 실제 S3 객체 데이터를 보유하게 된다.

**const data =**
* data 에서 s3Object.Body가 S3 객체에서 얻은 이미지 파일이며, 추가 처리를 위해 Sharp 라이브러리에 로드되고 있음을 나타내고 있다.
* 'sharp()' 함수는 다양한 이미지 조작 작업을 연결할 수 있는 Sharp 개체를 반환한다.
* 'resize(200)' 메서드가 Sharp 개체에서 호출되고, 너비가 200픽셀이 되도록 이미지 크기가 조정된다. 높이는 원본 이미지의 비율에 따라 자동으로 조정된다.
* 크기 조정 후 jpeg({ mozjpeg: true }) 메서드가 호출되는데, 이는 결과 이미지가 MozJPEG 인코더를 사용하여 JPEG 형식으로 변환되어야 함을 나타낸다. MozJPEG는 고품질의 최적화된 JPEG 이미지를 생성하는 것으로 알려진 인기 있는 JPEG 인코더이다.
* 마지막으로 toBuffer() 메소드가 호출되어 처리된 이미지를 버퍼 객체로 변환한다. 이 버퍼에는 필요에 따라 추가로 사용하거나 저장할 수 있는 이미지의 이진 데이터가 포함되어 있다.

**const result =**
* 버킷 이름을 'serverless-resize-photobook'으로 하드코딩하였다.
* ContentType을 통해 객체가 JPEG 형식의 이미지임을 나타내는 'image/jpeg'로 설정된다.
* Body는 이전에 사진의 사이즈를 변경하기 위해 선언했던 'data'로 지정한다. data는 버퍼 형태의 이미지 데이터를 보유하고 있다.
* ACL: 업로드된 개체에 대한 액세스 제어 권한을 지정한다. 이 경우 업로드된 개체를 공개적으로 읽을 수 있음을 의미하는 'public-read'로 설정한다.
* Promise가 해결되면 업로드 작업의 결과가 await 키워드를 사용하여 result 변수에 할당된다. 'result' 변수는 ETag(엔티티 태그) 및 기타 메타데이터와 같은 업로드된 객체에 대한 정보를 보유하게 된다.

```bash
$ npm i aws-sdk
$ npm i sharp
```

위 코드를 작성만 하면 정상적으로 작동되지 않는다. 우리가 참조한 애플리케이션들을 다운로드 해주어야 한다. 위 명령을 차례로 실행해서 설치해주어야 s3 버킷과 sharp를 사용할 수 있다. 이제 빌드와 배포를 진행해보자.

```yaml
2023-05-14T11:49:28.176Z	1d2a889d-0fca-4582-8b20-5e16348aa009	ERROR	Invoke Error 	
{
    "errorType": "AccessDenied",
    "errorMessage": "Access Denied",
    "code": "AccessDenied",
    "message": "Access Denied",
    "region": null,
    "time": "2023-05-14T11:49:28.117Z",
    "requestId": "735WBTVKKJ0ENPCT",
    "extendedRequestId": "TWaytELIEkAu0UkrOF87v66pLC9dWW8NRds2OlKNV96fhL7W2bK4hge6bfeaVvY6ddbtvRjRzsc=",
    "statusCode": 403,
    "retryable": false,
    "retryDelay": 79.12852316993937,
    "stack": [
        "AccessDenied: Access Denied",
        "    at Request.extractError (/var/task/node_modules/aws-sdk/lib/services/s3.js:711:35)",
        "    at Request.callListeners (/var/task/node_modules/aws-sdk/lib/sequential_executor.js:106:20)",
        "    at Request.emit (/var/task/node_modules/aws-sdk/lib/sequential_executor.js:78:10)",
        "    at Request.emit (/var/task/node_modules/aws-sdk/lib/request.js:686:14)",
        "    at Request.transition (/var/task/node_modules/aws-sdk/lib/request.js:22:10)",
        "    at AcceptorStateMachine.runTo (/var/task/node_modules/aws-sdk/lib/state_machine.js:14:12)",
        "    at /var/task/node_modules/aws-sdk/lib/state_machine.js:26:10",
        "    at Request.<anonymous> (/var/task/node_modules/aws-sdk/lib/request.js:38:9)",
        "    at Request.<anonymous> (/var/task/node_modules/aws-sdk/lib/request.js:688:12)",
        "    at Request.callListeners (/var/task/node_modules/aws-sdk/lib/sequential_executor.js:116:18)"
    ]
}
```

위 코드를 빌드, 배포를 해보면 로그에 위와 같은 'AccessDenied' 오류가 찍힌다. s3 버킷을 참조할 권한이 부족하다는 것인데, IAM의 사용자에 AmazonS3FullAccess 권한을 부여해도 이 오류는 없어지지 않는다.

```yaml
      Policies:
        # Give Lambda basic execution Permission to the helloFromLambda
      - AWSLambdaBasicExecutionRole
      - AmazonS3FullAccess
```

AmazonS3FullAccess 권한을 최상위 폴더 밑에 template.yaml의 코드에 위와 같이 부여해주면 된다.  

![](https://res.cloudinary.com/duruq2coh/image/upload/v1684067362/gitio/post/serverless/16_z0mu03.png)

권한을 부여한 후 다시 빌드, 배포를 해주면 아까 새로 생성했던 버킷에 200으로 resize된 이미지가 업로드 되어 있는 것을 확인할 수 있다.

### 끝 입니다. ㅋ