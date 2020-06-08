---
layout: post
title: Teams Bot开发系列：Teams的Activity处理
---

上一篇文章讲了activity处理的流程，我们bot的核心处理逻辑放在ActivityHandler的子类里，通过重载`OnMessageActivityAsync()`方法来实现。

这篇文章我来讲一下对于Teams的bot来说，整个处理的逻辑会有哪些不同点。

通过之前的文章，大家应该已经知道，Teams bot是Azure bot service支持的众多bot聊天平台里的一种channel（注意：这里的channel指bot service里的channel，和Teams里的channel是完全不同的概念）。但是Teams实际上提供了很多特有的事件和动作。使用bot sdk的通用模型，我们当然可以处理这些事情，但是Teams作为微软的主打产品，微软的bot sdk当然要为它提供更多的开发便利性。

SDK提供了一个针对Teams的ActivityHandler。这个handler有下面这些特殊的ConversationUpdateActivity的处理函数

| 事件 | 函数 | 说明 |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedAsync` | 当Teams的channel被创建 |
| channelDeleted | `OnTeamsChannelDeletedAsync` | 当Teams的channel被删除 |
| channelRenamed | `OnTeamsChannelRenamedAsync` | 当Teams的channel被重命名 |
| teamRenamed | `OnTeamsTeamRenamedAsync` | 当Teams的一个team被重命名 |
| MembersAdded | `OnTeamsMembersAddedAsync` | 当Teams的一个team中有新用户加入 |
| MembersRemoved | `OnTeamsMembersRemovedAsync` | 当Teams的一个team中有用户被移除 |

除了ConversationUpdateActivity这些Teams的特殊事件，handler还提供了一些Teams特有的invoke动作的处理

| Invoke类型                    | 函数                              | 说明                                                  |
| :-----------------------------  | :----------------------------------- | :----------------------------------------------------------- |
| CardAction.Invoke               | `OnTeamsCardActionInvokeAsync`       | 关于卡片的动作，比如卡片上一个按钮被点击了 |
| fileConsent/invoke              | `OnTeamsFileConsentAcceptAsync`      | 用户同意了上传文件 |
| fileConsent/invoke              | `OnTeamsFileConsentAsync`            | 用户要上传文件. |
| fileConsent/invoke              | `OnTeamsFileConsentDeclineAsync`     | 用户拒绝了上传文件. |
| actionableMessage/executeAction | `OnTeamsO365ConnectorCardActionAsync` | O365连接器的卡片动作 |
| signin/verifyState              | `OnTeamsSigninVerifyStateAsync`      | 登入验证状态 |
| task/fetch                      | `OnTeamsTaskModuleFetchAsync`        | Teams的Task Module的获取 |
| task/submit                     | `OnTeamsTaskModuleSubmitAsync`       | Teams的Task Module的提交 |

上面表格中的`OnTeamsFileConsentAsync`实际上是`OnTeamsFileConsentAcceptAsync`和`OnTeamsFileConsentDeclineAsync`的一个综合处理，你可以重载`OnTeamsFileConsentAsync`，或者分别重载 accept 和 decline 函数。下面的sdk代码可以让你有直观的了解

```cs
protected virtual async Task<InvokeResponse> OnTeamsFileConsentAsync(ITurnContext<IInvokeActivity> turnContext, FileConsentCardResponse fileConsentCardResponse, CancellationToken cancellationToken)
{
    switch (fileConsentCardResponse.Action)
    {
        case "accept":
            await OnTeamsFileConsentAcceptAsync(turnContext, fileConsentCardResponse, cancellationToken).ConfigureAwait(false);
            return CreateInvokeResponse();
        case "decline":
            await OnTeamsFileConsentDeclineAsync(turnContext, fileConsentCardResponse, cancellationToken).ConfigureAwait(false);
            return CreateInvokeResponse();
        default:
            throw new InvokeResponseException(HttpStatusCode.BadRequest, $"{fileConsentCardResponse.Action} is not a supported Action.");
    }
}
```

对于喜欢把问题研究透彻的朋友可能会问，Teams的ActivityHandler到底是怎么处理的？让我们跳入sdk源代码一探究竟。

```cs
public class TeamsActivityHandler : ActivityHandler
{
    protected override async Task<InvokeResponse> OnInvokeActivityAsync(ITurnContext<IInvokeActivity> turnContext, CancellationToken cancellationToken)
    {
        ...
        switch (turnContext.Activity.Name)
        {
            case "fileConsent/invoke":
                return await OnTeamsFileConsentAsync(turnContext, SafeCast<FileConsentCardResponse>(turnContext.Activity.Value), cancellationToken).ConfigureAwait(false);

            case "task/fetch":
                return CreateInvokeResponse(await OnTeamsTaskModuleFetchAsync(turnContext, SafeCast<TaskModuleRequest>(turnContext.Activity.Value), cancellationToken).ConfigureAwait(false));

            case "task/submit":
                return CreateInvokeResponse(await OnTeamsTaskModuleSubmitAsync(turnContext, SafeCast<TaskModuleRequest>(turnContext.Activity.Value), cancellationToken).ConfigureAwait(false));

            ......

            default:
                return await base.OnInvokeActivityAsync(turnContext, cancellationToken).ConfigureAwait(false);
        }
        ...
    }

    protected override Task OnConversationUpdateActivityAsync(ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
    {
        ...
        switch (channelData.EventType)
        {
            case "channelCreated":
                return OnTeamsChannelCreatedAsync(channelData.Channel, channelData.Team, turnContext, cancellationToken);

            case "channelDeleted":
                return OnTeamsChannelDeletedAsync(channelData.Channel, channelData.Team, turnContext, cancellationToken);

            case "channelRenamed":
                return OnTeamsChannelRenamedAsync(channelData.Channel, channelData.Team, turnContext, cancellationToken);

            case "teamRenamed":
                return OnTeamsTeamRenamedAsync(channelData.Team, turnContext, cancellationToken);

            default:
                return base.OnConversationUpdateActivityAsync(turnContext, cancellationToken);
        }
        ...
    }
}
```

从上面的代码里可以看到没有什么特别的magic，TeamsActivityHandler重载了`OnConversationUpdateActivityAsync`，并且根据`channelData.EventType`判断出不同teams的事件，然后调用相应的方法。对于invoke也类似，重载了`OnInvokeActivityAsync`，根据turnContext.Activity.Name来调用不同的方法。

回到我们的EchoBot代码，让EchoBot从`TeamsActivityHandler`继承下来，然后我们可以添加`OnTeamsChannelRenamedAsync`方法。把EchoBot设置到Teams里，修改安装了EchoBot的channel的名字，就可以看到这个方法被促发的。

```cs
public class EchoBot : TeamsActivityHandler
{
    protected virtual Task OnTeamsChannelRenamedAsync(ChannelInfo channelInfo, TeamInfo teamInfo, ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
    {
        var replyText = "Channel renamed.";
        await turnContext.SendActivityAsync(MessageFactory.Text(replyText, replyText), cancellationToken);
    }
}
```
