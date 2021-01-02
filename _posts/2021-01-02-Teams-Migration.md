---
layout: post
title: 迁移聊天记录到Teams
---

有一些朋友问我teams是否支持将其他平台/系统里的聊天记录迁移某个channel里，答案是肯定的，teams团队在去年年中的时候就提供了这个功能。这个功能是通过graph api来完成的，我们今天就来看看如何迁移聊天记录到teams里。

首先，我们需要确保我们的app有了一个teams的权限：`Teamwork.Migrate.All`，然后确保下面调用的api都是在beta版本下。

1. 创建一个team

需要注意的是这个team的creation mode是一个特殊的值：`migration`。如下：

```
POST https://graph.microsoft.com/beta/teams

Request body:
{
  "@microsoft.graph.teamCreationMode": "migration",
  "template@odata.bind": "https://graph.microsoft.com/beta/teamsTemplates('standard')",
  "displayName": "Tony test team",
  ...
}
```

2. 创建一个channel

等上面的team创建完成后，我们再创建一个channel。它同样creation mode是一个特殊的值：`migration`。如下：

```
POST https://graph.microsoft.com/beta/teams/{team-id}/channels

Request body:
{
  "@microsoft.graph.channelCreationMode": "migration",
  "displayName": "Test channel",
  "description": "test purpose only",
  "membershipType": "standard",
  ...
}
```

3. 一条条的导入聊天记录

当一条消息是纯文本的时候，使用如下格式：

```
POST https://graph.microsoft.com/beta/teams/{team-id}/channels/{channel-id}/messages

Request body:
{
   "createdDateTime":"2019-02-04T19:58:15.511Z",
   "from":{
      "user":{
         "id": "user id",
         "displayName": "Thomas",
         "userIdentityType": "aadUser"
      }
   },
   "body":{
      "contentType": "html",
      "content": "How is it going"
   }
}
```

如果这条消息含有图片的话，我们可以把图片内嵌到这条消息里：

```
POST https://graph.microsoft.com/beta/teams/{team-id}/channels/{channel-id}/messages

Request body:
{
  "body": {
        "contentType": "html",
        "content": "<img height=\"160\" src=\"../hostedContents/1/$value\" width=\"200\" style=\"vertical-align:bottom; width:176px; height:250px\">"
    },
    "hostedContents":[
        {
            "@microsoft.graph.temporaryId": "1",
            "contentBytes": "iVBORw0KGgoA.........",
            "contentType": "image/png"
        }
    ]
}
```

4. 当我们完成了聊天消息的导入后，我们就需要把channel和team的状态改成正常的状态，这样这个team和channel就可以开始正常的使用了

修改team的状态，结束迁移。

```
POST https://graph.microsoft.com/beta/teams/{team-id}/completeMigration
```

```
POST https://graph.microsoft.com/beta/teams/{team-id}/channels/{channel-id}/completeMigration
```

额外的一些工作：需要往我们刚才创建的team里增加用户，这样这个用户就可以开始使用这个team。这个也可以通过graph api来实现：

```
POST https://graph.microsoft.com/beta/teams/{team-id}/members

Request body:
{
    "@odata.type": "#microsoft.graph.aadUserConversationMember",
    ...
}
```
