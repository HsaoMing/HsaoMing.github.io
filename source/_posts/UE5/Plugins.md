---
title: Plugins
math: true
date: 2023-05-12 16:25:46
tags: Multiplayer Shooter
---

插件是为特定目的设计的代码和数据的集合，插件由一个或多个模块组成。该文章主要介绍了如何创建简易的联机插件。
<!--more-->

### Create Plugin
在 Edit->Plugins->Add 中可以添加空白插件，我创建了一个名为 MultiplayerSessions 的插件。为了实现联机功能，需要在 MultiplayerSessions.uplugin、MultiplayerSessions.Build.cs 中添加相关的 Plugin 和 Moudle。

```C#
// MultiplayerSessions.uplugin
"Plugin": [
    {
        "Name": "OnlineSubsystem",
        "Enabled": true
    },
    {
        "Name": "OnlineSubsystemSteam",
        "Enabled": true
    }
]

// MultiplayerSessions.Build.cs
PublicDependencyModuleNames.AddRange(
    new string[]
    {
        "Core",
        "OnlineSubsystem",
        "OnlineSubsystemSteam"
    }
    );
```

### Add Dependencies