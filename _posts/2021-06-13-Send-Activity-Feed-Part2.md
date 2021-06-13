---
layout: post
title: 发送不同类型的ActivityFeed
---

上一篇文章讲到了如何使用最新的Graph API来给一个用户发送一个简单的 Activity Feed。我们这篇文章来详细讲一下发送三种不同类型的消息。

### 发送 Chat 相关的 Activity Notification

API 为 `POST https://graph.microsoft.com/beta/chats/{chat-id}/sendActivityNotification`

http请求的内容为：

```json
{
    "topic": {
        "source": "entityUrl",
        "value": "https://graph.microsoft.com/beta/chats/{chat-id}"
    },
    "activityType": "taskCreated",
    "previewText": {
        "content": "New Task Created"
    },
    "recipient": {
        "@odata.type": "microsoft.graph.aadUserNotificationRecipient",
        "userId": "569363e2-1111-2222-3333-16f245c5d66a"
    },
    "templateParameters": [
        {
            "name": "taskId",
            "value": "Task 12322"
        }
    ] 
}
```

其中 `activityType` 和 `templateParameters` 我已经在前面一篇文章中介绍过，`previewText` 从名字上也很容易理解，是推送的消息的文本内容，`recipient` 是接受推送消息的用户的 id 信息。

这里比较关键的是 `topic`，这个指明了这个推送的消息是针对哪个聊天 (chat) 或者哪个 聊天消息 (chatMessage) 的，如果是针对 chat 的，那么 `topic` 里的 `value` 就是 `https://graph.microsoft.com/beta/chats/{chat-id}` 这种格式，如果是 chatMessage，格式就是 `https://graph.microsoft.com/beta/chats/{chat-id}/messages/{message-id}`

大家可能会问，如何获得这个 chat id 和 message id 呢，这两个可以在发送 message 是返回，或者也可以通过调用 graph api 来获取。

### 发送 Team 相关的 Activity Notification

API 为 `POST https://graph.microsoft.com/beta/teams/{teamId}/sendActivityNotification`

http请求的内容和之前的很像：

```json
{
    "topic": {
        "source": "entityUrl",
        "value": "https://graph.microsoft.com/beta/teams/{team-id}"
    },
    "activityType": "pendingFinanceApprovalRequests",
    "previewText": {
        "content": "Internal spending team has a pending finance approval requests"
    },
    "recipient": {
        "@odata.type": "microsoft.graph.aadUserNotificationRecipient",
        "userId": "569363e2-1111-2222-3333-16f245c5d66a"
    },
    "templateParameters": [
        {
            "name": "pendingRequestCount",
            "value": "5"
        }
    ]
}
```

它支持的 entity url 有好几种：
* team，格式为 `https://graph.microsoft.com/beta/teams/{team-id}`
* channel，格式为 `https://graph.microsoft.com/beta/teams/{team-id}/channels/{channel-id}`
* chatMesage，格式为 `https://graph.microsoft.com/beta/teams/{team-id}/channels/{channel-id}/messages/{message-id}`
* teamsTab，格式为 `https://graph.microsoft.com/beta/teams/{team-id}/channels/{channel-id}/tabs/{tab-id}`

### 发送 User 相关的 Activity Notification

API 为 `POST https://graph.microsoft.com/beta/users/{userId}/teamwork/sendActivityNotification`

http请求的内容和之前的很像：

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

它支持的 entity url 有两种：
* teamsAppInstallation，格式为 `https://graph.microsoft.com/beta/users/{user-id}/teamwork/installedApps/{installation-id}`
* teamsCatalogApp，格式为 `https://graph.microsoft.com/beta/appCatalogs/teamsApps/{app-id}`
