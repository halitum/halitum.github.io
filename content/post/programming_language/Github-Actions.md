---
title: "初窥 Github Actions 工作流"
date: 2024-02-01
description: "Github Actions 工作流入门：什么时候 + 做什么事"
tags: ["GitHub", "CI/CD"]
categories: ["编程语言"]
---

# 初窥 Github Actions 工作流

一句话总结 Github Actions：什么时候 + 做什么事

什么时候 = git push…等操作发生时

做什么事 = CI/CD等——可以在指定服务器上运行

---

在 git 项目根目录下添加 .github/workflows/egg.yaml 文件，之后推送至github时即会自动启用 Actions

在 .yaml 中定义Actions 的工作内容：

```yaml
on: 什么时候（push等）

jobs: 做什么事
  runs-on: 在什么环境运行（使用 self-hosted 在指定服务器上运行）
  job1: 
    steps: 
      - run: 执行什么命令（eg：pwd）
      - run: ...
  job2:
    steps: 
      - run: conda env list
      ......
```

