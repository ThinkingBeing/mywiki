---
layout: post
title: Harness Engineering - AI Agent 可靠的工程师
description: Harness engineering 是将不可靠的AI编码代理变成生产系统的学科，核心是通过确定性脚本、护栏和状态管理让AI做决策而非执行
date: 2026-04-11
categories: harness
---

# Harness Engineering: Turning AI Agents Into Reliable Engineers

> 来源: strategizeyourcareer.com | https://strategizeyourcareer.com/p/harness-engineering-ai-agents | 作者: Fran

## 1. 文章基本信息

- **标题**: Harness Engineering: Turning AI Agents Into Reliable Engineers
- **来源**: strategizeyourcareer.com (Substack)
- **链接**: https://strategizeyourcareer.com/p/harness-engineering-ai-agents
- **作者**: Fran (亚马逊软件工程师)
- **发布日期**: 2026年4月

## 2. 文章要解决的核心问题（论点）

如何将不可靠的AI编码代理变成能够持续、稳定交付生产代码的系统？

## 3. 问题所处的背景上下文

- **问题起源**: 作者在亚马逊开发自动化项目，需要更新数千行的大型JSON配置文件。团队使用AI Agent直接修改JSON，结果每次都破坏文件结构，但没有报错。模型使用的是 Opus 4.6，一百万上下文窗口，不是模型能力的问题。

- **根本原因**: 当上下文窗口充满数千行结构化数据时，模型的注意力被稀释。它能正确修改目标节点，但丢失了对兄弟键、嵌套括号和整体结构完整性的跟踪。

- **业界现状**: 大多数工程师与AI编码工具的交互方式是：打开IDE，输入 prompt，审查输出，重复。对于小文件和孤立任务这种模式有效，但一旦涉及大量文件，整个方法就失效了。

## 4. 文章的核心观点

**核心观点**: Harness Engineering（套索工程）是让AI代理可靠运行的系统设计学科，核心是让AI做决策而非执行。

### 关键观点一：提示工程无法解决结构化问题

告诉模型"保留周围结构"就像让人蒙着眼杂技然后给更多手位指令。问题不在指令，而在上下文过载本身。

### 关键观点二：AI是大脑，脚本是手

关键洞察：让Agent调用脚本作为工具，而不是直接生成JSON。AI告诉脚本要改什么，脚本精确执行。这意味着AI是决策者（像CEO指示方向），不需要自己做基础工作。

### 关键观点三：四个支柱

**支柱一：状态管理（State Management）**
- AI代理默认无状态，每次API调用从零开始
- 长时间任务会"失忆"，20步前的事忘了，重复同样的错误
- 解决方案：序列化上下文快照，类似游戏存档，崩溃后可恢复

**支柱二：上下文架构（Context Architecture）**
- 不让AI直接处理结构化数据，而是通过确定性脚本
- Agent调用工具而非生成代码
- 参考 Agent Skills 标准，让agent需要时再加载技能定义

**支柱三：护栏（Guardrails）**
- 用linter、结构测试、CI job在合并前验证
- 代理不是被"劝阻"不做某事，而是被物理阻止
- 错误信息应包含修复指导，直接注入到agent上下文

**支柱四：熵管理（Entropy Management）**
- AI会复制坏模式，产生"AI slop"：冗余逻辑、冗长实现、幻觉变量
- 长期累积导致"context poisoning"
- 需要定期的清理代理（类似垃圾回收），保持代码库健康

### 关键观点四："无聊"的代码库更适合AI

Stable APIs、可预测模式、简单架构比巧妙抽象更容易被AI理解。每增加一层复杂性，AI导航难度就增加一分。

## 5. 观点对应的论据

| 观点 | 论据 |
|------|------|
| 实践效果 | 亚马逊团队：100+ PRs/月；OpenAI：5个月100万行代码+1500个PRs；Anthropic：50天52个功能 |
| 确定性脚本 | JQ命令失败只返回0或1退出码，无法让LLM恢复；Python脚本可提供详细错误信息 |
| 熵管理 | 技术债务像金钱债务，每天付一点保持健康，长期累积要花大量时间修复 |

## 6. 我的评价和启发

### 可信度评估

- 高可信。作者是亚马逊一线工程师，文章基于真实生产实践
- 提供了具体数字支撑（100+ PRs/月）
- 与Anthropic官方文章相互印证，方法论一致

### 认同与不认同

**认同**:
- 四个支柱框架清晰完整，覆盖了AI代理可靠运行的核心要素
- "AI是大脑，脚本是手"的比喻非常精准
- 熵管理是经常被忽视但极其重要的一点

**不认同/需观察**:
- 状态管理的序列化方案是否足够高效？
- 清理代理的频率和时机如何把握？

### 对我的启发

1. **架构思维**: 不要求AI做它不擅长的事（精确结构化操作），让它做擅长的事（决策、判断）
2. **约束设计**: 用系统设计而非prompt来约束AI行为
3. **前瞻性**: 设计时假设"未来AI更强"，不要把当前局限编码进基础设施

### 可以进一步探索的方向

- 如何自动化判断何时需要清理代码？
- 不同类型任务的harness设计模式差异
- 如何平衡自动化和人类审核

## 7. 延伸阅读

- [Harness Design for Long-Running Apps](https://www.anthropic.com/engineering/harness-design-long-running-apps)
- [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)