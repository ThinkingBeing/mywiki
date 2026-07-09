---
layout: post
title: "Addy Osmani《Loop Engineering》解读：从提示 Agent 到设计提示 Agent 的系统"
description: "Addy Osmani 认为 AI 编程的杠杆点正在从手写 prompt 转向设计 loop：由自动化、worktree、skills、connectors、sub-agents 和外部 memory 组成的系统，持续发现任务、驱动 Agent、验证结果并记录状态。"
date: 2026-06-07
created_at: 2026-07-09 09:20:58 +0800
categories: AI Agents
---

# Addy Osmani《Loop Engineering》解读：从提示 Agent 到设计提示 Agent 的系统

> 来源: Addy Osmani Blog | [原文链接](https://addyosmani.com/blog/loop-engineering/) | 发布日期: 2026-06-07

## 1. 文章基本信息

- **标题**: Loop Engineering
- **来源**: AddyOsmani.com
- **链接**: https://addyosmani.com/blog/loop-engineering/
- **作者**: Addy Osmani
- **发布日期**: 2026-06-07
- **主题**: AI 编程 Agent、Loop Engineering、Agent Harness、Claude Code、Codex、Skills、Sub-agents、MCP、AI-assisted engineering

## 2. 文章要解决的核心问题（论点）

这篇文章试图回答的问题是：**当 coding agent 越来越能自主执行任务时，工程师的核心工作是否会从“手写 prompt”变成“设计会自动 prompt Agent 的循环系统”？**

一句话概括：**Loop Engineering 的本质，是把人从逐轮提示 Agent 的位置上替换下来，转而设计一个能发现任务、分配任务、验证结果、记录状态并决定下一步的外层系统。**

## 3. 问题所处的背景上下文

过去两年，很多 AI 编程实践的基本模式是 turn-by-turn：工程师写一个 prompt，补充上下文，阅读 Agent 的回复，再写下一条 prompt。Agent 很强，但人始终坐在驾驶位，每一轮都要手动推进。

Addy 观察到，2026 年前后这个工作姿态开始变化：Peter Steinberger 提出“不要再提示 coding agents，而要设计提示 agents 的 loops”；Anthropic Claude Code 负责人 Boris Cherny 也说自己的工作变成“写 loops”。

这背后的变化是：

1. **Agent 工具开始产品化循环能力**  
   过去要自己写 bash、cron、状态文件、PR 脚本；现在 Codex app、Claude Code 等工具已经内置 automations、`/goal`、worktree、skills、sub-agents、MCP connectors 等能力。

2. **单个 Agent harness 已经不够描述完整工作流**  
   Addy 之前写过 Agent Harness Engineering，关注的是单个 Agent 的运行环境。Loop Engineering 则更高一层：harness 是 Agent 在里面跑的环境，loop 是按计划启动 harness、派生 helper、持续喂任务的外层控制结构。

3. **工程师的杠杆点上移**  
   不是 prompt 不重要，而是最高杠杆从“如何写好某一条 prompt”转移到“如何设计一个会持续生成和验证 prompt 的系统”。

## 4. 文章的核心观点

### 4.1 Loop Engineering 是 Prompt Engineering 之上的控制结构

Addy 对 loop engineering 的定义很直接：**替换掉“你作为提示 Agent 的人”，改为设计一个系统来提示 Agent。**

一个 loop 可以理解为递归目标：你定义一个目的，AI 迭代执行直到完成。它不是单次回答，而是一个会跨多轮运行的控制结构：

- 找到要做的工作；
- 把工作交给 Agent；
- 检查结果；
- 写下已完成状态；
- 决定下一步。

这也解释了它和 prompt/context/harness 的关系：

| 层次 | 优化对象 | 工作单元 |
|---|---|---|
| Prompt Engineering | 单条指令怎么写 | 人手输入的一轮对话 |
| Context Engineering | 进入上下文窗口的文档、历史、工具说明 | 一次回答的条件 |
| Harness Engineering | 单个 Agent 的运行环境、工具、权限、反馈 | 一次 Agent run |
| Loop Engineering | 何时启动 Agent、交给谁、如何验证、如何继续 | 跨多轮、可自运行的系统 |

### 4.2 一个可用 loop 需要五个构件，加一个外部记忆

文章最实用的部分，是把 loop 拆成五个 building blocks，再加一个 memory：

1. **Automations**：按计划触发，自动发现和 triage 工作。  
2. **Worktrees**：隔离多个 Agent 的并行修改，避免文件冲突。  
3. **Skills**：把项目知识、构建步骤、约定和历史坑点写下来，避免每次重新解释。  
4. **Plugins / Connectors**：通过 MCP 等机制连接 issue tracker、数据库、Slack、CI、staging API 等真实工具。  
5. **Sub-agents**：拆分 maker 和 checker，让写代码的 Agent 不要给自己打分。  
6. **Memory**：外部状态，例如 markdown 文件、Linear board、GitHub issue，用来记录什么做完了、什么还没做。

Addy 特别强调 memory 的朴素重要性：**Agent 会忘，repo 不会忘。** 长期运行的系统不能把状态放在对话窗口里，而要放在磁盘、issue board 或其他外部系统里。

### 4.3 Automations 是 heartbeat，`/goal` 是停止条件机制

Automations 让 loop 不再是“一次性运行”，而是有节奏的后台过程。Codex app 里可以设置项目、prompt、频率和运行环境；Claude Code 则通过 `/loop`、schedule/cron、hooks、GitHub Actions 等方式达到类似效果。

文章还强调 `/goal` 这个原语：

- `/loop` 是按节奏重复运行；
- `/goal` 是持续运行直到一个可验证条件成立。

例如不是写“改善 auth 测试”，而是写“`test/auth` 下所有测试通过且 lint 干净”。更关键的是，Claude Code 的 `/goal` 会在每轮后用一个单独的小模型判断是否完成，避免“写代码的模型自己宣布完成”。这其实就是 maker-checker 分离在停止条件上的应用。

### 4.4 Worktree 解决机械冲突，但不能解决人的审查瓶颈

当多个 Agent 同时改一个仓库时，最大的现实问题之一是文件冲突。Git worktree 提供了自然的隔离：每个 Agent 在独立目录和分支上工作，共享同一份历史，但不会直接踩彼此的文件。

Codex 内置 worktree 支持；Claude Code 也可以通过 `git worktree`、`--worktree` 和 sub-agent 的 `isolation: worktree` 实现。

但 Addy 提醒：worktree 只解决机械冲突，不解决 review bottleneck。真正限制并行度的，不是你能开多少个 Agent，而是你能认真审多少个 PR。

### 4.5 Skills 让 intent 不再反复付费

Skills 的价值是把项目知识外部化：构建命令、测试命令、代码风格、禁止事项、历史事故、架构约定。没有 skills，Agent 每一轮都像金鱼一样从零理解项目，并用自信猜测填补你的意图空白。

Addy 把这和 “intent debt” 联系起来：如果人的真实意图没有被写在系统外部，Agent 就会反复重新推断，而且越自动化，错误推断扩散越快。

所以 skills 不只是提示词模板，而是让组织知识在 loop 中复利的机制。

### 4.6 Connectors 让 loop 进入真实环境，也带来真实风险

如果 loop 只能看文件系统，它的能力很有限。Connectors（通常基于 MCP）让 Agent 可以读取 issue tracker、查询数据库、调用 staging API、在 Slack 发消息、更新 Linear ticket、打开 PR。

这让 Agent 从“告诉你该怎么做”变成“在真实系统里完成动作”。但这也意味着权限、安全、审计、回滚都变得重要。一个能连接生产系统的 loop，不再是玩具，而是需要按照工程系统治理。

### 4.7 Sub-agents 的关键价值是 maker-checker 分离

文章认为，loop 里最重要的结构动作之一，是拆开“提出/实现方案的人”和“检查方案的人”。原因很简单：写代码的模型太容易相信自己的答案。

常见分工是：

- explorer：调查问题；
- implementer：实现修改；
- verifier/reviewer：运行测试、检查 diff、对照 spec、给出 PASS/FAIL。

这和传统工程里的 code review、QA、CI gate 是同构的。只不过在 loop 里，因为人不一定实时盯着，可信 verifier 的价值更高。

### 4.8 Loop 不会替代工程师，反而放大工程师是否理解系统

文章最后的警告很清醒：loop 改变工作方式，但不会把工程师从责任中删除。Addy 提到三个风险会随着 loop 变强而更尖锐：

1. **Verification 仍然是你的责任**：无人值守的 loop 也会无人值守地犯错。  
2. **Comprehension debt 会增长更快**：代码生成越快，人对系统真实理解的缺口可能越大。  
3. **Cognitive surrender 很舒服也很危险**：当 loop 自己跑起来，人很容易停止判断，直接接受输出。

他的结论是：**Build the loop. Stay the engineer.** 要设计 loop，但要像一个仍然打算对系统负责的工程师那样设计，而不是像一个只想按下启动键的人。

## 5. 观点对应的论据

文章的论据主要来自工具生态的能力收敛，以及作者对 AI 编程工作流的实践观察。

### 5.1 Codex 与 Claude Code 的能力正在同构

Addy 用一张映射表说明，虽然名称不同，但两个工具的 loop building blocks 已经非常相似：

| Primitive | 在 loop 中的作用 | Codex app | Claude Code |
|---|---|---|---|
| Automations | 定时发现与 triage | Automations tab、Triage inbox、`/goal` | `/loop`、`/goal`、hooks、GitHub Actions |
| Worktrees | 隔离并行任务 | 每个 thread 内置 worktree | `git worktree`、`--worktree`、sub-agent isolation |
| Skills | 固化项目知识 | `SKILL.md`、`$name`、隐式调用 | `SKILL.md` |
| Plugins / connectors | 连接真实工具 | MCP connectors、plugins | MCP servers、plugins |
| Sub-agents | 分工、审查、并行 | `.codex/agents/` TOML | `.claude/agents/`、agent teams |
| State / Memory | 记录进度 | Markdown、Linear 等 | `AGENTS.md`、progress files、Linear via MCP |

这个同构说明：loop engineering 不是某个产品的特殊功能，而是 AI coding tools 正在共同收敛到的一种工程形态。

### 5.2 一个端到端 loop 的例子

文章给出的典型 loop 是：

1. 每天早上 automation 在仓库上运行；
2. prompt 调用 triage skill；
3. skill 读取昨天的 CI failures、open issues、recent commits；
4. findings 写入 markdown 文件或 Linear board；
5. 对值得处理的问题，thread 打开隔离 worktree；
6. 一个 sub-agent draft fix；
7. 另一个 sub-agent 根据 skills 和测试审查 draft；
8. connectors 打开 PR、更新 ticket；
9. 无法处理的内容进入 triage inbox；
10. state file 记录已尝试、已通过、仍未完成的事项，供明天继续。

这个例子很重要，因为它展示的不是“一个聪明 Agent”，而是一个小型控制面板：调度、状态、执行、审查、外部集成都在其中。

## 6. 我的评价和启发

### 6.1 这篇文章的价值：把“Agent 自动化”从魔法叙事拉回系统设计

我认为这篇文章最有价值的地方，是它没有把 loop engineering 描述成“Agent 终于可以替代工程师”的故事，而是把它拆成传统工程系统熟悉的部件：cron、worktree、状态文件、CI、reviewer、connectors、权限边界。

换句话说，Agent 的新能力不是摆脱工程，而是更需要工程。越是自动化，越需要：

- 清晰的任务边界；
- 可验证的停止条件；
- 外部状态；
- 独立验证者；
- 审计和回滚；
- 人类 review。

这使得 Loop Engineering 更像控制系统设计，而不是 prompt 技巧。

### 6.2 我特别认同：最高杠杆点从“单条 prompt”转到“生成 prompt 的系统”

Prompt 仍然重要，但在长期任务中，单条 prompt 的质量不再是唯一瓶颈。真正决定效果的是：

- 什么事件触发 Agent？
- 它读取哪些状态？
- 它如何选择下一项任务？
- 它在哪里执行？
- 谁检查它？
- 什么证据允许它宣布完成？
- 失败后如何记录并进入下一轮？

这也是为什么两个使用同一模型的人，效果可能完全不同。差异不只是“谁更会写 prompt”，而是谁更会把 prompt 放进一个可运行、可观察、可纠错的系统。

### 6.3 文章最值得警惕的点：comprehension debt

AI 编程最隐蔽的债务，不一定是 bug，而是理解债务。代码越来越快进入仓库，但人对这些代码的 mental model 没有同步增长。

Loop 会放大这个问题：以前你至少还在每轮 prompt 中参与思考；现在 loop 可能在你不看时生成 PR、修复测试、更新依赖。如果没有强制摘要、设计说明、风险说明和人工 review，团队很容易变成“仓库在进化，但人类理解在退化”。

因此，我会给任何 coding loop 增加一个硬要求：**每次代码变更不仅要 tests pass，还要产出人类可读的变更理由、替代方案、影响面和未验证风险。**

### 6.4 落地建议：先做只读 loop，再逐步授予写权限

如果要实践 Addy 的思路，我不会一开始就让 Agent 自动改代码。更稳妥的路径是：

1. **只读 triage loop**：每天汇总 CI failures、open issues、recent commits，只写报告，不改代码。  
2. **draft loop**：允许在隔离 worktree 上生成修复分支，但必须人工 review。  
3. **verified PR loop**：加入独立 verifier sub-agent，只有通过测试和 spec 检查才进入人工 review。  
4. **低风险 auto-merge**：仅对依赖小版本升级、lint fix、flaky test retry 等低风险任务，在绿色 CI 和审计日志下自动合并。  
5. **核心业务仍需人类判断**：架构、产品语义、安全边界、数据迁移等不应完全交给无人值守 loop。

### 6.5 与 LangChain 那篇《The Art of Loop Engineering》的关系

这篇 Addy 的文章更像“原始定义和工程直觉”：解释为什么工程师要从 prompt writer 变成 loop designer，并列出五个构件。

LangChain 的《The Art of Loop Engineering》则更像产品化框架：把 loop 分成 Agent loop、Verification loop、Event-driven loop、Hill-climbing loop，并映射到 LangChain/LangSmith 原语。

两篇文章可以合起来读：

- Addy 讲的是 coding agent 工作方式如何变化；
- LangChain 讲的是生产 Agent 系统如何分层工程化；
- 共同结论是：**Agent 的价值越来越来自围绕模型构建的循环，而不是单次模型调用。**

## 7. 延伸阅读

- [Addy Osmani: Loop Engineering](https://addyosmani.com/blog/loop-engineering/)
- [Addy Osmani: Agent Harness Engineering](https://addyosmani.com/blog/)
- [LangChain: The Art of Loop Engineering](https://www.langchain.com/blog/the-art-of-loop-engineering)
- [Claude Code 文档](https://docs.anthropic.com/)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- Geoffrey Huntley 的 Ralph loop / long-running agent 相关讨论

## 8. 总结

这篇文章最值得带走的一句话是：**不要只做提示 Agent 的人，要做设计“提示 Agent 的系统”的人。**

但这不是让工程师退出循环，而是要求工程师把循环设计得更工程化：有外部状态、有权限边界、有隔离环境、有独立验证、有成本控制、有人工审查。否则 loop 只是把错误、token 成本和理解债务也一起自动化。