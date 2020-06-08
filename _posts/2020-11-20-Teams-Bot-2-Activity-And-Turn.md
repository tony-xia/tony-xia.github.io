---
layout: post
title: Teams Bot系列开发：Activity和Turn
---

这篇文章我们来说一下Activity和Turn这两个bot framework中最重要的两个概念，同时也介绍一下TurnContext和BotAdapter

## Activity
一个activity是聊天双方的一个信息载体，它可以是一条消息，也可以是一个动作。比如用户给bot发送一条文字消息，这就是一个activity，bot给用户回复一张图片，那这是另一个activity。

Activity是bot framework里最重要的概念，让我们来一起看一下c# sdk里对activity的定义。真实感受一下

```cs
    public class Activity
    {
        public string Type { get; set; }
        public string Id { get; set; }
        public DateTimeOffset? Timestamp { get; set; }
        public DateTimeOffset? LocalTimestamp { get; set; }
        public string LocalTimezone { get; set; }
        public string ServiceUrl { get; set; }
        public string ChannelId { get; set; }
        public ChannelAccount From { get; set; }
        public ConversationAccount Conversation { get; set; }
        public ChannelAccount Recipient { get; set; }
        public string TextFormat { get; set; }
        public string AttachmentLayout { get; set; }
        public IList<ChannelAccount> MembersAdded { get; set; }
        public IList<ChannelAccount> MembersRemoved { get; set; }
        public IList<MessageReaction> ReactionsAdded { get; set; }
        public IList<MessageReaction> ReactionsRemoved { get; set; }
        public string TopicName { get; set; }
        public bool? HistoryDisclosed { get; set; }
        public string Locale { get; set; }
        public string Text { get; set; }
        public string Speak { get; set; }
        public string InputHint { get; set; }
        public string Summary { get; set; }
        public SuggestedActions SuggestedActions { get; set; }
        public IList<Attachment> Attachments { get; set; }
        public IList<Entity> Entities { get; set; }
        public object ChannelData { get; set; }
        public string Action { get; set; }
        public string ReplyToId { get; set; }
        public string Label { get; set; }
        public string ValueType { get; set; }
        public object Value { get; set; }
        public string Name { get; set; }
        public ConversationReference RelatesTo { get; set; }
        public string Code { get; set; }
        public DateTimeOffset? Expiration { get; set; }
        public string Importance { get; set; }
        public string DeliveryMode { get; set; }
        public IList<string> ListenFor { get; set; }
        public IList<TextHighlight> TextHighlights { get; set; }
        public SemanticAction SemanticAction { get; set; }
        public string CallerId { get; set; }
    }
```

除了上面这么多的属性，还有很多方法和扩展方法，是一个非常大的类，我们以后详细一一说明。

## Turn
下面是对于turn的官方说明：

> In a conversation, people often speak one-at-a-time, taking turns speaking. With a bot, it generally reacts to user input. Within the Bot Framework SDK, a turn consists of the user's incoming activity to the bot and any activity the bot sends back to the user as an immediate response. You can think of a turn as the processing associated with the arrival of a given activity.

一个turn就是对一个activity的相关处理。比如说，对于用户发给我们bot一条消息，我们把这条消息转发给另一个用户，同时对消息本身点赞，那这两个操作一起是一个turn，是针对用户发来的消息的处理的turn。

## Turn上下文(Turn Context)

我认为TurnContext是仅次于Activity的概念，turn context对象跟随着整个消息的处理过程。地位非常类似以前asp.net的HttpContext概念。

```cs
    public interface ITurnContext
    {
        BotAdapter Adapter { get; }
        TurnContextStateCollection TurnState { get; }
        Activity Activity { get; }
        bool Responded { get; }

        Task<ResourceResponse> SendActivityAsync(...);
        Task<ResourceResponse> UpdateActivityAsync(...);
        Task DeleteActivityAsync(...);

        ITurnContext OnSendActivities(...);
        ITurnContext OnUpdateActivity(...);
        ITurnContext OnDeleteActivity(...);
    }
```

可以看到TurnContext不算庞大，其他的属性和方法看名字比较容易理解。其中一个主要概念是BotAdapter

## BotAdapter

从官方说明，我们可以看到这个是链接你bot endpoint和bot处理逻辑的一个桥梁，它封装了对于安全验证和对middleware的处理。当你的bot endpoint收到一个请求时，BotAdapter会验证这个请求是不是从正式渠道发送来的请求，如果是，它会创建Turn Context，然后开始各个middleware的处理。


> 要注意的一点是：Activity是贯穿于Azure bot service和bot framework sdk整条pipeline的，但是turn，turn context和BotAdapter，目前版本（v4）只是存在于bot framework sdk，是帮助开发人员处理bot的一个东西，如果你不想用turn，想自己处理整个bot的聊天，你可以只使用 `Microsoft.Bot.Schema`（包含Activity定义），不添加`Microsoft.Bot.Builder`库（含有turncontext和BotAdapter定义）。
