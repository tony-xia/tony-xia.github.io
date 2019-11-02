---
layout: post
title: Team photo的新api
---

Graph API的更新速度真是快，今年9月中旬又增加了关于Team photo的两个新的api。

* https://docs.microsoft.com/en-us/graph/api/team-get-photo
* https://docs.microsoft.com/en-us/graph/api/team-update-photo

今天就给大家介绍一下如何使用这两个新的api。
实际上说起来也简单，因为graph api的设计理解一脉相承。这两个api一个是获取一个team的图标（photo），一个是更新修改team的图标，所以对graph api熟悉的开发这立刻就可以想到他们对应的url是：
```
GET /beta/teams/{id}/photo
PUT /beta/teams/{id}/photo
```

对于GET获取图标的api需要特别提一下，上面这个api主要是获取图标的meta信息，返回信息类似于：
```
HTTP/1.1 200 OK
Content-type: application/json
{
    "@odata.mediaContentType": "image/jpeg",
    "id": "240X240",
    "width": 240,
    "height": 240
}
```

如果想要获取真正的图片文件，那要使用这个api：
```
GET /beta/teams/{id}/photo/$value
```
这个api会返回图片的二进制流。

另外值得一提的是，graph api设计很贴心，对于图片的获取，还提供了resize功能，也就是说调用方可以指定想要的图片的长和宽，api会返回对应大小的图片。
```
GET /beta/teams/{id}/photo/{size}
GET /beta/teams/{id}/photo/{size}/$value
```
这里的{size}目前支持这几种：48x48, 64x64, 96x96, 120x120, 240x240, 360x360, 432x432, 504x504, 和648x648。比如我想要获取一个id为738c7842-d884-4456-9b77-e2c0a73fb200的team的图标文件，就可以使用这个call
```
GET https://graph.microsoft.com/beta/teams/738c7842-d884-4456-9b77-e2c0a73fb200/photo/240x240/$value
```

对于更新team图标，目前的api对于图片大小有一个4MB的限制，所以如果你的图片超过这个大小，再上传前需要做一个压缩处理。

最近teams还增加了很多其他实用的api，我会在后面慢慢介绍。
