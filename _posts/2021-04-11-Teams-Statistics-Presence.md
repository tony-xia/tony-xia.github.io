---
layout: post
title: Teams数据统计 - 用户在线离线状态
---

前几天我在wechat的moments里看到以为朋友发了腾迅会议的对用户个人的年度数据统计，看上去很有大数据感。

![TeamsStatistics](../images/post20210411/001.png)
![TeamsStatistics](../images/post20210411/002.png)
![TeamsStatistics](../images/post20210411/003.png)


实际上 Teams 也具备的类似的能力，只是它把这个能力开放给了开发人员，我们可以通过强大的 Graph API，获取大量的数据信息（当然，为了保证数据安全，你的app必须获得 tenant 管理员的同意）。

我接下来的几篇文章就集中介绍一下如何获取这些信息，有兴趣的开发者可以轻松使用这些 api 来提供用户的数据统计功能。

我们这篇文章先来介绍一下用户在线离线状态 presence。

获取某一个用户的状态

```
GET /users/{id}/presence
```

获取多个用户的状态
```
POST /communications/getPresencesByUserId

Request body:
{
	"ids": ["fa8bf3dc-eca7-46b7-bad1-db199b62afc3", "66825e03-7ef5-42da-9069-724602c31f6b", ... ]
}
```

这两个 api 都需要一个权限 `Presence.Read.All`。下面是api的返回内容：

```json
{
	"value": [{
			"id": "fa8bf3dc-eca7-46b7-bad1-db199b62afc3",
			"availability": "Busy",
			"activity": "InAMeeting"
		},
		{
			"id": "66825e03-7ef5-42da-9069-724602c31f6b",
			"availability": "Away",
			"activity": "Away"
		}
	]
}
```

可以看到 teams 把用户的状态做的很细，有两个字段 `availability` 和 `activity`。

* `availability` 可能的值有：Available, AvailableIdle, Away, BeRightBack, Busy, BusyIdle, DoNotDisturb, Offline, PresenceUnknown
* `activity` 可能的值有：Available, Away, BeRightBack, Busy, DoNotDisturb, InACall, InAConferenceCall, Inactive, InAMeeting, Offline, OffWork, OutOfOffice, PresenceUnknown, Presenting, UrgentInterruptionsOnly

这么多值，分别代表什么意思呢？在Teams里这些状态如下表：

|User configured|App configured|
|:--- |:---|
| Available| Available|
|| Available, Out of Office. (当用户设置了自动回复功能，Teams就会设置成Out of office状态) |
|  Busy |  Busy  |
|| In a call |
|| In a meeting |
|| On a call, out of office|
|  Do not disturb ||
|| Presenting|
|| Focusing. 当用户在我们的日历里设置了focus时间，Teams 就会显示这个状态 |
| Away| Away|
|| Away Last Seen *time* |
|  Be right back| |
| Appear offline| Offline. 当用户没有在任何设备登入，几分钟后就会显示这个状态 | |
|| Status unknown|
|| Out of Office |
|||

知道了这些，各位是不是已经在心里有这个统计 app 的想法了？比如可以弄一个 Azure Function，并且使用 timer trigger，每隔几分钟或者几小时，就调用上面的 api，来获取公司里用户的状态，然后保存到数据库中，后面的统计就可以从数据库里 query 了，当然为了统计的效率，可能需要对数据存储做一些优化，比如某个用户的状态如果没有变化，就不重复记录。统计时也可能需要一些复杂的 sql 语句。不过一旦你有了用户在线离线的数据，统计则是水到渠成的事情了。
