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
> 一年前，组织已经开始使用AI 以难以想象的速度编写代码，但围绕代码的流程并没有以同样的速度变化。

Many engineering teams still have the same approval gates, reviews, handoffs, and policies, stalling productivity gains made by using agentic coding solutions like [Claude Code](https://claude.com/product/claude-code).
> 许多工程团队仍然有相同的审批gate、评审、交接和政策，阻碍了使用 Claude Code 等智能体式编码解决方案所带来的生产力提升。

The software development lifecycle (SDLC) is the process that takes software from idea to production. Most organizations run some version of the same six stages, covering planning, design, building, testing, deploying, and maintaining software. Traditionally, each stage is a discrete phase owned by a different role. Product managers write requirements, technical architects turn them into designs, engineers build the designs, QA teams at regulated enterprises verify it, releases teams ship it, and operations monitors what is running. Work moves between the phases through documents, tickets, and sign-offs.
> 软件开发生命周期 (SDLC) 是软件从构思到生产的过程。大多数组织都运行某种版本的六阶段流程，涵盖规划、设计、构建、测试、部署和维护软件。传统上，每个阶段都是由不同角色负责的离散阶段。产品经理编写需求，技术架构师将其转化为设计，工程师构建设计，受监管企业的 QA 团队验证它，发布团队上线，运营监控线上运行内容。工作通过文档、工单和签字审批在各阶段之间流转。

The traditional software development lifecycle (SDLC) is process-heavy to ensure accountability and control at each step. However, the traditional SDLC was designed to maximize efficiency in an era where the most time-consuming and expensive stage was writing and implementing code, which is no longer the case. PRDs, estimation rituals, and product security reviews all existed to force alignment during what could be weeks, months, or quarters of development work.
> 传统的软件开发生命周期 (SDLC) 流程繁重，以确保每个步骤的责任和控制。然而，在最耗时、最昂贵的阶段是编写和实现代码的时代，传统的 SDLC 旨在最大限度地提高效率，但现在情况已不再如此。 PRD、估算仪式和产品安全评审都是为了在可能需要数周、数月或几个季度的开发工作中强制保持对齐。

The traditional SDLC also features controls that assume every step is performed by humans. The organizations generating the most value have rebuilt their process around what agentic AI can now do, while ensuring that humans stay in the loop. In this guide, we walk through several of our Applied AI team's best practices for integrating Claude internally across each stage of the SDLC to accelerate development and make processes run faster, inspired by working with our customers.
> 传统的 SDLC 还具有假设每个步骤都是由人类执行的控制机制。产生最大价值的组织已经围绕智能体 AI现在可以做什么重建了他们的流程，同时确保人类仍参与关键环节。在本指南中，我们将介绍应用AI 团队的一些最佳实践，在与客户合作的启发下，在 SDLC 的每个阶段内部集成 Claude，以加速开发并使流程运行得更快。

When code is no longer the bottleneck and the build phase runs faster than the traditional SDLC allows for, three things become true:
> 当代码不再是瓶颈并且构建阶段的运行速度比传统 SDLC 允许的速度更快时，以下三件事就会成为现实：

* The bottleneck moves to the steps to the left and right of the build phase. This is mainly plan, review/test, and deploy, which still run at human speed.
  瓶颈移动到构建阶段左侧和右侧的步骤。这主要是计划、评审/测试和部署，仍然以人类的速度运行。

* The controls stop matching reality and become intractable. Reviewing each line by hand made sense when a person had written it, but it can't keep up once agents write most of the diff.
  控制不再符合现实并变得棘手。当一个人编写每一行时，手动检查每一行是有意义的，但一旦智能体编写了大部分diff，它就无法跟上。

* Governance costs increase because exceptions still route through meetings and committees that meet weekly or monthly.
  治理成本增加，因为例外情况仍然通过每周或每月举行的会议和委员会进行。

![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a8739a1b934ffe55bfc9715_44592f18.png)

Build is no longer the constraint — the human-speed steps around it are. Human-speed stages keep their length while build collapses to hours.
> 构建不再是约束——围绕它的人类速度环节才是。人类速度阶段保持原有长度，而构建阶段压缩到几个小时。

Let's use a security bottleneck as an example. Security teams are sized for human output, so when agents multiply code output, either the review queue builds or code ships under-reviewed. A regulated organization can't accept either outcome, so its security and policy checks have to keep pace with the agents.
> 让我们以安全瓶颈为例。安全团队的规模根据人力输出而定，因此，当智能体放大代码输出时，要么评审队列构建，要么代码在评审不足的情况下发布。受监管的组织不能接受任何一个结果，因此其安全和政策检查必须与智能体保持同步。

To better realize the productivity gains of and secure agentic AI, the traditional SDLC lifecycle requires the same level of transformation as the implementation phase has undergone.
> 为了更好地提高智能体 AI 的生产力并确保其安全，传统的 SDLC 生命周期需要进行与实现阶段相同级别的转型。

Table of contents
> 目录

1. [Code is no longer the bottleneck](#sd-c1)
  代码不再是瓶颈

2. [Plays](#sd-c2)
  Plays：可执行动作

3. [Stage 1 — Plan](#sd-s1)
  第一阶段——计划

4. [Stage 2 — Design](#sd-s2)
  第二阶段——设计

5. [Stage 3 — Build](#sd-s3)
  第三阶段——构建

6. [Stage 4 — Test](#sd-s4)
  第 4 阶段 — 测试

7. [Stage 5 — Deploy](#sd-s5)
  第五阶段 — 部署

8. [Stage 6 — Maintain](#sd-s6)
  第六阶段——维护

9. [Closing thoughts](#sd-c9)
  结语

## 2、What is an AI-native SDLC?（什么是 AI 原生 SDLC？）

The AI-native SDLC is a reimagined process that combines the old control objectives with new enforcement. Instead of a linear flow, the process becomes a loop, and AI is embedded at each point. The AI-native SDLC promotes automated handover and triggering of subsequent plays, helping to address the manual and clunky nature of handoff between the phases of the traditional SDLC.
> AI 原生 SDLC 是一个重新构想的流程，将旧的控制目标与新的执行相结合。流程不再是线性流转，而是循环，并且AI 被嵌入到每一个节点。 AI 原生 SDLC 促进了后续play的自动切换和触发，有助于解决传统 SDLC 阶段之间手动、笨重的交接的问题。

You'll also hear this shift called the agentic SDLC, the AI SDLC, or simply agentic software development — the labels differ, but they describe the same thing.
> 你还会听到这种转变，称为智能体式 SDLC、AI SDLC，或简称为智能体式软件开发 — 标签不同，但它们描述的是同一件事。

![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a8858c2eccce183e7553cf2_53b010df.png)

### 2.1、The shifts across the six stages of an AI-native SDLC（AI 原生 SDLC 六个阶段的变化）

The table below highlights the ends of the spectrum between traditional SDLC and AI-native SDLC, supported by Claude. Most organizations sit somewhere between the two columns.
> 下表重点介绍了传统 SDLC 和 Claude 支持的 AI 原生 SDLC 之间的两端。大多数组织都处在两列之间。

| Stage | Traditional SDLC | AI-native SDLC |
| --- | --- | --- |
| Plan | Requirements gathered by committee, distilled through workshops and sign-offs, written up by hand | Claude synthesizes pain points straight from the sources and captures them within `intent.md` which is human readable and machine actionable |
| Design | Spec written by analysts, parsed by designers | Requirements and design compressed into one working session with an agent, guided by standards encoded as skills, versioned in git |
| Build | Tests and code are handwritten and documentation is written after the main development happens | Tests and code are generated by AI and institutional knowledge is maintained as versioned machine-readable `CLAUDE.md` files and skills |
| Test | QA gates at stage boundaries | Continuous evals woven through implementation |
| Deploy | Humans review every line of code and governance occurs in review cycles, often inconsistently | Layers of agentic review with human review reserved for regulated and critical code. Governance is enforced as the AI acts, with hooks as approval gates |
| Maintain | Humans watch production for bugs | Agents monitor live deployments. Any breached control band is diagnosed and written back into the loop as a new `intent.md` |

| 阶段 | 传统SDLC | AI 原生SDLC |
| --- | --- | --- |
| 规划 | 委员会收集的要求，通过研讨会和签字提炼，手写 | Claude 直接从源头合成痛点，并在 `intent.md` 中捕获它们，这是人类可读且机器可操作的 |
| 设计 | 规格由分析师编写，由设计师解析 | 需求和设计被压缩为与智能体的一次工作会话，以编码为skill的标准为指导，并在 git 中进行版本化 |
| 构建 | 测试和代码是手写的，文档是在主要开发发生后编写的 | 测试和代码由 AI 生成，机构知识作为版本化的机器可读 `CLAUDE.md` 文件和skill进行维护 |
| 测试 | 阶段边界的 QA 门 | 通过实施不断进行eval |
| 部署 | 人类评审每一行代码，治理发生在评审周期中，通常不一致 | 为受监管和关键的代码保留多层智能体评审和人工评审。当AI 行动时，治理就会被强制执行，并以hook作为批准门 |
| 维护 | 人类观察生产中的bug | 智能体监控实时部署。任何违反的控制区间都会被诊断出来并作为新的 `intent.md` 写回到循环中 |

The thread running through the right-hand column is the committed artifact. Each stage ends by writing one to version control (including `intent.md`, `spec.md`, `plan.md`, the diff and its tests, the PR with its review findings, and the incident record) and the next stage begins by reading it. For the early stages, .md files are the predominant artifact because a product owner and an agent can both read and act on the same file. From Build onward, the artifact is code and its records. The chain of commits is also the audit trail: who asked for what, what the agent produced, and who approved it.
> 穿过右侧列的主线是已提交的工件。每个阶段都以向版本控制写入一个工件（包括 `intent.md` 、 `spec.md` 、 `plan.md` 、 diff 及其测试、 PR 及其评审结果以及事件记录）结束，下一阶段从读取它开始。在早期阶段，.md 文件是主要工件，因为产品负责人和智能体都可以读取并操作同一个文件。从构建阶段开始，工件变成代码及其记录。提交链也是审计轨迹：谁请求了什么、智能体生成了什么以及谁批准了它。

Humans remain accountable for every decision that requires judgment. In the agentic SDLC world, the human attention shifts along with the artifacts that must be reviewed.
> 人类仍然对每一个需要判断的决定负责。在智能体式 SDLC 世界中，人类注意力随着必须评审的工件而变化。

Every stage commits an artifact the next stage can read. Together, the intent, the spec, the plan, the diff and the review findings are the audit trail.
> 每个阶段都会提交一个下一个阶段可以读取的工件。意图、规范、计划、diff和评审结果共同构成了审计跟踪。

## 3、Plays（Plays：可执行动作）

The plays are the core of the playbook and are grouped into six non-linear stages (Plan, Design, Build, Test, Deploy, Maintain), which together cover the complete lifecycle.
> 这些 play 是 playbook 的核心，分为六个非线性阶段（规划、设计、构建、测试、部署、维护），共同覆盖完整生命周期。

Each play covers:
> 每个 play 包含：

* What changes;
  有什么变化；

* Getting started;
  如何开始;

* Concrete steps for implementation;
  具体实施步骤；

* Governance considerations; and
  治理考虑；和

* How you measure whether it worked.
  你如何衡量它是否有效。

The steps are modular and organizations may choose to prioritize transforming different stages at different times based on their unique needs. Each play names its dependencies under "Prerequisites," which the dependency graph further illustrates.
> 这些步骤是模块化的，组织可以根据自己的独特需求，在不同时间优先推进不同阶段的转型。每个 play 都会在“Prerequisites”下列出依赖关系，依赖图会进一步说明这一点。

A stage ends by committing an artifact with the commit initiating the next stage. An accepted `intent.md` triggers the requirements and design pass, an approved `spec.md` triggers plan mode, a merged PR triggers the pipeline, and a breached control band in production writes the next `intent.md` and so the loop continues.
> 一个阶段以提交一个工件而结束，该提交启动下一个阶段。接受的 `intent.md` 触发需求和设计通过，批准的 `spec.md` 触发计划模式，合并的 PR 触发流水线，生产中违反的控制区间写入下一个 `intent.md` ，因此循环继续。

First, you prompt each step by hand with the end state being a loop in which each accepted artifact fires the next gate. Human attention concentrates at the gates, reviewing what the agent flagged rather than starting each stage from scratch.
> 起初，你手动触发每个步骤；最终目标是形成一个循环：每个被接受的工件都会触发下一个 gate。人的注意力集中在 gate 上，检查智能体标记的内容，而不是从头开始每个阶段。

![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a8855c75344623fc81efcb8_5d5a3c05.png)

The plays are listed with stage; the arrows give the order to adopt them in. The two are not the same. Start with any clay play — nothing points into it, so it needs nothing first. For any other play, the arrows pointing into it are the plays to adopt before it.
> 剧目均附有舞台；箭头给出了采用它们的顺序。两者不一样。从任何粘土游戏开始——无要点，所以它首先不需要任何东西。对于任何其他戏剧，指向它的箭头是在它之前采用的戏剧。

**01**
> **01**

## 4、Plan（规划）

Ideas stop waiting for someone to write them up. Intent is captured once, in the originator's own words, as a version-controlled artifact the next stage can act on.
> 想法不再等待有人把它们写下来。用发起者自己的话说，意图被捕获一次，作为下一阶段可以采取行动的版本控制工件。

### 4.1、Capture as intent.md（捕获为 intent.md）

The `intent.md`, which kicks off the software development process can enter through different routes. A person has an idea, a ticket is filed, or an incident is surfaced via an alert (see Stage 6: Maintenance).
> 启动软件开发过程的 `intent.md` 可以通过不同途径进入：有人有了想法、提交了一张工单，或者通过告警发现了一个事件（见第 6 阶段：维护）。

When a person has an idea, they brainstorm with Claude and produce a markdown proto-spec. In the traditional SDLC, the same person must then convince a member of the product team to write the idea up with them or on their behalf.
> 当一个人有了想法，他们会与 Claude 头脑风暴并生成一个 Markdown 原型规范。在传统 SDLC 中，同一个人必须说服产品团队成员与其一起，或代其，把这个想法写出来。

The proto-spec generated by Claude is human readable, version-controlled, and immediately consumable by the next stage. The proto-spec is saved as an `intent.md`.
> Claude 生成的原型规范是人类可读、受版本控制、可立即供下一阶段使用的。该原型规范保存为 `intent.md`。

Regardless of whether the intent originates from an event trigger or an agent, the same steps apply: the product owner reviews and corrects the agent-written `intent.md` before it is committed.
> 无论意图来自事件触发还是智能体，流程都一样：产品负责人会在提交前评审并修正智能体编写的 `intent.md`。

Traditional：An idea passes through backlog entries, user stories, story points, and refinement meetings before anyone can act on it. Ownership transfers at each handoff, so what reaches engineering is several steps removed from what the originator meant.
> 传统方式：一个想法要经过 backlog 条目、用户故事、故事点和 refinement 会议，才有人能采取行动。每一次交接都会转移所有权，因此到达工程团队手中的内容，已经与最初发起人的真实意图隔了好几层。

AI-native： originator brainstorms with Claude and writes the result down as `intent.md`, a proto-spec in the originator's own terms. The artifact contains what is wanted, why, and under which constraints. Repeat processes are encoded via skills.
> AI 原生方式：发起者与 Claude 进行头脑风暴，并将结果写成 `intent.md`，这是发起者自己措辞下的原型规范。该工件包含需要什么、为什么需要，以及在什么约束下需要。重复流程由 skill 编码。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

None.
> 无。

**Infrastructure**
> **基础设施**

Claude access for people who are not engineers (claude.ai or [Cowork](https://claude.com/product/cowork)); an agreed `intent.md` template; a shared, version-controlled home for intent that the product owner watches. For a single product the simplest home is an `intent/` folder in the product repo. This setup keeps the artifact chain next to the code derived from it. A dedicated intent repo is only worth the overhead when intent spans many repositories, and in a monorepo it is a directory. The Stage 3: Build sidebar covers how this home relates to a Jira or requirements tool that already holds the record.
> 非工程师人员可通过 Claude 访问（claude.ai 或 Cowork）；配好一个 `intent.md` 模板；再准备一个供产品负责人查看的、受版本控制的共享主页。对于单个产品，最简单的主页就是产品仓库里的 `intent/` 文件夹。这样可以把工件链放在它派生出的代码旁边。只有当 intent 横跨多个仓库时，专用 intent 仓库才值得这份开销；在单体仓库里，它就是一个目录。第 3 阶段：Build 侧栏解释了这个主页如何与已经留痕的 Jira 或需求工具配合。

Setting this up is a one-time task for the platform or engineering team. A technical team member needs to stand up the intent home and decide who can write to it, since many contributors will come from across the organization.
> 对于平台或工程团队来说，进行此设置是一项一次性任务。技术团队成员需要建立意图主页并决定谁可以向其写入内容，因为许多贡献者将来自整个组织。

Once the repository exists, contributors without git experience don't need to use git directly. Instead a connector to the version-control system (e.g. GitHub) lets Claude commit markdown files on their behalf from claude.ai or Cowork.
> 一旦仓库就位，没有 git 经验的贡献者也不需要直接操作 git。相反，版本控制系统（例如 GitHub）的连接器可以让 Claude 代表他们从 claude.ai 或 Cowork 提交 Markdown 文件。

#### 4.1.1、How to execute it（如何执行）

1. The originator describes the problem to Claude in their own words. The originator may describe what they cannot do today, who is affected by the idea, what better looks like, or what is out of scope. No formal language is required.
  发起者用自己的话向Claude描述了这个问题。发起者可能会描述他们今天不能做什么、谁会受到这个想法的影响、什么看起来更好，或者什么超出了范围。不需要正式语言。

2. Brainstorm until the idea is concrete. Claude asks the questions an analyst would ask: scope, users, constraints, and what success looks like.
  集思广益，直到想法具体化。Claude提出了分析师会问的问题：范围、用户、限制以及成功是什么样子。

3. Ask Claude to write the result as `intent.md` using the organization's template, which can be encoded as a skill set up by a technical team member and signed off by a lead. This can cover the problem, proposed outcome, affected users and systems, constraints, and open questions.
  要求 Claude 使用组织的模板将结果编写为 `intent.md`，该模板可以编码为由技术团队成员设置并由主管签署的skill。这可以涵盖问题、建议的结果、受影响的用户和系统、约束和悬而未决的问题。

4. The originator corrects anything Claude misunderstood.
  创始人纠正了Claude的任何误解。

5. Commit `intent.md` to the shared home. Author and timestamp join the record, and the product owner picks the idea up from there.
  将 `intent.md` 提交到共享home 目录。作者和时间戳加入记录，产品负责人从那里选取想法。

```
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

#### 4.1.2、Governance considerations（治理考虑）

The evidence is the committed `intent.md`, which lists the author, the timestamp and the full revision history. It's logged in the git history of the intent home. The product owner approves, and the accept or reject decision that sends the intent into Stage 2: Design is recorded as the merge or the closing review.
> 证据是提交的 `intent.md` ，其中列出了作者、时间戳和完整的修订历史记录。它已记录在意图主页的 git 历史记录中。产品负责人批准，并将意图发送到第 2 阶段：设计的接受或拒绝决策被记录为合并或结束评审。

How to measure it
> 如何测量

**Leading indicator**
> **领先指标**

Time from first conversation to a committed `intent.md`, read from git history on the intent home, which records author and time stamp. The expectation is to fall from a multi-week elicitation and refinement cycle to hours.
> 从第一次对话到提交 `intent.md` 的时间，从意图主页上的 git 历史记录中读取，其中记录了作者和时间戳。人们期望将为期数周的启发和细化周期缩短为几个小时。

**Lagging indicator**
> **滞后指标**

The survival rate, or the share of `intent.md` files that the product owner accepts into Stage 2: Design rather than closes. The accept or reject decision is recorded as the merge of the artifact or the closed review. Additionally, the number of changes made to the `intent.md` that are made after the first `spec.md` commit for the same change.
> 生存率，或产品负责人接受进入第 2 阶段：设计而不是关闭的 `intent.md` 文件的份额。接受或拒绝决定被记录为工件的合并或封闭式评审。此外，在第一次 `spec.md` 提交同一更改后对 `intent.md` 进行的更改数。

**02**
> **02**

## 5、Design（设计）

Requirements and design collapse into one session. Policy is applied while the spec is written, not discovered in a review weeks later.
> 需求和设计合并为一个会话。政策是在规范编写时应用的，而不是在几周后的评审中发现的。

### 5.1、Requirements and design（需求与设计）

Once approved by the product owner, Claude takes the accepted `intent.md` and produces a requirements and design spec. This is guided by the organization's [skills](https://code.claude.com/docs/en/skills) for brand, security, compliance, and UX.
> 一旦获得产品负责人的批准，Claude 就会采用已接受的 `intent.md` 并生成需求和设计规范。这是由组织的品牌、安全、合规性和用户体验skill指导的。

The product owner reviews that spec, but doesn't write it. The goal of this process is to create a spec the engineering team can plan against, with flagged areas of concern.
> 产品负责人评审该规范，但不编写它。此过程的目标是创建工程团队可以根据其进行计划的规范，并标记出关注的领域。

Front-end work is the clearest example. Once the `intent.md` is accepted, the product owner mocks the design up in [Claude Design](https://claude.com/product/design) (beta) from the `intent.md`, iterates on the mock, and then exports it to Claude Code to build.
> 前端工作就是最明显的例子。一旦 `intent.md` 被接受，产品负责人就会在 Claude Design（测试版）中从 `intent.md` 模拟设计，迭代模拟，然后将其导出到 Claude Code 进行构建。

Traditional：Requirements and design are separate phases run by separate teams. Analysts formalize the idea into requirements and designers then parse those back into a design. The separation exists for accountability, but it is slow and lossy.
> 传统方式：需求和设计由不同团队分阶段完成。分析师把想法形式化为需求，设计师再把这些需求解析回设计。分离有助于问责，但速度慢且信息有损。

AI-native：两个 phases happen in a single prompted session. Claude takes `intent.md` and produces a requirements and design spec, constrained by the organization's skills, with areas of concern flagged.
> AI-native两个阶段都发生在单个提示会话中。 Claude 采用 `intent.md` 并根据组织的skill制定了需求和设计规范，并标记了关注的领域。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

Write an `intent.md` file, with brand, security, compliance, and UX policies written as skills.
> 编写一个 `intent.md` 文件，将品牌、安全、合规性和 UX 政策写入skill。

**Infrastructure**
> **基础设施**

A product owner with Claude access. No engineering skill is required.
> 具有 Claude 访问权限的产品负责人。不需要工程skill。

#### 5.1.1、How to execute it（如何执行）

1. The product owner opens a session with the organization's skills available and attaches the `intent.md`.
  产品负责人使用组织可用的skill打开一个会话，并附加 `intent.md` 。

2. The product owners prompt points at the `intent.md`, names the constraints, and demands flagged concerns. Run it by hand at first, then codify it as an organization-level slash command. From there make the acceptance of `intent.md` in the intent home the trigger, with a non-interactive job that fires on the merge, run the pass with the organization's skills loaded, and commit `spec.md` as a pull request (the CI/CD play in Stage 5: Deploy covers the plumbing). From that point the product owner's first involvement is the review.
  产品负责人提示指向 `intent.md` ，命名约束，并要求标记的问题。首先手动运行它，然后将其编码为组织级斜线命令。从那里开始，在 Intent Home 中接受 `intent.md` 作为触发器，并在合并时触发非交互式作业，在加载组织skill的情况下运行传递，并将 `spec.md` 作为拉取请求提交（第 5 阶段：部署中的 CI/CD 操作涵盖了流水线）。从那时起，产品负责人的第一个参与就是评审。

3. The same product owner reviews the spec against the idea. Does the spec solve the stated problem, and are the open questions from `intent.md` answered or carried forward?
  同一个产品负责人根据这个想法评审规范。该规范是否解决了所述问题，并且 `intent.md` 中的开放问题是否得到回答或继续？

4. Work through the flagged concerns first as they are the points an analyst would have escalated. The product owner resolves each one with its policy owner before engineering sees the spec.
  首先解决标记的问题，因为它们是分析师会升级的要点。在工程人员看到规范之前，产品负责人会与其政策负责人解决每一个问题。

5. Commit `spec.md` alongside `intent.md`. The file pair records what was asked for and what was decided.
  与 `intent.md` 一起提交 `spec.md` 。文件对记录了要求的内容和决定的内容。

6. The product owner decides whether the spec and intent progress to build, consulting a technical lead for anything the organization classes as higher risk. A human team mate always makes this call, and accepting the spec is what starts the plan mode play in Stage 3: Build.
  产品负责人决定是否继续构建规范和意图，并就组织归类为较高风险的任何内容咨询技术主管。人类队友总是会做出这个决定，接受规范就是在“阶段 3：构建”中开始计划模式的开始。

#### 5.1.2、What it looks like (the prompt)（示例 prompt）

```
Read the attached intent.md and produce a requirements and design spec for integrating it into our existing codebase. Apply the skills available to you so the plan conforms to our brand guidelines, security policies and UX standards. Document the spec fully as spec.md, ready to hand to the engineering team. Describe clearly any areas of concern, especially where you cannot satisfy contradicting policies.
```

#### 5.1.3、Governance considerations（治理考虑）

Instead of being discovered in a review weeks later, the live policy is read and applied while the spec is written. The organization's skills are applied as constraints on the spec. The spec, the prompt that produced it, and the skill versions in force are all logged in version control. The product owner signs off the spec, and routes flagged concerns to the named policy owners.
> 实时政策不是在几周后的评审中被发现，而是在编写规范时读取并应用。组织的skill被用作规范的约束。规范、生成规范的提示以及有效的skill版本都记录在版本控制中。产品负责人签署规范，并将标记的问题发送给指定的政策 owner。

How to measure it
> 如何测量

**Leading indicator**
> **领先指标**

Elapsed time between the `intent.md` commit and the `spec.md` commit for the same change (two git timestamps), compared with the old requirements-plus-design cycle.
> 与旧的需求加设计周期相比，同一更改的 `intent.md` 提交和 `spec.md` 提交之间经过的时间（两个 git 时间戳）。

**Lagging indicator**
> **滞后指标**

Requirements rework after build starts. Count `spec.md` commits dated after the first `plan.md` commit for the same change. Git log will give this directly.
> 构建开始后需求返工。计算同一更改的第一个 `plan.md` 提交之后的 `spec.md` 提交。 Git log 会直接给出这个。

**03**
> **03**

## 6、Build（构建）

Nothing is implemented without an accepted plan. Institutional knowledge becomes files the agent reads, and the guardrails run as code rather than as habits.
> 没有已接受的计划，不进行任何实现。组织知识变成智能体可读取的文件，护栏以代码形式运行，而不是依赖习惯。

### 6.1、Claude Code plan mode as the default starting point（将 Claude Code plan mode 作为默认起点）

Engineers start Claude Code sessions in [plan mode](https://code.claude.com/docs/en/permission-modes), give Claude the approved `spec.md` from Stage 2: Design, and let it interview them, iterating on the plan until the engineer is happy with it.
> 工程师以 plan mode 启动 Claude Code 会话，把第 2 阶段 Design 中批准的 `spec.md` 交给 Claude，并让 Claude 访谈自己、反复迭代计划，直到工程师满意。

Traditional：An engineer reads the design and starts writing code. How the change will be made, down to which files and which tests, stays in the engineer's head or at best a ticket comment. Nobody else can review it. The first thing a reviewer sees is the finished diff, and by then rework is slow.
> 传统方式：工程师阅读设计后开始写代码。变更将如何实现、具体改哪些文件、如何测试，通常停留在工程师脑中，最多写在工单评论里。没人能评审它。评审者第一次看到的是完成后的 diff，而此时返工成本已经很高。

AI-native：工作 starts with a written plan that Claude produces in plan mode, where it can read the codebase without changing anything. The engineer corrects the plan before code is written, and the approved version is committed as `plan.md` for later stages to check against.
> AI 原生方式：工作从 Claude 在 plan mode 中生成的书面计划开始；此时 Claude 可以读取代码库，但不能修改任何内容。工程师在写代码前修正计划，并将批准版本提交为 `plan.md`，供后续阶段检查。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

The intent artifact (`intent.md` or `spec.md`) if one exists, and the `CLAUDE.md` file helps.
> intent 工件（如 `intent.md` 或 `spec.md`，如果存在）以及 `CLAUDE.md` 文件会有帮助。

**Infrastructure**
> **基础设施**

Claude Code with access to the repository.
> 能访问仓库的 Claude Code。

#### 6.1.1、How to execute it（如何执行）

1. The engineer starts the session in plan mode with Claude.
  工程师以 plan mode 启动 Claude 会话。

2. The engineer gives Claude the `intent.md` and the `spec.md` and asks for an implementation plan that names the files that change, the order of the work, and the tests that prove it.
  工程师提供 `intent.md` 和 `spec.md`，要求 Claude 输出实现计划，列明要修改的文件、工作顺序和证明其正确性的测试。

3. Interrogate the plan by asking what the change could break, which step is most risky, and what other options Claude chose not to do.
  追问计划：这个变更可能破坏什么？哪一步风险最大？Claude 没有选择的其他方案是什么？

4. Iterate until an engineer who has never seen the conversation could implement the change from the plan alone.
  反复迭代，直到一个从未看过这段对话的工程师也能只根据该计划完成实现。

5. Commit the approved plan as `plan.md`. The plan joins the audit trail, and the PR review play (Stage 5: Deploy) checks the eventual diff against it.
  将批准后的计划提交为 `plan.md`。计划加入审计轨迹，PR review play（第 5 阶段 Deploy）会检查最终 diff 是否符合它。

6. Accept the plan and let Claude implement. With a solid plan, the implementation is often a single pass.
  接受计划，让 Claude 实现。有了扎实计划，实现通常可以一次完成。

7. When implementation departs from the plan, update `plan.md` in the same commit. Consider using a hook to enforce synchronization between the two.
  当实现偏离计划时，在同一个提交中更新 `plan.md`。可以考虑使用 hook 强制二者同步。

#### 6.1.2、What it looks like (plan.md)（示例 plan.md）

```
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

#### 6.1.3、Governance considerations（治理考虑）

Design review happens before any code is generated, when changing course is still a matter of editing a document. Plan mode enforces this itself, since Claude cannot edit files until the engineer accepts the plan. The plan and its revisions are logged along with who accepted it. Routine changes are approved by the engineer, and anything the organization classes as higher risk goes to a tech lead or architect.
> 设计评审发生在生成任何代码之前，而更改过程仍然是编辑文档的问题。计划模式本身强制执行此操作，因为在工程师接受计划之前，Claude无法编辑文件。该计划及其修订以及接受者都被记录下来。例行变更由工程师批准，组织归类为较高风险的任何内容都会交给技术主管或架构师。

How to measure it
> 如何测量

**Leading indicator**
> **领先指标**

Share of changes that merge from the first implementation pass, and time from plan approval to merged PR with the required data within the PR metadata.
> 从第一次实现阶段合并的变更份额，以及从计划批准到将 PR 与 PR 元数据中所需数据合并的时间。

**Lagging indicator**
> **滞后指标**

Rework cycles per change, again from the PR metadata, and how often the merged diff still matches the committed `plan.md`.
> 每次更改的返工周期，再次来自 PR 元数据，以及合并的diff仍然与提交的 `plan.md` 匹配的频率。

### 6.2、Claude Code on auto mode（Claude Code auto mode）

Claude Code can also run in auto mode, where the engineer approves the plan and, once happy and iterated upon, Claude applies each change without a per-edit prompt. As the guardrails from the later plays mature (a tuned `CLAUDE.md`, skills that encode policy, hooks that block unsafe actions, and a test suite Claude can run), auto-accept becomes the default for routine work: a tight `spec.md`, a small blast radius, and code the tests already cover.
> Claude Code 还可以在自动模式下运行，工程师批准该计划，一旦满意并进行迭代，Claude 就会应用每个更改，而无需每次编辑提示。随着后来的护栏逐渐成熟（经过对齐的 `CLAUDE.md` 、编码政策的skill、阻止不安全操作的hook以及 Claude 可以运行的测试套件），自动接受成为日常工作的默认设置：紧密的 `spec.md` 、小的爆炸半径以及测试已经涵盖的代码。

The shift is now away from the user watching the agent make the edits and reviewing actions, towards the review of artifacts after longer autonomous sessions. Auto-accept mode further enables parallelism across individuals and the team when used with worktrees and is fundamental to running the SDLC autonomously and closing the loop as described in Stage 6: Maintenance.
> 现在的转变是从用户观看智能体进行编辑和评审操作，转向在较长的自主会话后评审工件。当与worktree一起使用时，自动接受模式进一步实现了个人和团队之间的并行性，并且是自主运行 SDLC 和关闭循环的基础，如第 6 阶段：维护中所述。

Sidebar
> 侧边栏

### 6.3、Legacy systems and the source of truth（遗留系统与事实来源）

*Applies to every artifact the process produces.*
> *适用于流程产生的每个工件。*

Existing SDLC processes likely already track artifacts, just not in markdown files. Work items may be in Jira, requirements in a tool with regulatory traceability built in, designs in Figma, and change approvals with a change board. Those systems are hard to displace because auditors and regulators already accept them and other teams depend on them, so the AI-native SDLC has to fit around what exists.
> 现有的 SDLC 流程可能已经在跟踪工件，只是不在 Markdown 文件中。工作项目可能在 Jira 中，需求在内置监管可追溯性的工具中，设计在 Figma 中，并通过变更板进行变更批准。这些系统很难被取代，因为审计员和监管机构已经接受它们，并且其他团队也依赖它们，因此AI 原生 SDLC 必须适应现有的系统。

When transitioning to the AI-native SDLC, for every artifact the process produces, name one system as the source of truth, with everything else holding a copy or a link to the original. The configurations below can be set up to have one source of truth, with the choice differing per artifact:
> 当过渡到 AI 原生 SDLC 时，对于流程产生的每个工件，将一个系统命名为事实来源，其他所有系统都保留原始副本或链接。下面的配置可以设置为只有一个事实来源，每个工件的选择都不同：

**The repo as the source of truth.** The markdown artifacts are the authoritative record and the legacy system references files within commits. This can be one of the cleanest configurations for engineering-led organizations, as all records live in one tool with one timestamp authority.
> **仓库作为事实来源。** Markdown 工件是提交中的权威记录和遗留系统引用文件。对于工程主导的组织来说，这可能是最干净的配置之一，因为所有记录都位于具有一个时间戳权限的一个工具中。

**The legacy system as the source of truth.** Jira, ServiceNow, or the requirements tool holds the authoritative record and the markdown artifacts are working copies. Claude reads the record at the start of the session and writes the outcome back through an [MCP](https://code.claude.com/docs/en/mcp) connector in the same session that produced the spec or the plan.
> **遗留系统是事实的来源。** Jira、ServiceNow 或需求工具拥有权威记录，Markdown 工件是工作副本。 Claude 在会话开始时读取记录，并通过生成规范或计划的同一会话中的 MCP 连接器将结果写回。

**Linkage as the minimum bar.** All artifacts note the record ID and all legacy records contain the commit SHA of the markdown file. Linkage is a good place to start when transitioning to the AI-native SDLC, accepting that there are two sources of truth.
> **链接作为最低要求。** 所有工件都会注明记录 ID，并且所有遗留记录都包含 Markdown 文件的 commit SHA。在过渡到 AI 原生 SDLC 时，链接是一个很好的起点，接受有两个事实来源。

Both the legacy system and the markdown-first system can coexist, so long as there is a link between the two or one is declared the source of truth.
> 遗留系统和Markdown-first 系统都可以共存，只要两者之间存在链接或者其中一个被声明为事实来源。

### 6.4、The CLAUDE.md（CLAUDE.md 文件）

[`CLAUDE.md`](https://code.claude.com/docs/en/memory) gives Claude the context a new joiner would need, covering conventions, commands, architecture, and the mistakes the team sees most often. Knowledge that used to sit in people's heads and on wikis becomes a file the agent reads at the start of every session, maintained by the whole team and iterated on whenever a mistake is made.
> `CLAUDE.md` 为 Claude 提供了新成员需要知道的上下文，涵盖约定、命令、架构以及团队最常遇到的错误。过去存在于人脑和 wiki 中的知识，现在变成智能体每次会话开始时读取的文件，由整个团队维护，并在每次出错时迭代。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

None.
> 无。

**Infrastructure**
> **基础设施**

A repo, Claude Code installed, and one engineer who knows the codebase well.
> 一个仓库，安装了 Claude Code，还有一位熟悉代码库的工程师。

#### 6.4.1、How to execute it（如何执行）

1. Run `/init` in the repo. Claude generates a starting `CLAUDE.md` from what it finds.
  在仓库中运行 `/init`。 Claude 根据它发现的内容生成一个起始 `CLAUDE.md` 。

2. Cut the generated file down to what a new joiner would need on day one. Keep the build, test and lint commands, the conventions that matter, and the things Claude keeps getting wrong.
  将生成的文件缩减为新成员第一天所需的内容。保留 build、测试和 lint 命令、重要的约定以及 Claude 经常出错的地方。

3. Check `CLAUDE.md` into git at the repo root so the whole team shares one version and changes are reviewed like code.
  将 `CLAUDE.md` 提交到仓库根目录的 git 中，以便整个团队共享一个版本，并像代码一样评审更改。

4. A working rule helps here. When Claude makes a mistake twice, the correction goes into `CLAUDE.md`.
  一个有用的规则是：当 Claude 同一个错误犯了两次，就把修正写入 `CLAUDE.md`。

5. Keep it under a page, because Claude reads all of it at the start of a session and anything stale is taking up context for no benefit.
  保持在一页以内，因为 Claude 会在会话开始时读取全部内容，任何过时信息都会占用上下文且没有收益。

#### 6.4.2、What it looks like (CLAUDE.md)（CLAUDE.md 示例）

```
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

#### 6.4.3、Governance considerations（治理考虑）

`CLAUDE.md` is version controlled, so the instructions the agent works to are reviewable and auditable. Team conventions are applied through the file, changes to it are logged in git history, and code owners approve those changes in PR review.
> `CLAUDE.md` 受版本控制，因此智能体遵循的指令可被评审和审计。团队约定通过该文件应用，文件变更记录在 git 历史中，并由 code owners 在 PR review 中批准。

How to measure it
> 如何测量

**Leading indicator**
> **领先指标**

How often Claude repeats a mistake `CLAUDE.md` should have caught. The corrections or changes to the `CLAUDE.md` should be tracked within the git history.
> Claude 重复犯本应由 `CLAUDE.md` 捕获的错误的频率。对 `CLAUDE.md` 的修正或变更应在 git 历史中跟踪。

**Lagging indicator**
> **滞后指标**

Time to first merged PR for a new member of the team from PR history.
> 新团队成员首次合并 PR 所需的时间，可从 PR 历史读取。

### 6.5、Skills as institutional knowledge（Skills 作为组织知识）

Skills are how an organization makes its institutional knowledge operational. The instructions are explicit, version-controlled, applied broadly, and updated centrally when policy changes. The rule of thumb: write a skill for institutional knowledge that must be applied consistently; don't write a skill for components that belong in `CLAUDE.md` or a prompt.
> Skills 是组织将机构知识操作化的方式。指令是显式的、受版本控制的、可广泛应用的，并且在政策变化时集中更新。经验法则是：需要一致应用的机构知识写成 skill；属于 `CLAUDE.md` 或 prompt 的组件不要写成 skill。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

None required. Having a `CLAUDE.md` helps, because it keeps the agent's working knowledge in the repo, but a skill does not depend on it.
> 不需要。拥有 `CLAUDE.md` 会有所帮助，因为它将智能体的工作知识保留在仓库中，但 skill 并不依赖于它。

**Infrastructure**
> **基础设施**

One policy with a named owner and a written source of truth.
> 一项有明确 owner 和书面事实来源的政策。

#### 6.5.1、How to execute it（如何执行）

1. Pick one piece of knowledge that is enforced inconsistently today. This could be a security standard, an API design convention, or a brand rule.
  选择一项目前执行不一致的知识。这可以是安全标准、API 设计约定或品牌规则。

2. Write it as a skill, a folder containing a `SKILL.md` whose frontmatter says when it triggers and whose body says what to do. An engineer writes it from the policy owner's source of truth, using Claude to help.
  将其写为一个 skill，一个包含 `SKILL.md` 的文件夹，frontmatter 说明何时触发，其正文说明要做什么。工程师根据政策 owner 的事实来源编写，可以让 Claude 协助。

3. Put the skill in the repo at `.claude/skills/<name>/` so it ships with the code, or distribute it organization-wide through a [plugin](https://code.claude.com/docs/en/plugin-marketplaces).
  将 skill 放在 `.claude/skills/<name>/` 的仓库中，随代码一起发布，或者通过插件在组织范围内分发。

4. Test that the skill triggers. Ask Claude to do the relevant task in different ways and confirm the skill loads each time.
  测试 skill 是否触发。用不同方式要求 Claude 执行相关任务，并确认每次都会加载 skill。

5. When the policy changes, change the skill and have the policy owner sign off the change.
  当政策发生变化时，更新 skill，并由政策 owner 签署变更。

6. Engineers pick up the new version automatically in their next session.
  工程师在下一次会话中自动获得新版本。

#### 6.5.2、What it looks like (.claude/skills/secure-api-review/SKILL.md)（.claude/skills/secure-api-review/SKILL.md 示例）

```
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

#### 6.5.3、Governance considerations（治理考虑）

A skill is a control, though an advisory one. It makes Claude likely to apply the policy while the code is written, and nothing forces a session to comply with it. A policy that must always hold needs something deterministic behind the skill, such as a hook that blocks the action or a review pass that re-checks the policy at the PR. The skill makes violations rare and the hook makes them close to impossible. Skill invocations are logged in session traces, and the policy owner reviews skill changes like code.
> skill 是一种控制，尽管是一种建议性的。它使得 Claude 可能在编写代码时应用该政策，并且没有机制强制会话合规。必须始终保持的政策需要 skill 背后有确定性机制，例如阻止动作的 hook 或在 PR 中重新检查政策的 review pass。skill 让违规变少，hook 让违规几乎不可能发生。skill 调用会记录在session traces中，政策 owner 会像代码一样评审 skill 变更。

How to measure it
> 如何测量

**Leading indicator**
> **领先指标**

Time from the policy owner approving a policy change to the updated skill merging, taken from the PR on the skill folder.
> 从政策 owner批准政策变更到更新后的 skill 合并的时间，取自 skill 文件夹上的 PR。

**Lagging indicator**
> **滞后指标**

PR reviews findings that cite the policy, which should fall towards zero once the skill is applying the policy while the code is written. Where the findings don't fall towards zero, either the skill isn't triggering or its text has drifted from the official policy.
> PR review 中引用该政策的发现，一旦 skill 在编写代码时应用该政策，该发现应降至零。如果结果没有下降到零，则要么该 skill 没有触发，要么它的文本偏离了官方政策。

### 6.6、Hooks as build-time guardrails（Hooks 作为构建期护栏）

A skill is an advisory control while a [hook](https://code.claude.com/docs/en/hooks) is the deterministic layer behind it. Most of Claude's actions are file edits and shell commands during implementation, so the build phase is where hooks can end up firing most often.
> skill 是一种建议性控制，而 hook 是其背后的确定性层。 Claude 的大部分操作都是在实现过程中进行文件编辑和 shell 命令，因此构建阶段是hook最常触发的阶段。

Build-phase hooks can:
> 构建阶段 hook 可以：

* Block edits to protected paths such as generated classes or a frozen package;
  阻止对受保护路径（例如生成的类或冻结的包）进行编辑；

* Run the formatter and linter after file edits so drift never accumulates;
  在文件编辑后运行 formatter 和 linter，避免漂移积累；

* Keep credentials out of the diff.
  将凭证保留在 diff 之外。

Back any skill whose policy has to hold without exception. A hook runs on each action that matches it, so build-phase hooks should be fast and scoped to the file that changed. Heavier checks such as the full test suite belong at the commit or the PR.
> 为任何必须无例外成立的政策，在 skill 背后加上确定性机制。hook 在与其匹配的每个操作上运行，因此构建阶段 hook 应该快速并且范围仅限于更改的文件。更重的检查（例如完整的测试套件）属于提交或 PR。

A hook that asks a human for approval belongs with the gates in Stage 5: Deploy, because an approval prompt during the build puts a person back on the critical path of all the sessions running in parallel.
> 请求人工批准的 hook 属于第 5 阶段 Deploy 的 gate，因为构建期间的审批提示会把人重新放回所有并行会话的关键路径上。

### 6.7、Parallel sessions and subagents（并行会话与 subagents）

One engineer can drive several streams of work at once.
> 一名工程师可以同时推进多条工作流。

A parallel session is another full Claude Code instance, working a separate task in its own [git worktree](https://code.claude.com/docs/en/worktrees). Each independent session knows nothing about the others, and the engineer steering them is the only thing they share.
> 并行会话是另一个完整的 Claude Code 实例，在自己的 git worktree 中执行单独的任务。每个独立的会话对其他会话一无所知，指导它们的工程师是它们唯一共享的东西。

A [subagent](https://code.claude.com/docs/en/sub-agents) runs inside a single session as a scoped helper with its own context window and tool limits and suits jobs that recur in multiple tasks such as verifying the app runs as expected.
> subagent 作为有作用域的助手在单个会话中运行，具有自己的上下文窗口和工具限制，适合在多个任务中重复出现的作业，例如验证应用按预期运行。

Parallel sessions raise the number of tasks an engineer can have in flight, while subagents keep each session focused on its own task. The engineer's job is steering and reviewing all of them.
> 并行会话增加了工程师可以执行的任务数量，而 subagents 则使每个会话都专注于自己的任务。工程师的工作是指导和评审所有这些。

Traditional：One engineer works one task at a time and spends a significant portion of their day or week on builds, tests and reviewers. Switching between tasks while waiting is possible, but the context switch is tiring enough that few people choose to.
> 传统方式：一名工程师一次只做一个任务，每天或每周的大量时间花在 build、test 和等待 reviewer 上。等待期间当然可以切换任务，但上下文切换足够累，所以很少有人愿意这样做。

AI-native：One engineer runs several Claude sessions at once, each in its own worktree on its own task. Repeated jobs become subagents with their own context and tool limits. The engineer's job shifts to orchestrating, and eventually, to building and monitoring loops.
> AI 原生方式：一名工程师同时运行多个 Claude 会话，每个会话都在自己的 worktree 中执行自己的任务。重复任务变成拥有独立上下文和工具限制的 subagents。工程师的工作转向编排，并最终转向构建和监控循环。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

The `CLAUDE.md`, since all sessions read the file. The feedback loop (Stage 4: Test) also helps here, because less supervision from the engineer is needed when a session can verify its own work.
> `CLAUDE.md` ，因为所有会话都读取该文件。反馈循环（第 4 阶段：测试）在这里也有帮助，因为当会话可以验证自己的工作时，需要工程师的监督较少。

**Infrastructure**
> **基础设施**

A git repository, since isolation comes from worktrees and permission settings tuned so sessions are not waiting on approval prompts for commands the organization considers safe.
> git 仓库，因为隔离来自worktree和对齐的权限设置，因此会话不会等待组织认为安全的命令的批准提示。

#### 6.7.1、How to execute it（如何执行）

1. The engineer splits the work into tasks that touch different files, using the plan from the plan mode play (Stage 3: Build) to see where the work is independent. Tasks that share files run in a single session, one after another.
  工程师将工作拆分为涉及不同文件的任务，使用计划模式播放（第 3 阶段：构建）中的计划来查看工作的独立性。共享文件的任务在单个会话中依次运行。

2. Each parallel task gets its own worktree, for example `claude --worktree feature-auth` in one terminal and `claude --worktree fix-rate-limit` in another. A worktree is a separate checkout on its own branch, which stops sessions colliding on files.
  每个并行任务都有自己的worktree，例如一个终端中的 `claude --worktree feature-auth` 和另一个终端中的 `claude --worktree fix-rate-limit` 。worktree是其自己分支上的单独签出，它可以阻止会话在文件上发生冲突。

3. Two or three sessions is a sensible starting point. The practical ceiling is how many streams one person can review properly, so add sessions only while review is keeping up.
  两到三场会议是一个明智的起点。实际的上限是一个人可以正确评审的流数量，因此仅在评审跟上时才添加会话。

4. Turn repeated jobs into subagents, as defined in markdown files in `.claude/agents/`, each with a name, a description of when to use it, and the tools it may touch. Examples include a code simplifier that strips needless complexity after the main agent finishes, a verifier that runs the app and checks behavior, a researcher that explores the codebase and reports back without flooding the main context. Check the definitions into git so the whole team shares them.
  将重复的作业转换为subagent，如 `.claude/agents/` 中的 markdown 文件中所定义，每个subagent都有一个名称、何时使用它的描述以及它可能涉及的工具。示例包括在主智能体完成后消除不必要的复杂性的代码简化器、运行应用并检查行为的验证器、探索代码库并在不淹没主上下文的情况下返回报告的研究人员。将定义签入 git，以便整个团队共享它们。

#### 6.7.2、What it looks like (.claude/agents/verifier.md)（.claude/agents/verifier.md 示例）

```
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

#### 6.7.3、Governance considerations（治理考虑）

More sessions means more output, so the controls have to come from configuration in the repo. Hooks and permission settings there apply to all sessions, and what a session does is logged and attributed to the engineer who ran it.
> 更多会话意味着更多输出，因此控件必须来自仓库中的配置。那里的hook和权限设置适用于所有会话，并且会话所做的事情被记录并归因于运行它的工程师。

How to measure it
> 如何测量

**Leading indicator**
> **领先指标**

Concurrent sessions per engineer while review quality holds, counted from the OpenTelemetry export, and the share of the day spent steering rather than waiting.
> 每个工程师在评审质量时的并发会话数，从 OpenTelemetry 导出中计算，以及花费在引导而不是等待上的时间份额。

**Lagging indicator**
> **滞后指标**

Changes merged per engineer per week read alongside the rework rate as determined per the PR history.
> 每个工程师每周合并的更改与根据 PR 历史确定的返工率一起读取。

**04**
> **04**

## 7、Test（测试）

Every session checks its own work before a human sees it, and the configuration that steers the agent gets regression-tested like the code it writes.
> 每个会话都会在人类看到之前检查自己的工作，并且引导智能体配置会像它编写的代码一样进行回归测试。

### 7.1、Give Claude a feedback loop（给 Claude 一个反馈循环）

Always give Claude a way to verify its own work, whether tests, a build, or a screenshot diff. A session checks its own work and fixes its own mistakes before an engineer sees them.
> 始终给 Claude 一种验证自己工作的方式：测试、构建或截图 diff。会话在人类看到之前先检查自己的工作，并修复自己的错误。

The feedback loop should not be confused with a verifier subagent (Stage 3: Build). The feedback loop runs through the whole task as many times as the work. The verifier subagent, on the other hand, is one way to package the final check by running a fresh context window once the session believes the work is done. This way the verdict is not colored by the assumptions that produced the code.
> 反馈循环不要与第 3 阶段 Build 中的 verifier subagent 混淆。反馈循环贯穿整个任务，可反复运行。Verifier subagent 则是在会话认为工作完成后，用新的上下文窗口包装最终检查，避免判断被生成代码时的假设污染。

Traditional：The signal that code works arrives late. CI minutes later, a tester days later, production weeks later. With an agent producing the code, a late signal means a person has to check all of its output, and that person becomes the bottleneck.
> 传统方式：代码是否工作的信号来得很晚：几分钟后的 CI、几天后的测试人员、几周后的生产环境。智能体产出代码时，迟来的信号意味着人必须检查其所有输出，这个人就变成瓶颈。

AI-native： session is given a way to check its own work before a person sees it. Run the tests, run the build, take the screenshot. Claude iterates until the check passes, so what reaches the engineer has already passed it. Setting the loop up falls to the engineer running the session, and the steps below are written for them.
> AI 原生方式：会话在交给人之前就拥有检查自身工作的能力。运行测试、构建、截图。Claude 迭代直到检查通过，因此到达工程师面前的内容已经通过验证。这个循环由运行会话的工程师来设置，下面步骤也是写给他们的。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

None.
> 无。

**Infrastructure**
> **基础设施**

A test suite and a build that run locally with one command each. For the UI work, a way for Claude to see the result is crucial, either a browser tool or a screenshot utility wired in via MCP.
> 测试套件和构建都能各自用一个命令在本地运行。对于 UI 工作，让 Claude 看到结果至关重要，可以是浏览器工具，也可以是通过 MCP 接入的截图工具。

#### 7.1.1、How to execute it（如何执行）

1. If checking the work today takes a sequence of commands and some environment knowledge, wrap it in a single target such as "make test" or "npm test" that exits non-zero on failure.
  如果今天检查工作需要一串命令和环境知识，把它包装成单一目标，例如 `make test` 或 `npm test`，并在失败时返回非零退出码。

2. In the `CLAUDE.md`'s Commands section, list each command with an example of a healthy output.
  在 `CLAUDE.md` 的 Commands 部分列出每个命令，以及健康输出示例。

3. State a target and make it quantifiable so Claude can check the work without asking you, for example: "All tests in test\_status.py pass," "the screenshot matches the attached mock," or "the endpoint returns 200 with the new field".
  陈述一个可量化目标，让 Claude 无需询问即可检查工作，例如：“`test_status.py` 中所有测试通过”、“截图匹配附加 mock”、“endpoint 返回 200 且包含新字段”。

4. For bug fixes, write the failing test first. Ask Claude to reproduce the bug as a test, run it, and confirm it fails for the reason you expect. Commit that test. Only then ask Claude to make it pass without editing the test, with the test-file hook from the final step enforcing the restriction. A test that existed before the fix, and that the agent couldn't rewrite, is proof the bug is gone.
  对 bug fix，先写失败测试。让 Claude 将 bug 复现为测试，运行并确认它按预期原因失败。提交该测试。然后才要求 Claude 在不编辑测试的情况下让它通过，并用最后一步中的 test-file hook 强制限制。修复前已经存在且智能体不能改写的测试，就是 bug 已消失的证据。

5. For UI work, close the loop with a visual check. Give Claude a browser or screenshot tool, give it the mock, and let it iterate. Implement, screenshot, compare, and adjust. Two or three rounds is normal, and the result should improve with each one.
  对 UI 工作，用视觉检查闭环。给 Claude 浏览器或截图工具、mock，让它实现、截图、比较、调整。两三轮很正常，结果应逐轮改善。

6. Make verification part of "done." Instruction lives in `CLAUDE.md`. Run the tests before reporting a task complete, and show the output.
  把验证作为“完成”的一部分。指令写在 `CLAUDE.md` 中：报告任务完成前运行测试，并展示输出。

7. Finally, the loop itself needs protecting, because an agent fixing code must not be able to weaken the check on that code. A hook that blocks edits to test files during a fix task does this. The alternative is to check the diff in review and reject any change that touches a test.
  最后，循环本身需要保护，因为修代码的智能体不能削弱验证该代码的检查。修复任务中阻止编辑测试文件的 hook 可以做到这一点。替代方案是在 review 中检查 diff，拒绝任何触碰测试的变更。

#### 7.1.2、What it looks like (CLAUDE.md verification block)（CLAUDE.md 验证块示例）

```
## Verifying your work

- Build: make build (must finish with "Build succeeded")
- Test: make test (all green; never skip or delete a failing test)
- Lint: make lint (zero warnings)

Run all three before reporting any task complete, and paste the output.
If a test fails, fix the code, not the test.
```

Governance considerations
> 治理考虑

**What is enforced**
> **强制执行什么**

Verification before a task is reported done, and the block on the agent editing test files during a fix, both implemented as hooks where the organization wants them guaranteed.
> 被强制的是：任务报告完成前的验证，以及修复任务中阻止智能体编辑测试文件的限制。在组织需要保证这些规则时，二者都可以通过 hooks 实现。

**What the evidence is**
> **证据是什么**

The literal output of "make test," the build log, or the screenshot diff that Claude ran and pasted, so the evidence comes from the toolchain.
> 证据是 Claude 实际运行并粘贴的 `make test` 输出、构建日志或截图 diff，因此证据来自真实工具链。

**Where it is logged**
> **记录在哪里**

In the session transcript, which the OpenTelemetry export forwards to the organization's observability stack, and in the PR's check run, where the reviewer and any later auditor can both see it.
> 在 OpenTelemetry 导出转发到组织的可观察性堆栈的会话记录中，以及在 PR 的检查运行中，review者和任何后来的审计员都可以看到它。

**Who approves**
> **谁批准**

The code owner reviewing the PR, who can concentrate on intent and risk because the mechanical evidence is already attached.
> 评审 PR 的code owners可以专注于意图和风险，因为已经附加了机械证据。

How to measure it
> 如何测量

**Leading indicator**
> **领先指标**

First-pass CI success rate for agent-written changes, which the CI system already supports.
> 智能体编写的更改的首次通过 CI 成功率，CI 系统已支持。

**Lagging indicator**
> **滞后指标**

Review time per PR (from the PR metadata), which should fall once the tests catch what reviewers used to catch, and the change failure rate from an incident tracker.
> 每个 PR 的评审时间（来自 PR 元数据），一旦测试捕获了评审者过去捕获的内容，该时间就会下降，以及事件跟踪器的更改失败率。

### 7.2、Continuous evals in CI（CI 中的持续 evals）

Evals are the AI-native equivalent of stage-gate QA. In practice that means a suite that runs whenever the agent's configuration changes. When a new model is swapped in or a prompt is rewritten, the eval suite says whether the agent still does the work to the same standard.
> eval是 AI 原生的阶段 gate QA 的等价物。实际上，这意味着只要智能体配置发生更改，套件就会运行。当换入新模型或重写提示时，eval套件会说明智能体是否仍然按照相同的标准进行工作。

The evals should be seen as a live suite. As models improve, cases that once discriminated stop doing so and new ones must be added that arise from ongoing monitoring.
> eval应该被视为现场套件。随着模型的改进，曾经受到歧视的案例将不再受到歧视，并且必须添加因持续监测而产生的新案例。

Depending on the use case, some teams may prefer to run these evals offline on a set cadence rather than on every change. The steps below are for continuous evaluations.
> 根据用例，一些团队可能更喜欢按照设定的节奏离线运行这些eval，而不是每次更改时运行。以下步骤用于持续eval。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

The `CLAUDE.md` and feedback loop (Stage 4: Test).
> `CLAUDE.md` 和反馈循环（第 4 阶段：测试）。

**Infrastructure**
> **基础设施**

CI that can run Claude Code non-interactively, and an API key with budget for eval runs.
> 可以非交互方式运行 Claude Code 的 CI，以及带有eval运行预算的 API 密钥。

#### 7.2.1、How to execute it（如何执行）

1. The platform engineer collects 20 to 50 real tasks from recent work with its expected/accepted outcome.
  平台工程师从最近的工作中收集 20 到 50 个实际任务及其预期/可接受的结果。

2. Write each task as an eval, meaning the prompt plus the checks that define acceptable (tests pass, lint clean, behavior unchanged, policy followed).
  将每个任务编写为eval，意味着提示加上定义可接受的检查（测试通过、lint 清理、行为不变、遵循政策）。

3. The suite runs non-interactively in CI on a schedule and on any change to `CLAUDE.md`, skills or hooks, since that configuration steers the agent and deserves the regression testing that code gets.
  该套件在 CI 中按计划以及对 `CLAUDE.md` 、skill或hook进行任何更改时以非交互方式运行，因为这些配置会引导智能体，值得像代码一样做回归测试。

4. Gate configuration changes on the results. A skill change that drops the pass rate gets reviewed before it merges.
  门配置改变对结果的影响。降低通过率的skill变更在合并之前会经过评审。

5. Each production incident gets an eval, written by the team that owned the incident, and stays in the suite as a regression test.
  每个生产事件都会有一个eval，由负责该事件的团队编写，并保留在套件中作为回归测试。

#### 7.2.2、What it looks like (.github/workflows/agent-evals.yml)（.github/workflows/agent-evals.yml 示例）

```
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

#### 7.2.3、Governance considerations（治理考虑）

Evals give QA a gate that keeps up with agent output. The pass-rate threshold is enforced as a merge check, runs are logged so results can be compared over time, and the team that owns the configuration change approves it.
> eval为 QA 提供了一个跟上智能体输出的入口。通过率阈值作为合并检查强制执行，记录运行，以便可以随时间比较结果，并且拥有配置更改的团队批准它。

How to measure it
> 如何测量

**Leading indicator**
> **领先指标**

The eval pass rate over time, reported by the suite on every run, and how long a production incident takes to become a permanent eval.
> 随着时间的推移，套件在每次运行时报告的eval通过率，以及生产事件需要多长时间才能成为永久eval。

**Lagging indicator**
> **滞后指标**

Regressions caught in CI compared with regressions found in production derived from the incident tracker.
> CI 中捕获的回归与事件跟踪器生成的生产中发现的回归进行比较。

**05**
> **05**

## 8、Deploy（部署）

Review runs in both directions, and governance is enforced as the agent acts. The agent does everything up to the production gate and nothing past it.
> 评审是双向进行的，治理是在AI 行动时强制执行的。智能体负责生产前的所有工作，但不会越过它。

### 8.1、AI in the PR review loop（PR review 循环中的 AI）

Claude both gives and receives reviews. It reviews incoming PRs against the organization's policies and addresses review comments on its own PRs. This allows engineers to focus on behavior in their PR review, which boils down to judging intent and risk.
> Claude既给出评价又接受评价。它根据组织的政策评审传入的 PR，并处理对其自己的 PR 的评审意见。这使得工程师能够在公关评审中关注行为，归结为判断意图和风险。

Traditional：Review capacity was planned around human output. A PR waits for a reviewer to read all of it, review quality varies with the reviewer's load, and the author chases while the backlog grows.
> 传统方式：评审能力按人类产出规划。PR 等待 reviewer 读完整个 diff，评审质量随 reviewer 负载波动；backlog 增长时，作者还要不断催促。

AI-native：所有 PRs get an identical set of review passes, with findings ranked by severity. Human attention moves up a level, to whether the change does what the plan intended and whether the risk is acceptable.
> AI-native所有 PR 都会获得一组相同的评审通过，结果按严重程度排名。人们的注意力上升到一个层次，关注变革是否达到了计划的预期以及风险是否可以接受。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

An updated `CLAUDE.md` file from Stage 3: Build; skills if the review passes enforce written policies, defined subagents.
> 第 3 阶段：构建的更新的 `CLAUDE.md` 文件；如果评审通过，则执行书面政策和定义的subagent。

**Infrastructure**
> **基础设施**

A repo with the Claude integration installed, either the managed [Code Review](https://code.claude.com/docs/en/code-review) (research preview) service enabled by an admin or the [claude-code-action](https://code.claude.com/docs/en/github-actions) running in your own CI, with model calls through AWS Bedrock, Google Vertex or Microsoft Foundry where needed (the CI/CD play covers the deployment options). Branch protection policies that require a code owner's approval are also worthwhile.
> 安装了 Claude 集成的仓库，可以是由管理员启用的托管代码评审（research preview）服务，也可以是在你自己的 CI 中运行的 claude-code-action，并在需要时通过 AWS Bedrock、Google Vertex 或 Microsoft Foundry 进行模型调用（CI/CD 播放涵盖了部署选项）。需要code owners批准的分支保护政策也是值得的。

#### 8.1.1、How to execute it（如何执行）

1. The managed Code Review service is the fastest start. An admin enables it and selects repositories. Run the review in your own CI with the claude-code-action when you need control of the pipeline or want API calls routed through your own cloud agreement (the CI/CD play covers that plumbing).
  托管代码评审服务是最快的启动方式。管理员启用它并选择仓库。当你需要控制流水线或希望通过你自己的云协议路由 API 调用时，请使用 claude-code-action 在你自己的 CI 中运行评审（CI/CD 操作涵盖了该流水线）。

2. The tech lead writes the review policy as `REVIEW.md` at the repo root, divided into the passes the organization cares about: bugs and logical errors; security and vulnerabilities; compliance against the spec (`spec.md` from the requirements play), the implementation plan (`plan.md` from the plan mode play) and design principles. `REVIEW.md` also defines what counts as Important as opposed to a Nit, and what to skip.
  技术负责人将评审政策写入仓库根目录中的 `REVIEW.md` 中，分为组织关心的阶段：bug和逻辑bug；安全和漏洞；遵守规范（来自需求的 `spec.md`）、实施计划（来自计划模式的 `plan.md`）和设计原则。 `REVIEW.md` 还定义了与 Nit 相对应的重要内容以及要跳过的内容。

3. The tech lead sets the human threshold. Findings do not approve or block a PR on their own, and branch protection still requires approval from a code owner. A platform engineer who wants to gate merges on findings can read the severity counts that the check run publishes as a machine-readable tally.
  技术主管设定了人类的gate。发现本身并不批准或阻止 PR，并且分支保护仍然需要code owners的批准。想要对结果进行合并的平台工程师可以读取检查运行作为机器可读计数发布的严重性计数。

4. When a reviewer or the author tags `@claude` on a review comment, Claude addresses the comment and pushes the fix. The PR thread records both the request and the change. This fix loop runs through the claude-code-action. In the managed service, commenting `@claude review` requests a fresh review instead. For PRs Claude opened, go further and let Claude babysit the PR to merge. Teams wrap the loop in a custom slash command that sweeps the unresolved review comments and failing checks on the PR, addresses them and pushes the fixes, until the PR is green and waiting only on code owner approval.
  当review者或作者在review评论上标记 `@claude` 时，Claude 会处理该评论并推送修复。 PR 线程记录请求和更改。此修复循环贯穿Claude代码操作。在托管服务中，评论 `@claude review` 会请求重新评审。对于 Claude 打开的 PR，更进一步，让 Claude 照顾 PR 进行合并。团队将循环包装在自定义斜线命令中，该命令清除未解决的评审评论和 PR 上的失败检查，解决它们并推送修复，直到 PR 变为绿色并仅等待code owners批准。

5. Review findings feed back into `CLAUDE.md`. When a review flags a mistake for the second time, the correction goes into `CLAUDE.md` as part of that review, and because review reads `CLAUDE.md` the mistake is caught from the next PR onwards. Review also flags when a change has made `CLAUDE.md` outdated.
  评审结果反馈到 `CLAUDE.md` 。当评审第二次标记bug时，更正会作为该评审的一部分进入 `CLAUDE.md` ，并且由于评审读取 `CLAUDE.md` ，因此从下一个 PR 开始会发现bug。当更改使 `CLAUDE.md` 过时时，评审还会进行标记。

6. Once a month the tech lead tunes the setup by rating findings so the reviewer improves and by capping Nit volume in `REVIEW.md`. Generated paths and anything CI already enforces are excluded.
  技术负责人每月一次通过对结果进行评分来对齐设置，以便review者进行改进，并在 `REVIEW.md` 中限制 Nit 量。生成的路径和 CI 已经强制执行的任何内容都被排除。

#### 8.1.2、What it looks like (REVIEW.md)（REVIEW.md 示例）

```
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

#### 8.1.3、Governance considerations（治理考虑）

Separation of duties is preserved, because the agent that wrote the code has no way to approve it. The review policy in `REVIEW.md` is applied to all PRs, and findings, fixes, ratings and approvals are logged in the PR history, so the PR is the audit record. Approval comes from a human through branch protection, informed by the findings.
> 职责分离得以保留，因为编写代码的智能体无法批准它。 `REVIEW.md` 中的评审政策适用于所有 PR，并且发现结果、修复、评分和批准都记录在 PR 历史记录中，因此 PR 是评审记录。根据发现，批准由人类通过分支保护完成。

For how these controls compose at production scale, see [securing an AI-native SDLC at Anthropic](https://claude.com/blog/how-anthropic-secures-its-ai-native-software-development-lifecycle).
> 有关这些控件如何在生产规模上组成的信息，请参阅在 Anthropic 上保护 AI 原生 SDLC。

How to measure it
> 如何测量

**Leading indicator**
> **领先指标**

Time to first review, which should fall to minutes, and the share of review comments resolved without a human touching the branch with data stored directly on Git.
> 首次评审时间应缩短至几分钟，并且无需人工接触直接存储在 Git 上的数据的分支即可解决评审评论的问题。

**Lagging indicator**
> **滞后指标**

Defects and vulnerabilities caught before merge set against those escaping to production, from the PR history and the incident tracker.
> 从 PR 历史记录和事件跟踪器中，在合并之前发现的缺陷和漏洞与逃逸到生产环境的缺陷和漏洞相对应。

### 8.2、Hooks as approval gates（Hooks 作为审批 gate）

The build phase used hooks as guardrails, allowing or blocking actions with no human involved (Stage 3: Build). A hook can also ask, pausing the action until a specific person approves, which is what release gating needs.
> 构建阶段使用hook作为护栏，允许或阻止无人参与的操作（第 3 阶段：构建）。hook还可以询问，暂停操作直到特定人员批准，这正是发布 gate控所需要的。

The play sits in Stage 5: Deploy because the release gate is the clearest case, but hooks are not deploy-specific: they run wherever Claude acts. For example, hooks can block edits to migrations and infra without a change ticket during Stage 3: Build, and stop the agent editing test files during a fix task in Stage 4: Test.
> 这个 play 位于第 5 阶段：部署，因为发布 gate是最清晰的情况，但hook不是特定于部署的：它们在 Claude 行动的任何地方运行。例如，hook 可以在第 3 阶段：构建期间阻止对迁移和基础设施的编辑，而无需变更工单，并在第 4 阶段：测试的修复任务期间停止智能体编辑测试文件。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

None.
> 无。

**Infrastructure**
> **基础设施**

A written list of the approvals the change process requires.
> 变更过程所需批准的书面列表。

#### 8.2.1、How to execute it（如何执行）

1. Engineering leadership, with change management and compliance, lists the human approval gates that must survive, such as change management sign-off, release authorization, and edits to protected paths.
  工程领导层与变更管理和合规团队一起，列出必须保留的人工审批 gate，例如变更管理签核、发布授权，以及对受保护路径的编辑。

2. The platform engineer expresses each gate as a hook, a script that runs before Claude acts that can allow, ask, or block.
  平台工程师将每个 gate 表达为一个 hook：它是在 Claude 行动前运行的脚本，可以允许、询问或阻止。

3. Team hooks go in `.claude/settings.json` in git, and non-negotiable hooks go in managed settings owned by the platform or IT admin, where individual engineers cannot switch them off.
  团队 hooks 放在 git 中的 `.claude/settings.json`；不可协商的 hooks 放在平台或 IT admin 拥有的 managed settings 中，单个工程师无法关闭。

4. A block should explain itself, so when a hook stops an action the reason and the route to approval appear in Claude's output.
  block 应该自解释：当 hook 阻止某个动作时，原因和审批路径会出现在 Claude 的输出里。

#### 8.2.2、What it looks like (.claude/settings.json)（它看起来像什么（.claude/settings.json））

```
{
    "hooks": {
      "PreToolUse": [
        {
          "matcher": "Bash",
          "hooks": [
            { "type": "command",
              "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/production-gate.sh" }
          ]
        }
      ]
    }
}
```

#### 8.2.3、And the gate itself (.claude/hooks/production-gate.sh)（gate 脚本示例：.claude/hooks/production-gate.sh）

```
#!/bin/bash
# Production deploys require a named release authorization
cmd=$(jq -r '.tool_input.command' < /dev/stdin)
if [[ "$cmd" == *"deploy"* && "$cmd" == *"production"* ]]; then
   if [ -z "$RELEASE_APPROVAL" ]; then
     echo "Production deploys need a release authorization." >&2
     exit 2 # exit 2 blocks the action; the message goes to Claude
   fi
fi
exit 0
```

#### 8.2.4、Governance considerations（治理考虑）

Hooks are the approval gates. The gate condition is enforced every time, for everyone. Allow and block decisions are logged with a timestamp. The gate also defines what counts as approval, whether that's an approved change ticket or the release manager's sign-off.
> Hooks 就是审批 gate。gate 条件会对每个人、每一次执行强制生效。allow/block 决策会带时间戳记录。gate 还定义了什么算批准，例如已批准的变更工单或 release manager 的签字。

Worked example
> 工作示例

### 8.3、Managed settings for a regulated enterprise（受监管企业的托管设置示例）

*Deployed by the platform team via MDM or the admin console; engineers cannot edit or override any of it.*
> *由平台团队通过MDM或管理控制台部署；工程师无法编辑或覆盖其中任何内容。*

```json
{ "permissions": { "deny": [ "Read(.env\*)", "Read(./secrets/\*\*)", "WebFetch", "Bash(curl \*)", "Bash(wget \*)" ], "allow": [ "Bash(git \*)", "Bash(make build)", "Bash(make test)", "Bash(make lint)" ], "disableBypassPermissionsMode": "disable" }, "allowManagedPermissionRulesOnly": true, "sandbox": { "enabled": true, "failIfUnavailable": true, "allowUnsandboxedCommands": false, "network": { "allowedDomains": ["git.internal.example.com", "registry.npmjs.org"] }, "credentials": { "files": [ { "path": "~/.ssh", "mode": "deny" }, { "path": "~/.aws/credentials", "mode": "deny" } ], "envVars": [ { "name": "GITHUB\_TOKEN", "mode": "deny" } ] } }, "allowManagedHooksOnly": true, "disableSideloadFlags": true, "allowManagedMcpServersOnly": true, "strictKnownMarketplaces": [ { "source": "github", "repo": "example-corp/approved-plugins" } ], "requiredMinimumVersion": "2.1.193" }
```

What each line buys, in control terms
> 从控制角度来看，每条线都购买什么

`permissions.deny` keeps secrets out of the agent's context and blocks arbitrary network egress through tools; `permissions.allow` pre-approves the safe inner loop so the deny list doesn't turn into prompt fatigue.
> `permissions.deny` 将密钥保留在智能体上下文之外，并通过工具阻止任意网络出口； `permissions.allow` 预先批准安全内部循环，因此拒绝列表不会变成 prompt fatigue。

`disableBypassPermissionsMode` plus `allowManagedPermissionRulesOnly` means no engineer, project file or command-line flag can widen the rules.
> `disableBypassPermissionsMode` 加 `allowManagedPermissionRulesOnly` 意味着没有工程师、项目文件或命令行标志可以扩大规则。

`sandbox` closes the gap permissions cannot. A tool-level deny on WebFetch doesn't stop a shell command reaching the network; the OS-level domain allowlist blocks egress outright.
> `sandbox` 弥补了 permissions 无法覆盖的缺口。工具层面对 WebFetch 的 deny，并不能阻止 shell 命令访问网络；OS 级域名 allowlist 会直接阻断出站访问。

`failIfUnavailable` and `allowUnsandboxedCommands` make the sandbox a gate: Claude Code refuses to start when the sandbox cannot initialize, and a command that fails inside the sandbox cannot be retried outside it.
> `failIfUnavailable` 和 `allowUnsandboxedCommands` 让沙箱成为一扇 gate：当沙箱无法初始化时，Claude Code 拒绝启动，并且在沙箱内失败的命令无法在沙箱外重试。

`credentials` closes the gap the deny rules leave open. `permissions.deny` governs Claude's file tools, but a sandboxed shell command could still read `~/.ssh` or `~/.aws/credentials` by default; this block denies those reads and strips the named secrets from the environment of every sandboxed command.
> `credentials` 弥补了拒绝规则留下的空白。 `permissions.deny` 管理 Claude 的文件工具，但默认情况下沙盒 shell 命令仍然可以读取 `~/.ssh` 或 `~/.aws/credentials` ；该块拒绝这些读取，并从每个沙盒命令的环境中删除指定的密钥。

`allowManagedHooksOnly` means the approval gates from this play are the only hooks that run; nothing local can add to or replace them.
> `allowManagedHooksOnly` 表示只有这个 play 中的审批 hooks 会运行；本地配置无法添加或替换它们。

`disableSideloadFlags` and `strictKnownMarketplaces` mean every skill, agent, hook and MCP server on an engineer's machine arrived through the organization's approved plugin marketplace, never from a home directory.
> `disableSideloadFlags` 和 `strictKnownMarketplaces` 表示工程师机器上的每个 skill、agent、hook 和 MCP server 都来自组织批准的 plugin marketplace，而不是来自 home 目录。

`allowManagedMcpServersOnly` makes the agent's tool surface an allowlist owned by the platform team.
> `allowManagedMcpServersOnly` 让智能体的工具面成为平台团队拥有的 allowlist。

`requiredMinimumVersion` refuses to start on a version below the approved floor, so the controls are enforced by a build the organization has actually assessed.
> `requiredMinimumVersion` 拒绝在低于批准下限的版本上启动，因此控制是由组织实际eval的构建强制执行的。

Consider the above a starting point to tailor, rather than a recommendation to copy. Every deny trades against capability, and the right balance depends on the data classification of the repo. The settings reference documents every key, including the managed-only ones: [code.claude.com/docs/en/settings](https://code.claude.com/docs/en/settings)
> 请把上面的配置视为定制起点，而不是照抄建议。每一条 deny 都会牺牲一部分能力，正确平衡取决于仓库的数据分类。settings reference 记录了每个 key，包括 managed-only key：code.claude.com/docs/en/settings

How to measure it (for the hooks themselves)
> 如何测量（对于hook本身）

**Leading indicator**
> **领先指标**

Time spent waiting on each approval gate. Every hook decision is written to the OpenTelemetry export with a timestamp and an allow or block verdict, so the wait is visible per gate.
> 每个审批 gate 上花费的等待时间。每个 hook 决策都会以时间戳和 allow/block 结果写入 OpenTelemetry export，因此每个 gate 的等待时间都可见。

**Lagging indicator**
> **滞后指标**

Gate violations reaching production before and after hooks from the incident tracker.
> 从 incident tracker 中比较 hooks 前后进入生产环境的 gate 违规数量。

### 8.4、CI/CD integration and deployment（CI/CD 集成与部署）

Run Claude Code non-interactively inside the CI/CD pipeline, sandbox the execution so long-running agents run safely, expose deployment through MCP integrations, and rehearse the rollback paths before the agent ever needs them.
> 在 CI/CD 流水线内以非交互方式运行 Claude Code，对执行进行沙箱处理，以便长时间运行的智能体安全运行，通过 MCP 集成暴露部署能力，并在智能体需要回滚路径之前演练回滚路径。

Traditional：Pipelines run deterministic scripts, and anything that needs judgment waits for a human. For example, triaging the flaky test, writing the changelog, or working out why the build broke. Deployment and rollback are runbooks a human follows under pressure.
> 传统方式：流水线运行确定性脚本，任何需要判断的事情都要等待人类。例如，分流 flaky test、编写 changelog、或弄清楚 build 为什么失败。部署和回滚是人类在压力下执行的 runbook。

AI-native：Claude runs non-interactively inside the pipeline for the judgment steps, in a sandbox with scoped credentials. Deployment tooling is exposed to the agent through MCP, so the workflow that wrote and tested the change can also ship it and roll it back, inside gates the organization defines per environment.
> AI 原生方式：Claude 在流水线中以非交互方式执行判断步骤，运行在带限定范围凭证的沙箱里。部署工具通过 MCP 暴露给智能体，因此写出并测试变更的工作流，也可以在组织按环境定义的 gate 内发布和回滚它。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

Claude in the PR review loop and hooks as approval gates, because the gates must exist before automation accelerates anything through them.
> Claude 已进入 PR review loop，并且 hooks 已作为审批 gate 存在；因为在自动化加速任何流程穿过 gate 之前，gate 必须先存在。

**Infrastructure**
> **基础设施**

A CI platform with the claude-code-action installed, or any runner that can call `claude -p`; model access through the API, or Bedrock, Foundry, or Vertex where traffic must stay on the organization's cloud agreement; MCP servers for the deployment targets; a sandbox profile for agent jobs with no standing production credentials.
> 安装了 claude-code-action 的 CI 平台，或任何可以调用 `claude -p` 的运行器；通过 API、Bedrock、Foundry 或 Vertex 进行模型访问，其中流量必须遵守组织的云协议；用于部署目标的 MCP 服务器；用于没有常设生产凭证的智能体任务的沙箱profile。

#### 8.4.1、How to execute it（如何执行）

1. The platform engineer starts with read-only judgment steps. Use `claude -p` in a pipeline job to triage a failed build, summarize a flaky test, or draft the changelog.
  平台工程师从只读判断步骤开始。在流水线作业中使用 `claude -p` 来分类失败的构建、总结不稳定的测试或起草变更日志。

2. Add write steps behind the existing gates for jobs like fixing lint, updating generated docs, or addressing review comments via the `@claude` mentions. Anything the agent writes arrives as a PR through branch protection, and the agent has no route to push to main.
  在现有 gate 后添加写入步骤，用于修复 lint、更新生成文档、或通过 `@claude` mention 处理 review comments。智能体写出的任何内容都会通过分支保护以 PR 形式到达，而且智能体没有路径直接推送到 main。

3. Execution is sandboxed. Agent jobs run in containers under a network policy with short-lived scoped tokens, and hold no production credentials by default.
  执行过程运行在沙箱中。智能体任务在容器里运行，受网络策略约束，使用短期、限定范围的 tokens，并且默认不持有生产凭证。

4. Expose deployment through MCP. Deploy, status, and rollback become tools, scoped per environment, so the agent's deployment powers are an allowlist rather than a shell script with credentials.
  通过 MCP 暴露部署能力。deploy、status 和 rollback 变成按环境限定范围的工具，因此智能体的部署权限是一份 allowlist，而不是带凭证的 shell 脚本。

5. Tier the autonomy by environment. In development, the agent deploys freely. In production, the agent prepares the release and the release manager authorizes it, and a hook enforces the production gate. Staging sits somewhere in the middle.
  按环境划分自主性层级。在开发环境中，智能体可以自由部署；在生产环境中，智能体准备发布，由 release manager 授权，并由 hook 强制执行生产 gate；staging 则介于两者之间。

6. Rollback should be the most rehearsed path in the pipeline, a single command that the agent can run and that is exercised regularly in staging. The closing the loop play (Stage 6: Maintenance) calls this rollback when a control band is breached, so it has to be proven in advance.
  回滚应该是流水线中最常演练的路径，是智能体可以运行、并在 staging 中定期演练的单个命令。当控制区间被突破时，闭环 play（第 6 阶段：维护）调用此回滚，因此必须提前证明。

#### 8.4.2、What it looks like (pipeline step)（pipeline step 示例）

```
- name: Triage failed build
  if: failure()
  run: >
    claude -p "Read the build log at out/build.log. Identify the most
    likely cause, say whether the failure looks flaky or real, and write a
    three-line summary for the PR thread." >> triage.md
```

#### 8.4.3、Governance considerations（治理考虑）

The governing principle is that the agent may act up to the production gate and cannot pass it. The controls below enforce this principle.
> 治理原则是：智能体可以行动到生产 gate 之前，但不能越过它。下面的控制措施强制执行这一原则。

* Branch protection turns anything the agent writes into a PR, with no direct path to main.
  分支保护将智能体写入的任何内容都转换为 PR，而没有直接到达 main 的路径。

* The production deploy hook blocks the release until a named release manager authorizes it. Each non-interactive run acts under the agent's own identity, so the pipeline log separates what the agent did from what the engineer who triggered it did.
  生产部署 hook 会阻止发布，直到指定的发布经理授权为止。每个非交互运行都以智能体自己的身份进行操作，因此流水线日志将智能体所做的操作与触发它的工程师所做的操作分开。

* Per-environment permission tiers set how much the agent may do on the way to the gate.
  每个环境的权限级别设置智能体在到达 gate 之前可以做多少事。

How to measure it
> 如何测量

**Leading indicator**
> **领先指标**

The share of pipeline failures triaged without paging a human taken from the CI/CD pipeline logs.
> 从 CI/CD 流水线日志读取：无需呼叫人类即可完成分流的流水线故障比例。

**Lagging indicator**
> **滞后指标**

DevOps Research and Assessment (DORA) measures, which the CI system and deployment tooling already emit.
> DevOps Research and Assessment（DORA）指标；CI 系统和部署工具已经会产生这些指标。

**06**
> **06**

## 9、Maintain（维护）

The loop closes. A trigger invokes Claude with no person in the invocation path, and what it finds re-enters the pipeline as `intent.md`.
> 闭环形成。触发器在调用路径中无人参与的情况下调用 Claude，而 Claude 发现的内容会作为 `intent.md` 重新进入流水线。

### 9.1、Maintenance and closing the loop（维护与闭环）

So far, we've discussed how to add Claude to each stage of the SDLC process, with each stage requiring a human to launch the initial steps. This stage, however, shifts the focus to autonomous running of Claude to close the loop.
> 到目前为止，我们讨论的是如何把 Claude 加入 SDLC 流程的每个阶段，并且每个阶段都需要人类启动初始步骤。但这个阶段会把重点转向 Claude 的自主运行，用它来闭合循环。

For example, a continuously running monitoring agent could, off the back of a bug ticket being raised, create an `intent.md`, and flow through the requirements, plan, build test and review phases. Stage 6: Maintenance runs headless, with an independent confidence gate between stages, a deterministic check or an adversarial reviewing agent, deciding whether the previous stage's output continues or is escalated to a human.
> 例如，一个持续运行的监控智能体可以在 bug 工单被提出后创建 `intent.md`，并流经需求、计划、构建、测试和评审阶段。第 6 阶段 Maintain 以 headless 方式运行，在阶段之间通过独立的置信 gate、确定性检查或对抗性评审智能体，决定上一阶段输出是继续推进还是升级给人类。

Traditional：Maintenance is a reactive phase. All tickets or incidents wait on a person to act on it and restart the process. An alert fires at 3 a.m. and can be missed, a ticket can sit in the backlog until someone picks it up, and post-mortem actions may not reach the codebase at all if another fire starts first.
> 传统方式：维护是一个被动阶段。所有工单或事故都要等待某个人采取行动并重启流程。凌晨 3 点的告警可能被错过，工单可能一直留在 backlog 里直到有人接手；如果另一场火先烧起来，post-mortem 行动项甚至可能根本进不了代码库。

AI-native：A trigger such as a control-band breach, a ticket, a channel message or a schedule invokes Claude without a person in the path. Claude diagnoses, acts only through gated routes, and writes what it finds as `intent.md`, which then goes through the stages described above. People triage and review that work, and no longer have to start it.
> AI 原生方式：控制区间突破、工单、频道消息或定时任务等触发器，会在路径中无人参与的情况下调用 Claude。Claude 进行诊断，只通过带 gate 的路径行动，并把发现写成 `intent.md`，随后进入上述阶段。人类负责分流和评审这些工作，而不再需要亲自启动它。

### 9.2、Closing the loop（闭环）

A deterministic script watches production and invokes Claude when a control band is breached. Monitoring of a breach is a helpful example of the pattern for the loop running autonomously, while the [Claude Tag](https://claude.com/product/tag) (public beta) section at the end of the stage covers work arriving through different channels.
> 一个确定性脚本监控生产环境，并在控制区间被突破时调用 Claude。监控突破是自主闭环模式的一个有用示例；本阶段末尾的 Claude Tag（public beta）部分则涵盖通过不同渠道到达的工作。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

`Intent.md` which gives the loop a structured output to restart. Claude accelerated PR reviews, hooks as an action boundary, and a rollback path for CI/CD (which the highest autonomy tier invokes).
> `Intent.md` 为循环提供用于重启流程的结构化输出。还需要 Claude 加速 PR reviews、作为行动边界的 hooks，以及 CI/CD 的回滚路径（最高自主性层会调用它）。

**Infrastructure**
> **基础设施**

A metrics store the detection script can query (Prometheus, the CI system's API, or equivalents), read access to the repository, a way to run Claude Code non-interactively in CI, or the [Agent SDK](https://platform.claude.com/docs/en/agent-sdk/overview) for a service that receives webhooks.
> 检测脚本可查询的指标存储（Prometheus、CI 系统 API 或同类系统）、仓库读取权限、在 CI 中非交互运行 Claude Code 的方式，或用于接收 webhook 服务的 Agent SDK。

#### 9.2.1、How to execute it（如何执行）

1. The service owner or platform engineer picks one metric with a stable rolling baseline, such as CI test failure rate, post-deploy 5xx rate, or PR cycle time.
  服务所有者或平台工程师选择一种具有稳定滚动基线的指标，例如 CI 测试失败率、部署后 5xx 率或 PR 周期时间。

2. They write the detection script, typically mean and standard deviation over a rolling window with rules (Western Electric or similar) so the bands catch slow drift as well as spikes. The script is version controlled and unit tested, and detection stays entirely deterministic, with no model involved.
  他们编写检测脚本，通常是在滚动窗口上计算均值和标准差，并配合规则（Western Electric 或类似规则），让控制区间既能捕捉缓慢漂移，也能捕捉尖峰。脚本受版本控制并经过单元测试；检测过程保持完全确定性，不涉及模型。

3. Response tiers are defined in version-controlled config (`bands.yaml` below). At 1σ the script only logs, at 2σ it invokes Claude read-only to diagnose, and at 3σ Claude may act, though only by opening a PR into the review gate or triggering a pre-approved runbook.
  响应层级定义在版本控制配置中（如下方 `bands.yaml`）。1σ 时脚本只记录日志；2σ 时以只读方式调用 Claude 诊断；3σ 时 Claude 可以行动，但只能通过向 review gate 打开 PR，或触发预先批准的 runbook。

4. The trigger layer can be a scheduled workflow in GitHub or GitLab, a webhook from the existing monitoring stack, or a Cron Job inside the network. Claude runs stateless, either as a non-interactive step on a CI runner or as an Agent SDK service in a sandboxed container, and the CI/CD play covers the deployment and model-access options. Because the run is stateless and non-interactive, a loop can begin and end without anyone starting it.
  触发层可以是 GitHub 或 GitLab 中的 scheduled workflow、现有监控栈的 webhook，或网络内部的 Cron Job。Claude 以无状态方式运行，可以是 CI runner 上的非交互步骤，也可以是沙箱容器中的 Agent SDK 服务；CI/CD play 覆盖部署和模型访问选项。因为运行是无状态且非交互的，一个循环可以在无人启动的情况下开始并结束。

5. The agent writes its diagnosis as `intent.md` in the Stage 1: Plan format, covering the anomaly and its evidence, a proposed outcome, the affected systems and any open questions. From there the finding goes through the pipeline like anything else.
  智能体按照第 1 阶段 Plan 的格式，将诊断写成 `intent.md`，覆盖异常及其证据、建议结果、受影响系统和开放问题。从这里开始，该发现就像其他变更一样进入流水线。

6. The service owner or on-call engineer triages the queue, routing product-facing findings to the product owner. Fix now, schedule, or dismiss. Dismissals tune the bands and help to reduce noise.
  服务 owner 或 on-call 工程师对队列进行分流，将面向产品的发现路由给产品负责人。可以选择立即修复、排期或驳回。驳回会调优控制区间，并帮助减少噪音。

7. When a fix ships, add an eval for the incident (the continuous evals play) to ensure that such issues are protected against going forwards.
  修复发布后，为该事故添加一个 eval（来自 continuous evals play），确保此类问题未来受到保护。

#### 9.2.2、What it looks like (for example, a bands.yaml monitoring CI test failure rate)（示例：监控 CI 测试失败率的 bands.yaml）

```
metric: ci_test_failure_rate
baseline: rolling_30d
rules: western_electric
tiers:
  1sigma: { action: log }
  2sigma: { action: diagnose,
            tools: "Read,Grep,Bash(gh run view *)" }
  3sigma: { action: propose,
            routes: [pull_request, runbook:rollback-deploy] }
```

#### 9.2.3、Governance considerations（治理考虑）

The tier boundaries are enforced from version-controlled config, with permissions and managed settings denying production access. Invocations, findings and triage decisions are logged with a timestamp. A service owner triages and approves findings, resulting changes go through the normal PR review gate, and the runbooks the agent may trigger were approved in advance.
> 分层边界通过版本控制配置强制执行，权限和 managed settings 会拒绝生产访问。调用、发现和分流决策都会带时间戳记录。服务 owner 对发现进行分流和批准；由此产生的变更会通过正常 PR review gate；智能体可能触发的 runbook 也已提前获批。

How to measure it
> 如何测量

**Leading indicator**
> **领先指标**

Time from band breach to an `intent.md` in the triage queue, against the old time from incident to post-mortem action. The detection script's log has the breach timestamp and tier of incident.
> 从控制区间突破到 `intent.md` 进入分流队列的时间，对比过去从事故到 post-mortem 行动项的时间。检测脚本日志包含突破时间戳和事故层级。

**Lagging indicator**
> **滞后指标**

The share of findings that become merged fixes (triage queue against actual PR history), and repeat incidents of the same class, which should fall as the fixes add cases to the eval suite.
> 最终变成已合并修复的发现占比（分流队列对照实际 PR 历史），以及同类重复事故数量；随着修复把案例加入 eval suite，这些指标应当下降。

#### 9.2.4、Examples（示例）

* When the CI test failure rate breaches 3σ, the agent quarantines the flaky test or opens a revert PR, and the review gate decides.
  当 CI 测试失败率突破 3σ 时，智能体会隔离不稳定的测试或打开revert PR，然后由评审门决定。

* When the post-deploy 5xx rate breaches 3σ with a deployment in the window, the agent triggers the existing rollback pipeline.
  当部署后 5xx 速率突破 3σ，且时间窗口内存在部署时，智能体会触发现有回滚流水线。

* When PR cycle time trips a drift rule, the agent writes a report for engineering leadership, which shows the harness works for process metrics as well as production ones.
  当 PR 周期时间超出漂移规则时，智能体会为工程领导层编写一份报告，其中显示这个 harness 不仅适用于生产指标，也适用于流程指标。

Detection stays deterministic. Claude is invoked once a band is breached, and the tier sets what it may do.
> 检测保持确定性。一旦控制区间被突破，Claude 就会被调用，而层级决定它可以做什么。

### 9.3、Recurring codebase scans（周期性代码库扫描）

A security scan is a point-in-time statement about a codebase under a particular model, and both halves go stale: the code changes every week, and each model generation finds vulnerabilities the previous one missed. The AI-native answer is to run the scan on a schedule, without a human in the invocation path, and to send what it finds through the same gates as any other change to the codebase.
> 安全扫描是在特定模型下对代码库作出的时间点判断，而且两部分都会过时：代码每周变化，新一代模型也会发现上一代模型漏掉的漏洞。AI 原生答案是按计划运行扫描，调用路径中无人参与，并把发现通过与其他代码变更相同的 gate 送入流程。

[Claude Security](https://claude.com/product/claude-security) is the hosted form of scheduled scanning. Connect a GitHub repository, and scans run on Claude Mythos 5 in Anthropic's infrastructure, with each finding validated before it is reported and a confidence rating attached. Suggested patches are reviewed and applied in Claude Code on the web. The organization gets the findings without needing access to the model itself.
> Claude Security 是定期扫描的托管形态。连接 GitHub 仓库后，扫描会在 Anthropic 基础设施中的 Claude Mythos 5 上运行；每个发现都会在报告前被验证，并附带置信度评分。建议 patch 会在 Claude Code on the Web 中评审和应用。组织无需直接访问模型本身，也能获得发现。

Traditional：Security scanning is an event with a scan launched before a release or an audit. The report goes to a tracker, and the backlog is worked down by hand until the next event. Code written in between is covered by whatever the PR review caught.
> 传统方式：安全扫描是一个事件，通常在发布或审计前启动。报告进入 tracker，然后人工处理 backlog，直到下一次事件发生。两次扫描之间写出的代码，只能依赖 PR review 捕获的问题覆盖。

AI-native：Scans run on a schedule against every connected repository, on the most capable model available, with findings validated before anyone reads them. Each finding is handled the way a breached control band is: a fix that fits in one PR goes through the review gate, and anything larger becomes an `intent.md`. Coverage is dated from the last run, not from the first
> AI 原生方式：扫描按计划针对每个已连接仓库运行，使用可用的最强模型，并在任何人读取结果前验证发现。每个发现都像控制区间突破一样处理：能放进一个 PR 的修复进入 review gate；更大的问题写成 `intent.md`。覆盖范围从最近一次运行算起，而不是第一次运行。

Getting started
> 如何开始

**Prerequisites**
> **前置条件**

The PR review gate and hooks as approval gates ([Stage 5: Deploy](#sd-s5)), so that findings go through review like any other change. The `intent.md` format from [Stage 1: Plan](#sd-s1) for findings too large for a single PR.
> PR review gate 和作为审批 gate 的 hooks（第 5 阶段 Deploy），这样发现就能像其他变更一样经过 review。对于大到无法放进单个 PR 的发现，使用第 1 阶段 Plan 的 `intent.md` 格式。

**Infrastructure**
> **基础设施**

Claude Security is available to Claude Enterprise organizations in public beta. It needs the Anthropic GitHub App installed on the target repositories (cloud-hosted github.com), Claude Code on the Web enabled, Extra Usage turned on with a spend limit set, premium seats for the people who run scans, and the feature switched on by an admin at `claude.ai/admin-settings/claude-code`. Scans are billed on consumption at Mythos 5 rates, so the spend limit should match the size and number of repositories.
> Claude Security 在公共测试版中可供 Claude Enterprise 组织使用。它需要在目标仓库（云托管的 github.com）上安装 Anthropic GitHub 应用，启用 Web 上的 Claude Code，打开额外使用并设置支出限制，为运行扫描的人员提供高级席位，并且由 `claude.ai/admin-settings/claude-code` 的管理员打开该功能。扫描按 Mythos 5 费率按消耗计费，因此支出限制应与仓库的大小和数量相匹配。

#### 9.3.1、How to execute it（如何执行）

1. The security lead connects the repositories and organizes them into projects by repo, service, or team, so ownership of findings is clear from the start.
  安全主管连接仓库，并按仓库、服务或团队将它们组织到项目中，因此结果的所有权从一开始就很明确。

2. Run a first full scan of the most critical repositories, including ones that have been scanned before by other tools or by earlier models. Treat the first scan as the baseline. The first scan will likely surface findings in code that was considered clean.
  对最关键的仓库进行首次完整扫描，包括之前由其他工具或早期模型扫描过的仓库。将第一次扫描作为基线。第一次扫描可能会发现被认为是干净的代码。

3. Set a schedule per project. Weekly is a sensible default for actively developed services; scope scans to a directory or branch where a repository is large or mixed.
  为每个项目设定一个时间表。对于积极开发的服务来说，每周是一个明智的默认设置；范围扫描到仓库较大或混合的目录或分支。

4. Triage findings with the confidence rating in hand. Dismiss with a reason, so the dismissal is recorded and the same finding does not return as new on the next run.
  根据现有的置信度对结果进行分类。驳回有原因，因此驳回会被记录下来，并且相同的结果不会在下一次运行中作为新的结果返回。

5. For a bounded finding, open the suggested patch in Claude Code on the Web, review it, and send it through the PR review gate like any other change. The agent that proposed the fix has no route to approve it.
  对于边界清晰的发现，在 Claude Code on the Web 中打开建议 patch，评审它，并像其他变更一样送入 PR review gate。提出修复的智能体没有路径批准它。

6. For anything wider than one patch, such as an architectural weakness or a pattern repeated across services, write it up as `intent.md` in the Stage 1 format and start it at Plan.
  对于范围超过单个 patch 的问题，例如架构弱点或跨服务重复出现的模式，用第 1 阶段格式写成 `intent.md`，并从 Plan 开始。

7. When a fix is released to production, add an eval for the vulnerability class to the suite from the continuous evals play, so the configuration that steers the agent is tested against that class from then on.
  修复发布到生产环境后，把该漏洞类别作为 eval 加入 continuous evals play 的套件中，这样从此以后，引导智能体的配置都会针对这一类别接受测试。

8. Export findings as CSV or Markdown, or use webhooks, to keep the organization's existing tracker and audit systems as the system of record where auditors already expect them.
  将结果导出为 CSV 或 Markdown，或使用 Webhooks，以将组织现有的跟踪器和审计系统保留为审计人员期望的记录系统。

#### 9.3.2、Governance considerations（治理考虑）

The scan runs under the organization's admin controls meaning what repositories are connected, who holds a scan seat, and the spend limit are all set centrally. Every finding has a validation result and a confidence rating, and every dismissal has a reason, so the scan history is an audit record of what was found, fixed, and consciously accepted.
> 扫描在组织 admin controls 下运行，也就是说连接哪些仓库、谁持有扫描席位、支出限制是多少，都会集中设置。每个发现都有验证结果和置信度评分，每次驳回都有原因，因此扫描历史就是“发现了什么、修复了什么、以及有意识接受了什么”的审计记录。

Fixes reach production through the PR review gate and branch protection rather than from the scan itself. Claude Security augments existing static analysis and dependency scanning. The deterministic checks stay in CI, and the model-driven scan covers the context-dependent vulnerabilities those checks are not built to find.
> 修复通过 PR review gate 和分支保护进入生产，而不是由扫描本身直接进入生产。Claude Security 增强现有静态分析和依赖扫描。确定性检查继续留在 CI 中，而模型驱动扫描覆盖那些传统检查本来就不擅长发现的上下文相关漏洞。

How to measure it
> 如何测量

**Leading indicator**
> **领先指标**

Share of connected repositories on a schedule, and time from a finding being reported to its patch entering the PR review gate, read from the scan history and the PR metadata.
> 已连接仓库中按计划扫描的比例，以及从发现被报告到其 patch 进入 PR review gate 的时间；这些可从扫描历史和 PR metadata 中读取。

**Lagging indicator**
> **滞后指标**

Vulnerabilities found by the scheduled scan set against those found in production or by external report, from the incident tracker; and the trend in findings per scan on repositories that have been through several runs, which should fall as fixes and evals accumulate.
> 通过计划扫描集针对生产中发现的漏洞或通过事件跟踪器的外部报告发现的漏洞；以及经过多次运行的仓库每次扫描结果的趋势，随着修复和eval的积累，该结果应该会下降。

### 9.4、Claude on call with Claude Tag（Claude Tag 参与 on-call）

Incidents can also arrive via other means such as workplace communication apps, like Slack or Teams. Incidents can look like a 10pm Slack message for an urgent fix on an incident channel and can now be actioned immediately. Claude Tag (public beta currently available in Slack) makes Claude a member of those channels under its own identity, so each new incident gets a first responder and the response itself becomes part of the loop and memory for future incidents.
> 事件还可以通过其他方式到达，例如工作场所通信应用，如 Slack 或 Teams。事件可能看起来像晚上 10 点 Slack 消息，用于紧急修复事件通道，现在可以立即采取行动。 Claude Tag（目前在 Slack 中提供公共测试版）使 Claude 以自己的身份成为这些通道的成员，因此每个新事件都会有一个第一响应者，并且响应本身会成为未来事件循环和内存的一部分。

The conversation and institutional knowledge stay in the channel, with anyone in the channel able to guide and action the response. Any team member can test hypotheses, explore new options and investigate in real time with the channel history adding to the auditability. Through access to MCP Claude verifies the metric is back at baseline and confirms it in the thread, writes the post-mortem to a version-controlled lessons file that future investigations can read.
> 对话和组织知识保留在频道中，频道里的任何人都能引导并推进响应。任何团队成员都可以实时测试假设、探索新选项和调查问题，频道历史也增加了可审计性。通过 MCP，Claude 可以验证指标是否回到基线并在线程中确认，还会把 post-mortem 写入受版本控制的 lessons 文件，供未来调查读取。

Incidents are not the only work Claude Tag picks up. Tagged on a ticket over MCP or asked in the channel, Claude triages the work the same way. A small, well-bounded fix arrives as a PR through the review gate, and anything larger is written up as `intent.md` for Stage 1: Plan, at which point the loop starts feeding itself. See: [how Claude Tag runs on-call for CI/CD at Anthropic](https://claude.com/blog/ai-ci-cd-on-call).
> 事故并不是 Claude Tag 接手的唯一工作。无论是在 MCP 上的工单里 tag Claude，还是在频道中询问，Claude 都会以同样方式分流工作。小而边界清晰的修复会通过 review gate 以 PR 形式到达；更大的问题则会写成第 1 阶段 Plan 的 `intent.md`，此时循环开始自我驱动。参见：Claude Tag 如何在 Anthropic 负责 CI/CD on-call。

![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a8760aded54a2a8319cd5b9_fe6d780d.png)

The channel is the audit trail: request, diagnosis, human authorization and fix all stay where the incident was handled.
> 频道就是审计轨迹：请求、诊断、人工授权和修复都保留在事故处理的地方。

## 10、Closing thoughts（结语）

Models and harnesses have become more advanced, allowing organizations to not just transform how they produce code, but the entire software development lifecycle.
> 模型和 harness 已经变得更先进，使组织不仅能改变代码生产方式，还能转型整个软件开发生命周期。

This transformation keeps human judgement central to the process and considers the governance and regulation requirements of large enterprise organizations.
> 这种转变使人类判断成为流程的核心，并考虑大型企业组织的治理和监管要求。

This guide consolidated many of the real best practices our Applied AI team executes on a daily basis for our customers, and we hope you found it a practical and actionable resource.
> 本指南整合了 Applied AI 团队每天为客户实践的许多真实最佳实践，希望你会觉得它实用且可操作。

The loop keeps running. Human judgement stays above it.
> 循环持续运行。人类判断始终位于其上。

### 10.1、Resources and acknowledgments（资源与致谢）

The documentation below is what a platform team needs to set those controls up, in roughly the order you would roll them out.
> 下面这些文档，是平台团队设置这些控制机制所需的材料，大致按你推出它们的顺序排列。

- [Set up Claude Code for your organization — the admin decision map; start here](https://code.claude.com/docs/en/admin-setup)
  > 为组织设置 Claude Code：管理员决策图，建议从这里开始。

- [Settings reference and precedence, including every managed-only key](https://code.claude.com/docs/en/settings)
  > Settings 参考与优先级，包括所有 managed-only key。

- [Server-managed settings from the Claude admin console](https://code.claude.com/docs/en/server-managed-settings)
  > 来自 Claude admin console 的 server-managed settings。

- [Permissions](https://code.claude.com/docs/en/permissions)
  > 权限配置。

- [Sandboxing — OS-level filesystem and network isolation](https://code.claude.com/docs/en/sandboxing)
  > 沙箱：OS 级文件系统和网络隔离。

- [Hooks — guide](https://code.claude.com/docs/en/hooks-guide) / [Hooks — reference](https://code.claude.com/docs/en/hooks)
  > Hooks 指南与参考。

- [Skills](https://code.claude.com/docs/en/skills)
  > Skills 文档。

- [Plugins and private marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
  > Plugins 和私有 marketplaces：用于在组织范围内分发 skills 与 hooks。

- [Managed MCP](https://code.claude.com/docs/en/managed-mcp)
  > Managed MCP：集中控制智能体的工具面。

- [Enterprise deployment overview — Bedrock, Vertex, Foundry](https://code.claude.com/docs/en/third-party-integrations)
  > 企业部署概览：Bedrock、Vertex、Foundry。

- [Enterprise network configuration](https://code.claude.com/docs/en/network-config)
  > 企业网络配置。

- [Monitoring (OpenTelemetry)](https://code.claude.com/docs/en/monitoring-usage)
  > Monitoring / OpenTelemetry。

- [Analytics dashboard](https://code.claude.com/docs/en/analytics)
  > Analytics dashboard。

- [Compliance API](https://platform.claude.com/docs/en/manage-claude/compliance-api)
  > Compliance API：企业活动流、聊天检索与删除。

- [Security model](https://code.claude.com/docs/en/security)
  > Security model。

Thanks to Jim Blackhurst, Will Steuk, and Jamal Arif for their contributions to this guide, which was inspired by and built on much of their previous work.
> 感谢 Jim Blackhurst、Will Steuk 和 Jamal Arif 对本指南的贡献，本指南的灵感来自于他们之前的大部分工作，并以此为基础。
