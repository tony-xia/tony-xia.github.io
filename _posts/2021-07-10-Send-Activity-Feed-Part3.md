---
layout: post
title: 发送ActivityFeed的隐藏功能
---

前两篇文章介绍了如何发送 activity notification，这篇文章主要介绍两个隐藏功能，实际上所谓的隐藏功能是指大家在阅读官方文档是会忽略的两个点，但是实际上也是很实用的两个功能点。

## text 类型的 topic

之前文章中提到我们的 activity notification 支持三种类型，他们分别的url是：

```
POST https://graph.microsoft.com/beta/chats/{chat-id}/sendActivityNotification
POST https://graph.microsoft.com/beta/teams/{teamId}/sendActivityNotification
POST https://graph.microsoft.com/beta/users/{userId}/teamwork/sendActivityNotification
```

他们的http request body基本类似：

```json
{
    "topic": {
        "source": "entityUrl",
        "value": "https://graph.microsoft.com/beta/users/{user-id}/teamwork/installedApps/{installation-id}"
    },
    "activityType": "taskCreated",
    "previewText": {
        "content": "New Task Created"
    },
    "templateParameters": [
        {
            "name": "taskId",
            "value": "Task 342342"
        }
    ]
}
```

区别在于不同的类型，他们对应的topic里的value的格式不同。实际上 Teams 的 Graph API 还给我们提供了一种通用的 topic 类型：`text`，如下：

```json
{
    "topic": {
        "source": "text",
        "value": "Deployment Approvals Channel",
        "webUrl": "https://teams.microsoft.com/l/message/19:448cfd2ac2a7490a9084a9ed14cttr78c@thread.skype/1605223780000?tenantId=c8b1bf45-3834-4ecf-971a-b4c755ee677d&groupId=d4c2a937-f097-435a-bc91-5c1683ca7245&parentMessageId=1605223771864&teamName=Approvals&channelName=Azure%20DevOps&createdTime=1605223780000"
    },
    ...
}
```

可以看到，在 `source` 字段指定 `text`，然后在 `value` 里填入你想要的任何文字内容，再加一个 `webUrl` 就可以了。需要注意的是，在这种模式下， `webUrl` 是必须的。

有了这种类型，实际上你就可以推送任何内容了，不再局限于 team 里的某个 resource。

## Activity Notification的修改

在一个推送的请求里，实际上还有一个隐藏的属性 `chainId`

```json
{
    "topic": { ... },
    "activityType": "...",
    "previewText": { ... },
    "templateParameters": [ ... ],
    "chainId": 3279238
}
```

`chainId` 是一个64位的整数，来唯一的指定你这次推送的id，如果你需要修改你之前推送的notification内容，可以再次调用 graph api，只要传入一样的 `chainId` 就可以了，就可以将之前的notificaiton更新。

也就是说如果你的 app 需要更新之前发的 notification 内容，那你在发推送的时候需要生成一个唯一的64位整数，并且保存下来，下次要更新的时候再查询到那个id，并用它再发送新的 notification 内容即可。
