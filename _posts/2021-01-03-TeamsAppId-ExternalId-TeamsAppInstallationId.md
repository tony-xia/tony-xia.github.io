---
layout: post
title: Teams AppId, InstallationId 和 ExternalId 的区别
---

大家如果看teams的 graph api 开发文档，可能会把 app id, installation id 和 external id 搞混，我自己一开始的时候就有点被搞晕了，再加上app manifest里面的 id 和 bot id，基本就彻底晕掉了。

那我们今天这篇文章就来讲讲这几种 id 。

首先当我们开发一个 team app 的时候，都需要编写一个 manifest json 文件，在这个文件里就有必须要指定一个 id，还有一个 bot id，bot id就是你创建的 microsoft app id，我们通常把这两个id 使用一样的值，但是实际上一个 teams app里面可以有多个 bot，也就是说你可以在manifest json文件里指定多个 bot id。

当我们把我们的 teams app提交到teams里后（提交一个zip文件，zip里含有manifest json文件），teams系统就会自动生成一个 app id，这个就是teams文档里提到的 app id，要注意的是这个app id和我们在manifest 文件里写的id 不是一个东西。同时 teams 把 manifest 里的 id 也保存下来，叫做了 external id。

我们看一下微软文档里的写法： [https://docs.microsoft.com/en-us/graph/api/resources/teamsapp?view=graph-rest-1.0&preserve-view=true#properties](https://docs.microsoft.com/en-us/graph/api/resources/teamsapp?view=graph-rest-1.0&preserve-view=true#properties)

#### TeamsApp Resource Type Properties

| Property            | Type     | Description |
|:------------------- |:-------- |:----------- |
| id                  | string   | The catalog app's generated app ID (different from the developer-provided ID in the [Microsoft Teams zip app package](/microsoftteams/platform/concepts/apps/apps-package). |
| externalId          | string   | The ID of the catalog provided by the app developer in the [Microsoft Teams zip app package](/microsoftteams/platform/concepts/apps/apps-package). |

搞清楚了 app id 和 external id 的区别后，我们再来看一下 installation id 就简单了。

一个 app 可以被安装到不同的 tenant 下的不同的 team 里，每一次安装，就会对应的生成一个 installtion id。也就是说一个 app id 会对应到多个 installation id，他们是一对多的关系。

我们来看一个实际的例子。

```
GET https://graph.microsoft.com/v1.0/teams/{team-id}/installedApps?$expand=teamsApp

Response:
HTTP/1.1 200 OK
Content-type: application/json
{
    "@odata.count": 1,
    "value": [
        {
            "id": "NjkwM2Z....",   // installation id
            "teamsApp": {
                "id": "11111111-25e0-4569-8ebe-13601cb55a18",    // app id
                "externalId": "22222222-f94e-4d80-ba90-5594b641a8ee",  // external id (在manifest里指定的 id)
                "displayName": "YPA",
                "distributionMethod": "sideloaded"
            }
        }
    ]
}
```

通过上面的例子，大家应该都清楚他们的关系了，当然，如果你能自己动手调用一下graph api，看一下在你自己的tenant里各个 id 的情况，那肯定理解更加深刻。
