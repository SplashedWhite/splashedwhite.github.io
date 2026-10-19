---
title: gpt新模型没出现的更新cli即可
tags: [代码,ChatGPT,codex,ai,cli,gpt]
categories: 记录
banner_img: /img/分类背景.jpg
date: 2026-10-19 14:55:00
---

# <center>gpt新模型没出现的更新cli即可</center>

新模型也是如此

💦一帖
没有gpt6 sol的更新一下cli就好了

win+R打开cmd：

cmd命令为：
```
codex update
```

查看版本的命令为：
```
codex --version
```

然后看看版本，实测codex-cli 0.156.1是有的

mac也是一样的：

`Command + 空格`，搜索并打开“终端”，输入`codex update`就行了。

或者用这个`npm install -g @openai/codex@latest`也是一样的。

输入`codex update`就是执行的`npm install -g @openai/codex@latest`

`codex update`和`npm install -g @openai/codex@latest`的效果是一样的。如果`codex update`不行的话，那就输入npm那个。