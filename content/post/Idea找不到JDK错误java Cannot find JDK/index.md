---
title: "Idea找不到JDK错误java Cannot find JDK"
date: 2024-10-28T19:30:19+08:00
draft: false
tags: ["IDEA","编译器","java"]
categories: ["java"]
typora-root-url: ./
typora-copy-images-to: ./
---

### Idea找不到JDK错误`java Cannot find JDK`

在使用`Settings Repository (Deprecated)`插件时，会出现找不到JDK错误。明明`Project Struture`中

`SDK`，`Project SDK`还有`Module Sdk`都已经配置完好，甚至Maven都可以正常编译，测试。

究其原因是Idea中的一个bug，也许不再会修复，因为`Settings Repository`插件已经废弃

#### 解决方法

1. 检查目录`%APPDATA%\JetBrains\IntelliJIdea<Version>\settingsRepository\repository\jdk.table.xml`中配置都正确

2. 检查目录`<projectdir>\.idea\misc.xml`配置也都正确
3. 检查目录`%APPDATA%\JetBrains\IntelliJIdea<Version>\options`，该目录中同样存在一个`jdk.table.xml`文件副本，而该副本配置不对（可能是以前的配置）。
4. 复制`%APPDATA%\JetBrains\IntelliJIdea<Version>\settingsRepository\repository\jdk.table.xml`文件到`%APPDATA%\JetBrains\IntelliJIdea<Version>\options`中，不需要重启，问题解决

#### 问题引用

[https://intellij-support.jetbrains.com/hc/en-us/community/posts/360009821700/comments/20662762472210](https://intellij-support.jetbrains.com/hc/en-us/community/posts/360009821700/comments/20662762472210)