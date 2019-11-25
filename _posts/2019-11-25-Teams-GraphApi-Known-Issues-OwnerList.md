---
layout: post
title: Teams团队的成员列表API的已知问题
---

如果大家经常使用Graph API来对Teams进行操作管理的话，有时候会遇到一些奇怪的问题，我前两天还在Stack Overflow上回答了一个用户的问题，这个问题我自己也遇到过。所以我想用这篇文章来分享一下，万一以后大家遇到类似的问题，就有所防备。

这个问题实际上是一个已知问题，官方的解释在[这里](https://docs.microsoft.com/en-us/graph/api/resources/teams-api-overview?view=graph-rest-beta#membership-changes-in-microsoft-teams)

> We recommend that when you add an owner, you also add that user as a member. If a team has an owner who is not also a member, ownership and membership changes might not show up immediately in Microsoft Teams. In addition, different apps and APIs will handle that differently. For example, Microsoft Teams will show teams that the user is either a member or an owner of, while the Microsoft Teams PowerShell cmdlets and the /me/joinedTeams API will only show teams the user is a member of. To avoid confusion, add all owners to the members list as well.

讲的是说，如果我们需要给一个团队（team）增加一个owner，那你不单单需要把这个用户加入到owner列表，还需要把他加入到member成员列表。如果你直加了一个列表，那就可能有问题。不同的程序对于这个用户的判断会不一致。

比如：你把用户A加入到一个团队（team）的owner列表，但是没有把这个用户加入到成员member列表。那么，你使用powershell的cmdlet，和/me/joinedTeams这个API就会认为这个用户A还不在这个团队里。

所以，为了防止这类事情的发生，建议的做法是：如果你要把用户加入到owner列表，那同时你也应该把这个用户加入到成员列表。

另外，还有一个已知问题，我之前也遇到了。官方是这么说的：

> Known issue: when DELETE /groups/{id}/owners is called, the user is also removed from the /groups/{id}/members list. To work around this, we recommend that you remove the user from both owners and members, then wait 10 seconds, then add them back to members.

意思是：当你想把一个用户移出owner列表，也就是说这个用户不在是这个团队的拥有者。我们通常会使用这个API

```
DELETE /groups/{id}/owners/{userId}/$ref
```

但是诡异的地方是，如果你这么做了之后，这个用户会自动从这个团队的member成员列表里移出，也就是说，他连普通的成员身份都没有了。也就是说相当于自动帮你调用了`DELETE /groups/{id}/members/{userId}/$ref`接口。

目前官方建议的workaround是当你调用`DELETE /groups/{id}/owners/{userId}/$ref`把这个用户移出owner拥有者列表后，等10秒钟（注意，官方建议要等10秒钟，不能立刻），然后再调用下面这个接口来吧用户加回到member成员列表。

```
POST /groups/{id}/members/$ref
```

希望这两个已知问题能尽快得到产品团队的解决，也希望这篇文章能在大家遇到类似的问题时，对大家有帮助。另外，强烈推荐[Microsoft Teams开发入门和实践](https://edu.csdn.net/course/detail/26738) 这套视频教程，从入门到精通Teams开发！

