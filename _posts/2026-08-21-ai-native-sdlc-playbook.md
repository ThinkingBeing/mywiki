---
layout: post
title: "【翻译】The AI-Native SDLC playbook（AI 原生 SDLC 实战手册）"
description: "Anthropic 认为 AI Agent 已让编码不再是最大瓶颈，企业需要围绕 intent.md、spec.md、plan.md、反馈闭环、Hooks、Skills、evals 和人工治理重构整个 SDLC。"
date: 2026-08-21
created_at: 2026-08-31 10:32:35 +0800
categories: anthropic ai-agent software-engineering sdlc
source: https://claude.com/blog/the-ai-native-sdlc-playbook
author: Louis Claxton
---

## 1、Code is no longer the bottleneck（代码不再是瓶颈）

Organizations have started using AI to write code at a speed unthinkable one year ago, yet the processes around the code haven't changed at the same pace.
> 组织已经开始用 AI 以一年前难以想象的速度编写代码，但围绕代码的流程并没有以同样的速度变化。

Many engineering teams still have the same approval gates, reviews, handoffs, and policies, stalling productivity gains made by using agentic coding solutions like Claude Code .
> 许多工程团队仍然沿用相同的审批关卡、评审、交接和政策，这些流程阻碍了 Claude Code 这类智能体式编码方案带来的生产力提升。

The software development lifecycle (SDLC) is the process that takes software from idea to production. Most organizations run some version of the same six stages, covering planning, design, building, testing, deploying, and maintaining software. Traditionally, each stage is a discrete phase owned by a different role. Product managers write requirements, technical architects turn them into designs, engineers build the designs, QA teams at regulated enterprises verify it, releases teams ship it, and operations monitors what is running. Work moves between the phases through documents, tickets, and sign-offs.
> 软件开发生命周期（SDLC）是软件从想法走向生产环境的过程。大多数组织都运行某种版本的六阶段流程，覆盖规划、设计、构建、测试、部署和维护。传统上，每个阶段都是由不同角色负责的离散阶段：产品经理写需求，技术架构师将其转化为设计，工程师实现设计，受监管企业中的 QA 团队进行验证，发布团队负责上线，运营团队监控线上运行情况。工作通过文档、工单和签字审批在各阶段之间流转。

The traditional software development lifecycle (SDLC) is process-heavy to ensure accountability and control at each step. However, the traditional SDLC was designed to maximize efficiency in an era where the most time-consuming and expensive stage was writing and implementing code, which is no longer the case. PRDs, estimation rituals, and product security reviews all existed to force alignment during what could be weeks, months, or quarters of development work.
> 传统 SDLC 流程繁重，是为了在每一步确保责任归属和控制。然而，传统 SDLC 的设计目标，是在“写代码和实现代码”最耗时、最昂贵的时代最大化效率；而今天，这一点已经不再成立。PRD、估算仪式和产品安全评审，过去都用于在持续数周、数月甚至数季度的开发工作期间强制对齐。

The traditional SDLC also features controls that assume every step is performed by humans. The organizations generating the most value have rebuilt their process around what agentic AI can now do, while ensuring that humans stay in the loop. In this guide, we walk through several of our Applied AI team's best practices for integrating Claude internally across each stage of the SDLC to accelerate development and make processes run faster, inspired by working with our customers.
> 传统 SDLC 的控制机制还假定每一步都由人类执行。真正获得最大价值的组织，已经围绕智能体 AI 现在能做的事情重建流程，同时确保人类仍然参与关键环节。本指南会介绍 Anthropic Applied AI 团队在内部各个 SDLC 阶段集成 Claude 的若干最佳实践，这些实践也来自我们与客户合作的经验，目标是加速开发并让流程更快运转。

When code is no longer the bottleneck and the build phase runs faster than the traditional SDLC allows for, three things become true:
> 当代码不再是瓶颈、构建阶段比传统 SDLC 允许的速度更快时，有三件事会变成现实：

The bottleneck moves to the steps to the left and right of the build phase. This is mainly plan, review/test, and deploy, which still run at human speed.
> 1. **瓶颈转移到构建阶段左右两侧。** 主要是规划、评审/测试和部署，这些环节仍以人类速度运行。
The controls stop matching reality and become intractable. Reviewing each line by hand made sense when a person had written it, but it can't keep up once agents write most of the diff.
> 2. **控制机制不再匹配现实，变得难以执行。** 当代码由人类逐行编写时，人工逐行评审是合理的；但当大部分 diff 由智能体生成后，这种方式无法跟上速度。
Governance costs increase because exceptions still route through meetings and committees that meet weekly or monthly.
> 3. **治理成本上升。** 因为例外情况仍然要通过每周或每月才开一次的会议和委员会处理。

Build is no longer the constraint — the human-speed steps around it are. Human-speed stages keep their length while build collapses to hours. Let's use a security bottleneck as an example. Security teams are sized for human output, so when agents multiply code output, either the review queue builds or code ships under-reviewed. A regulated organization can't accept either outcome, so its security and policy checks have to keep pace with the agents.
> 构建不再是约束，围绕构建的人类速度环节才是约束。构建阶段压缩到数小时后，其他阶段仍保持原来的长度。以安全瓶颈为例：安全团队是按人类产出规模配置的，因此当智能体放大代码产出时，要么评审队列堆积，要么代码在评审不足的情况下发布。受监管组织无法接受任何一种结果，因此其安全与政策检查必须跟上智能体的速度。

To better realize the productivity gains of and secure agentic AI, the traditional SDLC lifecycle requires the same level of transformation as the implementation phase has undergone.
> 为了更好地实现智能体 AI 的生产力收益并确保其安全，传统 SDLC 生命周期需要经历与实现阶段同等级别的转型。

What is an AI-native SDLC?
> ---

## 2、Table of contents（什么是 AI 原生 SDLC？）

The AI-native SDLC is a reimagined process that combines the old control objectives with new enforcement. Instead of a linear flow, the process becomes a loop, and AI is embedded at each point. The AI-native SDLC promotes automated handover and triggering of subsequent plays, helping to address the manual and clunky nature of handoff between the phases of the traditional SDLC.
> AI 原生 SDLC 是一种重新构想的软件开发流程：它将旧有的控制目标与新的执行方式结合起来。流程不再是线性流转，而是变成一个循环，并且 AI 被嵌入到每一个节点。AI 原生 SDLC 推动自动化交接和后续动作的触发，从而解决传统 SDLC 各阶段之间手动、笨重的交接问题。

You'll also hear this shift called the agentic SDLC, the AI SDLC, or simply agentic software development — the labels differ, but they describe the same thing.
> 你也会听到类似说法，例如智能体式 SDLC、AI SDLC，或简单称为智能体式软件开发。叫法不同，但描述的是同一件事。

### 2.1、Plays（Plays：可执行动作）

The table below highlights the ends of the spectrum between traditional SDLC and AI-native SDLC, supported by Claude. Most organizations sit somewhere between the two columns.
> 传统 SDLC 与 Claude 支持的 AI 原生 SDLC 可以被看作一条光谱的两端。多数组织会处在两者之间。

| 阶段 | 传统 SDLC | AI 原生 SDLC |
|---|---|---|
| 规划 | 需求由委员会收集，通过工作坊和签字审批提炼，再手写成文档 | Claude 直接从源头综合痛点，并捕获到 `intent.md` 中；该文件既适合人读，也可被机器执行 |
| 设计 | 分析师编写规格，设计师再解析 | 需求和设计压缩到一次与智能体的工作会话中，由编码成 skills 的标准引导，并在 git 中版本化 |
| 构建 | 测试和代码手写，文档在主要开发之后补写 | 测试和代码由 AI 生成，组织知识以版本化、机器可读的 `CLAUDE.md` 文件和 skills 维护 |
| 测试 | QA 在阶段边界设置关卡 | 持续 evals 被编织进实现过程 |
| 部署 | 人类逐行审查代码，治理通过评审周期进行，且经常不一致 | 多层智能体评审，人类评审只保留给受监管和关键代码；治理通过 hooks 在 AI 行动时强制执行 |
| 维护 | 人类监控生产环境中的 bug | 智能体监控线上部署；任何控制区间被突破，都会被诊断并写回循环，成为新的 `intent.md` |

Stage Traditional SDLC AI-native SDLC Plan Requirements gathered by committee, distilled through workshops and sign-offs, written up by hand Claude synthesizes pain points straight from the sources and captures them within intent.md which is human readable and machine actionable Design Spec written by analysts, parsed by designers Requirements and design compressed into one working session with an agent, guided by standards encoded as skills, versioned in git Build Tests and code are handwritten and documentation is written after the main development happens Tests and code are generated by AI and institutional knowledge is maintained as versioned machine-readable CLAUDE.md files and skills Test QA gates at stage boundaries Continuous evals woven through implementation Deploy Humans review every line of code and governance occurs in review cycles, often inconsistently Layers of agentic review with human review reserved for regulated and critical code. Governance is enforced as the AI acts, with hooks as approval gates Maintain Humans watch production for bugs Agents monitor live deployments. Any breached control band is diagnosed and written back into the loop as a new intent.md
> 贯穿右侧列的主线，是“已提交的工件”。每个阶段结束时都会向版本控制提交一个工件，包括 `intent.md`、`spec.md`、`plan.md`、diff 及其测试、带评审发现的 PR、以及事故记录；下一个阶段则从读取该工件开始。早期阶段主要使用 `.md` 文件，因为产品负责人和智能体都能读取并操作同一个文件。从构建阶段开始，工件变成代码及其记录。提交链同时也是审计轨迹：谁提出了什么，智能体产出了什么，谁批准了它。

The thread running through the right-hand column is the committed artifact. Each stage ends by writing one to version control (including intent.md , spec.md , plan.md , the diff and its tests, the PR with its review findings, and the incident record) and the next stage begins by reading it. For the early stages, .md files are the predominant artifact because a product owner and an agent can both read and act on the same file. From Build onward, the artifact is code and its records. The chain of commits is also the audit trail: who asked for what, what the agent produced, and who approved it.
> 人类仍然对所有需要判断的决策负责。在智能体式 SDLC 中，人类注意力会随着必须被评审的工件一起移动。

Humans remain accountable for every decision that requires judgment. In the agentic SDLC world, the human attention shifts along with the artifacts that must be reviewed.
> **每个阶段都提交一个下个阶段可读取的工件。意图、规格、计划、diff 和评审发现共同构成审计轨迹。**

Every stage commits an artifact the next stage can read. Together, the intent, the spec, the plan, the diff and the review findings are the audit trail.
> ---

## 3、Stage 1 — Plan（Plays：可执行动作）

The plays are the core of the playbook and are grouped into six non-linear stages (Plan, Design, Build, Test, Deploy, Maintain), which together cover the complete lifecycle.
> 这些 play 是本手册的核心，被分为六个非线性阶段：Plan、Design、Build、Test、Deploy、Maintain，合起来覆盖完整生命周期。

Each play covers:
> 每个 play 都包含：

What changes;
> - 发生了什么变化；
Getting started;
> - 如何开始；
Concrete steps for implementation;
> - 具体实施步骤；
Governance considerations; and
> - 治理考虑；
How you measure whether it worked.
> - 如何衡量效果。

The steps are modular and organizations may choose to prioritize transforming different stages at different times based on their unique needs. Each play names its dependencies under "Prerequisites," which the dependency graph further illustrates.
> 这些步骤是模块化的。组织可以根据自身需要，优先改造不同阶段。每个 play 都会在“Prerequisites”中说明依赖关系，依赖图会进一步展示这些关系。

A stage ends by committing an artifact with the commit initiating the next stage. An accepted intent.md triggers the requirements and design pass, an approved spec.md triggers plan mode, a merged PR triggers the pipeline, and a breached control band in production writes the next intent.md and so the loop continues.
> 一个阶段以提交工件结束，而该提交会启动下一个阶段：被接受的 `intent.md` 触发需求和设计流程；批准后的 `spec.md` 触发 plan mode；合并后的 PR 触发流水线；生产环境中的控制区间突破会写出下一个 `intent.md`，循环由此持续。

First, you prompt each step by hand with the end state being a loop in which each accepted artifact fires the next gate. Human attention concentrates at the gates, reviewing what the agent flagged rather than starting each stage from scratch.
> 一开始，你可以手动提示每一步；最终目标则是建立一个循环：每个被接受的工件都会触发下一个关卡。人类注意力集中在关卡上，评审智能体标记出来的内容，而不是从零开始启动每个阶段。

The plays are listed with stage; the arrows give the order to adopt them in. The two are not the same. Start with any clay play — nothing points into it, so it needs nothing first. For any other play, the arrows pointing into it are the plays to adopt before it. 01 Plan
> ---

# 1、Stage 2 — Design（01 规划：Plan）

Ideas stop waiting for someone to write them up. Intent is captured once, in the originator's own words, as a version-controlled artifact the next stage can act on.
> 想法不再等待某个人把它写出来。意图被一次性捕获，使用发起人自己的语言，形成一个受版本控制、下个阶段可直接操作的工件。

## 1.1、Stage 3 — Build（捕获为 `intent.md`）

The intent.md , which kicks off the software development process can enter through different routes. A person has an idea, a ticket is filed, or an incident is surfaced via an alert (see Stage 6: Maintenance).
> 启动软件开发流程的 `intent.md` 可以来自不同路径：某个人有一个想法、有人提交了工单，或者某个事件通过告警浮现出来（见第 6 阶段：维护）。

When a person has an idea, they brainstorm with Claude and produce a markdown proto-spec. In the traditional SDLC, the same person must then convince a member of the product team to write the idea up with them or on their behalf.
> 当某个人有一个想法时，他可以与 Claude 头脑风暴，并生成一个 Markdown 格式的原型规格。在传统 SDLC 中，同一个人还必须说服产品团队成员与自己一起，或代表自己，将这个想法正式写出来。

The proto-spec generated by Claude is human readable, version-controlled, and immediately consumable by the next stage. The proto-spec is saved as an intent.md .
> Claude 生成的原型规格既适合人类阅读，也受版本控制，并且可以立即被下一阶段消费。这个原型规格保存为 `intent.md`。

Regardless of whether the intent originates from an event trigger or an agent, the same steps apply: the product owner reviews and corrects the agent-written intent.md before it is committed.
> 无论意图来自事件触发还是来自智能体，同样的步骤都适用：产品负责人在提交前评审并修正智能体写出的 `intent.md`。

Traditional An idea passes through backlog entries, user stories, story points, and refinement meetings before anyone can act on it. Ownership transfers at each handoff, so what reaches engineering is several steps removed from what the originator meant.
> - **传统方式：** 一个想法要经过 backlog 条目、用户故事、故事点和 refinement 会议，才有人能采取行动。每一次交接都会转移所有权，因此到达工程团队手中的内容，已经与最初发起人的真实意图隔了好几层。
AI-native The originator brainstorms with Claude and writes the result down as intent.md , a proto-spec in the originator's own terms. The artifact contains what is wanted, why, and under which constraints. Repeat processes are encoded via skills.
> - **AI 原生方式：** 发起人与 Claude 头脑风暴，并将结果写成 `intent.md`，即用发起人自己的语言表达的原型规格。该工件包含想要什么、为什么要做、以及约束条件。重复流程通过 skills 编码。

### 1.1.1、Stage 4 — Test（如何开始）

Prerequisites None.
> **前置条件：** 无。

Infrastructure Claude access for people who are not engineers (claude.ai or Cowork ); an agreed intent.md template; a shared, version-controlled home for intent that the product owner watches. For a single product the simplest home is an intent/ folder in the product repo. This setup keeps the artifact chain next to the code derived from it. A dedicated intent repo is only worth the overhead when intent spans many repositories, and in a monorepo it is a directory. The Stage 3: Build sidebar covers how this home relates to a Jira or requirements tool that already holds the record.
> **基础设施：** 非工程人员也能访问 Claude（claude.ai 或 Cowork）；一致认可的 `intent.md` 模板；一个产品负责人会关注的、共享且受版本控制的 intent 存放位置。对单一产品而言，最简单的位置是产品仓库中的 `intent/` 文件夹。这能让工件链与由其衍生出的代码放在一起。只有当 intent 跨越多个仓库时，专门的 intent 仓库才值得引入；在 monorepo 中，它只是一个目录。

Setting this up is a one-time task for the platform or engineering team. A technical team member needs to stand up the intent home and decide who can write to it, since many contributors will come from across the organization.
> 这项设置是平台或工程团队的一次性任务。技术团队成员需要建立 intent 的存放位置，并决定谁可以写入，因为许多贡献者会来自组织的不同部门。

Once the repository exists, contributors without git experience don't need to use git directly. Instead a connector to the version-control system (e.g. GitHub) lets Claude commit markdown files on their behalf from claude.ai or Cowork.
> 仓库建立后，没有 git 经验的贡献者不需要直接使用 git。相反，连接到版本控制系统（如 GitHub）的 connector 可以让 Claude 代表他们从 claude.ai 或 Cowork 提交 Markdown 文件。

### 1.1.2、Stage 5 — Deploy（如何执行）

The originator describes the problem to Claude in their own words. The originator may describe what they cannot do today, who is affected by the idea, what better looks like, or what is out of scope. No formal language is required.
> 1. 发起人用自己的语言向 Claude 描述问题：今天做不到什么，谁受影响，更好的状态是什么，哪些内容不在范围内。不需要正式语言。
Brainstorm until the idea is concrete. Claude asks the questions an analyst would ask: scope, users, constraints, and what success looks like.
> 2. 持续头脑风暴，直到想法足够具体。Claude 会提出分析师会问的问题：范围、用户、约束、成功标准。
Ask Claude to write the result as intent.md using the organization's template, which can be encoded as a skill set up by a technical team member and signed off by a lead. This can cover the problem, proposed outcome, affected users and systems, constraints, and open questions.
> 3. 要求 Claude 使用组织模板将结果写成 `intent.md`。这个模板可以由技术团队成员编码为 skill，并由负责人签字确认。模板可覆盖问题、预期结果、受影响用户和系统、约束、开放问题。
The originator corrects anything Claude misunderstood.
> 4. 发起人纠正 Claude 理解错误的地方。
Commit intent.md to the shared home. Author and timestamp join the record, and the product owner picks the idea up from there.
> 5. 将 `intent.md` 提交到共享位置。作者和时间戳进入记录，产品负责人从这里接手。

# 2、Intent: claims status self-service（理赔状态自助服务意图）
> 示例：

```md
# Intent: claims status self-service
Author: J. Ortiz (claims operations). Status: draft.

## Problem
Customers phone the contact center to ask where their claim is.
Handlers spend roughly a third of call time on status-only queries.

## Proposed outcome
Customers see claim status, next step and expected date in the portal.

## Affected users and systems
Claims handlers, portal team, claims-core API.

## Constraints
No new PII in the portal session. Existing authentication only.

## Open questions
Do third-party loss adjusters need access too?
```

### 2.1、Stage 6 — Maintain（治理考虑）

Author: J. Ortiz (claims operations). Status: draft.
> 证据是已提交的 `intent.md`，它列出了作者、时间戳和完整修订历史，并记录在 intent 存放位置的 git 历史中。产品负责人进行批准，而让 intent 进入第 2 阶段设计的接受或拒绝决定，会以 merge 或关闭 review 的形式记录下来。

### 2.2、Closing thoughts（结语）

## 2.1、Problem（问题）
> - **领先指标：** 从第一次对话到提交 `intent.md` 的时间，可从 intent 存放位置的 git 历史读取。预期应从多周的需求启发和 refinement 周期缩短到数小时。
Customers phone the contact center to ask where their claim is.
> - **滞后指标：** `intent.md` 的存活率，即有多少 `intent.md` 被产品负责人接受进入第 2 阶段设计，而不是关闭。接受或拒绝通过工件合并或 review 关闭记录。另外，统计同一变更在第一个 `spec.md` 提交之后，对 `intent.md` 做出的修改数量。

Handlers spend roughly a third of call time on status-only queries.
> ---

# 3、The shifts across the six stages of an AI-native SDLC（六个阶段的变化）

## 3.1、Proposed outcome（预期结果）
> 需求和设计合并到一次会话中。政策在写规格时就被应用，而不是几周后在评审中才被发现。

## 3.2、Capture as intent.md（捕获为 `intent.md`）

Customers see claim status, next step and expected date in the portal.
> 在产品负责人批准后，Claude 会读取已接受的 `intent.md`，生成需求与设计规格。这一过程由组织针对品牌、安全、合规和 UX 编写的 skills 引导。

## 3.3、Affected users and systems（受影响的用户和系统）
> 产品负责人评审这个规格，但不亲自编写它。该流程的目标，是创建一个工程团队可以据此规划的规格，并标记出值得关注的区域。

Claims handlers, portal team, claims-core API.
> 前端工作是最清晰的例子。`intent.md` 被接受后，产品负责人可以在 Claude Design（beta）中基于 `intent.md` 生成设计 mock，迭代 mock，然后将其导出到 Claude Code 进行构建。

## 3.4、Constraints（约束）
> - **传统方式：** 需求和设计由不同团队分阶段完成。分析师把想法形式化为需求，设计师再把这些需求解析回设计。分离有助于问责，但速度慢且信息有损。
No new PII in the portal session. Existing authentication only.
> - **AI 原生方式：** 两个阶段在一次提示会话中完成。Claude 读取 `intent.md`，在组织 skills 的约束下生成需求和设计规格，并标记关注点。

### 3.4.1、Getting started（如何开始）

## 3.5、Open questions（开放问题）
> **前置条件：** 已编写 `intent.md`；品牌、安全、合规和 UX 政策已写成 skills。

Do third-party loss adjusters need access too? Governance considerations
> **基础设施：** 有 Claude 访问权限的产品负责人。不要求工程技能。

### 3.5.1、How to execute it（如何执行）

The evidence is the committed intent.md , which lists the author, the timestamp and the full revision history. It's logged in the git history of the intent home. The product owner approves, and the accept or reject decision that sends the intent into Stage 2: Design is recorded as the merge or the closing review.
> 1. 产品负责人开启一个会话，确保组织 skills 可用，并附上 `intent.md`。
Time from first conversation to a committed intent.md , read from git history on the intent home, which records author and time stamp. The expectation is to fall from a multi-week elicitation and refinement cycle to hours.
> 2. 产品负责人的 prompt 指向 `intent.md`，明确约束，并要求标记关注点。一开始手动运行，随后将其编码为组织级 slash command。再进一步，把 intent home 中 `intent.md` 的接受动作作为触发器，通过非交互任务加载组织 skills、运行该 pass，并以 pull request 形式提交 `spec.md`。从那时起，产品负责人的第一次介入就是 review。
The survival rate, or the share of intent.md files that the product owner accepts into Stage 2: Design rather than closes. The accept or reject decision is recorded as the merge of the artifact or the closed review. Additionally, the number of changes made to the intent.md that are made after the first spec.md commit for the same change.
> 3. 同一产品负责人根据最初想法评审 spec：它是否解决了已陈述的问题？`intent.md` 中的开放问题是否被回答或被继续保留？
02 Design
> 4. 优先处理被标记的关注点，因为这些就是分析师原本会升级处理的问题。产品负责人在工程团队看到 spec 前，先与政策 owner 一起解决这些问题。
Requirements and design collapse into one session. Policy is applied while the spec is written, not discovered in a review weeks later.
> 5. 将 `spec.md` 与 `intent.md` 一起提交。二者记录了“提出了什么”和“决定了什么”。
Once approved by the product owner, Claude takes the accepted intent.md and produces a requirements and design spec. This is guided by the organization's skills for brand, security, compliance, and UX.
> 6. 产品负责人决定 spec 和 intent 是否进入构建阶段；若组织将某些内容归为高风险，则咨询技术负责人。这个决定始终由人类队友作出；接受 spec 会启动第 3 阶段构建中的 plan mode play。

The product owner reviews that spec, but doesn't write it. The goal of this process is to create a spec the engineering team can plan against, with flagged areas of concern.
> 示例 prompt：

```text
Read the attached intent.md and produce a requirements and design spec for integrating it into our existing codebase. Apply the skills available to you so the plan conforms to our brand guidelines, security policies and UX standards. Document the spec fully as spec.md, ready to hand to the engineering team. Describe clearly any areas of concern, especially where you cannot satisfy contradicting policies.
```

### 3.5.2、How to measure it（如何衡量）

Front-end work is the clearest example. Once the intent.md is accepted, the product owner mocks the design up in Claude Design (beta) from the intent.md , iterates on the mock, and then exports it to Claude Code to build.
> 实时政策在规格编写过程中被读取和应用，而不是几周后在评审中才被发现。组织 skills 作为 spec 的约束。spec、生成它的 prompt，以及当时生效的 skill 版本，全部记录在版本控制中。产品负责人签署 spec，并将被标记的问题路由给对应政策 owner。

### 3.5.3、Leading indicator（如何衡量）

Traditional Requirements and design are separate phases run by separate teams. Analysts formalize the idea into requirements and designers then parse those back into a design. The separation exists for accountability, but it is slow and lossy.
> - **领先指标：** 同一变更中，从 `intent.md` 提交到 `spec.md` 提交的经过时间（两个 git 时间戳），与旧的需求加设计周期比较。
AI-native Both phases happen in a single prompted session. Claude takes intent.md and produces a requirements and design spec, constrained by the organization's skills, with areas of concern flagged.
> - **滞后指标：** 构建开始后的需求返工。统计同一变更在第一个 `plan.md` 提交之后的 `spec.md` 提交次数，可直接从 git log 获取。

Prerequisites Write an intent.md file, with brand, security, compliance, and UX policies written as skills.
> ---

# 4、Lagging indicator（03 构建：Build）

Infrastructure A product owner with Claude access. No engineering skill is required.
> 没有被接受的计划，就不进行实现。组织知识变成智能体可读取的文件，护栏以代码方式运行，而不是依赖习惯。

## 4.1、Requirements and design（需求与设计）

The product owner opens a session with the organization's skills available and attaches the intent.md .
> 工程师以 plan mode 启动 Claude Code 会话，将第 2 阶段的已批准 `spec.md` 交给 Claude，让它访谈自己，反复迭代计划，直到工程师满意。

The product owners prompt points at the intent.md , names the constraints, and demands flagged concerns. Run it by hand at first, then codify it as an organization-level slash command. From there make the acceptance of intent.md in the intent home the trigger, with a non-interactive job that fires on the merge, run the pass with the organization's skills loaded, and commit spec.md as a pull request (the CI/CD play in Stage 5: Deploy covers the plumbing). From that point the product owner's first involvement is the review.
> - **传统方式：** 工程师阅读设计后开始写代码。变更将如何实现、具体改哪些文件、如何测试，通常停留在工程师脑中，最多写在工单评论里。没人能评审它。评审者第一次看到的是完成后的 diff，而此时返工成本已经很高。
The same product owner reviews the spec against the idea. Does the spec solve the stated problem, and are the open questions from intent.md answered or carried forward?
> - **AI 原生方式：** 工作从书面计划开始。Claude 在 plan mode 中读取代码库但不能修改任何内容。工程师在代码编写前纠正计划，并将批准版本提交为 `plan.md`，供后续阶段检查。

### 4.1.1、Claude Code plan mode as the default starting point（将 Claude Code plan mode 作为默认起点）

Work through the flagged concerns first as they are the points an analyst would have escalated. The product owner resolves each one with its policy owner before engineering sees the spec.
> **前置条件：** intent 工件（如 `intent.md` 或 `spec.md`，如果存在）以及 `CLAUDE.md` 文件会有帮助。

Commit spec.md alongside intent.md . The file pair records what was asked for and what was decided.
> **基础设施：** 能访问仓库的 Claude Code。

### 4.1.2、Claude Code on auto mode（如何执行）

The product owner decides whether the spec and intent progress to build, consulting a technical lead for anything the organization classes as higher risk. A human team mate always makes this call, and accepting the spec is what starts the plan mode play in Stage 3: Build.
> 1. 工程师以 plan mode 启动 Claude 会话。
What it looks like (the prompt)
> 2. 工程师提供 `intent.md` 和 `spec.md`，要求 Claude 输出实现计划，列明要修改的文件、工作顺序和证明其正确性的测试。
Read the attached intent.md and produce a requirements and design spec for integrating it into our existing codebase. Apply the skills available to you so the plan conforms to our brand guidelines, security policies and UX standards. Document the spec fully as spec.md, ready to hand to the engineering team. Describe clearly any areas of concern, especially where you cannot satisfy contradicting policies. Governance considerations
> 3. 追问计划：这个变更可能破坏什么？哪一步风险最大？Claude 没有选择的其他方案是什么？
Instead of being discovered in a review weeks later, the live policy is read and applied while the spec is written. The organization's skills are applied as constraints on the spec. The spec, the prompt that produced it, and the skill versions in force are all logged in version control. The product owner signs off the spec, and routes flagged concerns to the named policy owners.
> 4. 反复迭代，直到一个从未看过这段对话的工程师也能只根据该计划完成实现。
Leading indicator Elapsed time between the intent.md commit and the spec.md commit for the same change (two git timestamps), compared with the old requirements-plus-design cycle.
> 5. 将批准后的计划提交为 `plan.md`。计划加入审计轨迹，PR review play 会检查最终 diff 是否符合它。
Lagging indicator Requirements rework after build starts. Count spec.md commits dated after the first plan.md commit for the same change. Git log will give this directly.
> 6. 接受计划，让 Claude 实现。有了扎实计划，实现通常可以一次完成。
03 Build
> 7. 当实现偏离计划时，在同一个提交中更新 `plan.md`。可以考虑使用 hook 强制二者同步。

Nothing is implemented without an accepted plan. Institutional knowledge becomes files the agent reads, and the guardrails run as code rather than as habits.
> 示例 `plan.md`：

```md
# Plan: claims status self-service (from intent.md 2026-06-02)

## Files that change
portal/src/claims/StatusPanel.tsx (new), claims-api/routes/status.py,
claims-api/tests/test_status.py

## Order of work
1. Add the status endpoint behind existing auth.
2. Panel against the endpoint.
3. Wire into the portal nav.

## Risks
The claims-core API rate-limits at 50 rps; the panel must cache.

## Proof
test_status.py covers the four claim states; screenshot matches the
approved mock.
```

### 4.1.3、Sidebar Legacy systems and the source of truth（侧栏：遗留系统和事实来源）

Engineers start Claude Code sessions in plan mode , give Claude the approved spec.md from Stage 2: Design, and let it interview them, iterating on the plan until the engineer is happy with it.
> 设计评审发生在代码生成前，此时改变方向仍只是编辑文档。Plan mode 本身强制了这一点，因为在工程师接受计划之前，Claude 不能编辑文件。计划及其修订、谁接受了它，都会被记录。常规变更由工程师批准；组织归类为高风险的内容则交给技术负责人或架构师。

### 4.1.4、Skills as institutional knowledge（Skills 作为组织知识）

Traditional An engineer reads the design and starts writing code. How the change will be made, down to which files and which tests, stays in the engineer's head or at best a ticket comment. Nobody else can review it. The first thing a reviewer sees is the finished diff, and by then rework is slow.
> - **领先指标：** 从第一次实现 pass 就能合并的变更比例，以及从计划批准到 PR 合并的时间；所需数据应在 PR metadata 中。
AI-native Work starts with a written plan that Claude produces in plan mode, where it can read the codebase without changing anything. The engineer corrects the plan before code is written, and the approved version is committed as plan.md for later stages to check against.
> - **滞后指标：** 每个变更的返工周期，同样来自 PR metadata；以及合并后的 diff 仍然匹配已提交 `plan.md` 的频率。

## 4.2、Hooks as build-time guardrails（Hooks 作为构建期护栏）

Prerequisites The intent artifact ( intent.md or spec.md ) if one exists, and the CLAUDE.md file helps.
> Claude Code 也可以运行在 auto mode。工程师批准计划并迭代满意后，Claude 可以在每次修改时不再逐一提示，而是直接应用变更。随着后续 play 中的护栏成熟——调优后的 `CLAUDE.md`、编码政策的 skills、阻止不安全行为的 hooks、以及 Claude 可运行的测试套件——auto-accept 会成为常规工作的默认方式：紧凑的 `spec.md`、较小的爆炸半径、已有测试覆盖的代码。

Infrastructure Claude Code with access to the repository.
> 变化的方向是：用户不再盯着智能体编辑并逐个评审动作，而是在更长的自主会话结束后评审工件。Auto-accept 模式结合 worktrees 还能进一步实现个人和团队层面的并行化，是让 SDLC 自主运行并闭环的基础。

## 4.3、Parallel sessions and subagents（并行会话与 subagents）

The engineer starts the session in plan mode with Claude.
> 适用于流程产生的每个工件。

The engineer gives Claude the intent.md and the spec.md and asks for an implementation plan that names the files that change, the order of the work, and the tests that prove it.
> 现有 SDLC 流程可能已经跟踪工件，只是这些工件不在 Markdown 文件中。工作项可能在 Jira，需求可能在带监管可追溯性的工具中，设计可能在 Figma，变更审批可能在变更委员会系统中。这些系统很难被替代，因为审计员和监管者已经接受它们，其他团队也依赖它们。因此 AI 原生 SDLC 必须围绕既有系统适配。

Interrogate the plan by asking what the change could break, which step is most risky, and what other options Claude chose not to do.
> 过渡到 AI 原生 SDLC 时，要为每一种工件指定一个系统作为事实来源，其他系统只保存副本或指向原件的链接。可选配置包括：

Iterate until an engineer who has never seen the conversation could implement the change from the plan alone.
> - **仓库作为事实来源。** Markdown 工件是权威记录，遗留系统引用提交中的文件。这对工程主导型组织可能是最干净的配置，因为所有记录都在一个工具中，并使用同一个时间戳权威。
Commit the approved plan as plan.md . The plan joins the audit trail, and the PR review play (Stage 5: Deploy) checks the eventual diff against it.
> - **遗留系统作为事实来源。** Jira、ServiceNow 或需求工具保存权威记录，Markdown 工件是工作副本。Claude 在会话开始时读取记录，并在同一会话中通过 MCP connector 将结果写回。
Accept the plan and let Claude implement. With a solid plan, the implementation is often a single pass.
> - **链接作为最低要求。** 所有工件注明记录 ID，所有遗留记录包含 Markdown 文件的 commit SHA。这是转型早期的好起点，承认暂时存在两个事实来源。

When implementation departs from the plan, update plan.md in the same commit. Consider using a hook to enforce synchronization between the two.
> 遗留系统和 Markdown-first 系统可以共存，只要二者之间有链接，或者明确指定了一个事实来源。

## 4.4、Give Claude a feedback loop（给 Claude 一个反馈循环）

What it looks like (plan.md)
> `CLAUDE.md` 向 Claude 提供新成员需要知道的上下文，包括约定、命令、架构，以及团队最常遇到的错误。过去存在人脑和 wiki 中的知识，现在变成智能体在每次会话开始时读取的文件，由整个团队维护，并在每次出现错误时迭代。

### 4.4.1、Continuous evals in CI（CI 中的持续 evals）

# 5、Plan: claims status self-service（from intent.md 2026-06-02）
> **前置条件：** 无。

## 5.1、Files that change（变更文件）
> **基础设施：** 一个仓库、已安装 Claude Code，以及一位熟悉代码库的工程师。

### 5.1.1、AI in the PR review loop（PR review 循环中的 AI）

portal/src/claims/StatusPanel.tsx (new), claims-api/routes/status.py,
> 1. 在仓库中运行 `/init`。Claude 会根据它发现的内容生成初始 `CLAUDE.md`。
claims-api/tests/test _status.py
> 2. 精简生成文件，只保留新成员第一天需要知道的内容：构建、测试、lint 命令；重要约定；Claude 经常搞错的事项。
## 5.2、Order of work（工作顺序）
> 3. 将 `CLAUDE.md` 提交到仓库根目录，让整个团队共享一个版本，像代码一样评审其变更。
1. Add the status endpoint behind existing auth.
> 4. 一个有用的规则是：当 Claude 同一个错误犯了两次，就把修正写入 `CLAUDE.md`。
2. Panel against the endpoint.
> 5. 保持在一页以内，因为 Claude 会在会话开始时读取全部内容，任何过时信息都会占用上下文且没有收益。

3. Wire into the portal nav.
> 示例：

```md
# Payments service

## Commands
- Build: make build
- Test: make test (unit), make itest (integration, needs docker)
- Lint: make lint (runs in CI; fix before pushing)

## Conventions
- Java 21, Spring Boot 3. No new Lombok.
- Money is always BigDecimal, never double.
- Every endpoint needs an integration test in src/itest.

## Architecture
- api/ holds REST controllers, core/ holds domain logic,
adapters/ talks to external systems.
- Kafka events are defined in schemas/; never edit generated classes.

## Things Claude gets wrong
- Do not bump dependency versions; the platform team owns them.
- The legacy v1/ package is frozen; changes go in v2/.
```

### 5.2.1、Hooks as approval gates（Hooks 作为审批 gate）

## 5.3、Risks（风险）
> `CLAUDE.md` 受版本控制，因此智能体遵循的指令可被评审和审计。团队约定通过该文件应用，文件变更记录在 git 历史中，由 code owners 在 PR review 中批准。

The claims-core API rate-limits at 50 rps; the panel must cache.
> - **领先指标：** Claude 重复犯本应由 `CLAUDE.md` 捕获的错误的频率。
## 5.4、Proof（验证）
> - **滞后指标：** 新团队成员第一次合并 PR 的时间。

## 5.5、Worked example Managed settings for a regulated enterprise（Skills 作为组织知识）

test_ status.py covers the four claim states; screenshot matches the
> Skills 是组织将机构知识操作化的方式。指令是显式的、受版本控制的、可广泛应用的，并且在政策变化时集中更新。经验法则是：需要一致应用的机构知识写成 skill；属于 `CLAUDE.md` 或 prompt 的组件不要写成 skill。

### 5.5.1、CI/CD integration and deployment（CI/CD 集成与部署）

approved mock. Governance considerations
> 1. 选择一项今天执行不一致的知识，例如安全标准、API 设计约定或品牌规则。
Design review happens before any code is generated, when changing course is still a matter of editing a document. Plan mode enforces this itself, since Claude cannot edit files until the engineer accepts the plan. The plan and its revisions are logged along with who accepted it. Routine changes are approved by the engineer, and anything the organization classes as higher risk goes to a tech lead or architect.
> 2. 将其写成 skill：一个包含 `SKILL.md` 的文件夹，frontmatter 说明何时触发，正文说明要做什么。工程师根据政策 owner 的事实来源编写，可以让 Claude 协助。
Leading indicator Share of changes that merge from the first implementation pass, and time from plan approval to merged PR with the required data within the PR metadata.
> 3. 将 skill 放在仓库的 `.claude/skills/<name>/` 中，随代码一起发布；或通过 plugin 在组织范围内分发。
Lagging indicator Rework cycles per change, again from the PR metadata, and how often the merged diff still matches the committed plan.md .
> 4. 测试 skill 是否触发：用不同方式要求 Claude 执行相关任务，确认每次都会加载 skill。
Claude Code can also run in auto mode, where the engineer approves the plan and, once happy and iterated upon, Claude applies each change without a per-edit prompt. As the guardrails from the later plays mature (a tuned CLAUDE.md , skills that encode policy, hooks that block unsafe actions, and a test suite Claude can run), auto-accept becomes the default for routine work: a tight spec.md , a small blast radius, and code the tests already cover.
> 5. 政策变化时更新 skill，并由政策 owner 签署变更。
The shift is now away from the user watching the agent make the edits and reviewing actions, towards the review of artifacts after longer autonomous sessions. Auto-accept mode further enables parallelism across individuals and the team when used with worktrees and is fundamental to running the SDLC autonomously and closing the loop as described in Stage 6: Maintenance.
> 6. 工程师在下一次会话中自动获得新版本。

Applies to every artifact the process produces.
> 示例 `.claude/skills/secure-api-review/SKILL.md`：

```md
---
name: secure-api-review
description: Apply the API security standard. Use whenever creating or
modifying an external-facing endpoint, reviewing API code, or
generating an OpenAPI spec.
---

# Secure API review
When you create or change an API endpoint:
1. Authentication: every endpoint requires the gateway JWT;
no anonymous routes outside /health.
2. Input validation: validate request bodies against the OpenAPI
schema and reject unknown fields.
3. Audit: every state-changing endpoint emits an audit event with
actor, action, entity and timestamp.
4. Data classification: fields tagged pii in the schema must never
appear in logs or error messages.
Run scripts/check-endpoints.sh and include its output in your summary.
```

### 5.5.2、Maintenance and closing the loop（维护与闭环）

Existing SDLC processes likely already track artifacts, just not in markdown files. Work items may be in Jira, requirements in a tool with regulatory traceability built in, designs in Figma, and change approvals with a change board. Those systems are hard to displace because auditors and regulators already accept them and other teams depend on them, so the AI-native SDLC has to fit around what exists.
> Skill 是一种控制，但属于建议性控制。它会让 Claude 更可能在写代码时应用政策，但不能强制会话合规。必须始终成立的政策，需要 skill 背后有确定性机制，例如阻止动作的 hook，或在 PR 中重新检查政策的 review pass。Skill 让违规变少，hook 让违规几乎不可能发生。Skill 调用会记录在会话 traces 中，政策 owner 像评审代码一样评审 skill 变更。

### 5.5.3、Closing the loop（闭环）

When transitioning to the AI-native SDLC, for every artifact the process produces, name one system as the source of truth, with everything else holding a copy or a link to the original. The configurations below can be set up to have one source of truth, with the choice differing per artifact:
> - **领先指标：** 从政策 owner 批准政策变更，到更新后的 skill 合并的时间。
The repo as the source of truth. The markdown artifacts are the authoritative record and the legacy system references files within commits. This can be one of the cleanest configurations for engineering-led organizations, as all records live in one tool with one timestamp authority.
> - **滞后指标：** PR review 中引用该政策的发现应趋近于零。若没有下降，要么 skill 没有触发，要么其文本已经偏离官方政策。

## 5.6、Examples（Hooks 作为构建期护栏）

The legacy system as the source of truth. Jira, ServiceNow, or the requirements tool holds the authoritative record and the markdown artifacts are working copies. Claude reads the record at the start of the session and writes the outcome back through an MCP connector in the same session that produced the spec or the plan.
> Skill 是建议性控制，hook 是背后的确定性层。Claude 在实现过程中大多数动作是文件编辑和 shell 命令，因此构建阶段通常是 hooks 最常触发的地方。

Linkage as the minimum bar. All artifacts note the record ID and all legacy records contain the commit SHA of the markdown file. Linkage is a good place to start when transitioning to the AI-native SDLC, accepting that there are two sources of truth.
> 构建阶段 hooks 可以：

Both the legacy system and the markdown-first system can coexist, so long as there is a link between the two or one is declared the source of truth.
> - 阻止编辑受保护路径，例如生成类或冻结包；
The CLAUDE.md
> - 在文件编辑后运行 formatter 和 linter，避免漂移积累；
CLAUDE.md gives Claude the context a new joiner would need, covering conventions, commands, architecture, and the mistakes the team sees most often. Knowledge that used to sit in people's heads and on wikis becomes a file the agent reads at the start of every session, maintained by the whole team and iterated on whenever a mistake is made.
> - 防止凭证进入 diff。

Prerequisites None.
> 任何必须无例外成立的政策，都应由 hook 支撑。Hook 在匹配动作时运行，因此构建阶段 hooks 应当快速，并限定到发生变更的文件。更重的检查，如完整测试套件，属于提交或 PR 层面。

Infrastructure A repo, Claude Code installed, and one engineer who knows the codebase well.
> 需要人类批准的 hook 应放在第 5 阶段部署中的 gate，而不是构建阶段；否则审批提示会把人重新放回所有并行会话的关键路径。

## 5.7、Recurring codebase scans（周期性代码库扫描）

Run /init in the repo. Claude generates a starting CLAUDE.md from what it finds.
> 一个工程师可以同时驱动多个工作流。

Cut the generated file down to what a new joiner would need on day one. Keep the build, test and lint commands, the conventions that matter, and the things Claude keeps getting wrong.
> 并行会话是另一个完整的 Claude Code 实例，在独立 git worktree 中处理单独任务。每个独立会话彼此不了解，唯一共享的是负责引导它们的工程师。

Check CLAUDE.md into git at the repo root so the whole team shares one version and changes are reviewed like code.
> Subagent 则运行在单个会话内部，是带独立上下文窗口和工具限制的范围化助手，适合反复出现的任务，例如验证应用是否按预期运行。

A working rule helps here. When Claude makes a mistake twice, the correction goes into CLAUDE.md .
> 并行会话提高单个工程师同时推进的任务数量；subagents 则让每个会话专注于自身任务。工程师的职责转向引导和评审它们。

### 5.7.1、Governance considerations（治理考虑）

Keep it under a page, because Claude reads all of it at the start of a session and anything stale is taking up context for no benefit.
> 1. 工程师根据 plan mode 生成的计划，将工作拆分为修改不同文件的任务。共享文件的任务应放在同一会话中顺序执行。

示例 `CLAUDE.md`：

2. 每个并行任务获得自己的 worktree，例如在一个终端运行 `claude --worktree feature-auth`，另一个终端运行 `claude --worktree fix-rate-limit`。Worktree 是独立分支上的独立 checkout，可防止会话相互冲突。
3. 先从两三个会话开始。实际上限取决于一个人能认真评审多少工作流，只有在 review 跟得上时才增加会话数量。
4. 将重复任务变成 subagents，定义在 `.claude/agents/` 的 Markdown 文件中，包含名称、何时使用、可访问工具。例如：代码简化器、验证器、研究员。将定义提交到 git，让团队共享。

# 6、Payments service（支付服务）

## 6.1、Commands（命令）

- Build: make build
> 示例 `.claude/agents/verifier.md`：

```md
---
name: verifier
description: Runs the app and checks the change works before the session
reports done
tools: Bash, Read
---

Start the app with make run. Exercise the changed behavior and the two
nearest neighboring flows. Report what you ran, what you saw, and any
behavior that does not match plan.md. Do not fix anything; report only.
```

### 6.1.1、Claude on call with Claude Tag（Claude Tag 参与 on-call）

- Test: make test (unit), make itest (integration, needs docker)
> 会话越多，输出越多，因此控制必须来自仓库中的配置。Hooks 和权限设置适用于所有会话，且每个会话的行为会被记录并归因到运行它的工程师。

- Lint: make lint (runs in CI; fix before pushing)
> - **领先指标：** 在 review 质量保持的情况下，每位工程师的并发会话数；以及每天花在引导而非等待上的时间。
## 6.2、Conventions（约定）
> - **滞后指标：** 每位工程师每周合并的变更数量，并与返工率一起分析。

- Java 21 , Spring Boot 3. No new Lombok.
> ---

# 7、Resources and acknowledgments（资源与致谢）

- Money is always BigDecimal, never double.
> 每个会话在人类看到之前先检查自己的工作；引导智能体的配置像其写出的代码一样接受回归测试。

## 7.1、Give Claude a feedback loop（给 Claude 一个反馈循环）

- Every endpoint needs an integration test in src/itest.
> 始终给 Claude 一种验证自己工作的方式：测试、构建或截图 diff。会话在人类看到之前先检查自己的工作，并修复自己的错误。

## 7.2、Architecture（架构）
> 反馈循环不要与第 3 阶段中的 verifier subagent 混淆。反馈循环贯穿整个任务，可反复运行。Verifier subagent 则是在会话认为工作完成后，用新上下文窗口包装最终检查，避免判断被产生代码时的假设污染。

- api/ holds REST controllers, core/ holds domain logic,
> - **传统方式：** 代码是否工作的信号来得很晚：几分钟后的 CI、几天后的测试人员、几周后的生产环境。智能体产出代码时，迟来的信号意味着人必须检查其所有输出，这个人就变成瓶颈。
adapters/ talks to external systems.
> - **AI 原生方式：** 会话在交给人之前就拥有检查自身工作的能力。运行测试、构建、截图。Claude 迭代直到检查通过，因此到达工程师面前的内容已经通过了验证。

### 7.2.1、How to execute it（如何执行）

- Kafka events are defined in schemas/; never edit generated classes.
> 1. 如果今天检查工作需要一串命令和环境知识，把它包装成一个单一目标，例如 `make test` 或 `npm test`，并在失败时返回非零退出码。
## 7.3、Things Claude gets wrong（Claude 常犯的错误）
> 2. 在 `CLAUDE.md` 的 Commands 部分列出每个命令，以及健康输出示例。
- Do not bump dependency versions; the platform team owns them.
> 3. 陈述一个可量化目标，让 Claude 无需询问即可检查工作，例如“`test_status.py` 中所有测试通过”、“截图匹配附加 mock”、“endpoint 返回 200 且包含新字段”。
- The legacy v1/ package is frozen; changes go in v2/. Governance considerations
> 4. 对 bug fix，先写失败测试。让 Claude 将 bug 复现为测试，运行并确认它按预期原因失败。提交该测试。然后才要求 Claude 在不编辑测试的情况下让它通过，并用 hook 强制禁止编辑测试文件。修复前已经存在且智能体不能改写的测试，是 bug 已消失的证据。
CLAUDE.md is version controlled, so the instructions the agent works to are reviewable and auditable. Team conventions are applied through the file, changes to it are logged in git history, and code owners approve those changes in PR review.
> 5. 对 UI 工作，用视觉检查闭环。给 Claude 浏览器或截图工具、mock，让它实现、截图、比较、调整。两三轮很正常，结果应逐轮改善。
Leading indicator How often Claude repeats a mistake CLAUDE.md should have caught. The corrections or changes to the CLAUDE.md should be tracked within the git history.
> 6. 把验证作为“完成”的一部分。指令写在 `CLAUDE.md` 中：报告任务完成前运行测试，并展示输出。
Lagging indicator Time to first merged PR for a new member of the team from PR history.
> 7. 最后保护循环本身。修代码的智能体不应削弱验证该代码的检查。修复任务中阻止编辑测试文件的 hook 可以做到这一点。替代方案是在 review 中检查 diff，拒绝任何触碰测试的变更。

Skills are how an organization makes its institutional knowledge operational. The instructions are explicit, version-controlled, applied broadly, and updated centrally when policy changes. The rule of thumb: write a skill for institutional knowledge that must be applied consistently; don't write a skill for components that belong in CLAUDE.md or a prompt.
> 示例 `CLAUDE.md` 验证块：

```md
## Verifying your work
- Build: make build (must finish with "Build succeeded")
- Test: make test (all green; never skip or delete a failing test)
- Lint: make lint (zero warnings)
Run all three before reporting any task complete, and paste the output.
If a test fails, fix the code, not the test.
```

### 7.3.1、治理与衡量（治理与衡量）

Prerequisites None required. Having a CLAUDE.md helps, because it keeps the agent's working knowledge in the repo, but a skill does not depend on it.
> 被强制的是：任务报告完成前的验证，以及修复任务中阻止智能体编辑测试文件的限制。证据是 Claude 运行并粘贴的 `make test` 输出、构建日志或截图 diff，来自实际工具链。记录位置包括会话 transcript、OpenTelemetry export，以及 PR check run。

Infrastructure One policy with a named owner and a written source of truth.
> - **领先指标：** 智能体编写变更的首次 CI 通过率。
Pick one piece of knowledge that is enforced inconsistently today. This could be a security standard, an API design convention, or a brand rule.
> - **滞后指标：** 每个 PR 的 review 时间，以及变更失败率。

## 7.4、Continuous evals in CI（CI 中的持续 evals）

Write it as a skill, a folder containing a SKILL.md whose frontmatter says when it triggers and whose body says what to do. An engineer writes it from the policy owner's source of truth, using Claude to help.
> Evals 是 AI 原生版本的阶段关卡 QA。实践中，它意味着当智能体配置发生变化时运行一套测试：换新模型、改 prompt 时，eval suite 会判断智能体是否仍以相同标准完成工作。

Put the skill in the repo at .claude/skills/<name>/ so it ships with the code, or distribute it organization-wide through a plugin .
> Evals 应被视为活的测试套件。随着模型进步，曾经能区分好坏的案例可能失效，新的案例应来自持续监控。

### 7.4.1、How to execute it（如何执行）

Test that the skill triggers. Ask Claude to do the relevant task in different ways and confirm the skill loads each time.
> 1. 平台工程师收集最近真实工作中的 20 到 50 个任务，以及其预期/已接受结果。
When the policy changes, change the skill and have the policy owner sign off the change.
> 2. 将每个任务写成 eval，即 prompt 加上定义“可接受”的检查：测试通过、lint 干净、行为不变、政策被遵循。
Engineers pick up the new version automatically in their next session.
> 3. 该套件在 CI 中非交互运行，并在 `CLAUDE.md`、skills 或 hooks 发生变更时运行，因为这些配置会引导智能体，值得像代码一样回归测试。
What it looks like (.claude/skills/secure-api-review/SKILL.md)
> 4. 用结果 gate 配置变更。若某个 skill 变更导致通过率下降，需要在合并前评审。
name: secure-api-review
> 5. 每个生产事故都应成为一个 eval，由负责该事故的团队编写，并作为回归测试留在套件中。

description: Apply the API security standard. Use whenever creating or
> 示例 `.github/workflows/agent-evals.yml`：

```yaml
name: Agent evals
on:
  pull_request:
    paths: ['CLAUDE.md', '.claude/**']
  schedule:
    - cron: '0 2 * * *'
jobs:
  evals:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install -g @anthropic-ai/claude-code
      - name: Run eval suite
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          for eval in evals/*.json; do
            claude -p "$(jq -r '.prompt' $eval)" \
              --allowedTools "Read,Edit,Bash(make test)" \
              --output-format json > result.json
            ./evals/check.sh "$eval" result.json
          done
```

### 7.4.2、治理与衡量（治理与衡量）

modifying an external-facing endpoint, reviewing API code, or
> Evals 给 QA 一个能跟上智能体输出速度的 gate。通过率阈值作为 merge check 执行，运行结果被记录以便长期比较，配置变更由负责团队批准。

generating an OpenAPI spec.
> - **领先指标：** 每次运行报告的 eval 通过率趋势，以及生产事故变成永久 eval 所需时间。
# 8、Secure API review（安全 API 审查）
> - **滞后指标：** CI 中捕获的回归与生产中发现的回归之间的对比。

When you create or change an API endpoint:
> ---

# 9、部署：Deploy（部署：Deploy）

1. Authentication: every endpoint requires the gateway JWT;
> 评审双向运行，治理在智能体行动时被强制执行。智能体可以做到生产 gate 之前的一切，但不能越过它。

## 9.1、AI in the PR review loop（PR review 循环中的 AI）

no anonymous routes outside /health.
> Claude 既给出评审，也接收评审。它根据组织政策评审传入 PR，并处理自己 PR 上的 review comments。这使工程师能专注于 PR review 中的行为判断，本质上就是判断意图和风险。

2. Input validation: validate request bodies against the OpenAPI
> - **传统方式：** 评审能力按人类产出规划。PR 等待评审者阅读全文，评审质量随负载波动，作者不断催促，积压越来越多。
schema and reject unknown fields.
> - **AI 原生方式：** 所有 PR 都获得相同的评审 passes，发现按严重性排序。人类注意力上移到：变更是否实现了计划意图、风险是否可接受。

### 9.1.1、How to execute it（如何执行）

3. Audit: every state-changing endpoint emits an audit event with
> 1. 最快的起点是托管 Code Review 服务。管理员启用并选择仓库。当需要控制流水线，或要通过自己的云协议路由 API 调用时，可在自己的 CI 中运行 `claude-code-action`。
actor, action, entity and timestamp.
> 2. 技术负责人在仓库根目录编写 `REVIEW.md`，将评审政策分为组织关心的 passes：bug 和逻辑错误；安全与漏洞；与 `spec.md`、`plan.md` 和设计原则的一致性。`REVIEW.md` 还定义什么算 Important，什么算 Nit，以及哪些内容跳过。
4. Data classification: fields tagged pii in the schema must never
> 3. 技术负责人设定人类阈值。评审发现本身不会批准或阻止 PR，分支保护仍要求 code owner 批准。若平台工程师想基于 findings gate merge，可读取 check run 发布的机器可读严重性计数。
appear in logs or error messages.
> 4. 当评审者或作者在 review comment 中标记 `@claude`，Claude 会处理评论并推送修复。PR thread 记录请求和变更。对于 Claude 打开的 PR，可以更进一步，让 Claude babysit PR 直到可合并。
Run scripts/check-endpoints.sh and include its output in your summary. Governance considerations
> 5. Review findings 会反馈到 `CLAUDE.md`。当某个错误第二次被 review 标出，就将修正写入 `CLAUDE.md`。因为 review 也读取 `CLAUDE.md`，下一个 PR 起就能捕获该错误。
A skill is a control, though an advisory one. It makes Claude likely to apply the policy while the code is written, and nothing forces a session to comply with it. A policy that must always hold needs something deterministic behind the skill, such as a hook that blocks the action or a review pass that re-checks the policy at the PR. The skill makes violations rare and the hook makes them close to impossible. Skill invocations are logged in session traces, and the policy owner reviews skill changes like code.
> 6. 技术负责人每月通过给 findings 评级和在 `REVIEW.md` 中限制 Nit 数量来调优设置。生成路径和 CI 已强制的内容应被排除。

Leading indicator Time from the policy owner approving a policy change to the updated skill merging, taken from the PR on the skill folder.
> 示例 `REVIEW.md`：

```md
# Review instructions
## Passes
Run three passes and tag each finding with its pass:
- Bugs: logic errors, broken edge cases, subtle regressions
- Security: injection risks, authentication gaps, PII in logs
- Compliance: the change matches spec.md, plan.md and our design principles

## What Important means here
Reserve Important for findings that would break behavior, leak data
or breach a policy. Style and naming are nits.

## Cap the nits
Report at most five nits per review; summarize the rest as a count.

## Do not report
Generated files under src/gen/ and anything CI already enforces.
```

### 9.1.2、治理与衡量（治理与衡量）

Lagging indicator PR reviews findings that cite the policy, which should fall towards zero once the skill is applying the policy while the code is written. Where the findings don't fall towards zero, either the skill isn't triggering or its text has drifted from the official policy.
> 职责分离得以保留，因为写代码的智能体无法批准代码。`REVIEW.md` 中的评审政策应用到所有 PR，发现、修复、评分和批准都记录在 PR 历史中，因此 PR 本身就是审计记录。批准由人类通过分支保护完成，并由 findings 辅助判断。

A skill is an advisory control while a hook is the deterministic layer behind it. Most of Claude's actions are file edits and shell commands during implementation, so the build phase is where hooks can end up firing most often.
> - **领先指标：** 首次 review 时间应降到分钟级；以及无需人类触碰分支即可解决的 review comments 比例。
Build-phase hooks can:
> - **滞后指标：** 合并前捕获的缺陷和漏洞，与逃逸到生产环境的缺陷和漏洞对比。

## 9.2、Hooks as approval gates（Hooks 作为审批 gate）

Block edits to protected paths such as generated classes or a frozen package;
> 构建阶段使用 hooks 作为护栏，在没有人参与的情况下允许或阻止动作。Hook 也可以“询问”：暂停动作，直到特定人员批准。这正是发布 gating 所需。

Run the formatter and linter after file edits so drift never accumulates;
> 该 play 位于第 5 阶段部署，因为 release gate 是最清晰的场景。但 hooks 并非部署专用：凡是 Claude 行动的地方都可以运行。例如，hooks 可以在构建阶段阻止没有变更工单的 migration 和 infra 修改，也可以在测试阶段阻止智能体编辑测试文件。

### 9.2.1、How to execute it（如何执行）

Keep credentials out of the diff.
> 1. 工程领导与变更管理、合规团队列出必须保留的人类审批关卡，例如变更管理签字、发布授权、受保护路径编辑。
Back any skill whose policy has to hold without exception. A hook runs on each action that matches it, so build-phase hooks should be fast and scoped to the file that changed. Heavier checks such as the full test suite belong at the commit or the PR.
> 2. 平台工程师将每个 gate 表达为 hook：一个在 Claude 行动前运行的脚本，可以 allow、ask 或 block。
A hook that asks a human for approval belongs with the gates in Stage 5: Deploy, because an approval prompt during the build puts a person back on the critical path of all the sessions running in parallel.
> 3. 团队 hooks 放在 git 中的 `.claude/settings.json`，不可协商的 hooks 放在平台或 IT 管理的 managed settings 中，工程师不能关闭。
One engineer can drive several streams of work at once.
> 4. Block 应解释原因。当 hook 阻止动作时，Claude 输出中应显示原因和获得批准的路径。

A parallel session is another full Claude Code instance, working a separate task in its own git worktree . Each independent session knows nothing about the others, and the engineer steering them is the only thing they share.
> 示例 `.claude/settings.json`：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/production-gate.sh" }
        ]
      }
    ]
  }
}
```

A subagent runs inside a single session as a scoped helper with its own context window and tool limits and suits jobs that recur in multiple tasks such as verifying the app runs as expected.
> 对应 gate：

```bash
#!/bin/bash
# Production deploys require a named release authorization
cmd=$(jq -r '.tool_input.command' < /dev/stdin)
if [[ " $cmd " == * "deploy" * && " $cmd " == * "production" * ]]; then
  if [ -z " $RELEASE_APPROVAL " ]; then
    echo "Production deploys need a release authorization." >&2
    exit 2
  fi
fi
exit 0
```

### 9.2.2、企业受监管场景示例（企业受监管场景示例）

Parallel sessions raise the number of tasks an engineer can have in flight, while subagents keep each session focused on its own task. The engineer's job is steering and reviewing all of them.
> Managed settings 可由平台团队通过 MDM 或 admin console 部署，工程师无法编辑或覆盖。示例配置包括：禁止读取 `.env` 和 secrets、禁止任意 WebFetch/curl/wget、只允许 `git`、`make build/test/lint` 等安全命令；禁止绕过权限；启用 sandbox；限制网络域名；拒绝读取 `~/.ssh`、`~/.aws/credentials` 并剥离敏感环境变量；只允许 managed hooks、approved plugin marketplace、managed MCP servers；要求最低 Claude Code 版本。

Traditional One engineer works one task at a time and spends a significant portion of their day or week on builds, tests and reviewers. Switching between tasks while waiting is possible, but the context switch is tiring enough that few people choose to.
> 这些设置在控制意义上带来的收益包括：

AI-native One engineer runs several Claude sessions at once, each in its own worktree on its own task. Repeated jobs become subagents with their own context and tool limits. The engineer's job shifts to orchestrating, and eventually, to building and monitoring loops.
> - `permissions.deny` 防止 secrets 进入智能体上下文，并阻止通过工具任意网络外连。
Prerequisites The CLAUDE.md , since all sessions read the file. The feedback loop (Stage 4: Test) also helps here, because less supervision from the engineer is needed when a session can verify its own work.
> - `permissions.allow` 预先批准安全的内循环，避免 deny list 造成提示疲劳。
Infrastructure A git repository, since isolation comes from worktrees and permission settings tuned so sessions are not waiting on approval prompts for commands the organization considers safe.
> - `disableBypassPermissionsMode` 和 `allowManagedPermissionRulesOnly` 防止工程师、项目文件或命令行 flag 放宽规则。
The engineer splits the work into tasks that touch different files, using the plan from the plan mode play (Stage 3: Build) to see where the work is independent. Tasks that share files run in a single session, one after another.
> - `sandbox` 解决权限无法覆盖的缺口：即使禁止 WebFetch，也要阻止 shell 命令直接访问网络。
Each parallel task gets its own worktree, for example claude --worktree feature-auth in one terminal and claude --worktree fix-rate-limit in another. A worktree is a separate checkout on its own branch, which stops sessions colliding on files.
> - `credentials` 进一步限制 sandboxed shell 读取本地凭证或继承敏感环境变量。
Two or three sessions is a sensible starting point. The practical ceiling is how many streams one person can review properly, so add sessions only while review is keeping up.
> - `allowManagedHooksOnly` 确保审批 gates 只能来自受管配置。
Turn repeated jobs into subagents, as defined in markdown files in .claude/agents/ , each with a name, a description of when to use it, and the tools it may touch. Examples include a code simplifier that strips needless complexity after the main agent finishes, a verifier that runs the app and checks behavior, a researcher that explores the codebase and reports back without flooding the main context. Check the definitions into git so the whole team shares them.
> - `disableSideloadFlags` 与 `strictKnownMarketplaces` 确保 skills、agents、hooks、MCP servers 来自组织批准的插件市场。
What it looks like (.claude/agents/verifier.md)
> - `allowManagedMcpServersOnly` 让智能体的工具面由平台团队维护 allowlist。
name: verifier
> - `requiredMinimumVersion` 阻止低于批准版本的客户端启动。

description : Runs the app and checks the change works before the session
> 这应被视为可裁剪的起点，而不是可直接照抄的建议。每个 deny 都会牺牲能力，正确平衡取决于仓库的数据分类。

## 9.3、CI/CD integration and deployment（CI/CD 集成与部署）

reports done
> 在 CI/CD 流水线中非交互运行 Claude Code，用 sandbox 让长时间运行的智能体安全执行，通过 MCP 暴露部署，并在智能体真正需要之前排练 rollback 路径。

tools : Bash, Read
> - **传统方式：** 流水线运行确定性脚本，凡是需要判断的事情等待人类，例如 triage flaky test、写 changelog、分析 build 为什么失败。部署和 rollback 是人类在压力下照着 runbook 执行。
Start the app with make run. Exercise the changed behavior and the two
> - **AI 原生方式：** Claude 在 pipeline 中非交互执行判断步骤，在带 scoped credentials 的 sandbox 中运行。部署工具通过 MCP 暴露给智能体，因此写代码、测试的 workflow 也能在组织定义的环境 gates 内发布和回滚。

### 9.3.1、How to execute it（如何执行）

nearest neighboring flows. Report what you ran, what you saw, and any
> 1. 平台工程师从只读判断步骤开始：在 pipeline job 中使用 `claude -p` triage 失败构建、总结 flaky test、草拟 changelog。
behavior that does not match plan.md. Do not fix anything; report only. Governance considerations
> 2. 在现有 gates 后添加写入步骤，例如修 lint、更新生成文档、通过 `@claude` 处理 review comments。智能体写出的任何内容都以 PR 形式进入分支保护，不能直接 push 到 main。
More sessions means more output, so the controls have to come from configuration in the repo. Hooks and permission settings there apply to all sessions, and what a session does is logged and attributed to the engineer who ran it.
> 3. 执行环境必须 sandboxed：agent jobs 在容器中运行，配合网络策略和短期 scoped tokens，默认不持有生产凭证。
Leading indicator Concurrent sessions per engineer while review quality holds, counted from the OpenTelemetry export, and the share of the day spent steering rather than waiting.
> 4. 通过 MCP 暴露部署。Deploy、status、rollback 变成按环境 scoped 的工具，使智能体的部署能力是 allowlist，而不是带凭证的 shell 脚本。
Lagging indicator Changes merged per engineer per week read alongside the rework rate as determined per the PR history.
> 5. 按环境分层自治程度。开发环境中智能体可自由部署；生产环境中智能体准备 release，由 release manager 授权，hook 强制执行 production gate；staging 介于两者之间。
04 Test
> 6. Rollback 应是 pipeline 中最常排练的路径，应是智能体可运行且在 staging 定期演练的单一命令。第 6 阶段闭环 play 会在控制区间突破时调用该 rollback，因此必须提前证明可用。

Every session checks its own work before a human sees it, and the configuration that steers the agent gets regression-tested like the code it writes.
> 示例 pipeline step：

```yaml
- name: Triage failed build
  if: failure()
  run: >
    claude -p "Read the build log at out/build.log. Identify the most
    likely cause, say whether the failure looks flaky or real, and write a
    three-line summary for the PR thread." >> triage.md
```

### 9.3.2、治理与衡量（治理与衡量）

Always give Claude a way to verify its own work, whether tests, a build, or a screenshot diff. A session checks its own work and fixes its own mistakes before an engineer sees them.
> 治理原则是：智能体可以行动到生产 gate 之前，但不能越过它。分支保护确保智能体写入内容都变成 PR，不能直达 main；production deploy hook 阻止发布，直到指定 release manager 授权；每个非交互运行都使用智能体自己的身份，pipeline 日志可区分智能体做了什么与触发它的工程师做了什么。

The feedback loop should not be confused with a verifier subagent (Stage 3: Build). The feedback loop runs through the whole task as many times as the work. The verifier subagent, on the other hand, is one way to package the final check by running a fresh context window once the session believes the work is done. This way the verdict is not colored by the assumptions that produced the code.
> - **领先指标：** 无需呼叫人类即可 triage 的 pipeline failures 比例。
Traditional The signal that code works arrives late. CI minutes later, a tester days later, production weeks later. With an agent producing the code, a late signal means a person has to check all of its output, and that person becomes the bottleneck.
> - **滞后指标：** DORA 指标，这些通常已由 CI 和部署工具产生。

AI-native The session is given a way to check its own work before a person sees it. Run the tests, run the build, take the screenshot. Claude iterates until the check passes, so what reaches the engineer has already passed it. Setting the loop up falls to the engineer running the session, and the steps below are written for them.
> ---

# 10、维护：Maintain（维护：Maintain）

Prerequisites None.
> 循环闭合。一个触发器在没有人类处于调用路径的情况下调用 Claude，而 Claude 发现的内容会作为 `intent.md` 重新进入流水线。

## 10.1、Maintenance and closing the loop（维护与闭环）

Infrastructure A test suite and a build that run locally with one command each. For the UI work, a way for Claude to see the result is crucial, either a browser tool or a screenshot utility wired in via MCP.
> 前文讨论的是如何将 Claude 加入 SDLC 的每个阶段，而每个阶段都需要人类启动初始步骤。本阶段则转向自主运行 Claude 来闭合循环。

If checking the work today takes a sequence of commands and some environment knowledge, wrap it in a single target such as "make test" or "npm test" that exits non-zero on failure.
> 例如，持续运行的监控智能体可以在 bug 工单产生后创建 `intent.md`，并流经需求、计划、构建、测试和评审阶段。第 6 阶段维护以 headless 方式运行，阶段之间有独立的置信 gate，由确定性检查或对抗性评审智能体判断上一阶段输出是继续推进，还是升级给人类。

In the CLAUDE.md 's Commands section, list each command with an example of a healthy output.
> - **传统方式：** 维护是反应式阶段。所有工单或事故都等待人类行动并重启流程。凌晨 3 点的告警可能被错过，工单可能一直待在 backlog，事后复盘行动可能因为下一场火灾而永远无法进入代码库。
State a target and make it quantifiable so Claude can check the work without asking you, for example: "All tests in test_status.py pass," "the screenshot matches the attached mock," or "the endpoint returns 200 with the new field".
> - **AI 原生方式：** 控制区间突破、工单、频道消息或计划任务等触发器，可以在没有人类处于路径中的情况下调用 Claude。Claude 诊断问题，只通过受 gate 约束的路径行动，并将发现写成 `intent.md`，进入前文阶段。人类负责 triage 和 review，而不再负责启动。

## 10.2、Closing the loop（闭环）

For bug fixes, write the failing test first. Ask Claude to reproduce the bug as a test, run it, and confirm it fails for the reason you expect. Commit that test. Only then ask Claude to make it pass without editing the test, with the test-file hook from the final step enforcing the restriction. A test that existed before the fix, and that the agent couldn't rewrite, is proof the bug is gone.
> 确定性脚本监控生产环境，并在控制区间被突破时调用 Claude。控制区间突破监控是自主循环的一个有用示例。

### 10.2.1、How to execute it（如何执行）

For UI work, close the loop with a visual check. Give Claude a browser or screenshot tool, give it the mock, and let it iterate. Implement, screenshot, compare, and adjust. Two or three rounds is normal, and the result should improve with each one.
> 1. 服务 owner 或平台工程师选择一个有稳定滚动基线的指标，如 CI 测试失败率、部署后 5xx 率、PR 周期时间。
Make verification part of "done." Instruction lives in CLAUDE.md . Run the tests before reporting a task complete, and show the output.
> 2. 编写检测脚本，通常基于滚动窗口的均值和标准差，并使用 Western Electric 或类似规则，以捕捉缓慢漂移和尖峰。脚本受版本控制并有单元测试；检测过程完全确定性，不涉及模型。
Finally, the loop itself needs protecting, because an agent fixing code must not be able to weaken the check on that code. A hook that blocks edits to test files during a fix task does this. The alternative is to check the diff in review and reject any change that touches a test.
> 3. 响应等级定义在版本化配置中，例如 `bands.yaml`：1σ 只记录日志，2σ 调用 Claude 只读诊断，3σ 允许 Claude 行动，但只能打开进入 review gate 的 PR 或触发预先批准的 runbook。
What it looks like (CLAUDE.md verification block)
> 4. 触发层可以是 GitHub/GitLab scheduled workflow、来自现有监控栈的 webhook，或网络内 Cron Job。Claude 以无状态方式运行，可以是 CI runner 上的非交互步骤，也可以是 sandboxed container 中的 Agent SDK 服务。
## 10.3、Verifying your work（验证你的工作）
> 5. 智能体将诊断写成第 1 阶段 Plan 格式的 `intent.md`，包含异常和证据、预期结果、受影响系统、开放问题。从那里开始，发现像其他变更一样进入流水线。
- Build: make build (must finish with "Build succeeded" )
> 6. 服务 owner 或 on-call 工程师 triage 队列：立即修复、排期或驳回。驳回会用于调优控制区间并减少噪声。
- Test: make test (all green; never skip or delete a failing test)
> 7. 修复上线后，为该事故添加 eval，确保未来防止同类问题。

- Lint: make lint (zero warnings)
> 示例 `bands.yaml`：

```yaml
metric: ci_test_failure_rate
baseline: rolling_30d
rules: western_electric
tiers:
  1sigma: { action: log }
  2sigma: { action: diagnose, tools: "Read,Grep,Bash(gh run view *)" }
  3sigma: { action: propose, routes: [ pull_request, runbook:rollback-deploy ] }
```

### 10.3.1、治理与衡量（治理与衡量）

Run all three before reporting any task complete, and paste the output.
> 等级边界由版本化配置强制执行，权限和 managed settings 拒绝生产访问。调用、发现和 triage 决策都带时间戳记录。服务 owner 负责 triage 和批准发现，后续变更经过正常 PR review gate，智能体可触发的 runbooks 预先获得批准。

If a test fails, fix the code, not the test. Governance considerations
> - **领先指标：** 从控制区间突破到 triage 队列中出现 `intent.md` 的时间，对比过去从事故到 post-mortem 行动的时间。
What is enforced Verification before a task is reported done, and the block on the agent editing test files during a fix, both implemented as hooks where the organization wants them guaranteed.
> - **滞后指标：** 成为已合并修复的发现比例，以及同类事故重复发生率。

What the evidence is The literal output of "make test," the build log, or the screenshot diff that Claude ran and pasted, so the evidence comes from the toolchain.
> 示例：

Where it is logged In the session transcript, which the OpenTelemetry export forwards to the organization's observability stack, and in the PR's check run, where the reviewer and any later auditor can both see it.
> - 当 CI 测试失败率突破 3σ，智能体隔离 flaky test 或打开 revert PR，由 review gate 决定。
Who approves The code owner reviewing the PR, who can concentrate on intent and risk because the mechanical evidence is already attached.
> - 当部署后 5xx 率在部署窗口内突破 3σ，智能体触发现有 rollback pipeline。
Leading indicator First-pass CI success rate for agent-written changes, which the CI system already supports.
> - 当 PR cycle time 触发漂移规则，智能体为工程领导编写报告，说明该 harness 也可用于流程指标。

Lagging indicator Review time per PR (from the PR metadata), which should fall once the tests catch what reviewers used to catch, and the change failure rate from an incident tracker.
> 检测保持确定性。Claude 只在控制区间突破后被调用，响应等级决定它可以做什么。

## 10.4、Recurring codebase scans（周期性代码库扫描）

Evals are the AI-native equivalent of stage-gate QA. In practice that means a suite that runs whenever the agent's configuration changes. When a new model is swapped in or a prompt is rewritten, the eval suite says whether the agent still does the work to the same standard.
> 安全扫描是关于某个代码库在某个模型下的时间点声明，而这两部分都会过时：代码每周都在变化，每一代模型也能发现前一代漏掉的漏洞。AI 原生答案是按计划运行扫描，让发现通过与其他代码变更相同的 gate。

The evals should be seen as a live suite. As models improve, cases that once discriminated stop doing so and new ones must be added that arise from ongoing monitoring.
> Claude Security 是托管形式的定期扫描。连接 GitHub 仓库后，扫描在 Anthropic 基础设施中使用 Claude Mythos 5 运行，每个 finding 在报告前都会验证，并附上置信评分。建议补丁可在 Claude Code on the Web 中评审和应用。组织无需直接访问模型，就能获得 findings。

Depending on the use case, some teams may prefer to run these evals offline on a set cadence rather than on every change. The steps below are for continuous evaluations.
> - **传统方式：** 安全扫描是发布或审计前启动的一次性事件。报告进入 tracker，backlog 由人手工处理，直到下一次扫描。期间写入的代码只受 PR review 覆盖。
Prerequisites The CLAUDE.md and feedback loop (Stage 4: Test).
> - **AI 原生方式：** 扫描按计划针对每个连接仓库运行，使用可用的最强模型，finding 在人类阅读前先验证。每个 finding 像控制区间突破一样处理：一 PR 能完成的修复进入 review gate；更大的问题写成 `intent.md`。

### 10.4.1、How to execute it（如何执行）

Infrastructure CI that can run Claude Code non-interactively, and an API key with budget for eval runs.
> 1. 安全负责人连接仓库，并按 repo、service 或 team 组织为 projects，确保 findings 从一开始就有明确 ownership。
The platform engineer collects 20 to 50 real tasks from recent work with its expected/accepted outcome.
> 2. 对最关键仓库运行首次完整扫描，即使它们已经被其他工具或早期模型扫描过。首次扫描应作为 baseline，可能会在此前被认为干净的代码中发现问题。
Write each task as an eval, meaning the prompt plus the checks that define acceptable (tests pass, lint clean, behavior unchanged, policy followed).
> 3. 为每个 project 设置计划。对活跃开发服务，weekly 是合理默认；大型或混合仓库可按目录或分支限定扫描范围。
The suite runs non-interactively in CI on a schedule and on any change to CLAUDE.md , skills or hooks, since that configuration steers the agent and deserves the regression testing that code gets.
> 4. 根据置信评分 triage findings。驳回应附原因，保证 dismissal 被记录，且同一 finding 不会在下一次扫描中作为新问题返回。
Gate configuration changes on the results. A skill change that drops the pass rate gets reviewed before it merges.
> 5. 对边界清晰的 finding，在 Claude Code on the Web 中打开建议补丁，评审后通过 PR review gate 进入流程。提出修复的智能体没有批准路径。
Each production incident gets an eval, written by the team that owned the incident, and stays in the suite as a regression test.
> 6. 对超出一个补丁范围的问题，如架构弱点或跨服务重复模式，按第 1 阶段格式写成 `intent.md`，从 Plan 开始。
What it looks like (.github/workflows/agent-evals.yml)
> 7. 修复发布到生产后，为该漏洞类别添加 eval。
name: Agent evals
> 8. 将 findings 导出为 CSV 或 Markdown，或使用 webhooks 与组织现有 tracker 和审计系统集成。

### 10.4.2、治理与衡量（治理与衡量）

on:
> 扫描在组织 admin 控制下运行：连接哪些仓库、谁持有扫描 seat、spend limit 都集中设置。每个 finding 有验证结果和置信评分，每个 dismissal 有原因，因此扫描历史就是发现、修复和有意识接受风险的审计记录。修复通过 PR review gate 和分支保护进入生产，而不是由扫描本身直接发布。

pull_request:
> - **领先指标：** 已按计划连接的仓库比例，以及 finding 被报告后进入 PR review gate 的时间。
paths: [ 'CLAUDE.md' , '.claude/**' ]
> - **滞后指标：** 定期扫描发现的漏洞与生产环境或外部报告发现的漏洞对比；以及经过多次扫描的仓库中，每次扫描 findings 数量趋势。

## 10.5、Claude on call with Claude Tag（Claude Tag 参与 on-call）

schedule:
> 事故也可能通过 Slack、Teams 等工作沟通应用进入。事故可能是一条晚上 10 点发在 incident channel 中的紧急修复消息，现在可以立即被处理。Claude Tag（当前 Slack public beta）让 Claude 以自己的身份成为频道成员，因此每个新事故都会有一个 first responder，而响应本身也成为未来事故循环和记忆的一部分。

- cron: '0 2 * * *'
> 对话和组织知识保留在频道中，频道中的任何人都能引导并触发响应。任何团队成员都可以实时测试假设、探索新选项、调查问题，频道历史也增加了可审计性。通过 MCP 访问，Claude 可以验证指标已经回到基线，并在线程中确认，将 post-mortem 写入受版本控制的 lessons 文件，供未来调查读取。

jobs:
> 事故并不是 Claude Tag 能接手的唯一工作。通过 MCP 在工单中标记它，或在频道中要求它 triage 工作，也会走同样流程。边界清晰的小修复以 PR 形式通过 review gate；更大的工作写成 `intent.md` 进入第 1 阶段 Plan。循环由此开始自我供给。

evals:
> 频道就是审计轨迹：请求、诊断、人类授权和修复都保留在处理事故的地方。

runs-on: ubuntu-latest
> ---

## 10.6、Closing thoughts（结语）

steps:
> 模型和 harness 已经变得更先进，使组织不仅能改变代码生产方式，也能改造整个软件开发生命周期。

- uses: actions/checkout@v4
> 这种转型仍然将人类判断置于流程中心，并考虑大型企业组织的治理和监管要求。

- run: npm install -g @anthropic-ai/claude-code
> 本指南整合了 Anthropic Applied AI 团队每天为客户实践的许多真实最佳实践，希望它成为一份实用且可执行的资源。

- name: Run eval suite
> **循环持续运行。人类判断保持在循环之上。**

env:
> ---

## 10.7、Resources and acknowledgments（资源与致谢）

ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
> 平台团队设置这些控制所需的文档大致包括：

run: |
> - Claude Code 组织设置：admin decision map
for eval in evals/*.json; do
> - Settings reference 与 precedence
claude -p "$(jq -r '.prompt' $eval)" \
> - Server-managed settings
--allowedTools "Read,Edit,Bash(make test)" \
> - Permissions
--output-format json > result.json
> - Sandboxing：操作系统层面的文件系统和网络隔离
./evals/check.sh "$eval" result.json
> - Hooks guide 与 reference
done Governance considerations
> - Skills
Evals give QA a gate that keeps up with agent output. The pass-rate threshold is enforced as a merge check, runs are logged so results can be compared over time, and the team that owns the configuration change approves it.
> - Plugins 与私有 marketplaces
Leading indicator The eval pass rate over time, reported by the suite on every run, and how long a production incident takes to become a permanent eval.
> - Managed MCP
Lagging indicator Regressions caught in CI compared with regressions found in production derived from the incident tracker.
> - Enterprise deployment：Bedrock、Vertex、Foundry
05 Deploy
> - Enterprise network configuration
Review runs in both directions, and governance is enforced as the agent acts. The agent does everything up to the production gate and nothing past it.
> - Monitoring：OpenTelemetry
Claude both gives and receives reviews. It reviews incoming PRs against the organization's policies and addresses review comments on its own PRs. This allows engineers to focus on behavior in their PR review, which boils down to judging intent and risk.
> - Analytics dashboard
Traditional Review capacity was planned around human output. A PR waits for a reviewer to read all of it, review quality varies with the reviewer's load, and the author chases while the backlog grows.
> - Compliance API
AI-native All PRs get an identical set of review passes, with findings ranked by severity. Human attention moves up a level, to whether the change does what the plan intended and whether the risk is acceptable.
> - Security model

Prerequisites An updated CLAUDE.md file from Stage 3: Build; skills if the review passes enforce written policies, defined subagents.
> 感谢 Jim Blackhurst、Will Steuk 和 Jamal Arif 对本指南的贡献。本指南受到他们过往大量工作的启发，并建立在这些工作之上。
