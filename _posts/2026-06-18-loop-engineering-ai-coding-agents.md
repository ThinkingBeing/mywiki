---
layout: post
title: "Loop Engineering：从手写提示词到设计自主 Agent 循环"
description: "Lushbinary 文章认为 AI 编程的杠杆点正从单次提示词转向可调度、可验证、有记忆的外层循环；真正的挑战不是让 Agent 更自动，而是把验证、状态、隔离和人类判断嵌入循环。"
date: 2026-06-17
created_at: 2026-06-18 07:43:49 +0800
categories: AI Agents
---

# Loop Engineering：从手写提示词到设计自主 Agent 循环

> 来源: Lushbinary | [原文链接](https://lushbinary.com/blog/loop-engineering-ai-coding-agents-guide/) | 发布日期: 2026-06-09

## 1. 文章基本信息

- **标题**: Loop Engineering: Designing Systems That Prompt AI Agents
- **来源**: Lushbinary Blog
- **链接**: https://lushbinary.com/blog/loop-engineering-ai-coding-agents-guide/
- **作者**: Lushbinary Team
- **发布日期**: 2026-06-09
- **主题**: AI 编程 Agent、Loop Engineering、Claude Code、OpenAI Codex、自动化软件开发

## 2. 文章要解决的核心问题（论点）

这篇文章试图回答的问题是：当 AI 编程 Agent 不再只是一次次响应人工提示，而是能够长期、定时、并行、自我检查地工作时，工程师应该如何设计这个“驱动 Agent 的系统”？

一句话概括：**Loop Engineering 的核心不是写出更神奇的提示词，而是设计一个能持续发现任务、调用 Agent、验证结果、记录状态并决定下一步的外层控制循环。**

## 3. 问题所处的背景上下文

过去两年，很多 AI 编程实践的基本姿势是“人类在驾驶位”：工程师写 prompt、补充上下文、阅读回复、继续追问、让 Agent 改代码、再运行测试。这个模式仍然有效，但它的瓶颈很明显：

1. **每一步都依赖人类推进**  
   Agent 可以很强，但如果每个回合都要人类输入下一条指令，吞吐量仍然受限于人的注意力。

2. **长会话容易退化**  
   一次长 Agent 会话会塞满旧推理、错误尝试、过期文件状态和无关上下文。上下文越长，模型越容易被历史污染。

3. **项目知识重复解释**  
   每次开新会话都要解释构建步骤、代码规范、测试命令、历史坑点。没有外部化的技能与记忆，Agent 每次都从零猜。

4. **工具开始内置循环能力**  
   文章提到，到 2026 年中，Claude Code 和 OpenAI Codex 都已经提供类似的构件：定时自动化、`/goal`、worktree 隔离、skills、sub-agents、MCP connectors 等。这意味着“循环”不再只是工程师自己写的一堆 shell 脚本，而正在产品化。

这篇文章实际上描述的是 AI 编程从“交互式助手”走向“半自主工程系统”的阶段变化。

## 4. 文章的核心观点

### 4.1 杠杆点从 Prompt Engineering 转移到 Loop Engineering

作者把演进分成三层：

| 层次 | 优化对象 | 工作单元 |
|---|---|---|
| Prompt Engineering | 单条指令怎么写 | 人手输入的一轮对话 |
| Context Engineering | 放入窗口的文件、历史、工具定义 | 一次回答的上下文条件 |
| Loop Engineering | 谁在何时生成提示、如何检查结果、如何继续下一步 | 跨多轮的自运行循环 |

文章强调，prompt 和 context 并没有消失。循环仍然由提示词构成，也仍然需要正确上下文。但最高杠杆点变成了：**设计一个能稳定产生正确提示、调用正确工具、用正确证据判定完成的系统。**

### 4.2 Ralph Technique 是 Loop Engineering 的原型

文章引用 Geoffrey Huntley 的 “Ralph” 技术：把编码 Agent 放进一个简单 `while` 循环里。每次启动一个全新的 Agent，会读取 `PLAN.md` 和 `STATUS.md`，选择一个未完成任务，实现、测试、提交、更新状态，然后退出。循环直到状态文件里出现“全部完成”。

这个方法看起来很笨，但关键洞察是：

- 每次运行都重置上下文，避免长会话污染；
- 状态不放在模型记忆里，而放在磁盘文件里；
- Agent 每轮只做一个小任务；
- 是否完成由外部状态和可验证结果决定。

作者认为，现代 Claude Code / Codex 的 loop、goal、automation、本质上是 Ralph 思路的产品化版本。

### 4.3 一个可用循环需要五个构件，加一个外部记忆

文章总结了五个 building blocks：

1. **Automations**：定时触发，自动发现任务和做 triage。  
2. **Worktrees**：让多个 Agent 在不同 git worktree 中并行工作，避免文件冲突。  
3. **Skills**：把项目知识、约定、构建步骤、踩坑记录写成可复用说明。  
4. **Plugins / Connectors**：通过 MCP 等机制连接 issue tracker、数据库、CI、Slack 等真实系统。  
5. **Sub-agents**：把“实现者”和“检查者”拆开，避免同一个模型给自己打分。  
6. **Memory**：放在上下文窗口之外的状态记录，例如 `TODO.md`、Linear board、GitHub issues。

其中最重要的是第 6 个：**模型会忘，仓库不会忘。** 长期循环必须把状态外部化，而不是寄希望于对话历史。

### 4.4 停止条件必须像合同，而不是愿望

文章对 `/goal` 或长期目标的提醒很关键：不要写“改善 checkout flow”这种模糊目标，而要写成可验证合同：

- 目标状态：例如 `src/billing` 覆盖率达到 90%。
- 证据：`npm test` 退出码为 0，coverage report 证明数字达标。
- 约束：不得修改公共 API，不得删除已有测试。
- 预算：最多 25 轮或 5 美元成本。

这说明 Loop Engineering 的核心能力之一，是把人类意图翻译成机器可判定的验收条件。

### 4.5 自动化越强，风险也越尖锐

作者没有把 loop 描述成“替代工程师”的魔法，而是指出三个风险会被放大：

1. **验证仍然是你的责任**：无人值守的循环也会无人值守地犯错。  
2. **理解债务增长更快**：代码更快进入仓库，但你未必真的理解它。  
3. **认知投降更舒适**：当 loop 能自己跑，人最容易停止判断，直接接受输出。

文章最后的姿态是：建设循环，但不要退出工程师角色。Loop 应该用于重复、低风险、可验证的任务；而判断、架构取舍、安全边界仍然需要人。

## 5. 观点对应的论据

### 5.1 工具生态已经支持循环原语

文章列举 Claude Code 和 OpenAI Codex 中逐渐收敛的能力：

- Claude Code：`/loop`、`/goal`、hooks、subagents、worktree isolation、GitHub Actions。
- Codex：Automations、`/goal`、内置 worktrees、skills、TOML sub-agents。
- 两者都可通过 MCP connectors 连接外部系统。

这说明 loop engineering 不是抽象概念，而是已经落实为工具功能。

### 5.2 Ralph loop 证明了“简单外层循环 + 外部状态”的有效性

原始 Ralph 技术用几行 shell 就能工作。它并不依赖复杂 orchestrator，而依赖：

- 清晰 spec；
- 小粒度 task；
- 干净上下文；
- 外部状态文件；
- 测试和提交作为完成证据。

这为后续产品化能力提供了一个非常朴素但有力的原型。

### 5.3 Maker-checker 分工缓解自我验证问题

文章强调“写代码的 Agent 不应该是唯一判定自己完成的 Agent”。所以需要 reviewer / verifier sub-agent，以不同提示、不同模型或不同权限进行对抗式检查。这与人类工程中的 code review、CI gate、QA 分工非常接近。

### 5.4 成熟度阶梯降低采用风险

作者建议不要直接跳到自动合并，而是逐级采用：

0. 手动提示；
1. 定时 triage，只写发现，不改代码；
2. draft 阶段，在隔离分支上生成修复；
3. verifier sub-agent 先审再给人；
4. 仅对低风险任务自动合并，例如依赖升级、lint、flaky test retry。

这个阶梯是文章里最实际的部分：它承认 loop 是高风险能力，因此要用渐进信任来获得可靠性。

## 6. 我的评价和启发

### 6.1 这篇文章真正有价值的地方：把“Agent 自动化”讲成了控制系统问题

很多关于 AI Agent 的文章会停留在“模型更强，所以能自动写代码”。这篇文章更有工程感：它关心调度、隔离、状态、验收、成本、权限、审查。这其实把 AI 编程从模型能力问题，重新放回软件工程和系统设计问题。

我认为“Loop Engineering”这个词虽然新，但它背后的思想并不神秘：

- cron / queue 是调度；
- worktree / branch 是隔离；
- TODO.md / issue board 是状态机；
- tests / CI 是判定函数；
- reviewer agent 是第二道检查；
- MCP connector 是 I/O 边界。

换句话说，真正的能力不是“让模型自己想办法”，而是用传统工程手段把模型限制在可观察、可回滚、可审计的闭环里。

### 6.2 我特别认同：上下文重置比长会话更重要

Ralph 技术最重要的洞察不是 while loop，而是“每一轮都是干净 Agent”。这对长期编码任务非常关键。长会话看似连续，实际上会积累大量认知垃圾；而干净上下文 + 外部状态文件，让每次 Agent 都从当前真实仓库状态出发。

这也解释了为什么很多 Agent workflow 最终都会回到类似模式：

- 把计划写到文件；
- 把完成状态写到文件；
- 每轮读取文件和仓库，而不是相信聊天历史；
- 一次只做可验证的小步。

这比“开一个超长上下文窗口让模型一直记住”更符合工程可靠性。

### 6.3 文章略显乐观的地方：工具命令和产品能力变化太快

文章列举了 Claude Code、Codex 的许多具体命令和能力。但这类 AI coding tool 更新极快，命令、权限、计费、自动化语义都可能变。文章自己也在末尾提醒要查官方文档。对读者来说，更应该学习抽象模式，而不是死记某个工具命令。

可迁移的不是 `/loop` 或 Automations 的具体 UI，而是这套结构：**触发器 → 任务发现 → 隔离执行 → 验证 → 状态更新 → 人类审查/下一轮。**

### 6.4 对个人或团队的启发：先把“每日重复认知劳动”做成只读 loop

如果要落地，我不会一开始就让 Agent 自动改代码。更现实的第一步是只读循环，例如：

- 每天早上汇总失败的 CI；
- 读取最新 issue，按根因聚类；
- 扫描依赖安全公告；
- 总结昨天合并的 PR 和潜在风险；
- 把发现写进 `TODO.md` 或 issue comment。

这个阶段不需要相信 Agent 写代码，只需要判断它的 triage 是否有用。等只读 loop 持续几天或几周表现稳定，再让它在 worktree 上 draft fix，最后才考虑更强权限。

### 6.5 最大风险不是 Agent 犯错，而是人类停止理解

文章提到的“comprehension debt”非常重要。AI 让代码生成变快后，人的理解速度没有同步变快。Loop engineering 如果没有配套 review、日志、变更摘要和架构文档，会让仓库迅速变成“代码很多但没人真正拥有”的状态。

所以我会给 loop 加一个额外要求：**每次产生代码变更，都必须产出人类可读的设计说明和风险说明。** 不只是 tests pass，还要回答：为什么这样改？替代方案是什么？影响面在哪里？哪些地方没有验证？这样才能对抗理解债务。

## 7. 可执行落地建议

如果你想尝试 Loop Engineering，可以按这个顺序开始：

1. **选一个低风险、重复、可验证任务**  
   例如 CI failure triage、依赖升级建议、issue 分类，而不是核心业务重构。

2. **写一个外部状态文件**  
   例如 `AGENT_TODO.md`，包含 Open / In Progress / Done / Blocked。不要让状态只存在聊天记录里。

3. **把项目约定写成 skill**  
   包括构建命令、测试命令、禁止事项、代码风格、常见坑。

4. **从只读自动化开始**  
   第一阶段只允许 Agent 读 CI、issue、日志并写报告，不允许改代码。

5. **引入 worktree draft**  
   让 Agent 在隔离分支里改，永远通过 PR 进入主分支。

6. **加入 verifier sub-agent**  
   verifier 必须运行测试、检查 diff、对照验收条件，而不是“看起来还行”。

7. **设置预算和硬停止条件**  
   包括最大轮数、最大 token 成本、最大运行时间、失败时如何降级。

8. **保留人类审查**  
   至少在相当长时间内，不要让 loop 自动合并核心业务代码。

## 8. 延伸阅读

- Addy Osmani: Loop Engineering
- Addy Osmani: Agent Harness Engineering
- Geoffrey Huntley: Ralph Wiggum as a “software engineer”
- Anthropic Claude Code documentation
- OpenAI Codex documentation
- Model Context Protocol documentation

## 9. 总结

这篇文章最值得带走的一句话是：**AI 编程的下一层能力，不是你更会 prompt，而是你更会设计 prompt 的生产系统。**

Loop Engineering 把 Agent 从“对话里的助手”变成“工程系统里的执行单元”。但越是自动化，越需要严格的工程边界：外部状态、可验证停止条件、隔离分支、maker-checker 分离、预算限制、人类审查。否则，它只是把错误、成本和理解债务也一起自动化了。
