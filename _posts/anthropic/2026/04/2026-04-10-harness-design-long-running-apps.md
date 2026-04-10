---
layout: post
title: "Harness design for long-running application development"
description: Anthropic 如何设计三代理架构（Planner、Generator、Evaluator）来解决长时间运行Agent的上下文丢失和自我评估失准问题
date: 2026-04-10
categories: anthropic
---

# Harness design for long-running application development

**原文链接**: https://www.anthropic.com/engineering/harness-design-long-running-apps

**作者**: Prithvi Rajasekaran（Anthropic Labs团队）

**发布日期**: 2026年3月24日

---

## 核心内容

这篇文章介绍了Anthropic如何设计"harness"（框架/脚手架）来让Claude Agent能够自主构建完整的应用程序。

### 两大核心问题及解决方案

#### 1. Context问题
- 长时间任务中，模型会因上下文填满而失去连贯性
- 某些模型会出现"context anxiety"——接近上下文限制时过早收尾
- **解决方案**：Context Reset（重置上下文 + 结构化handoff传递状态）

#### 2. 自评问题
- Agent评估自己工作时总是过度赞美，即使质量明显一般
- **解决方案**：分离Generator（生成器）和Evaluator（评估器），让独立评估器变得"挑剔"

---

## 三代理架构

| 代理 | 职责 |
|------|------|
| **Planner** | 将需求分解为详细的规格说明书 |
| **Generator** | 根据规格实现代码 |
| **Evaluator** | 通过Playwright测试运行结果，评估是否符合标准 |

### GAN风格的对抗设计

灵感来自生成对抗网络（GAN），设计了一个多代理结构：
- **Generator**：生成前端设计/代码
- **Evaluator**：独立评估输出质量

Evaluator使用Playwright MCP直接与页面交互，评估每个迭代的设计质量。

---

## 关键经验

### 1. Sprint结构
将大型任务分解为可管理的"冲刺"，每个sprint有具体的验收标准。

### 2. 模型越强，harness可以越简单
- 在Opus 4.5上，评估器能发现很多问题
- 升级到4.6后，模型能力提升，某些任务已能独立完成，评估器变得不那么必要
- **教训**：随着模型改进，应该重新审视harness，移除不再必要的组件

### 3. 前端设计实验
- 四个评分标准：工艺、设计、原创性、功能性
- 强调设计质量和原创性，惩罚通用的"AI slop"模式
- 包含"museum quality"等表述会引导生成器朝向特定视觉风格

---

## 成果展示

### 游戏引擎示例
- **单代理运行**：核心功能（游戏模式）根本无法工作
- **三代理架构**：可以移动角色并游玩，物理效果基本可用

### Digital Audio Workstation (DAW)示例
- 运行时间：约4小时，$124 token费用
- 包含：arrangement view、mixer、transport
- 内置Agent可以通过prompt驱动音乐创作功能

---

## 总结

1. **实验很重要**：针对正在构建的模型进行实验，阅读其在实际问题上的trace，调优性能

2. **任务分解有价值**：对于复杂任务，将任务分解并为每个方面应用专门的代理是有价值的

3. **harness组合的空间不会缩小**：随着模型改进，harness的组合方式会变化，但有趣的工作是不断寻找新的组合方式

---

## 核心观点（一句话总结）

通过设计Generator-Evaluator多代理架构，解决了AI Agent在长时间任务中的上下文丢失和自我评估失准问题，并且随着模型能力提升，harness可以逐步简化。