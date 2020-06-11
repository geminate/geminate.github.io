---
layout: post
title: npm scripts 同时执行多个监听任务
categories: Node
no-post-nav: true
---

在前端开发的过程中，一个比较常见的需求是启动项目 server 监听的同时启动一个 mock server 模拟数据。之前的做法一直是同时开两个控制台窗口分别运行两条 npm 命令。但这样做操作起来比较麻烦。

经过网上查询后，发现了几种回答(使用‘&’符号连接多个命令等)，但经过测试均不能在 windows 下完美实现。因此我去查询了npm 相关的包，经过测试后推荐使用 `concurrently` 或 `npm-run-all` 。

