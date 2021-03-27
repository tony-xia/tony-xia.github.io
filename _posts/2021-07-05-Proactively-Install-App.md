---
layout: post
title: 主动给团队或用户安装Teams App
---

在写这篇文章的时候，这个新功能还处在 Public Review，这意味着可能（很小的可能性）这里写的方法在正式发布前还会有一些改动。

之前有一些做teams app开发的朋友问过我，能不能主动给一个team或者一个用户安装一个指定的app，之前做不到，但现在可以了，方法如下：

1. 权限

先要确保你的app有 `TeamsAppInstallation.ReadWriteSelfForUser.All` 和 `TeamsAppInstallation.ReadWriteSelfForTeam.All`，从名字我们可以清楚的看到，一个权限是给一个用户主动安装app，第二是针对 team 的。

2. 找到要安装的 app 的 id

我在前一篇博客文章里解释过各种 id 的区别，简单的说，开发者在 teams app 的 manifest json 文件里指定的 id，并且不是 teams app id，在manifest里指定的 id 在teams graph api里叫做 external id，而 app id 是 teams 自动生成的一个 id。需要我们通过这个api来获取。

```
GET https://graph.microsoft.com/beta/appCatalogs/teamsApps?$filter=externalId eq '{11111111-2222-3333-4444-911d24850d7c}'

Response body:
{
  "value": [
    {
      "id": "b1c5353a-7aca-41b3-830f-27d5218fe0e5",
      "externalId": "11111111-2222-3333-4444-911d24850d7c",
      "name": "Test app name",
      "version": "0.0.1",
      ......
    }
  ]
}
```

上面的调用就是查询一个 external id 是等于 manifest ID 的一个 app，而在返回的内容里的 "id" 就是我们需要的 app id。

3. 检查这个 app 是否已经给 user 安装过

```
GET https://graph.microsoft.com/beta/users/{user-id}/teamwork/installedApps?$expand=teamsApp&$filter=teamsApp/id eq '{teamsAppId}'
```

4. 安装 app
如果上一步检测下来，user并没有安装过这个 app 的话，那么我们就可以开始安装 app 了。

```
POST https://graph.microsoft.com/beta/users/{user-id}/teamwork/installedApps

Request body:
{
   "teamsApp@odata.bind" : "https://graph.microsoft.com/beta/appCatalogs/teamsApps/{teamsAppId}"
}
```

上面 4 部都完成后，那个user就可以开始使用 app 了
