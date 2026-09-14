---
title: 从必应汇率卡片消失说起：Edge 搜索入口与 URL 参数
tags: [代码,bing,bing国际版,URL]
categories: 记录
banner_img: /img/分类背景.jpg
date: 2026-09-14 15:48:00
---

# <center>从必应汇率卡片消失说起：Edge 搜索入口与 URL 参数</center>

##### 记录切换地区和对比域名的过程。

这几天搜索汇率的时候，发现必应下面的汇率小卡片没了，之前一直是有的，结果过了好几天也没回来。

![汇率图片](img_posts/汇率图片.png)

我以为是地区的问题，就把它改成了澳大利亚，果然有了，然后我又改回中国，还是有的。

但是无论是在澳大利亚，还是在中国的区域，重新搜索，又没有了。然后一看地区，又回到了中国。

我就以为是bing路由到cn了，但是为什么改成中国还会有汇率卡片呢？并且之前也是一直有的啊。这个理由说不通。

我就去截了图片问gpt。gpt发现了一个平时不怎么注意的点。就是网址。

用表格来说就是：

| 地址栏中的域名 | 地区设置 | 汇率卡片             |
| -------------- | -------- | -------------------- |
| `cn.bing.com`  | 中国     | 没有                 |
| `www.bing.com` | 澳大利亚 | 有，默认换算成澳元   |
| `www.bing.com` | 改回中国 | 有，默认换算成人民币 |

没有汇率卡片的网址是[`cn.bing.com/search?q=`](https://cn.bing.com/search?q=)

然后有卡片的网址，是[`www.bing.com/search?q=`](https://www.bing.com/search?q=)

然后让我去测试是不是因为cn的问题，还给了我几个可疑的地方，比如`cookie`，本地代理的路由配置等问题。

我先去测试了网址的问题，结果确实如gpt所料，就是这个原因。

##### 怎样恢复地址栏搜索

因为我是在这个地址栏搜索的，所以得先看地址栏。

```
edge://settings/searchEngines
```

或者

```
edge://settings/privacy/services/search/searchEngines
```

进入地址栏。然后去看`bing`，估计就是这个地方重定向了。

| 搜索引擎 | 快捷方式 | 以%s代替查询的URL                                            |
| -------- | -------- | ------------------------------------------------------------ |
| 必应     | bing.com | {bing:cnBaseURL}search?q=%s&{bing:cvid}{bing:msb}{google:assistedQueryStats} |

我们看一下这个URL的解析：

| 模板部分                      | 作用                                       |
| ----------------------------- | ------------------------------------------ |
| `{bing:cnBaseURL}`            | 让 Edge 填入对应的 Bing 基础网址           |
| `search`                      | 访问搜索页面                               |
| `q=%s`                        | 把输入的关键词放进 `q` 参数                |
| `{bing:cvid}`、`{bing:msb}`   | 内部附加参数                               |
| `{google:assistedQueryStats}` | 与地址栏搜索建议、输入交互统计有关的占位符 |

看URL可以发现，`{bing:cnBaseURL}`为`Bing`的网址，这里写的是`cnBaseURL`。估计就是这个问题。

如果是直接修改，因为地区的问题，仍然会显示为`cnBaseURL`，所以可以新建一个。

| 搜索引擎 | 快捷方式 | 以%s代替查询的URL                                            |
| -------- | -------- | ------------------------------------------------------------ |
| 必应www  | bingwww  | https://www.bing.com/search?q=%s&{bing:cvid}{bing:msb}{google:assistedQueryStats} |

为什么要填写https://www.bing.com/ 而不是`{bing:cnBaseURL}`，是因为新建立搜索引擎的填写

`{bing:cnBaseURL}`的话，前面会自动加上`http://`，就导致没办法搜索了。所以用https://www.bing.com/ 来代替。

##### 搜索网址为什么那么长

然后就引出了下一个问题，就是按理来说搜索只需要https://www.bing.com/search?q=%s 就可以了。后面的这些东西是干什么的呢？

| 搜索引擎 | 快捷方式 | 以%s代替查询的URL                |
| -------- | -------- | -------------------------------- |
| 必应www  | bingwww  | https://www.bing.com/search?q=%s |

其实这样写也是可以的。

这些带有花括号 {...} 的字符串是 Chromium 内核浏览器（如 Edge、Chrome）的**动态宏占位符**。当你在地址栏敲下回车时，浏览器会自动捕获你当下的输入行为、推荐词点击情况以及会话状态，并将这些占位符替换成具体的跟踪与分析参数。

简短链接只保留了最基础的搜索词，而长链接附加了完整的**输入行为遥测与会话跟踪数据**。

我们以搜索后的作为对比：

搜索词语499里拉：

https://www.bing.com/search?q=499%E9%87%8C%E6%8B%89&FORM=ANAB01&PC=U531

https://www.bing.com/search?q=499%E9%87%8C%E6%8B%89&cvid=53234073e0f04b4da3d4812938c911a2&gs_lcrp=EgRlZGdlKgYIABBFGDsyBggAEEUYOzIGCAEQRRg8MgYIAhBFGD0yBggDEEUYPdIBCDEyNTNqMGo3qAIBsAIB&FORM=ANAB01&PC=U531

可以看出来，form字段和pc字段是都有的，cvid字段和gs_lcrp字段则是后面独有的。

### 参数解析

| **占位符 / 参数名**           | **对应实际参数**     | **作用与含义**                                               |
| ----------------------------- | -------------------- | ------------------------------------------------------------ |
| `{bing:cvid}`                 | `cvid=5323407...`    | **Conversation ID（会话关联 ID）**。Bing 分配的 32 位标识符，用于将你在地址栏的输入、下拉推荐与最终搜索结果页面在后台日志中串联起来。（内部用途尚不明确。） |
| `{google:assistedQueryStats}` | `gs_lcrp=EgRlZG...`  | **辅助搜索统计数据**。Chromium 原生的统计参数（经 Base64 编码的 Protobuf 数据），记录了你是否点击了下拉联想词、联想词排在第几位、输入用时等交互遥测。 |
| `{bing:msb}`                  | *(未触发时通常为空)* | **Microsoft Suggest Box**。主要用于标识微软搜索框特定联想状态或交互来源。（不确定，没找到资料） |
| *(浏览器内置固定参数)*        | `FORM=ANAB01`        | **来源入口标记（Form Code）**。告知 Bing 此次搜索来自于 Edge 浏览器的地址栏直接搜索（Omnibox）。 |
| *(浏览器内置固定参数)*        | `PC=U531`            | **客户端/渠道标识（Partner Code）**。通常指代特定版本或预装渠道的 Edge 浏览器。 |

对于这个`{google:assistedQueryStats}`来说，不意味着搜索会转交给 Google。Chromium 的实现允许它为非 Google 搜索网址补充统计参数。

并且Chromium 的实现显示，`{google:assistedQueryStats}` 可以展开成 `aqs`、`gs_lcrp` 等参数，所以模板名称和最终参数名称不一定相同。

##### 两种链接对日常搜索的影响

- **搜索结果基本是一致**：检索内容是 `q=搜索内容`，长模板额外携带了标识和搜索框统计参数；`q` 指定查询内容，搜索结果还可能受地区、语言等因素影响。
- **隐私方面**：
  - **长链接**：帮助搜索引擎优化联想词排序（机器学习反馈），但携带了较多浏览行为特征。
  - **短链接**：更干净、链接更短。