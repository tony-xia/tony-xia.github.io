---
layout: post
title: 面向全球用户的Teams app之Culture数字篇
---

我前几周在微软Ignite the Tour北京大会上，分享了如何开发一款面向全世界用户的Teams App，里面介绍了在开发Global Ready的app时会遇到的各种挑战，反响很好。所以我准备写几篇文章，将这些内容分享给没有时间参加大会的同学。

这篇我主要想讲一下，开发一款Global Ready的Teams App，在文化（Culture）方面会遇到的挑战。

假设我们现在开发了一款定时提醒软件，有一个用户给我们的bot发了一条消息：让bot在`2/10/2020 3pm`提醒此用户给朋友买生日礼物。当我们的app收到了这么一个日期时，我们如何判断这是几月几号呢？

很多在美国外企工作过的人会立刻反应过来说，这个是2020年2月10日。但是！一些在英文留学过的人会说，这个是2020年10月2日。到底哪个正确？

实际上这两个都是正确的，因为在美国等一些国家，他们的日期格式是`月 / 日 / 年`，但在另一些国家，比如英国，澳大利亚，他们的日期格式是`日 / 月 / 年`，所以，当我们去解析这个日期格式的时候需要特别注意当前用户所在的国家区域，根据用户的culture不同，对日期字符串进行不同的解析。

看完日期，再来看一下数字。当用户发文字消息给你的bot，说帮我转账`295,000`元，那是指多少钱？很多同学会问：这个难道不是29万5千吗？

实际上，不同国家对于数字的分割和不同，比如下面这几个：
* Canadian (French): 4 294 967 295,000
* German: 4 294 967.295,000
* Italian: 4.294.967.295,000
* Great Britain, United States: 4,294,967,295.00

可以看到有些是每三位一个逗号，有些是一个点，有些是空格，有些国家小数点是一个点，有些小数点是一个逗号。所以回到上面这个数字，到底是295元，还是29万5千元？这个和前面的日期格式一样，必须根据当前用户所在的culture来定。

我们再看一下不同国家的货币：
* Russia: 2,25 €
* Great Britain: €2.25
* Germany: € 2,25

可以看到，除了上面说的小数点不同，不同国家对于货币符号放在数字前面还是后面也很不同。像俄罗斯就是放在后面的。

对于上面这些情况，我们在开发的时候如何处理呢？我建议对于时间，数字的处理，使用成熟的library，千万不要自己去写，不要重复造轮子。别人的library已经试错，改进了成百上千次了。你自己开发的不经历类似的过程，很难达到类似的质量。

如果大家对Teams app开发感兴趣，强烈推荐中国微软的牛人Ares陈老师最近出了一套的Teams开发系列视频讲座：[Microsoft Teams开发入门和实践 https://aka.ms/teamsdev163study](https://aka.ms/teamsdev163study) ，从入门到精通Teams开发，对于准备从事或者正在从事Teams app开发的同学来说，必看！