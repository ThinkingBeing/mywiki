---
layout: post
title: Harness Engineering - AI Agent 可靠的工程师
description: Harness engineering 是将不可靠的AI编码代理变成生产系统的学科
date: 2026-04-11
categories: harness
---

# Harness Engineering - AI Agent 可靠的工程师

## 核心观点

大多数AI编码代理可以写出漂亮的演示，但很少能直接交付生产代码。区别在于 Harness Engineering：构建让AI代理可靠运行的系统。

## 问题的根源

当处理大型结构化文件（如数千行的JSON）时，LLM的上下文窗口会被稀释。模型能正确识别要修改的节点，但会丢失对兄弟键、嵌套括号和结构完整性的跟踪。结果是文件在修改点看起来正确，但在其他地方被破坏。

提示工程无法解决这个问题 — 就像让人蒙着眼杂技然后给更多详细的手位指令。

## 四个支柱

1. 状态管理：AI代理默认无状态，长时间运行会失忆。通过序列化上下文快照实现游戏存档式恢复。

2. 上下文架构：不让代理直接生成JSON，而是调用确定性脚本。AI负责决策，脚本负责执行。

3. 护栏：物理阻止代理违反约束。文件超大小限制则linter拒绝，结构测试失败则无法合并PR。

4. 熵管理：AI会复制坏模式，产生AI垃圾。需要定期清理代理扫描代码。

## 实践效果

- 亚马逊团队：每月100+ PRs
- OpenAI：5个月100万行代码 + 1500个PRs
- Anthropic：50天发布52个功能

---
来源: strategizeyourcareer.com/p/harness-engineering-ai-agents
