---
layout: post
title: Java版本的Bot Framework SDK
---

微软为了鼓励Java开发人员开发bot，在上个月推出了Java的Bot SDK v4.6版本，目前还在Preview版本，相信不用多久就可以赶上其他版本了。

我的java还停留在 n 年前的水平，但是处于好奇，决定玩一下这套sdk。

这套sdk目前建议的java版本是 1.8或者以上，我们打开最简单的EchoBot，可以看到入口的application如下：

```java
public class Application extends BotDependencyConfiguration {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Override
    public BotFrameworkHttpAdapter getBotFrameworkHttpAdaptor(Configuration configuration) {
        return new AdapterWithErrorHandler(configuration);
    }
}
```

使用spring，并且通过重载`getBotFrameworkHttpAdaptor`方法来让bot framework获取一个Adapter。

核心的EchoBot类从ActivityHandler继承，当用户发送一条消息的时候，`onMessageActivity()`函数会被调用，通过参数`turnContext`可以获取当前的activity信息，并且和c# sdk类似，通过调用`turnContext.sendActivity()`来回复消息。

```java
@Component
public class EchoBot extends ActivityHandler {

    @Override
    protected CompletableFuture<Void> onMessageActivity(TurnContext turnContext) {
        return turnContext.sendActivity(
            MessageFactory.text("Echo: " + turnContext.getActivity().getText())
        ).thenApply(sendResult -> null);
    }

    @Override
    protected CompletableFuture<Void> onMembersAdded(
        List<ChannelAccount> membersAdded,
        TurnContext turnContext
    ) {
        return membersAdded.stream()
            .filter(
                member -> !StringUtils
                    .equals(member.getId(), turnContext.getActivity().getRecipient().getId())
            ).map(channel -> turnContext.sendActivity(MessageFactory.text("Hello and welcome!")))
            .collect(CompletableFutures.toFutureList()).thenApply(resourceResponses -> null);
    }
}
```

在EchoBot里还重载了`onMembersAdded()`方法，当有一个用户加入会话时，这个方法会被调用，但是由于java像c#那种简单的property语法和await/async语法，所以可以看到整个方法代码读起来有点累。 :(

总体上看，java sdk和其他c#, js sdk在术语和结构上基本完全一致，有其他语言bot sdk开发经验的人，使用java应该也就是一两天的适应过程。希望java sdk也能尽快赶上其他语言的sdk。

参考：
[Bot SDK Java repo](https://github.com/microsoft/botbuilder-java)
