---
layout: post
title: Teams Bot如何判断用户所在的时区
---

一说到时间，就会联想到时区，夏令时等头痛的问题，不同国家有不同国家的规定。如果你希望你的Teams Bot可以判断出当前用户所在的时区，从而可以针对性的进行一些处理时，你要做好心理准备，这个复杂程度远远超过你的想象。因为一个用户这次在一个时区内登入Teams，下一次可能就在另一个时区了。

好消息是Teams已经帮我们做了很多事情，当Teams发送请求到我们的Bot时，payload里已经带了一些时间信息。我在我之前的一篇文章中也提到过。下面是一个标准的request body
``` json
{
    "name": "composeExtension/fetchTask",
    "type": "invoke",
    "timestamp": "2019-06-17T14:32:04.956Z",
    "localTimestamp": "2019-06-18T00:32:04.956+10:00",
    "id": "f:1361493733941541435",
    "channelId": "msteams",
    "serviceUrl": "https://smba.trafficmanager.net/apac/",
    "from": { },
    "conversation": {
        "isGroup": true,
        "conversationType": "channel",
        "tenantId": "aece5000-341d-493a-841d-f67e417f1447",
        "id": "19:bf1cbc367561473db0c3fe762c11b508@thread.skype"
    },
    "recipient": { },
    "entities": [
        {
            "locale": "en-US",
            "country": "US",
            "platform": "Windows",
            "type": "clientInfo"
        }
    ],
    "channelData": { },
    "value": {
        "commandId": "start",
        "commandContext": "compose",
        "context": {
            "theme": "default"
        }
    },
    "locale": "en-US"
}
```

上面这个payload里有两个关键的时间信息：

* timestamp：当前的UTC时间
* localTimestamp：当前用户所在的他/她的本地时间，加号后面的小时数针对UTC时间的offset

所以你的bot就知道了这个用户当前所在的时区和UTC相差多少。要注意我有两个重要点：
* "当前"。 我前面已经提到过，用户的所在地会变化，Teams告诉bot的是此时此刻的用户信息
* "所在的时区和UTC相差多少"。这里说的并不是时区(timezone)，为什么这么说？我们举个例子：假设现在是UTC时间0点0分0秒，用户A在UTC+11的时区，那TA的时间是11:00am，另一个用户B在UTC+10时区，但TA所在国家正好在经历夏令时，所以B的时间也是11:00am。在这两种情况下，Teams对于用户A和用户B发送给Bot的时间信息是一样的。但是实际上他们在不同时区。

如果你看到这里有点晕了，请回到上面这段再看一遍。如果你看懂了，那你会问：到底有没有版本得到用户所在的时区？我的目前的答案是没有特别好的方法，但是Graph API有一个取巧的方法。

Graph API包含了Outlook api，我们使用下面这个api：
```
GET https://graph.microsoft.com/beta/users/{id|userPrincipalName}/mailboxSettings
```

我们看一下它返回什么内容

``` json
{
  "@odata.context": "https://graph.microsoft.com/beta/$metadata#users('48d31887-5fad-4d73-a9f5-3c356e68a038')/mailboxSettings",
  "archiveFolder": "AAMkAGVmMDEzMTM4LTZmYWUtNDdkNC1hMDZiLTU1OGY5OTZhYmY4OAAuAAAAAAAiQ8W967B7TKBjgx9rVEURAQAiIsqMbYjsT5e-T7KzowPTAAAAAAFNAAA=",
  "timeZone": "Pacific Standard Time",
  "dateFormat": "M/d/yyyy",
  "timeFormat": "h:mm tt",
  "automaticRepliesSetting": {
    "status": "disabled",
    "externalAudience": "all",
    "internalReplyMessage": "",
    "externalReplyMessage": "",
    "scheduledStartDateTime": {
      "dateTime": "2019-10-05T12:00:00.0000000",
      "timeZone": "UTC"
    },
    "scheduledEndDateTime": {
      "dateTime": "2019-10-06T12:00:00.0000000",
      "timeZone": "UTC"
    }
  },
  "language": {
    "locale": "en-US",
    "displayName": "English (United States)"
  },
  "workingHours": {
    "daysOfWeek": [
      "monday",
      "tuesday",
      "wednesday",
      "thursday",
      "friday"
    ],
    "startTime": "08:00:00.0000000",
    "endTime": "17:00:00.0000000",
    "timeZone": {
      "name": "Pacific Standard Time"
    }
  }
}
```

是不是很强大？不旦旦有timezone信息，还有日期时间显示格式的偏好。还有工作日信息，不同国家对工作日的定义是不同的。还有标准工作时间的信息，不同公司对上下班时间的设置也会不同。

当然，使用Graph API并不是没有代价的，这个需要用户做额外的授权，不过Teams产品团队表示，以后会把对Graph API的授权和Teams app合在一起，这样用户在安装teams app的时候就同时完成了授权。期待这天早点到来。
