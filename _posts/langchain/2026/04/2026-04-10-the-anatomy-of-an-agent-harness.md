---
layout: post
title: "The Anatomy of an Agent Harness"
description: 解析 Agent 的核心组成部分：Model + Harness，Harness 如何通过文件系统、代码执行、内存管理、上下文工程等机制将模型智能转化为实用系统
date: 2026-04-10
categories: langchain
---

# The Anatomy of an Agent Harness

**原文链接**: https://blog.langchain.com/the-anatomy-of-an-agent-harness/

**作者**: Vivek Trivedy

---

## 核心观点

**Agent = Model + Harness**

如果你不是模型，那就是 harness。Harness 包括所有不是模型本身的代码、配置和执行逻辑。

- **模型**：包含智能
- **Harness**：使智能变得有用的系统

---

## 核心 Harness 组件

### 1. 文件系统（Filesystem）

**需求**：Agent 需要持久化存储来接口真实数据、卸载不适合放入上下文的信息、跨会话持久化工作。

**解决方案**：Harness 提供文件系统抽象和工具。

文件系统是最基础的 harness 原语，因为它解锁了：
- 持久化存储
- 版本控制（Git）
- 跨会话状态共享

### 2. 代码执行（Code Execution）

**需求**：Agent 需要能够自主解决问题，而不需要人类预先设计每一个工具。

**解决方案**：Bash + 代码执行工具。

这是"给模型一台电脑"的关键步骤，让模型可以通过代码自主设计工具，而不是被限制在预配置的工具集中。

### 3. 沙箱环境（Sandbox）

**需求**：Agent 需要安全的环境来执行操作。

**解决方案**：使用沙箱而非本地执行。

- 安全隔离执行代码
- 白名单命令
- 网络隔离
- 按需创建/销毁环境，支持横向扩展

### 4. 内存与知识（Memory & Knowledge）

**需求**：Agent 应该记住见过的东西，并访问训练时不存在的信息。

**解决方案**：
- **持久化内存**：AGENTS.md 文件标准，跨会话注入知识
- **最新知识**：Web Search 和 MCP 工具（如 Context7）访问知识截止日期后的信息

### 5. 上下文管理（Context Engineering）

**需求**：Agent 性能不应随工作时间推移而下降。

**问题**：Context Rot（上下文腐化）——模型在上下文窗口填满时推理和完成任务的能力变差。

**解决方案**：
- **Compaction（压缩）**：智能卸载和总结现有上下文
- **Tool Call Offloading**：将大型工具输出卸载到文件系统，只保留头尾token
- **Skills**：渐进式披露，避免启动时加载过多工具

### 6. 长期工作（Long-horizon Work）

**需求**：Agent 需要能够自主完成复杂工作，跨越长时间范围。

**解决方案**：
- **文件系统 + Git**：跨会话跟踪工作
- **Ralph Loop**：拦截模型退出尝试，强制继续工作
- **规划与自验证**：分解目标、检查正确性

---

## Model 与 Harness 的协同进化

1. 有用的原语被发现
2. 添加到 harness 中
3. 用于训练下一代模型

**但这有副作用**：改变工具逻辑会导致模型性能下降。

**重要发现**：最佳 harness 不一定是模型后训练时使用的 harness。例如 Opus 4.6 在 Claude Code 中得分远低于在其他 harness 中。

---

## 结论

- **模型会吸收更多功能**：随着模型变强，部分 harness 功能会被吸收
- **但 harness 仍然有价值**：良好的环境配置、正确的工具、持久化状态和验证循环使任何模型更高效
- **Harness 工程仍是活跃研究领域**

---

## 核心观点（一句话总结）

Agent = Model + Harness，Harness 是围绕模型构建的系统，通过文件系统、代码执行、内存管理、上下文工程等机制将模型智能转化为实用工具，即使模型变强，Harness 优化仍能显著提升性能。