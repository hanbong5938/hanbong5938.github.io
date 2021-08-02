---
layout: post
title: Node AWS S3 업로드
date: 2017-02-08
catalog: true
header-img: https://i.imgur.com/avC1Xor.jpg
tags:
- Node
---


Node.Js에서 AWS S3업로드 예제입니다. 본 예제의 전체 소스는 [GitHub](https://github.com/cheese10yun/node-yun)에서 참고할 수 있습니다.
***S3에 관련된 설정이 완료됐다는 가정하에 포스팅을 진행하겠습니다.***

#### 필수 패키지 설치
```bash
npm install formidable --save
npm install async --save
npm install aws-sdk --save
```

* <code><b>formidable</b></code> 파일 업로드를 위한 모듈
* <code><b>async</b></code>순차 실행을 위한 모듈
* <code><b>aws-sdk</b></code> S3 업로드를 위한 모듈

<code><b>HTML Form</b></code>
```html
<form  action="/api/v1/upload" enctype="multipart/form-data"  method="post">
    <input  type="file" name="img_files[]" accept="image/*">
    <button type="submit">Submit</button>
</form>
```

HTML 입력폼 소스입니다. 간단함으로 넘어가겠습니다.

<code><b>upload API</b></code>

```javascript
Upload = require('../service/UploadService'),
router.post('/upload', function (req, res) {
    var tasks = [
        function (callback) {
            Upload.formidable(req, function (err, files, field) {
                callback(err, files);
            })
        },
        function (files, callback) {
            Upload.s3(files, function (err, result) {
                callback(err, files);
            });
        }
    ];
    async.waterfall(tasks, function (err, result) {
        if(!err){
            res.json({success:true, msg:'업로드 성공'})
        }else{
            res.json({success:false, msg:'실패', err:err})
        }
    });
});
```

해당 upload 라우터로 요청이 들어오면 <code><b>tasks[...]</b></code>의 작업들이 <code><b>async</b></code>
모듈로 순차적으로 실행 됩니다. <code><b>tasks[...]</b></code> 작업순서는 다음과 같습니다.

1. **<code><b>formidable</b></code> 모듈를 이용해서 Node 서버로 파일을 업로드 시킵니다.**
2. **<code><b>aws-sdk</b></code> 모듈를 이용해서 AWS S3로 파일을 업로드 시킵니다.**
3. **<code><b>tasks[...]</b></code> 작업의 결과를 JSON타입으로 클라이언트에게 응답합니다.**

세부적인 작업은 <code><b>UploadService.js</b></code>에서 <code><b>callback</b></code>으로 진행됩니다.
아래에서 설명을 계속 진행하겠습니다.

<code><b>UploadService.js S3 & formidable설정 </b></code>

```javascript
var
    formidable = require('formidable'),
    AWS = require('aws-sdk'),
    Upload = {};
AWS.config.region = 'ap-northeast-2'; //지역 서울 설정
var s3 = new AWS.S3();
var form = new formidable.IncomingForm({
    encoding: 'utf-8',
    multiples: true,
    keepExtensions: false //확장자 제거
});
/*S3 버킷 설정*/
var params = {
    Bucket: 'BucketName',
    Key: null,
    ACL: 'public-read',
    Body: null
};
```

<code><b>aws_access_key_id, aws_secret_access_key</b></code>값을 소스코드에 입력하시는 것은 보안상 바람직하지 않습니다.
[AWS Document](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)를 참고하세요.

<code><b>var params = {...}</b></code> 객체는 AWS S3업로드에 대한 정보 입니다.

* Bucket :  S3 Bucket 이름을 지정합니다.
* Key : S3의 경로 및 파일 이름을 지정합니다.
* ACL : 파일 권한에 대한 설정입니다.
* Body : 업로드할 파일의 경로를 지정합니다.

<code><b>UploadService.js 업로드 로직</b></code>
```javascript
Upload.formidable = function (req, callback) {
    form.parse(req, function (err, fields, files) {
    });
    form.on('error', function (err) {
        callback(err, null);
    });
    form.on('end', function () {
        callback(null, this.openedFiles);
    });
    form.on('aborted', function () {
        callback('form.on(aborted)', null);
    });
};
Upload.s3 = function (files, callback) {
    params.Key = 'test/'+files[0].name;
    params.Body = require('fs').createReadStream(files[0].path);
    s3.upload(params, function (err, result) {
        callback(err, result);
    });
};
module.exports = Upload;
```

가장 핵심인 업로드 로직입니다. <code><b>formidable</b></code>, <code><b>s3</b></code> 메서드는 <code><b>callback</b></code> 메서드로
각작업의 결과를 넘겨줍니다.

<code><b>formidable 메서드 설명</b></code>   
라우터에서 넘겨준 <code><b>req</b></code> 객체를 기반으로 파일 업로드를 진행합니다.
파일 업로드 중 에러가 발생하게 되면 <code><b>form.on('error', ...)</b></code> 메서드를 통해서 에러를 <code><b>callback</b></code>으로 넘겨줍니다.

파일 업로드가 정상적으로 완료되면 <code><b>form.end(null, ...)</b></code>메서드가 호출되고 업로드한 파일의 정보(파일 사이즈, 파일 이름, 파일 경로 등등)가 <code><b>callback</b></code> 메서드를 통해서 으로 넘어가게 됩니다.  

###### <code><b>s3</b></code> 메서드 설명

<code><b>async.waterfall</b></code>를 통해서 넘겨받은 <code><b>files</b></code> 객체에는 위에서 설명한 파일 정보가 들어있는 객체입니다.

* <code><b>params.Body</b></code>값에는 위에서 업로드한 파일을 넘겨줍니다.
* <code><b>params.Key</b></code>값에는 실제 S3에 업로드될 path + 파일 이름을 작성합니다.


#### 실행 화면

![](https://i.imgur.com/P0bMJdM.png)
![](https://i.imgur.com/u2qStuu.png)

<code><b>params.Key</b></code> 값은 <code><b>test/[filename]</b></code> 입니다.
<code><b>test/</b></code>는 경로로 인식되며 해당 경로가 없는 경우에는 디렉터리를 자동으로 생성해서 파일을 목적지까지 안전하게? 전달됩니다.

#### 마무리
 회사에서 AWS S3업로드를 리펙토링 작업이 있어서 간단하게 정리해봤습니다.
 추가로 작업한 부분도 포스팅할 예정입니다.
 아무래도 AWS S3 이미지 업로드시 이미지를 최적화시키는 것이 될듯합니다.
 앞으로 계속 찾아뵙겠습니다.
 RSS 링크도 하단에 추가하였으니 추가해주시면 감사하겠습니다.
