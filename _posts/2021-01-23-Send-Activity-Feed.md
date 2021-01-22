---
layout: post
title: 推送ActivityFeed到Teams
---

几个月前，Teams 团队又推出了新的 Graph API，让 app 可以给用户发送 Activity Feed。我们来看看如何做。

首先，我们的app需要使用较新的 manifest 1.7版本，当然如果使用最新的1.8版本就更好了。在manifest json中添加 `webApplicationInfo` 和 `activities` 配置块

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/teams/v1.8/MicrosoftTeams.schema.json",
  "manifestVersion": "1.8",
  "version": "1.0.0",
  "id": "your app id",
  "packageName": "com.example.myapp",
  ...
  "webApplicationInfo": {
    "id": "AAD App ID",
    "resource": "Resource URL for acquiring auth token for SSO",
    "applicationPermissions": [ ... ],
  },
  "activities": {
    "activityTypes": [
      {
        "type":"taskCreated",
        "description":"Task Created Activity",
        "templateText":"{actor} created task {taskId} for you"
      },
      {
        "type":"teamMention",
        "description":"Team Mention Activity",
        "templateText":"{actor} mentioned team"
      }
    ]
  }
}
```

在 `webApplicationInfo` 中的 id 指的是 AzureAD app 的 ID (client ID)，`resource` 是 app 的 reply url (或者 redirect url)。

在 `activities` 中，type 是指 activity 的类型，它在整个manifest文件里需要唯一。description 是这类 activity 的简称，templateText 是 activity feed 推送的文本内容，它是一个模版，`{actor}` 在这里是一个特殊的类型，teams系统会自动填写，如果发送activity feed使用的是 delegated 验证的话，actor就显示delegated的用户的名字，如果是 application 验证的话，它显示的是 app 的名字。`{taskId}`是一个自定义的参数，你可以在发送 activity feed 的时候指定值。

我们准备好了manifest文件后，可以安装到teams 里，并且我们还需要确保一点：接受 activity feed 的用户也必须安装了我们的app，我之前有一篇文章讲的是如何主动给用户安装 app，有兴趣的读者可以参考那篇文章，这样就可以把你的app主动推给所有的用户进行安装。（当然，你的app首先需要取得相应的权限）。

准备工作做好后，我们就可以发送一个 Graph API的请求。如下：

```json
POST https://graph.microsoft.com/beta/chats/{chatId}/sendActivityNotification
Content-Type: application/json

{
    "topic": {
        "source": "entityUrl",
        "value": "https://graph.microsoft.com/beta/chats/{your-chat-id}"
    },
    "activityType": "taskCreated",
    "previewText": {
        "content": "A new task was created"
    },
    "recipient": {
        "@odata.type": "microsoft.graph.aadUserNotificationRecipient",
        "userId": "569363e2-1111-2222-3333-16f245c5d66a"
    },
    "templateParameters": [
        {
            "name": "taskId",
            "value": "123445"
        }
    ]
}
```

大家可以看到在 `templateParameters` 中，我们指定了taskId 参数，这个参数就对应到了之前 manifest 文件里 `activities` 节点下的 `{taskId}`。

```json
      {
        "type":"taskCreated",
        "description":"Task Created Activity",
        "templateText":"{actor} created task {taskId} for you"
      },
```

所以最终推送出去的文字内容就是 `SomeName created task 123445 for you`。
