---
layout: post
title: Bot Service自带的数据分析统计功能
---

每个产品上线后都希望自己能实时看到多少用户在使用我的产品，我的服务，有多少使用量，有没有遇到问题。市面上做用户数据、行为分析的公司也不少，但是大多数都需要我们修改一些代码来集成第三方的sdk库。

我的teams app上线后也急切希望看到有多少用户在使用的我的app，但是我又懒得改动代码。找了一下，突然发现实际上azure的bot service自带有一个简单的用户分析统计功能。

![analytics](../images/post20190831/001.png)

但是这个功能默认情况不开启，怎么开启呢？我们先需要创建一个application insights。

创建完后，在这个application insights里选择"API Access"这个菜单。

![analytics](../images/post20190831/002.png)

点击"Create API Key"。输入你自己好记得名字，就是你这个key是用来做什么的。把下面都打上勾

![analytics](../images/post20190831/003.png)

创建完后记住你的key。然后回到Bot Service资源，点击Settings菜单。

把刚才生成的那个key填入：Application Insights API key。另外还需要把这个Application Insightsde的ID号也填入：Application Insights Application ID字段

![analytics](../images/post20190831/004.png)

经过几天数据积累后，再回到Analytics菜单，你就可以看到你的用户数据啦

![analytics](../images/post20190831/005.png)

![analytics](../images/post20190831/006.png)

![analytics](../images/post20190831/007.png)

是不是很方便？还可以方便的查看用户流失率，方便及时调整你的teams app。
