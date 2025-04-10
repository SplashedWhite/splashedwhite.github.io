---
title: VScode插件Code Runner运行python输出窗口出现中文乱码
tags: [代码]
categories: 代码
banner_img: /img/关于背景.jpg
date: 2025-04-10 13:27:41
---
# <center>VScode插件Code Runner运行python输出窗口出现中文乱码</center>

## 问题描述
用Code Runner运行python程序时，输出界面中文显示乱码，如下图所示：

![问题描述](/img_9_code_runner/image-3.png)

但是如果用终端输出，那么中文就是正常的，如下图所示：

![终端输出](/img_9_code_runner/image-6.png)

## 原因
编码原因，更改为utf-8编码即可。

具体步骤如下：

## 解决办法
首先打开vscode的“**管理**”——“**设置**”

然后找到“**扩展**”

找到**Run Code configuration**

点击**Executor Map**下的  “**在settings.json中编辑**”  。如图所示：
![编辑settings.json](/img_9_code_runner/屏幕截图%202025-04-10%201339171.png)

![code-runner.executorMap](/img_9_code_runner/屏幕截图%202025-04-10%20132428.png)

找到"**code-runner.executorMap**"

找到"**python": "python -u**"

修改python的编码格式：将**python -u** 改为 **set PYTHONIOENCODING=utf8 && python**

![修改格式](/img_9_code_runner/image.png)

保存之后重新运行程序即可。

![成功](/img_9_code_runner/image-3.png)

参考出处：

[解决VS Code安装Code Runner插件后，输出窗口中文乱码问题（Python）](https://blog.csdn.net/ljb0077/article/details/112980095)
https://blog.csdn.net/ljb0077/article/details/112980095