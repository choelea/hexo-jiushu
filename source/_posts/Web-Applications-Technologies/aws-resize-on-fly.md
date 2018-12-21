---
title:  AWS 亚马逊云解决方案 - 动态压缩图片
description: AWS 亚马逊云解决方案 - 动态压缩图片
...



# 动态根据请求的尺寸生成图片
## 方案
![动态压缩图片](http://tech.jiu-shu.com/Web-Applications-Technologies/aws-resize-on-fly-architech.jpg)

**流程如下**
1. 用户向S3的桶（静态网站）发起resize的资源请求，桶设置了对应rule将不存在对应尺寸的资源重定向至OKCHEM-S3 (nodejs APP)
2. 由于对应的资源不存在，重定向至OKCHEM-S3
3. 浏览器向OKCHEM-S3发起请求
4. OKCHEM-S3 根据请求将图片进行resize， 然后上传至对应的桶
5. OKCHEM-S3 在完成上传后，永久重定向（301）到S3的URL。 （这个时候S3已经有了对应的资源）
6. 下次再次访问同样的资源，S3直接返回

参考： https://aws.amazon.com/cn/blogs/compute/resize-images-on-the-fly-with-amazon-s3-aws-lambda-and-amazon-api-gateway/   文章中缺失了关于API Gateway 部分的详细步骤。
## 测试图片
http://YOUR_BUCKET_WEBSITE_HOSTNAME_HERE/300×300/blue_marble.jpg

> 注意，S3 host static website的时候只支持http不支持https。 https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteEndpoints.html， 因此如果网站是https的，那么就需要采取CDN策略。
## 关键代码：

```javascript
var express = require('express');
var router = express.Router();
const config = require('../config')
const AWS = require('aws-sdk');
AWS.config.update(config.aws)
const S3 = new AWS.S3({
  signatureVersion: 'v4',
});
const Sharp = require('sharp');


/* Resize And upload to S3 */
router.get('/:BUCKET/resize', function(req, res, next) {
  const BUCKET = req.params.BUCKET;
  const URL = config.s3Hosts[BUCKET];
  const key = req.query.key;
  const match = key.match(/((\d+)x(\d+))\/(.*)/);
  const dimensions = match[1];
  const width = parseInt(match[2], 10);
  const height = parseInt(match[3], 10);
  const originalKey = match[4];
  S3.getObject({Bucket: BUCKET, Key: originalKey}).promise()
    .then(data => Sharp(data.Body)
      .resize(width, height)
      .toFormat('png')
      .toBuffer()
    )
    .then(buffer => S3.putObject({
        Body: buffer,
        Bucket: BUCKET,
        ContentType: 'image/png',
        Key: key,
      }).promise()
    )
    .then(() => {
      res.redirect(301, `${URL}/${key}`)
    })
    .catch(err => callback(err))
})
module.exports = router;

```

# AWS Congito - User Identity and Access Management
应用程序的域将为 https://<domain_prefix>.auth.<region>.amazoncognito.com。
您的应用程序的完整 URL 类似于此示例：https://example.auth.us-east-1.amazoncognito.com/login?redirect_uri=https://www.google.com&response_type=code&client_id=<client_id_value>

参考：https://docs.aws.amazon.com/zh_cn/cognito/latest/developerguide/cognito-user-pools-assign-domain.html
	
这里如果要修改域名, 比如把https://example.auth.us-east-1.amazoncognito.com 改成 https://auth.example.com, 就必须使用 类似  [Domain masking](https://en.wikipedia.org/wiki/Domain_masking) 这种手段， 好在Godaddy 这个域名提供商有个更简洁的方案：参考视频: [Godaddy Forwarding and Masking A Domain Name Tutorial](https://www.youtube.com/watch?v=Cfk_clv1_nY)

# AWS 资源文档集合
[AWS Serverless Authentication](http://tech.jiu-shu.com/Web-Applications-Technologies/aws-serverless-authentication.pdf)

[AWS Cognito Developer Document](https://aws.amazon.com/cn/documentation/cognito/)

[AWS Cognito Developer Guide](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-dg.pdf)

[Set Cache Control for Entire S3 Bucket](https://faragta.com/aws-s3/set-cache-control-for-entire-s3-bucket.html)

[Website Endpoints](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteEndpoints.html)
