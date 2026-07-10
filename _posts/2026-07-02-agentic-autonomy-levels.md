---
layout: post
title: "Agentic Autonomy Levels：智能体自主性等级"
description: "Addy Osmani 提出用 agency 与 orchestration 两个维度理解 AI Agent 自主性：真正成熟的方向不是盲目提高自主等级，而是让风险、可回滚性和可验证证据共同决定应使用哪一级自主。"
date: 2026-07-02
created_at: 2026-07-10 07:59:00 +0800
categories: AI-Agent
---

# Agentic Autonomy Levels：智能体自主性等级

> 来源: Addy Osmani Blog | [原文链接](https://addyosmani.com/blog/agentic-autonomy-levels/) | 发布日期: 2026-07-02

## 1. 文章基本信息

- **标题**: Agentic Autonomy Levels
- **来源**: Addy Osmani Blog
- **链接**: <https://addyosmani.com/blog/agentic-autonomy-levels/>
- **作者**: Addy Osmani
- **发布日期**: 2026-07-02
- **主题**: AI Agent、自主性等级、软件工程自动化、多智能体编排、验证与风险控制

## 2. 文章要解决的核心问题（论点）

这篇文章试图回答的问题是：**在 AI Agent 从“代码建议工具”演化为“可执行、可并行、可编排的软件工厂”之后，我们该如何判断某个任务应该交给智能体多高程度的自主性，以及需要什么验证机制来证明这种自主性是合理的？**

作者的核心论点可以概括为：**自主性不是越高越好，也不是单一阶梯；它应该由任务风险、可回滚性、成功标准和验证证据共同决定。成熟的软件工程团队追求的不是“最大自主”，而是“校准后的自主”。**

## 3. 问题所处的背景上下文

过去讨论 AI 编程工具时，重点往往是“提示词写得好不好”“模型会不会补全代码”。但 Addy 认为，Agentic Engineering 的重心已经从 **prompting** 转向 **operating**：

- Agent 不再只是给建议，而是可以运行命令、修改代码、执行测试、提交 PR。
- Claude Code、Codex 等工具开始支持 goal mode、loop、background session、subagent、hook、sandbox、review agent 等机制。
- 工程师不再只是和一个聊天窗口互动，而是在管理一个由多个 Agent、多个工作区、多个验证环节组成的执行系统。

这会带来一个新的问题：原来那种“一个 Agent 有多自主”的单轴评价方式不够用了。

Steve Yegge 曾提出过一个单轴自主性阶梯，用来衡量开发者对单个 Agent 的信任程度。Addy 认为这个框架仍有参考价值，但现在已经不够描述真实工作流。因为今天的能力不只来自单个 Agent 更“大胆”，还来自**多个 Agent 的并行协作与编排能力**。

因此，作者将自主性拆成两个维度：

1. **Agency（代理执行能力）**：单个 Agent 可以离人多远、独立完成多少探索与执行。
2. **Orchestration（编排能力）**：系统能否协调多个 Agent、多个任务、多个工作区，形成持续工作的流水线。

## 4. 文章的核心观点

### 4.1 自主性应该拆成两个轴：Agency 与 Orchestration

低 Agency 的 Agent 只是建议候选动作，并等待人类决策；中等 Agency 的 Agent 可以接手一个明确任务，并持续汇报证据；高 Agency 的 Agent 则围绕目标自主实验、测试、遇阻提问、换路径尝试，并最终带着证据回来。

Orchestration 轴则描述多 Agent 协作的复杂度：从单 Agent 单线程，到多个 Agent 在隔离 worktree 中并行工作，再到 manager agent 能从 backlog、issue tracker 或定时任务中持续拉取任务、分派 worker、验证输出、失败重试，只在异常时让人介入。

这也是本文区别于普通“AI 自主等级”讨论的关键：**一个强 Agent 和一套强 Agent 工厂不是同一件事。**

### 4.2 六个自主等级：从辅助到异常管理

作者提出了 0 到 5 六个等级：

- **Level 0: Assist（辅助）**  
  Agent 只提供建议，例如自动补全、inline edit、聊天中讨论方案。适合错误代价高、范围很小、需要人类形成判断的任务。验证主要由人完成。

- **Level 1: Supervised action（监督执行）**  
  Agent 可以编辑文件或运行命令，但关键动作需要人类批准。这是多数人当前使用 AI 编程工具的默认状态。风险在于“审批疲劳”：每个确认按钮看起来都差不多，人最终可能机械批准。

- **Level 2: Scoped task delegation（有边界的任务委派）**  
  把一个边界清晰的任务交给 Agent：目标、约束、完成定义都明确。人类可以打断，但不持续介入。验证开始从“人盯着看”转向 Agent 产出的证据，例如测试通过、类型检查、lint、截图、复现步骤等。

- **Level 3: Goal-driven autonomy（目标驱动自主）**  
  Agent 围绕一个目标持续 plan → act → test → review，直到达到停止条件。关键要求是：目标必须具体、可衡量、可自动验证。不要让 Agent “改善用户体验”这种模糊目标，而应设定类似“把 TTI 降到 1 秒以内”“消除 TypeScript 中所有 explicit any”这样的目标。

- **Level 4: Parallel delegation（并行委派）**  
  多个 Agent 并行处理任务的不同切片，每个 Agent 有隔离工作区、明确文件归属和自己的 review queue。难点不在“开很多 Agent”，而在任务拆分。否则会出现 false parallelism：看似并行，实际互相踩文件、重复决策、制造合并冲突。

- **Level 5: Managed-by-exception orchestration（异常管理式编排）**  
  人定义成功标准和策略，manager agent 根据触发器醒来，分派 worker，监控进度，验证输出，失败重试，必要时升级给更强 Agent 或人类。人类主要处理异常。这接近“软件工厂”：issue tracker/backlog 是输入，PR、修复结果和证据包是输出。

### 4.3 风险与可回滚性决定自主性的天花板

作者提出三个判断高自主是否合理的问题：

1. **我们多久能知道 Agent 做错了？**
2. **如果它做错了，我们能多干净地撤销？**
3. **什么证据能证明它做对了？**

如果答案是“很晚才知道、很难撤销、只能相信它的总结”，那就不应该称为高自主，而是高风险。

这也是文章最实用的部分：自主等级不应该按任务名称决定，而应按验证过程决定。一个有强测试、独立 reviewer agent、清晰 rollback path 的支付引擎重构，可能比一个没有权威真相来源的文档自动化任务更适合高自主。

### 4.4 每次 Agent 运行前都需要一份“契约”

作者建议，每次运行 Agent 前都应明确：

- **Goal**：要达成的结果，不是活动或手段。
- **Scope**：操作范围、允许的方法。
- **Non-goals**：哪些事情不做。
- **Tools and permissions**：能使用哪些工具，权限边界是什么。
- **Stopping condition**：何时停止，最好是可测量变量。
- **Evidence**：用什么测试、日志、截图、数据库记录或其他独立证据证明完成。
- **Escalation**：什么情况下升级给谁。
- **Budget**：时间、token、尝试次数、并行度上限。

这其实是在把“写 prompt”升级为“设计执行协议”。

### 4.5 成熟团队需要跟踪自主性的指标

作者建议为不同自主等级记录指标，例如：

- 人类介入之间的平均时间。
- 最长成功无人值守运行时间。
- 在 sandbox 中执行与升级执行的比例。
- 自动批准与拒绝比例。
- 每条人类指令对应的 Agent action 数。
- 澄清请求率、打断率。
- 每次接受变更的 review 时间。
- 返工率、缺陷逃逸率。
- 每次接受变更的 token 成本。

这些指标能避免团队凭感觉谈“自主性”。例如，一个需要人不断手动派活的单 Agent 系统，不是真 Level 4；一个保守但具备自动 intake、retry 和证据 gate 的系统，反而可能更接近 Level 5。

## 5. 观点对应的论据

### 5.1 来自现有工具能力的观察

文章多次引用 Claude Code 与 Codex 的产品形态：/plan、/goal、/loop、/background、/batch、subagents、hooks、checkpointing、worktrees、automations、auto-review、sandboxing 等。这些能力说明：Agent 工程已经从“一个对话框”进入“多个执行上下文 + 多个控制面”的阶段。

### 5.2 来自 Anthropic 使用研究的数据

作者引用 Anthropic 对 Claude Code 的研究：约 40 万个 sessions、23.5 万用户，时间跨度为 2025 年 10 月到 2026 年 4 月。研究发现，人类大约做 70% 的规划决策，而 Claude 做约 80% 的执行工作。

这支持了一个重要判断：**高自主不是把人排除在 loop 外，而是把人的角色从“执行每一步”转变为“决定方向、设置边界、审查证据”。**

### 5.3 来自工程实践的风险模型

作者用“支付引擎重构”和“文档自动化”作对比，说明任务表面风险与实际可自主程度并不完全一致。真正决定自主上限的是：

- 是否有可自动化验证的成功标准。
- 是否能快速发现错误。
- 是否能低成本回滚。
- 是否有独立验证者，而不是只相信执行 Agent 的总结。

### 5.4 来自反模式的经验总结

文章列出四个常见反模式：

1. **Autonomy as status（把自主性当身份徽章）**  
   团队把更高自主等级当作能力证明，导致系统跑得比验证能力更激进。

2. **Permission laundering（权限漂白）**  
   因审批疲劳而给 Agent 过宽权限。解决方式是 sandbox profile、scoped writable root、命令白名单、hook、auto-review。

3. **Summary substitution（用总结代替审查）**  
   Agent 的总结不能替代 diff、测试、日志、截图、风险说明等证据包。

4. **Fleet cosplay（舰队角色扮演）**  
   开了很多 Agent，但所有依赖和协调仍由人手工管理。这不是编排，只是把人的协调负担放大了。

## 6. 我的评价和启发

### 6.1 这篇文章最有价值的地方

我认为本文最大的价值，是把“Agent 自主性”从一个容易炫技的概念，拉回到工程管理问题：**自主等级必须被验证能力、权限边界、回滚能力和成本模型约束。**

很多关于 Agent 的讨论会默认“越自主越先进”，但 Addy 的观点更成熟：如果你无法快速知道它错了、无法撤销、无法独立证明它对了，那所谓高自主只是把风险隐藏起来。

这对当前使用 Claude Code、Codex、Cursor、OpenCode、Hermes 这类工具的人尤其重要。我们已经可以开很多子 Agent、后台任务和计划任务，但真正的瓶颈不是“能不能跑起来”，而是：

- 如何拆任务；
- 如何隔离工作区；
- 如何定义停止条件；
- 如何让证据自动产生；
- 如何让 reviewer 与 implementer 分离；
- 如何避免 token 成本和 review queue 失控。

### 6.2 我认同的观点

我非常认同“verification will always be the bottleneck”。模型能力提升会让执行越来越便宜，但验证不会自动消失。相反，执行越便宜，验证越容易成为瓶颈。

这和软件工程中的 CI/CD 很像：自动部署不是因为人类更大胆，而是因为测试、回滚、监控、灰度、告警成熟了。Agent 自主性也应该沿着同样路径发展。没有验证基础设施的 Level 5，本质上只是 Level 1 的审批疲劳升级版。

### 6.3 我会补充的一点

文章主要站在软件工程视角讨论 Agent，但这个框架也适用于个人知识管理、研究、运营监控、内容生产等场景。比如一个新闻监控 Agent，如果只是定时抓取并总结，是 Level 2/3；如果它能从多个源发现主题、去重、评估可信度、写摘要、失败重试、只在异常时通知人，才更接近 Level 5。

不过非代码场景有一个更难的问题：**“正确性证据”不像测试那样标准化**。代码可以跑测试，网页可以截图，性能可以测指标；但研究摘要、趋势判断、商业分析的证据往往更软。因此这些场景的自主性上限可能天然更低，除非我们能设计出结构化证据包，例如来源引用、交叉验证、反例搜索、置信度标注和人工抽检。

### 6.4 对实践的启发

如果要把这篇文章落到实践，我会把它转化成一个 Agent 任务启动模板：

```markdown
## Agent Run Contract

Goal:
Scope:
Non-goals:
Allowed tools/permissions:
Writable paths:
Stopping condition:
Evidence required:
Escalation rules:
Budget:
Rollback plan:
Reviewer:
```

对于低风险任务，可以轻量填写；对于高风险或高自主任务，这份 contract 应该像 PRD + runbook + acceptance criteria 的组合。

同时，团队应该维护一个“任务类型 → 默认自主等级”的表，但允许根据验证能力上调或下调。例如：

- 纯阅读总结：Level 2，除非需要跨源验证。
- 有测试覆盖的小型 bugfix：Level 2/3。
- 性能优化且有自动 benchmark：Level 3。
- 大规模重构：只有在文件 ownership、测试、review agent、rollback 都明确时才进入 Level 4。
- 自动处理 backlog：只有在 intake、分派、状态同步、失败升级、独立验证都编码完成后才进入 Level 5。

### 6.5 我不完全同意或需要警惕的地方

文章对“数百甚至数千 Agent 的连续工厂”保持开放态度，但我认为这里存在两个现实约束：

1. **组织吸收能力**：即使 Agent 能产出大量 PR，人类 review、产品决策、上线窗口、用户反馈处理也可能跟不上。
2. **系统性错误相关性**：多个 Agent 如果共享同一模型、同一上下文偏差、同一错误假设，数量增加不一定提高可靠性，反而可能放大同类错误。

所以 Level 5 的关键不只是 manager agent，而是异质化验证：不同模型、不同工具链、不同测试视角、不同权限边界，以及必要的人类抽检。

## 7. 延伸阅读

- Addy Osmani 原文：[Agentic Autonomy Levels](https://addyosmani.com/blog/agentic-autonomy-levels/)
- Steve Yegge: [Welcome to Gas Town](https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04)
- Anthropic Research: [Measuring Agent Autonomy](https://www.anthropic.com/research/measuring-agent-autonomy)
- Anthropic Research: [Claude Code Expertise](https://www.anthropic.com/research/claude-code-expertise)
- OpenAI: [Open-source Codex orchestration: Symphony](https://openai.com/index/open-source-codex-orchestration-symphony/)

## 8. 一句话总结

**AI Agent 的成熟路线不是让它“更自由”，而是让它在更清晰的契约、更小的权限边界、更强的证据链和更便宜的回滚机制中，承担与验证能力相匹配的自主性。**
