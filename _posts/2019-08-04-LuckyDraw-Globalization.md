---
layout: post
title: Teams Bot如何做全球化
---

Office365在全球有大量的用户，可以说是拥有最多用户的商业SaaS平台。Teams最近在发展迅猛，有1300万日活用户，已经超越了Slack。😄 [Microsoft Teams overtakes Slack with 13 million daily users](https://www.theverge.com/2019/7/11/20689143/microsoft-teams-active-daily-users-stats-slack-competition)

我在设计Teams LuckyDraw bot的时候就希望我的bot能够被全世界的用户所使用，这个实际上还是一个非常有挑战的事情。抛开各国的法律法规要求（比如欧盟的GDPR，国家领土归属问题）不说，抛开不同宗教文化要求不说，单单多语言，多时区就是十分复杂。

举几个语言的例子：
* 英语里有单复数，中文和一些语言里就没有。你在设计多语言界面的时候就需要注意
* 即使是英语，美式英语(en-US)，英式英语(en-UK)，印度英语(en-IN)，有很多区别
* 大多数语言是从左到右书写，但是有些是从右到左。界面设计特别需要注意

再举几个时区的例子：
* 有很多土地领域较大的国家，不像中国只有一个时区，同一个国家有几个时区是很正常的事情
* 有些国家有些区域有夏令时。比如澳大利亚的布里斯班和悉尼，虽然在一个时区，但是悉尼有夏令时，也就是说平时大家时间一样，但是一到夏天，时间就变得不同了
* 不同国家对于每周第一天是周一还是周日，定义不同
* Samoa和Tokelau这两个地方没有2011年12月30日这一天，他们直接从29日跳到了31日( [具体原因](https://www.bbc.com/news/world-asia-16351377) )。不知道这种事情会不会再次发生 （看到这里大家是不是和我当时一样，心里一万个。。。。）
* 日本从2019年5月1日开始新的calendar，为此，Windows等系统，.NET等运行环境，各类时间处理的库都全面升级，打补丁，出新版本

看了上面几个例子，是不是觉得很无语。是不是瞬间觉得那些国际化的SaaS平台有多伟大和复杂。

那我们来看看Teams app/bot如何处理时间问题，一个好消息是Teams已经帮我们处理了很多问题，来看一下Teams发送给bot的请求payload：

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

上面这个payload里有几个关键的值：
* timestamp：当前的UTC时间
* localTimestamp：当前用户所在的他/她的本地时间，加号后面的小时数针对UTC时间的offset
* locale：当前用户所使用的语言

有了这几个参数，实际上就你的bot就知道改如何处理了吧？

不过，在具体的bot设计中，你还需要时刻留意这几点：
* 一家公司（一个office365的tenant，可能有使用不同语言的人，可能分散在不同国家，在不同时区）
* 一个Team或者一个频道channel里的用户也可能使用不同语言，分散在不同国家，在不同时区
* 即使是同一个用户，他可能旅游或者出差到不同时区的不同国家，他可能在手机上的Teams是中文，但是桌面版本使用英文。

所以。。。所以大家要把你的Teams app走向全世界，需要精心设计，全面考虑。Good Luck!
