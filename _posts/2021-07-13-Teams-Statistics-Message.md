---
layout: post
title: Teams数据统计 - 聊天消息
---

前两篇文章介绍了如何对用户的在线状态和通话记录进行数据统计。这篇文章我们来看看如何统计用户的聊天消息。

在介绍具体 api 如何调用前，我们可以先看一下 Teams 里对于 Message 的层级结构，在 Teams 里，message有两种，一种是 Team 的 Channel 的 message，一种是对一对聊天或者群聊里的 message。他们的层级设计是不同的。

## Channel 里的消息

它的层次结构如下：
> Team -> Channel -> Message -> Reply
一个 tenant 里有多个 Team，每个 Team 里可以有多个 Channel，每个 Channel 里有多个 Message，每个 Message 可以有 0 到 n 个 reply消息。

所以我们使用 graph api 来获取信息的时候，我们先获取 teams 列表，因为 graph api目前没有一个简单的获取 team 列表的接口，所以必须要用下面这个获取 group 列表的接口，再加上 filter。这是一个[已知的问题](https://docs.microsoft.com/en-us/graph/known-issues?context=graph%2Fapi%2Fbeta&view=graph-rest-beta#get-teams-is-not-supported)，希望以后能被fix掉。

```
GET /groups?$filter=resourceProvisioningOptions/Any(x:x eq 'Team')
```
```json
Response:
{
    "@odata.context": "https://graph.microsoft.com/beta/$metadata#groups",
    "value": [
        {
            "id": "02bd9fd6-8f93-4758-87c3-1fb73740a315",
            "displayName": "HR Taskforce",
            "visibility": "Private"
            ...
        },
        ...
    ]
}
```

我们有了 team 列表后就可以开始获取每个 team 的 channel。使用如下接口：

```
GET https://graph.microsoft.com/beta/teams/{team-id}/channels
```
```json
Response:
{
  "value": [
    {
      "description": "description-value",
      "displayName": "display-name-value",
      "id": "02bd9fd6-1111-4758-87c3-1fb73740a315",
      ....
    },
    ...
  ]
}
```

有了 channel 后，我们就可以获取 channel 里的 message 列表。

```
GET https://graph.microsoft.com/beta/teams/{team-id}/channels/{channel-id}/messages
```
```json
Response:
{
    "value": [
        {
            "id": "1555375673184",
            "messageType": "message",
            "createdDateTime": "2019-04-16T00:47:53.184Z",
            "lastModifiedDateTime": "2019-05-04T19:58:15.511Z",
            "lastEditedDateTime": null,
            "deletedDateTime": null,
            "subject": "",
            "summary": null,
            "importance": "normal",
            "locale": "en-us",
            "from": {
                "user": {
                    "id": "bb8775a4-4d8c-42cf-a1d4-4d58c2bb668f",
                    "displayName": "Adele Vance",
                    "userIdentityType": "aadUser"
                }
            },
            "body": {
                "contentType": "html",
                "content": "<div><div>Nice to join this team. <at id=\"0\">Megan Bowen</at>, have we got the March report ready please?</div>\n</div>"
            },
            "attachments": [],
            "mentions": [
                {
                    "mentionText": "Megan Bowen",
                    "mentioned": {
                        "user": {
                            "id": "5d8d505c-864f-4804-88c7-4583c966cde8",
                            "displayName": "Megan Bowen",
                            "userIdentityType": "aadUser"
                        }
                    }
                }
            ],
            "reactions": []
        },
        ...
    ]
}
```

大家可以发现上面有用的信息非常多，比如：
* `createdDateTime`, `lastModifiedDateTime`, `lastEditedDateTime` 和 `deletedDateTime`，各种时间
* `from`：message是谁发的
* `mentions`：消息里有没有 @ 其他人
* `reactions`：消息有没有被点赞，谁在什么时候点了赞或者点了什么其他表情

由于一个channel里的 message 会很多，所以 graph api 还有一个分批获取 message 的接口，如果对这个接口感兴趣，可以参考这个文档：[https://docs.microsoft.com/en-us/graph/api/chatmessage-delta?view=graph-rest-beta&tabs=http](https://docs.microsoft.com/en-us/graph/api/chatmessage-delta?view=graph-rest-beta&tabs=http)

```
GET /teams/{team-id}/channels/{channel-id}/messages/delta
```

有了 message 后就可以获取每个 message 的replies 了。

```
GET /teams/{team-id}/channels/{channel-id}/messages/{message-id}/replies
```

## 对一对聊天或者群聊里的 message

它的层次结构比较简单，如下：
> User -> Chat -> Message

我们先需要枚举当前 tenant 下的所有的 user，然后对每一个 user 调用下面的接口来获取这个用户的聊天。

```
GET https://graph.microsoft.com/beta/users/{user-id}/chats
```

```json
Response:
{
    "value": [
        {
            "id": "19:meeting_MjdhNjM4YzUtYzExZi00OTFkLTkzZTAtNTVlNmZmMDhkNGU2@thread.v2",
            "topic": "Meeting chat sample",
            "createdDateTime": "2020-12-08T23:53:05.801Z",
            "lastUpdatedDateTime": "2020-12-08T23:58:32.511Z",
            "chatType": "meeting"
        },
        {
            "id": "19:561082c0f3f847a58069deb8eb300807@thread.v2",
            "topic": "Group chat sample",
            "createdDateTime": "2020-12-03T19:41:07.054Z",
            "lastUpdatedDateTime": "2020-12-08T23:53:11.012Z",
            "chatType": "group"
        },
        {
            "id": "19:d74fc2ed-cb0e-4288-a219-b5c71abaf2aa_8c0a1a67-50ce-4114-bb6c-da9c5dbcf6ca@unq.gbl.spaces",
            "topic": null,
            "createdDateTime": "2020-12-04T23:10:28.51Z",
            "lastUpdatedDateTime": "2020-12-04T23:10:36.925Z",
            "chatType": "oneOnOne"
        }
    ]
}
```

可以看到，上面的接口返回了各种对话类型 (chatType)：一对一聊天 `oneOnOne` ，群聊 `group` ，和会议里的聊天 `meeting`。有了chat列表后，我们就能对每一个chat来获取 message。
```
GET https://graph.microsoft.com/beta/users/{user-id}/chats/{chat-id}/messages
```

这个接口返回的内容和channel message返回的内容类似。

看到这里想必大家已经发现了 teams 的强大，和 graph api 的开放性，只要 app 有对应的权限，基本就能拿到任何数据，有了数据后，我们的统计报表就简单了。

