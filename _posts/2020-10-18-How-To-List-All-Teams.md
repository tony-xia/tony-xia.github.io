---
layout: post
title: 如何获取租户中所有的Team
---

大家在使用Graph API开发Teams App的时候，有时候会需要获取某个租户Tenant的所有team，在写这篇文章的时候Graph API并没有提供这么一个功能，没有一个类似于"GET /teams"的api。

在Micorsoft Graph官方文档的已知问题中，也提到了目前不支持这么一个api。

[https://docs.microsoft.com/en-us/graph/known-issues?view=graph-rest-beta#get-teams-is-not-supported](https://docs.microsoft.com/en-us/graph/known-issues?view=graph-rest-beta#get-teams-is-not-supported)


那如果我们开发的app想要获取team列表，有其他什么方法吗？

我们现在想一下Microsoft Teams中每个team是一个什么概念？在AzureAD中，一个team实际上就是一个AzureAD中的group。group是可以通过graph api来列举的。实际上在Group中有一个属性叫`resourceProvisioningOptions`，它的定义如下：
[https://docs.microsoft.com/en-us/graph/group-set-options](https://docs.microsoft.com/en-us/graph/group-set-options)

| Supported values for resourceProvisioningOptions	| Description                              | Default if not set                                |
| :-----------------------------------------------  | :--------------------------------------- | :------------------------------------------------ |
| Teams               | Provision this group as a team in Microsoft Teams. Additionally, this value can be added to the resourceProvisioningOptions string collection on group update through a `PATCH` operation, in order to convert an existing Microsoft 365 group to a team.       | The group is a regular Microsoft 365 group without Teams capabilities. |

可以知道，原来我们通过这个属性就可以来判断这个group是不是一个Teams里的team。

```
GET /groups?$select=id,resourceProvisioningOptions
```

这个接口会返回如下的内容
```json
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#groups",
    "value": [
        {
            "id": "00e897b1-70ba-4cb9-9126-fd5f95c4bb78",
            "resourceProvisioningOptions": []
        },
        {
            "id": "00f6e045-f884-4359-a617-d459ee626862",
            "resourceProvisioningOptions": [
                "Team"
            ]
        }
    ]
}
```

可以发现第一个group不是team，第二个group是，因为第二个group的 `resourceProvisioningOptions` 里含有 `Team`。

看到这里，大家可能会问，一个租户里的group有很多，我如果拿到了所有的group，再自己过滤是不是有点浪费。有没有更好的方法来直接返回所有的team？

有！我们需要借助方便的OData查询语法，如下：

```
GET /groups?$filter=resourceProvisioningOptions/Any(x:x eq 'Team')
```

上面这个调用就可以返回租户中的所有的team，实际上就是 `GET /teams` 要做的东西了。注意上面的 `$filter` 的表达式，如果大家想具体深入的学习这些查询条件的语法，可以查看这个文档，学会了后你会觉得使用graph api原来如此方便。可以把很多原来需要在查询端做的事情，都让graph api服务来完成。

[https://docs.microsoft.com/en-us/graph/query-parameters#filter-parameter](https://docs.microsoft.com/en-us/graph/query-parameters#filter-parameter)
