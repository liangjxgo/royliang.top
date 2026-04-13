---
title: "Blog Reborn: 从Hexo到Hugo，重新出发"
date: 2026-04-13
draft: false
description: "时隔8年，用Hugo重建个人博客，开启Java + AI Agent的新篇章"
tags: ["Hugo", "博客", "GitHub Pages"]
categories: ["tech"]
showToc: true
---

距离上一次写博客，已经过去8年了。

2018年那时候还是个刚入行不久的Java开发者，写了几篇关于ThreadLocal和区块链的文章就停更了。如今站在2026年，AI Agent正在重塑整个软件行业，我觉得是时候重新开始记录了。

## 为什么重建博客

几个原因：

1. **个人IP建设** — 技术博客是工程师最好的名片
2. **记录学习** — 从Java后端转型AI Agent方向，过程值得记录
3. **分享价值** — 踩过的坑、总结的经验，或许能帮到同样在路上的人

## 技术选型：Hugo + GitHub Actions

| 对比项 | Hexo (旧) | Hugo (新) |
|--------|-----------|-----------|
| 构建速度 | 较慢 | 极快 (< 100ms) |
| 依赖 | Node.js | 单个二进制文件 |
| 部署 | 手动build | GitHub Actions自动化 |
| 维护 | 需要维护源码分支 | 推送Markdown即发布 |

## 发布流程

```
写Markdown → git push → GitHub Actions自动构建 → 博客上线
```

就这么简单。

## 接下来会写什么

- Java工程师转型AI Agent的实战笔记
- Spring Boot + LangChain4j + Milvus的技术探索
- RAG、Agent工作流、工具调用的深度实践
- 生活感悟和读书笔记

欢迎来到 **Roy Liang** 的新博客。
