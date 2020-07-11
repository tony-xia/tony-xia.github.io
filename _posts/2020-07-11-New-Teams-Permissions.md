---
layout: post
title: 新的Teams API权限控制
---

这篇继续介绍BUILD大会里的内容：新的Teams API权限。这些新的权限让开发者可以更加细粒度的设置权限。

之前有些开发人员有问过我，为什么Graph API的权限这么多，为什么不针对Teams弄一个总的权限，这样不是更加简单吗？是，这样可以简化开发人员的工作，但是我们需要考虑到当一个企业IT安装teams app的时候，他/她会仔细审核你这个app所需要的权限，如果你app可以删除teams里的消息，甚至是可以删除一个team或者channel，我相信很多IT管理员就会很犹豫。所以我们作为一个teams app开发，我们应该仔细设置权限，应该只申请你app真正需要的权限，要求的权限越少，你的app就会有更多机会被安装到更多的企业中。

让我们来看一下几个最新的permission，这些permission在我写这篇文章的时候还是private preview，不过我相信很快就会public。

* **ChannelMessage.Delete**
这个权限允许app删除一个频道Channel里的消息Message，这个权限需要管理员审核，并且不支持Microsoft Account。所谓的Microsoft Account是指非AzureAD的企业账号。

* **ChannelMessage.Edit**
这个权限允许app修改一个频道Channel里的消息Message，同样，这个权限也需要管理员审核，并且也不支持Microsoft Account。

* **Chat.Send**
这个权限允许app以当前登入用户的身份来发送消息，这个消息有一定限制：目前允许的是一对一的对话，或者群聊。这个权限不需要管理员审核，并且不支持Microsoft Account。

* **Chat.Send.All**
这个权限和前面一个权限十分类似，区别在于这个权限是一个Application permissions，上面这个是Delegated permissions。这个权限允许app以一个用户的身份来发送消息，发送到一对一的对话，或者群聊。这个权限也不需要管理员审核，并且不支持Microsoft Account。

* **Teams.Create**
这个权限允许app创建team，它同时支持Delegated permission和Application permission。这个权限需要管理员审核，并且不支持Microsoft Account。

* **TeamsActivity.Read**
这个权限允许app读取登入用户的Activity feed。它不需要管理员审核，并且不支持Microsoft Account。

* **TeamsActivity.Read.All**
和上面权限类似，这个权限是Application permission，可以读取任何一个用户的activity feed。因为可以读取任何用户的信息，所以它需要管理员审核，并且不支持Microsoft Account。

* **TeamsActivity.Send**
这个权限允许app可以给任何用户发送activity，它需要管理员审核，并且不支持Microsoft Account。


参考：[Microsoft Graph permissions reference](https://docs.microsoft.com/en-us/graph/permissions-reference)
