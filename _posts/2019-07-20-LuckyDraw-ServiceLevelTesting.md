---
layout: post
title: Teams Bot的ServiceLevel测试
---

每一个Teams bot实际上就是一个web api服务，这个服务通过Bot Framework和Teams进行通讯，所以对于Teams app的测试就是对于一个api service的测试。

软件行业发展到如今，测试技术已经趋于成熟。单元测试，冒烟测试，整合测试。。。等等。那什么是Service level的测试。这里所谓的服务级的测试类似于Integration Test，就是指把整个api服务看成是一个黑盒，对这个服务的各个api接口作为最小单位，进行测试。与Integration Test不同之处在于，Service level测试更加侧重于服务本身，可以尽量mock掉服务的外部依赖项。

Service Level的测试在如今微服务的时代特别实用，如果使用大量的单元测试，把每个class的每个方法都层层保护，一旦将来改动了代码，对测试代码的更新也是一个较大的工作量，也就是说代码被测试限制的特别死。相反，微服务的时代因为每个服务都不会非常大，我们需要给代码一些改动的空间，我们关心的是每个api接口对于传入的输入，是否可以产生正确的输出。

而且，我在使用ServiceLevel测试对我的抽奖机器人进行测试的时候，能够很好的发现很多dead code，就是一些永远也不会被执行到的死代码。这些代码应该会删掉，保持代码的简洁。

那如何做呢？ASP.NET Core早就为我们准备好了ServiceLevel测试的利器：[TestServer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.testhost.testserver?view=aspnetcore-2.2)。微软官方文档里也有很多介绍如何使用TestServer来做整合测试，我们来看一个最简单的例子：

``` csharp
public class TestServerFixture : IDisposable
{
   private readonly TestServer _testServer;
   public HttpClient Client { get; }

   public TestServerFixture()
   {
      var builder = new WebHostBuilder()
      .UseEnvironment("Development")
      .UseStartup<Startup>();

      _testServer = new TestServer(builder);
      Client = _testServer.CreateClient();
   }

   public void Dispose()
   {
      Client.Dispose();
      _testServer.Dispose();
   }
}

[Fact]
public async Task WhenGetMethodIsInvokedWithoutAValidToken_GetShouldAnswerUnAuthorized()
{
   using (TestServerFixture fixture = new TestServerFixture())
   {
      // Act
      var response = await fixture.Client.GetAsync("/api/values/5");

     // Assert
     response
        .StatusCode
        .Should()
        .Be(HttpStatusCode.Unauthorized);
   }
}
```

当然，由于LuckyDraw bot里使用到了很多Azure table storage服务，我们在测试中，不应该使用真实的azure storage，不然多个测试用例并发执行的时候，数据肯定就乱掉了，而且会相互冲突，导致测试结果无法预料。所以在测试的时候我们需要把api服务的外部依赖项都mock掉，比如我在LuckyDraw bot里就mock了Bot connector，因为在测试中我们不能，也不应该真实的往teams里发送东西。

说了这么多，还是上代码，让大家对这个有一个更加直观的认识：

``` csharp
[Fact]
public async Task WhenEverythingIsGood_SendTextHelp_ReplyHelpMessage()
{
    using (var server = CreateServerFixture(ServerFixtureConfigurations.Default))
    using (var client = server.CreateClient())
    {
        var response = await client.SendTeamsText("<at>bot name</at>help");

        response.StatusCode.Should().Be(HttpStatusCode.OK);
        var createdMessages = server.Assert().GetCreatedMessages();
        createdMessages.Should().HaveCount(1);
        createdMessages[0].Activity.Text.Should().StartWith("Hi there, To start a lucky draw");
    }
}

public static async Task<HttpResponseMessage> SendTeamsText(
    this HttpClient httpClient,
    string text,
    string locale = null,
    double? offsetHours = null)
{
    var activity = new Activity
    {
        ServiceUrl = "https://service-url.com",
        ChannelId = "msteams",
        Type = ActivityTypes.Message,
        Text = text,
        Locale = locale ?? "en-us",
        LocalTimestamp = offsetHours.HasValue ? new DateTimeOffset(2018, 1, 1, 1, 1, 1, 1, TimeSpan.FromHours(offsetHours.Value)) : (DateTimeOffset?)null,
        From = new ChannelAccount("id", "name"),
        Recipient = new ChannelAccount("bot id", "bot name"),
        Conversation = new ConversationAccount(isGroup: true, id: "conv id", name: "conv name"),
        ChannelData = new TeamsChannelData
        {
            Tenant = new TenantInfo { Id = Guid.NewGuid().ToString() },
            Team = new TeamInfo { Id = Guid.NewGuid().ToString() },
            Channel = new ChannelInfo { Id = Guid.NewGuid().ToString() },
        }
    };

    return await httpClient.SendActivity(activity);
}
```

可以看到我们模拟了Teams的Activity，把我们自己生成的一个activity传递给了我们api接口，然后check了api发送给Teams的消息是不是我们想要的内容。
