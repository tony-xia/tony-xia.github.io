---
layout: post
title: Teams的manifest文件开始支持多语言
---

Teams发展速度飞快，Teams app的manifest文件schema也迎来了版本1.5，在这个版本里，很大的一个改进是支持多语言。

让我们一起来看看，如何在manifest文件里配置多语言。

1，我们需要先把manifest文件设置成v1.5

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.5/MicrosoftTeams.schema.json",
  "manifestVersion": "1.5",
  ...
}
```

2，设置默认的语言，假设我们这里设置成中文。注意，这里的localizationInfo是一个新的property

```json
{
  ...
  "localizationInfo": {
    "defaultLanguageTag": "zh-cn"
  }
  ...
}
```

3，增加你需要支持的其他语言

```json
{
  ...
  "localizationInfo": {
    "defaultLanguageTag": "zh-cn",
    "additionalLanguages": [
      {
        "languageTag": "fr-fr",
        "file": "fr-fr.json"
      },
      {
        "languageTag": "en-us",
        "file": "en-us.json"
      }
    ]
  }
  ...
}
```

4，在每一个不同语言的json文件里，我们就可以提供不同语言的说明文字了

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.5/MicrosoftTeams.Localization.schema.json",
  "name.short": "Le App",
  "name.full": "App pour Microsoft Teams",
  "description.short": "Créez d'excellentes applications pour Microsoft Teams avec App.",
  "description.full": "Créez de nouvelles applications Microsoft Teams, concevez et prévisualisez des cartes bot, et explorez la documentation avec App.",
  "staticTabs[0].name": "Editeur de manifest",
  "staticTabs[1].name": "Editeur de cartes",
  "staticTabs[2].name": "Bibliothèque de contrôles",
  "bots[0].commandLists[0].commands[0].title": "chercher",
  "bots[0].commandLists[0].commands[0].description": "Rechercher la documentation Teams pertinente"
}
```

需要主义的一点是，这里使用了数组的概念，比如：`staticTabs[1].name`对应的是staticTabs里第二个object里的name property。

目前Teams的App Studio还没有很好的支持这种多语言配置，不过相信按照Teams的发展速度，要不了多久就可以全面支持了，到时候大家就可以不用再直接编写json文件了。


