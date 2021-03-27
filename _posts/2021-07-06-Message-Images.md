---
layout: post
title: 给Teams消息附加图片的三种方式
---

Teams消息支持三种不同的方式来添加图片，这篇文章我们来一起看一下这三种方式。

1. Inline图片

```csharp
var imagePath = Path.Combine(Environment.CurrentDirectory, "abc.png");
var imageData = Convert.ToBase64String(File.ReadAllBytes(imagePath));
var image = new Attachment
{
    Name = @"Resources\abc.png",
    ContentType = "image/png",
    ContentUrl = $"data:image/png;base64,{imageData}",
};

reply = MessageFactory.Text("Hello world");
reply.Attachments = new List<Attachment>() { image };
```

可以看到这种方式讲图片内嵌到消息里，使用base64的方式来对图片进行编码，在url字段，把编后的base64字符串附加到“data:image/png;base64,”后面。

这种方式适合文件大小较小的图片，因为base64编码本身就会把数据扩大三分之一左右，如果一个图片有600kb，那base64之后，这条消息就起码有800kb多了。

2. 上传图片到服务器

```csharp
var imagePath = Path.Combine(Environment.CurrentDirectory, "abc.png");

var connector = turnContext.TurnState.Get<IConnectorClient>() as ConnectorClient;

var attachments = new Attachments(connector);

var response = await attachments.Client.Conversations.UploadAttachmentAsync(
    conversationId,
    new AttachmentData
    {
        Name = @"Resources\abc.png",
        OriginalBase64 = File.ReadAllBytes(imagePath),
        Type = "image/png",
    },
    cancellationToken);

var attachmentUri = attachments.GetAttachmentUri(response.Id);

var image = new Attachment
{
    Name = @"Resources\abc.png",
    ContentType = "image/png",
    ContentUrl = attachmentUri,
};

reply = MessageFactory.Text("Hello world");
reply.Attachments = new List<Attachment>() { image };
```

在这种方式下，先将本地的图片文件上传到conversation里，拿到返回的图片uri，然后在 attachment 的 ContentUrl 里指定图片的url。这种方式是我个人比较推荐的方式。当然这个图片也可以不在本地计算机里，也可以是互联网上的某张图片，我们先把图片下载下来，拿到图片的内容后，就可以调用 UploadAttachmentAsync() 来进行上传了。

3. 直接使用网上的图片url

```csharp
var image new Attachment
{
    Name = @"Resources\abc.png",
    ContentType = "image/png",
    ContentUrl = "https://www.blablabla.com/abc.png",
};

reply = MessageFactory.Text("Hello world");
reply.Attachments = new List<Attachment>() { image };
```

可以看到这种方式和第二种的最后一段很像，实际上第二种方式就是多了一步上传过程，拿到上传的图片url后，就和这第三种方式是一样的了。

那我为什么不推荐这种方式呢？因为这种方式实际上是让teams来自己去从网上download那张图片，但是如果teams没有权限访问那个图片url，那虽然这条消息可以发送成共，但是当用户查看这条消息的时候，就会无法显示图片。但第三种方式也有优点，如果这个图片会改变，那用户就能每次看到不同的内容了。

看完这三种方法后，我相信大家已经知道在什么场景下选择什么图片了。 :)
