---
layout: post
title: Teams bot的调用限制
---

上个月Teams团队发布了对Teams app/bot调用api的频率的限制。这也从侧面说明Teams app越来越多，Teams团队需要优先保证Teams本身的计算资源，来提供流畅的用户体验。

具体的每个限制指标在这里： [https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/bots/rate-limit](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/bots/rate-limit)

我解释一下目前的做法，teams app需要注意的地方，以及一些容易混淆的概念：

* Teams会对大家的api服务返回HTTP 429错误，可能大家之前很少遇到这个status code。429是Too Many Requests，就是请求数太多。服务器无法处理。当达到一些限制条件后，teams就会返回这个code

* 大家的api服务当遇到这个429时不要惊慌，这个是很平常的预料中的错误，需要做的是稍微等待一会儿，然后再次发送相同的请求给到Teams，如果你的api服务中已经有了retry机制，那就比较简单。如果没有，可以使用这段代码来重试。

```csharp
public class MyRetryStrategy : ITransientErrorDetectionStrategy
{
    public bool IsTransient(Exception ex)
    {
        var httpOperationException = ex as HttpOperationException;
        if (httpOperationException != null)
        {
            return httpOperationException.Response != null && (int)httpOperationException.Response.StatusCode == 429);
        }
        return false;
    }
}

var exponentialBackoffRetryStrategy = new ExponentialBackoff(5, TimeSpan.FromSeconds(2),
                        TimeSpan.FromSeconds(8), TimeSpan.FromSeconds(16), TimeSpan.FromSeconds(32));

// Setup Retry Policy
var retryPolicy = new RetryPolicy(new MyRetryStrategy(), fixedIntervalRetryStrategy);

await retryPolicy.ExecuteAsync(() => connector.Conversations.ReplyToActivityAsync((Activity)reply)).ConfigureAwait(false);
```

上面是C#的代码，对于其他语言，做法也类似，当接收到429代码时，等待一段时间，然后重试，再不行再等待一段时间，再重试。

* 这次的限制分为三个等级：
  * 第一级：你的bot对于一个聊天，这个聊天可以是：一个1对1的聊天，用户和bot的聊天，团队中一个频道的聊天
  * 第二级：所有bot对于一个聊天的限制，也就是说如果这个频道中有很多bot，大家共享这个限制。如果一个bot非常频繁的往频道中发送消息，那意味着其他bot会很大概率收到429出错代码
  * 第三级：你的bot针对datacenter的限制。这个大家可能不好理解。实际上Teams整个系统部署Azure上，但是并不是全部覆盖了所有的Azure数据中心，我们的bot在收到一个message payload时可以看到有一个字段。

```json
{
    "name": "composeExtension/fetchTask",
    "type": "invoke",
    "timestamp": "2019-06-17T14:32:04.956Z",
    "localTimestamp": "2019-06-18T00:32:04.956+10:00",
    "id": "f:1361493733941541435",
    "channelId": "msteams",
    "serviceUrl": "https://smba.trafficmanager.net/apac/",
    "from": {
        "id": "29:1l8B9m9SOOdHTqLDgmXvSrJyHfwd2ihooa7cxgtzJ8QjQ4WFC4mA_8K2Sa7jL-xUh7g4yh8sZIiDOX6vTtoaz6w",
        "name": "Tony Xia",
        "aadObjectId": "56c6599d-9216-4078-a8cf-3f039d36e1fd"
    },
    "conversation": {
        "isGroup": true,
        "conversationType": "channel",
        "tenantId": "aece5000-341d-493a-841d-f67e417f1447",
        "id": "19:bf1cbc367561473db0c3fe762c11b508@thread.skype"
    },
    "recipient": {
        "id": "28:89e9cdd8-f500-4696-a701-7c2323f62a86",
        "name": "TestMsgExt"
    },
    "entities": [
        {
            "locale": "en-US",
            "country": "US",
            "platform": "Windows",
            "type": "clientInfo"
        }
    ],
    "channelData": {
        "channel": {
            "id": "19:bf1cbc367561473db0c3fe762c11b508@thread.skype"
        },
        "team": {
            "id": "19:bf1cbc367561473db0c3fe762c11b508@thread.skype"
        },
        "tenant": {
            "id": "aece5000-341d-493a-841d-f67e417f1447"
        },
        "source": {
            "name": "compose"
        }
    },
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

大家又看到serviceUrl这个字段吗？

https://smba.trafficmanager.net/apac/

上面这个是表明这个是从哪个数据中心传来的数据，APAC就是亚洲太平洋地区。

所以这个条调用限制的意思就是你的bot往一个数据中心的调用次数有限制。大家可能会问：那如果我的teams bot很流行，这个数据中心的很多企业在用这个bot，这个限制有点不公平啊。

放心，首先你的bot很难达到这个限制，如果达到了，你可以向teams团队提申请，我相信他们肯定很愿意看到这种超级teams app，肯定愿意为你的bot放宽限制。😀

