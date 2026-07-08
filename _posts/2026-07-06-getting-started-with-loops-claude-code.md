---
layout: post
title: "Claude Code 官方：Getting started with loops 解读"
description: "Claude Code 团队把 coding agent 的 loop 定义为“重复执行直到满足停止条件的工作周期”，并按触发方式、停止条件、底层原语和适用任务划分为 turn-based、goal-based、time-based 与 proactive loops。"
date: 2026-07-06
created_at: 2026-07-08 08:10:50 +0800
categories: AI Agents
---

# Claude Code 官方：Getting started with loops 解读

> 来源: ClaudeDevs / X Article | [原文链接](https://x.com/ClaudeDevs/status/2074208949205881033) | 发布日期: 2026-07-06

## 1. 文章基本信息

- **标题**: Getting started with loops
- **来源**: ClaudeDevs（Claude Code 开发者官方账号）
- **链接**: https://x.com/ClaudeDevs/status/2074208949205881033
- **作者**: @delba_oliveira
- **发布日期**: 2026-07-06
- **公开数据（抓取时）**: 约 437 万浏览、1.47 万赞、3.23 万收藏、1730 转发、280 回复
- **主题**: Claude Code、AI coding agent、agent loops、/goal、/loop、/schedule、dynamic workflows、skills、token usage

## 2. 文章要解决的核心问题（论点）

这篇文章试图回答的问题是：**当大家都在谈“设计 loop”而不只是写 prompt 时，Claude Code 团队到底如何定义 loop、如何分类，以及工程师应该在什么场景使用哪一种 loop？**

一句话概括：**loop 是让 agent 重复执行“理解任务 → 调用工具 → 检查结果 → 继续或停止”的工作周期；设计 loop 的关键不是让 agent 无限自动化，而是明确触发条件、停止条件、验证方式和成本边界。**

## 3. 问题所处的背景上下文

AI 编程工具正在从“聊天式助手”走向“半自主工程系统”。过去我们主要优化的是单次 prompt：如何描述需求、如何给上下文、如何让模型输出更稳定。随着 Claude Code、Codex、Cursor 等工具越来越强，新的瓶颈变成了：**谁来决定下一步？谁来判断完成？谁来在无人值守时触发任务？谁来控制成本和质量？**

这就是 loop 话题兴起的原因。一个 agent 如果只能响应用户的一句话，它仍然受限于人的注意力；如果它能在明确边界内循环工作，就可以承担更多工程流程，例如每天总结消息、持续跟踪 PR、处理 bug report、迁移依赖、批量修复测试等。

但 loop 也带来风险：如果没有明确停止条件，agent 会浪费 token；如果没有可验证标准，它会过早停止或越做越偏；如果没有质量控制，多轮自动化会把小错误放大成系统性问题。因此 Claude Code 团队在这篇文章中给出了一套实用分类。

## 4. 文章的核心观点

### 4.1 Claude Code 对 loop 的定义

文章把 loop 定义为：**agent 重复执行一系列工作周期，直到满足某个停止条件。**

他们不是抽象地谈“自主性”，而是从四个工程维度拆解 loop：

1. **如何触发**：由用户 prompt 触发、由目标触发、由时间间隔触发，还是由事件/计划任务触发。
2. **如何停止**：模型自判完成、达到目标、达到最大轮数、任务队列清空、人工取消等。
3. **使用什么 Claude Code 原语**：普通对话、`/goal`、`/loop`、`/schedule`、dynamic workflows、auto mode 等。
4. **适合什么任务**：短任务、可验证任务、周期性任务、长期无人值守任务。

这个定义的价值在于：它把“agent 自动干活”拆成了可设计、可调试、可计费、可治理的工程构件。

### 4.2 Turn-based loops：每次 prompt 都是最基础的手动循环

- **触发方式**：用户发出 prompt。
- **停止条件**：Claude 判断任务完成，或需要更多上下文。
- **适用场景**：短任务、一次性任务、不属于固定流程的工作。
- **成本控制**：写更具体的 prompt，并把人工验证步骤沉淀为 skills，减少来回轮数。

文章提醒我们：即使普通对话也是一种 loop。你让 Claude 加一个 like button，它会读代码、修改、运行测试、检查结果，然后回复。接下来你再人工检查并发下一条 prompt。

这里的关键优化点不是“写更长 prompt”，而是把你每次手动检查的步骤写进 `SKILL.md`：比如如何启动应用、如何运行测试、如何打开页面验证、如何判断 UI 是否正确。验证越量化，Claude 越容易自检。

### 4.3 Goal-based loops：用 `/goal` 延长迭代，但必须给出可验证完成标准

- **触发方式**：实时手动 prompt。
- **停止条件**：目标达成，或达到最大轮数。
- **适用场景**：有清晰 exit criteria 的复杂任务。
- **成本控制**：给出具体完成标准和轮数上限，例如“最多尝试 5 次”。

当单轮对话不够时，可以用 `/goal` 让 Claude 继续迭代。文章强调，`/goal` 的核心不是“让 Claude 更努力”，而是**把 done 的定义外部化**。每当 Claude 想停下来，评估模型会检查目标是否达成；如果未达成，就让它继续。

因此，最适合 `/goal` 的标准通常是确定性的：测试全部通过、benchmark 分数超过阈值、lint 清零、页面出现某个文本、接口返回某个 JSON schema 等。模糊目标如“把代码写得更优雅”很难作为可靠停止条件。

### 4.4 Time-based loops：`/loop` 和 `/schedule` 让 agent 按时间间隔工作

- **触发方式**：指定时间间隔。
- **停止条件**：人工取消，或工作自然完成，例如 PR 合并、队列清空。
- **适用场景**：重复性工作，或需要轮询外部系统的任务。
- **成本控制**：拉长间隔，或尽量从定时轮询改为事件触发。

有些任务本身是周期性的：每天总结 Slack、每小时检查 CI、定时扫描 issue、追踪 PR review。`/loop` 可以在本机定时重跑 prompt；如果本机关机，loop 就停止。`/schedule` 则把 routine 搬到云端运行，更适合长期任务。

这类 loop 的关键问题不是“能不能自动跑”，而是“多久跑一次才合理”。如果被监控对象每天只变化几次，就没必要每分钟检查。过高频率只会消耗 token 和 API 调用。

### 4.5 Proactive loops：把多个原语组合成长期无人值守工作流

- **触发方式**：事件或计划任务，无需人类实时在场。
- **停止条件**：每个子任务完成后退出；routine 本身持续运行直到关闭。
- **适用场景**：稳定、重复、边界清晰的工作流，例如 bug report、issue triage、迁移、依赖升级。
- **成本控制**：用小模型处理路由和简单任务，把最强模型留给判断性环节。

文章举了一个处理用户反馈的组合：

1. 用 `/schedule` 定时检查新报告；
2. 用 `/goal` 定义完成标准，并用 skills 记录验证方式；
3. 用 dynamic workflows 编排多个 agent：triage、fix、review；
4. 用 auto mode 让 routine 不因权限确认而频繁暂停。

这其实就是从“单个 agent”进入“agent 系统设计”。真正的生产力提升不在于某一次生成代码，而在于把任务发现、分派、实现、审查、验证和停止连接成闭环。

## 5. 观点对应的论据

文章的论据主要来自 Claude Code 团队对产品原语的归纳，而不是外部实验数据。它用四类 loop 对应 Claude Code 中的具体功能：

| Loop 类型 | 触发 | 停止 | Claude Code 原语 | 适合任务 |
|---|---|---|---|---|
| Turn-based | 用户 prompt | Claude 自判完成或需要上下文 | 普通对话、skills | 短任务、一次性任务 |
| Goal-based | 用户设定目标 | 达成目标或达到轮数上限 | `/goal` | 有明确验证条件的复杂任务 |
| Time-based | 时间间隔 | 取消、PR 合并、队列清空等 | `/loop`、`/schedule` | 周期性任务、外部系统轮询 |
| Proactive | 事件或计划 | 子任务完成；routine 持续运行 | `/schedule`、`/goal`、dynamic workflows、auto mode | bug triage、迁移、依赖升级、反馈处理 |

此外，文章还给出两条贯穿所有 loop 的工程原则：

1. **质量来自系统，而不是单次输出。** 代码库要干净，文档要可达，skills 要写清楚验证标准，最好用第二个 agent 做 code review。
2. **成本来自边界不清。** 选择合适 primitive 和模型，定义停止标准，先小规模试跑，把确定性工作写成脚本，不要过度频繁运行 routine，并持续查看 `/usage`、`/goal`、`/workflows` 的 token 消耗。

## 6. 我的评价和启发

### 6.1 这篇文章的重要性：官方把“loop engineering”产品化了

我觉得这篇文章最值得注意的地方，是 Claude Code 团队把社区里比较松散的“loop engineering”概念翻译成了产品原语。以前我们说 loop，可能是在说 shell while 循环、cron job、agent 自我反思、多 agent 协作，甚至只是“多问几轮”。这篇文章给了一个更工程化的分类：触发、停止、原语、任务类型。

这会改变使用 Claude Code 的方式。未来不是所有任务都应该打开聊天框手动推进，而是应该先问：这个任务是一次性的、可验证的、周期性的，还是可以无人值守的？然后选择对应 loop。

### 6.2 我认同：停止条件比 prompt 更重要

文章反复强调 stop criteria，这点非常关键。Agent 系统失败的常见原因不是“模型不够聪明”，而是它不知道什么时候该停、凭什么判断完成、失败后应该重试几次。

一个好 loop 的提示词可能很短，但它必须有清晰的完成标准，例如：

- 测试命令必须通过；
- 页面必须出现某个结果；
- 生成文件必须满足 schema；
- PR 必须没有 lint error；
- 最多重试 N 次，失败后汇报阻塞原因。

这比“请认真检查”“请确保质量”可靠得多。

### 6.3 我会补充：loop 的状态管理同样重要

文章重点谈 Claude Code primitives，但如果真正落地长期 loop，还需要非常重视状态管理：哪些任务已经处理过？哪些通知已经发过？哪些 PR 已经评论过？失败次数记录在哪里？是否支持幂等？

如果状态只存在模型上下文里，routine 重启后就会丢失；如果没有去重，通知和 PR 评论会重复发送；如果没有失败记录，agent 会在同一个坏输入上无限重试。因此，生产级 loop 往往需要外部状态文件、数据库、issue label、git commit、queue offset 等机制。

### 6.4 对我们的启发：把个人工作流拆成四类 loop

这篇文章可以直接指导个人 AI 工作流设计：

1. **Turn-based**：临时问答、短代码修改、一次性解释。
2. **Goal-based**：写文章、修复杂 bug、让测试通过、完成一个可验收 feature。
3. **Time-based**：博客监控、GitHub Trending 监控、定时新闻摘要、定期检查 CI。
4. **Proactive**：自动 triage issue、自动跟进 PR、自动更新依赖、自动把监控结果生成报告。

其中最容易开始的是 time-based loop：选一个你每天重复看的信息源，把“抓取 → 去重 → 总结 → 通知”写成稳定脚本，再让 agent/cron 定时运行。等这个 loop 稳定后，再逐步把判断性环节交给更强模型。

### 6.5 风险：不要为了 loop 而 loop

文章也提醒：不是所有任务都需要复杂 loop。很多事情一次 prompt 足够。复杂 loop 会引入新的维护成本：权限、状态、重试、日志、费用、错误放大、误操作恢复。

我的判断是：只有当任务同时满足以下条件时，才值得升级为更自动的 loop：

- 重复发生；
- 输入来源稳定；
- 成功标准可验证；
- 出错后影响可控；
- 人类只需要看结果或做少量审批。

否则，用普通 turn-based loop 反而更安全。

## 7. 延伸阅读

- [Claude Code 文档](https://code.claude.com/docs/)
- [Claude Code Changelog](https://code.claude.com/docs/en/changelog)
- [ClaudeDevs 原帖 / X Article](https://x.com/ClaudeDevs/status/2074208949205881033)
- 本站旧文：[Loop Engineering：从手写提示词到设计自主 Agent 循环](/mywiki/2026/06/17/loop-engineering-ai-coding-agents.html)

## 8. 可执行清单：如何开始设计自己的 loop

1. 选一个你已经重复做的任务，不要一开始就选最复杂的。
2. 写下触发条件：用户手动、目标、时间、事件？
3. 写下停止条件：什么证据表明完成？最多尝试几次？失败后如何退出？
4. 把验证步骤写成脚本或 skill，而不是只写自然语言要求。
5. 先小规模试跑，观察哪里卡住、哪里过度行动、哪里浪费 token。
6. 再考虑自动化频率、模型选择、并行 agent 和 code review。

总结来说，这篇文章的核心不是教你“让 Claude 一直循环”，而是教你**把 agent 的重复工作变成一个有入口、有出口、有验证、有成本控制的工程系统**。
