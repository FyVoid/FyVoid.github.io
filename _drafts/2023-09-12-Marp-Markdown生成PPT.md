---
layout: post
title: "Marp-通过Markdown生成PPT"
date: 2023/09/12 17:20:00 +0800
---

> 最近发现一个能从markdown文档生成PPT格式的工具
>
> 准确的说是用markdown语法制作PPT的工具，不过因为并没有PPT好用，更适合从已有的Markdown文档快速生成Slide

## 环境配置

* VSCode
* VSCode插件：Marp for VSCode
* Markdown编辑器（推荐Typora）

> 学习本工具前推荐先掌握常用的markdown语法

## 从Markdown到PPT

由于Marp的简单功能内容很少，我们直接从一个[Markdown文件]({{site.baseurl}}/assets/Marp-example.md)和一个[背景图片]({{site.baseurl}}/assets/Marp-bgi.jpeg)开始构建一个PPT

### 基础设置

打开VSCode，在markdown文件前输入：

```
---
marp: true
theme: gaia
paginate: true
header: 用iverilog和gtkwave在Mac平台本地仿真
backgroundColor: #fff
---
```

以YAML front-matter对PPT进行初始设置

* marp: true 告诉vscode启用marp的预览，在markdown预览中就能看到生成PPT的效果
* theme 设置PPT的格式，可以设置为default、gaia或uncover，默认为default
* paginate 设置页码
* header 设置抬头内容，此处设置为标题
* backgroundColor 设置背景颜色

### directive

设置完后打开vscode的markdown预览，会发现：

![marp1]({{site.baseurl}}/assets/marp1.png)

只有一张PPT，并且十分混乱。这是因为默认下并不会自动根据Markdown文件中的标题分页，要按标题分页的话，输入

```
<!-- headingDivider: 2 -->
```

会发现PPT被按照二级标题分成了4张，headingDivider为marp中的一个**directive**，对其进行设置会改变自动生成的结果，设置为2，则1、2级标题会自动分页，对directive设置的格式显然为：
```
<!-- directiveName: directiveValue -->
```

当然，这不是我们想要的PPT，删除这一行，我们重新手动分页

### 分页

分页的语法非常简单，在需要分页的部分输入

```
---
```

就可以进行一次分页。

