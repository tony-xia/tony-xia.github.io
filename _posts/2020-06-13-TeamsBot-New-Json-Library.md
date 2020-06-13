---
layout: post
title: Teams Bot 如何使用新的 System.Text.Json 库
---

我最近把 LuckyDraw的代码升级到了 .net core 3.1，当然我也很想使用最新的微软json库，System.Text.Json这个库的性能比之前Newtonsoft.Json速度更快，而且就我本人爱好来说，更加喜欢System.Text.Json的命名，之前一直觉得 JObject, JArray, JToken 这些名字不够符合 c# 的 naming guideline。

微软 [这篇文章](https://docs.microsoft.com/en-us/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to) 很好的告诉大家如何将 Newtonsoft.Json 迁移到 System.Text.Json，但是如果你是用Bot SDK来开发teams bot，事情比你想象的复杂很多。

我们先来看一下bot sdk的sample code是怎么做的，打开EchoBot的代码，找到Startup.cs文件，你可以看到这么一行：

```cs
public class Startup
{
    ...
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers().AddNewtonsoftJson();
        ...
    }
}
```

现在大家明白了把，bot samples虽然都已经升级到了.net core 3.1，但是，它还是把mvc设置成使用Newtonsoft.Json。

那问题到底在哪里，为什么一定要使用Newtonsoft? 我们来看一下bot sdk源码，看一下bot framework里最核心的Activity的代码。

```cs
public partial class Activity
{
    ...
    [JsonProperty(PropertyName = "type")]
    public string Type { get; set; }

    [JsonProperty(PropertyName = "id")]
    public string Id { get; set; }

    [JsonProperty(PropertyName = "timestamp")]
    public System.DateTimeOffset? Timestamp { get; set; }

    [JsonProperty(PropertyName = "localTimestamp")]
    public System.DateTimeOffset? LocalTimestamp { get; set; }

    [JsonProperty(PropertyName = "localTimezone")]
    public string LocalTimezone { get; set; }

    [JsonProperty(PropertyName = "serviceUrl")]
    public string ServiceUrl { get; set; }

    [JsonProperty(PropertyName = "channelId")]
    public string ChannelId { get; set; }

    [JsonProperty(PropertyName = "from")]
    public ChannelAccount From { get; set; }

    ...
}
```

可以看到每个property都有一个`JsonProperty`的attribute，这个attribute是在Newtonsoft.Json里定义的，当序列化的时候，会使用指定的name作为json里的属性名字。当然在新的System.Text.Json里也有一个对应的attribute，叫`JsonPropertyName`，所以问题就来了，如果我们使用新的System.Text.Json来对一个activity对象进行serialize和deserialize，由于属性 Type 上只有`JsonProperty`并没有新的`JsonPropertyName`，serialize后，json就用了首字母大写的`{"Type":"blablabla"}`，如果是使用老的Newtonsoft.Json，那就是`{"type":"blablabla"}`。

当然不单单是JsonProperty这么一个问题，还有其他json序列化和反序列化的一些attribute也有类似问题，比如下面这两个：

```cs
namespace Microsoft.Bot.Builder.Dialogs.Adaptive.Templates
{
    [JsonConverter(typeof(ActivityTemplateConverter))]
    public class ActivityTemplate : ITemplate<Activity>
    {
        ...
    }
}
```

```cs
namespace Microsoft.Bot.Builder.Dialogs.Adaptive.Actions
{
    public class BeginDialog : BaseInvokeDialog
    {
        [JsonConstructor]
        public BeginDialog(...)
            : base(dialogIdToCall, options)
        {
            ...
        }
        ...
    }
}
```

正式由于上面这些问题，所以如果要继续拥护在mvc里使用新的System.Text.Json，同时又要使用bot sdk来做开发，那就必须在和bot sdk里一些对象打交道的时候使用老的Newtonsoft.Json。

比如以前可以这么写：

```cs
public class MessagesController : ControllerBase
{
    [HttpPost("messages")]
    public async Task<IActionResult> GetMessage([FromBody]Activity activity)
    {
        ...
    }
}
```

现在就要：

```cs
public class MessagesController : ControllerBase
{
    [HttpPost("messages")]
    public async Task<IActionResult> GetMessage()
    {
        Activity activity;
        using (var streamReader = new StreamReader(Request.Body))
        {
            var bodyString = await streamReader.ReadToEndAsync();
            activity = JsonConvert.DeserializeObject<Activity>(bodyString);
        }
        ...
    }
}
```

因为你不能再依赖于mvc来帮你deserialize出Activity对象，因为我们的mvc是使用新的System.Text.Json。

当我们要返回一个activity对象的时候，以前可以这样：

```cs
[HttpPost("messages")]
public async Task<IActionResult> GetMessage([FromBody]Activity activity)
{
    Activity repliedActivity;
    ...
    return Ok(repliedActivity);
}
```

现在就要：

```cs
[HttpPost("messages")]
public async Task<IActionResult> GetMessage()
{
    Activity repliedActivity;
    ...
    return OkFromNewtonsoftJson(repliedActivity);
}

private IActionResult OkFromNewtonsoftJson(object value)
{
    if (value == null)
    {
        return NoContent();
    }

    var json = JsonConvert.SerializeObject(value);
    return Content(json, "application/json", Encoding.UTF8);
}
```

因为我们不能再靠mvc来帮你serialize一个Activity对象，必须手动使用Newtonsoft.Json来序列化。

希望bot sdk和其他sdk，能够尽快的兼容新的Json库，这样才能使广大开发者拥抱 System.Text.Json。
