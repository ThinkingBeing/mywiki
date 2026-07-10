---
layout: post
title: "The New SDLC With Vibe Coding：从随性提示到 Agentic Engineering"
description: "这份白皮书提出 AI 正在把软件开发从写语法转向表达意图；真正能进入生产的软件团队，不是靠 vibe coding 的速度，而是靠 context engineering、harness、测试/eval、guardrails 和人类判断构成的新 SDLC。"
date: 2026-05-01
created_at: 2026-07-10 09:06:35 +0800
categories: AI-Agent
---

# The New SDLC With Vibe Coding：从随性提示到 Agentic Engineering

> 来源: Google Drive PDF | 文件标题: *The New SDLC With Vibe Coding: From ad-hoc prompting to Agentic Engineering* | 发布/版本时间: 2026-05 | 作者: Addy Osmani, Shubham Saboo, Sokratis Kartakis

## 1. 文档基本信息

- **标题**: The New SDLC With Vibe Coding: From ad-hoc prompting to Agentic Engineering
- **作者**: Addy Osmani, Shubham Saboo, Sokratis Kartakis
- **时间**: 2026 年 5 月
- **篇幅**: 51 页
- **主题**: Vibe Coding、Agentic Engineering、AI 驱动的软件开发生命周期、Context Engineering、Harness Engineering、AI coding agents、开发者角色转型
- **原始文件**: 用户提供的 Google Drive PDF
- **本地备份**: `/home/user/hermes/downloads/the-new-sdlc-with-vibe-coding.pdf`

## 2. 文档要解决的核心问题

这份白皮书试图回答一个非常现实的问题：**当 AI Coding Agents 已经能够生成多文件代码、运行测试、修复错误、提交 PR，传统软件开发生命周期（SDLC）应该如何重构，才能既利用 AI 的速度，又不牺牲生产软件所需的质量、可靠性和工程纪律？**

作者的核心结论是：

> 软件工程正在从“写代码语法”转向“表达意图、设计约束、验证输出”。Vibe coding 可以加速探索，但生产级软件需要 Agentic Engineering：用规格、上下文、测试、eval、guardrails、sandbox、observability 和人类判断构成一套能持续生产可靠软件的系统。

换句话说，AI 时代的软件工程瓶颈不再主要是“谁能更快写代码”，而是：

- 谁能更清楚地定义目标；
- 谁能把业务意图转化为机器可执行的规格；
- 谁能设计高质量上下文；
- 谁能建立自动化验证和反馈回路；
- 谁能判断哪些输出可以信任、哪些必须推翻。

## 3. 背景：为什么这份文档重要？

文档开头指出，编程史长期以来是一种“翻译”活动：人理解问题，设计抽象方案，再把方案翻译成机器可执行的语法。高阶语言、框架、云平台都在降低这层翻译成本，但开发者和机器之间的主要接口仍然是语法。

AI Coding Agents 正在改变这一点。作者引用的趋势数据包括：

- 到 2026 年初，约 85% 的专业开发者定期使用 AI Coding Agents；
- 51% 每天使用；
- 估计 41% 的新代码由 AI 生成。

从 autocomplete 到 inline code suggestion，再到 chat-based coding，再到可以 clone repo、规划多文件修改、运行测试、提交 PR 的自主 Agent，软件开发的主接口正在从 syntax 变成 intent。

但这也带来风险：如果开发者只是把需求随手丢给模型、接受返回结果、出错后复制错误消息继续让 AI 修，确实很快，却很难进入生产。作者把这种随性方式称为 **vibe coding**；与之相对的是 **agentic engineering**，即在规格、约束、测试、反馈、可观测性和人工判断保护下使用 AI。

## 4. 核心观点

### 4.1 Vibe Coding 与 Agentic Engineering 不是二元对立，而是一条光谱

文档把 AI 辅助开发分成三个层次：

| 维度 | Vibe Coding | Structured AI-Assisted Coding | Agentic Engineering |
|---|---|---|---|
| 意图表达 | 随意自然语言提示 | 带示例和约束的详细提示 | 正式 specs、架构文档、memory 文件 |
| 验证方式 | “看起来能跑” | 人工测试、抽查 | 自动测试、CI/CD gates、LM judges |
| 代码理解 | 可能不读生成代码 | 选择性审查关键路径 | 关注架构与质量，AI 负责实现细节 |
| 错误处理 | 复制错误给 AI | 人诊断根因，AI 实现修复 | Agent 在边界内自诊断，人处理架构问题 |
| 适用范围 | 原型、脚本、个人项目 | 既有代码库中的普通功能 | 生产系统、团队级开发 |
| 风险 | 高，适合可丢弃代码 | 中等 | 低，依赖系统性验证 |

这张光谱很关键，因为它避免了一个常见误解：**“用了 AI”并不自动等于“vibe coding”，也不自动等于“agentic engineering”。区别不在工具，而在 AI 输出周围有没有结构、验证和判断。**

作者有一句判断很硬：如果没有 tests 和 evals，再复杂的 prompt 也仍然只是 vibe coding。

### 4.2 Context Engineering 是真正的新技能

文档强调，AI 生成代码质量越来越少取决于“提示词是不是巧妙”，越来越多取决于“上下文是否足够好”。这就是 **context engineering**。

作者把 Agent 需要的上下文分为六类：

1. **Instructions**：角色、目标、边界。
2. **Knowledge**：文档、架构图、领域知识。
3. **Memory**：短期会话记录与长期项目状态。
4. **Examples**：few-shot 行为示例和代码库中的参考模式。
5. **Tools**：API、脚本、外部服务的精确定义。
6. **Guardrails**：硬约束、格式规则、安全校验。

这里最有洞察的是对 **static context** 和 **dynamic context** 的区分：

- Static context：总是加载，例如 system instructions、AGENTS.md、CLAUDE.md、GEMINI.md、全局 memory。优点是稳定，缺点是每次都消耗 token，且容易稀释重点。
- Dynamic context：按需加载，例如 skills、工具结果、RAG 文档、窗口化历史。优点是高效，缺点是需要设计触发和检索机制。

这和我们在实际使用 Hermes、Claude Code、Codex 时的经验高度一致：把所有规则都塞进 system prompt 会导致上下文膨胀；把流程知识拆成可按需加载的 skill，才更接近可持续的工程系统。

### 4.3 新 SDLC：实现被压缩，需求、架构、验证仍由人主导

文档认为，AI 没有简单地让旧 SDLC “更快”，而是让 SDLC 的边界重新排列。

- **需求与规划**：AI 可以从产品简报生成 user stories、识别边界条件、生成 API schema、快速做原型。需求不再只是团队间交接的文档，而变成人与 AI 的对话，并同时产出 specification 和初始实现。
- **设计与架构**：这是最顽固的人类中心阶段。架构涉及一致性 vs 可用性、复杂度 vs 灵活性、买 vs 建等长期权衡，AI 可以执行架构，但不能完全替代人类做判断。
- **实现**：AI 能显著加速多文件修改和功能开发，但收益并不总是线性。文档引用 METR 研究指出，部分经验开发者使用 AI 反而慢了 19%，原因在于验证、调试和纠错成本上升。
- **测试与 QA**：测试 AI 生成代码不仅要看最终代码是否通过，还要看 Agent 的执行轨迹是否合理。也就是说，output evaluation 与 trajectory evaluation 都重要。
- **Code review 与部署**：AI 可以做第一轮 reviewer，检查 bug、风格、安全和性能问题，但不能取代对设计、可维护性和战略一致性的人工判断。
- **维护与演进**：AI 能读懂遗留代码、识别相关文件、按既有模式修改，从而降低过去“不敢碰”的技术债迁移和重构成本。

这部分的核心是：**AI 最先压缩的是 implementation，但真正决定生产质量的 requirements、architecture、verification 并没有消失，反而变得更重要。**

### 4.4 Factory Model：开发者的产出从“代码”变成“生产代码的系统”

文档提出一个很重要的心智模型：**Factory Model**。

在这个模型里，现代开发者不再像工人一样亲手装配每个零件，而更像工厂经理：设计流水线、设定质量标准、监控产出。

这个“软件工厂”包括：

- 规格和上下文，定义要构建什么；
- Agent，把规格转化为实现；
- 测试和质量门，验证正确性；
- 反馈回路，把失败交回给 Agent 修正；
- Guardrails，把 Agent 限制在安全和可预测边界内。

这与 Addy Osmani 另一篇关于 Agentic Autonomy Levels 的文章形成互补：自主性不是让 Agent 自由发挥，而是在一个能验证、能回滚、能观察、能分派的系统里运行。

### 4.5 Harness Engineering：模型不是系统，模型周围的东西才决定 Agent 能否完成任务

文档最值得工程团队反复阅读的概念是 **Harness**。

作者指出，很多团队容易把“模型”当成“系统”：新模型出来，Agent 变强；旧模型不行，Agent 变差。但这会导致错误投资。真正决定 Agent 行为的，往往是模型周围的 harness：

- instructions 和 rule files，例如 AGENTS.md、CLAUDE.md、GEMINI.md、skill 文件；
- tools，例如函数、MCP server、API；
- sandbox 和执行环境；
- orchestration logic，例如 sub-agent、model routing、handoff；
- guardrails/hooks，例如 tool call 前、文件编辑后、commit 前运行的确定性检查；
- observability，例如 logs、traces、evals、成本和延迟监控。

作者给出一个非常明确的公式：

> Agent = Model + Harness

这意味着，一个 raw model 不是 Agent；只有当 harness 给它状态、工具执行、反馈循环和可执行约束之后，它才成为真正的 Agent。

文档还引用 benchmark 例子说明 harness effect：有团队只改 harness、不换模型，就让 coding agent 在 Terminal Bench 2.0 从 Top 30 之外进入 Top 5；LangChain 也通过改 system prompt、tools 和 middleware，在固定模型上把分数提高 13.7 分。

这对工程团队的启发是：Agent 失败时，不要第一反应就是“模型不行”。很多失败其实是配置失败：缺工具、规则模糊、guardrail 缺失、上下文噪音太多、没有观测。

### 4.6 开发者角色：从 Conductor 到 Orchestrator

文档把开发者与 AI 协作的角色分成两种模式：

#### Conductor：指挥家模式

开发者在 IDE 中实时引导 AI：看着代码生成，随时提示、修改、纠偏。适合复杂逻辑、棘手 debug、不熟悉代码库等场景。它保持了传统工程师熟悉的控制感，但吞吐量受限，因为人仍然在指挥每个细节。

#### Orchestrator：编排者模式

开发者在更高抽象层工作：定义目标，把任务交给后台或多个 Agent，稍后审查结果并纠偏。适合已定义清楚的 bugfix、按既有模式实现功能、迁移、测试生成等任务。

Orchestrator 需要的不是更熟练的语法，而是：

- Specification：能否把任务定义清楚；
- Decomposition：能否把大任务拆成适合 Agent 的单元；
- Evaluation：能否快速判断输出质量；
- System design：能否设计约束、测试和反馈回路。

这也是文档后面那句结论的铺垫：**Generation is solved. Verification, judgment, and direction are the new craft.**

### 4.7 80% 问题：AI 快速生成大部分代码，但最后 20% 决定生产质量

文档提出 AI-assisted development 的一个长期挑战：**80% problem**。

AI 可以快速生成一个功能的大约 80%，但剩下 20%——边界条件、错误处理、集成点、业务正确性、长期维护影响——仍然需要深层上下文和人类判断。

更危险的是，AI 错误已经从语法错误转向概念错误：

- 对业务逻辑做错假设；
- 需求模糊时不澄清；
- 漏掉 edge cases；
- 产生看起来合理、甚至通过基础测试、但长期架构有问题的代码。

因此成熟开发者不是“接受 AI 生成的一切以追求速度”，而是把自己的注意力保留给 AI 最不擅长的部分：歧义、架构权衡、正确性验证。

### 4.8 Coding Agents 的三类工作位置

文档把日常 coding agents 分成三类：

1. **Editor agents**：在 IDE 中提供补全、解释、局部修改，如 GitHub Copilot、Cursor、Windsurf、JetBrains AI Assistant。适合保持 flow。
2. **Terminal agents**：从命令行接收自然语言目标，拥有文件系统访问、能运行测试并迭代，如 Claude Code、Codex CLI、OpenCode、Cline。适合多文件工作、探索陌生代码库、需要运行工具的任务。
3. **Background agents**：在云端 sandbox 中接收任务，运行数小时并产出 PR，如 Google Jules、Copilot agent mode、Cursor background agents。适合描述清楚、可异步交付的任务。

作者强调：起点取决于任务，而不是哪个工具看起来自主性更高。

### 4.9 构建生产级 Agent：从原型到产品的生命周期正在坍缩

文档还讨论了当“要构建的软件本身就是 Agent”时会发生什么。比如客服 bot、研究助手、合规监控工具。这些不是一次性脚本，而是需要自己的工具、memory、eval、deployment、observability 的产品。

作者以 Google Agents CLI / ADK 为例说明：开发者可以用同一个 coding agent，通过 skills 完成 scaffold、编写 agent code、生成 evalset、运行 eval、部署到 Agent Runtime、接入 observability。过去从原型到生产需要单独学习框架和云控制台，现在越来越多变成终端中的自然语言工作流。

这部分有一定 Google 生态推广色彩，但趋势判断是成立的：**原型、评估、部署、观测、迭代正在被压缩到同一条工作流中。**

### 4.10 AI 开发经济学：Vibe Coding 是低 CapEx、高 OpEx；Agentic Engineering 是高 CapEx、低 OpEx

文档最后从经济学角度分析 AI 开发：不要只看“写代码速度”，要看 TCO（Total Cost of Ownership）。

Vibe coding 表面上成本低：一个 AI 订阅、几个随意 prompt 就能开始。但它隐藏了高 OpEx：

- **Token burn rate**：反复把大文件塞给模型、让模型修自己的错误，消耗大量 token；
- **Maintenance tax**：随意生成的代码缺乏结构一致性，几个月后人类要反向理解 AI 生成的“意大利面”；
- **Security remediation**：没有 eval harness，漏洞更容易进入生产，修复成本指数级上升。

Agentic engineering 相反：前期要投入 specs、API schema、测试、上下文结构、harness、guardrails，CapEx 更高；但一旦工厂建好，每次交付和维护的边际成本会下降。

作者还指出，context engineering 本身是财务杠杆：把 100,000 token repo 每次都塞进 prompt 在规模化时不可行；高信噪比上下文、dynamic context、skills、MCP tool calling、智能模型路由，都是降低 OpEx 的手段。

## 5. 观点对应的论据

文档主要使用四类论据：

1. **行业采用数据**：AI coding agents 的普及率、新代码中 AI 生成比例、开发者日常使用情况。
2. **工具形态变化**：从 IDE 补全、终端 Agent 到后台 Agent，说明 AI 已经从“建议工具”进入“执行系统”。
3. **研究与 benchmark**：引用 METR 关于 AI 工具可能让经验开发者在某些任务上变慢的研究；引用 Terminal Bench / LangChain 的 harness 改进结果说明系统设计影响巨大。
4. **工程经验模型**：通过 vibe coding vs agentic engineering、factory model、harness、80% problem 等框架，把零散现象组织成可操作的心智模型。

这些论据中，最有说服力的是 harness effect 和 80% problem，因为它们与现实工程体验高度一致：模型越强，越容易让人忽略系统边界；生成越快，越容易让验证成为瓶颈。

## 6. 我的评价和启发

### 6.1 这份文档真正重要的不是“vibe coding”，而是它给出了 AI-Native SDLC 的工程化语言

“Vibe coding”这个词容易被媒体化，好像重点是开发者可以随便说几句话就生成产品。但这份文档的真正价值，是把讨论从“AI 写代码快不快”推进到“怎样构建一个可靠的软件生产系统”。

我认为它最重要的三个关键词是：

- **Context Engineering**：让 Agent 拥有正确上下文；
- **Harness Engineering**：让 Agent 在正确工具、权限、沙箱、hook、观测中运行；
- **Verification**：用 tests、evals、review、traces 和 guardrails 证明输出可信。

这三者合起来，才是从 vibe coding 进入 agentic engineering 的门槛。

### 6.2 对个人开发者的启发

如果是个人开发者，最可执行的建议不是马上追逐更多 Agent，而是从三个小动作开始：

1. 给项目写一个简洁但严格的 `AGENTS.md` / `CLAUDE.md` / `GEMINI.md`；
2. 把重复流程沉淀为 skill，而不是每次重新 prompt；
3. 对要交付的代码先写 tests/evals，再让 AI 实现。

尤其是“每当 Agent 做了不该做的事，就往规则里加一条”这个建议很实用。它把使用 AI 的经验从个人脑子里转移到项目资产中。

### 6.3 对团队负责人的启发

对 engineering leaders 来说，最大风险是只采购 AI 工具，却不投资 harness。这样会得到“速度幻觉”：短期 demo 很快，长期技术债、review 压力、安全风险和 token 成本上升。

团队应该把以下内容当作代码一样管理：

- system prompts；
- AGENTS.md / rule files；
- eval suites；
- skill libraries；
- MCP server connections；
- review checklists；
- agent run traces 和成本指标。

这些不是“提示词小技巧”，而是新 SDLC 的基础设施。

### 6.4 我最认同的观点：AI 放大工程文化

文档说：**AI amplifies your engineering culture.** 这句话很准确。

有强测试、清晰架构标准、健康 code review 的团队，会被 AI 放大优势；没有这些基础的团队，会被 AI 放大混乱。AI 不会自动让一个混乱团队变成高效团队，它只是让混乱以更高速度发生。

这也解释了为什么有些人使用 AI 感觉效率暴涨，有些人感觉越来越累：前者可能已经有清晰任务边界和验证体系，后者则被迫审查大量看似合理但缺乏上下文约束的输出。

### 6.5 我需要保留意见的地方

文档对 AI coding agents 的生产化速度较乐观，尤其是“原型到生产从数周缩短到数小时”的判断。我认为这在某些内部工具、低风险 agent、强云平台约束场景下成立，但对高风险业务系统仍需谨慎。

原因有三点：

1. **业务正确性难以自动评估**：很多错误不是代码错，而是业务语义错。
2. **安全与权限边界更复杂**：Agent 一旦接入真实数据和工具，风险从代码 bug 扩展到数据泄露、越权操作、供应链攻击。
3. **组织吸收速度有限**：AI 能生成更多 PR，但人类 review、产品决策、上线审批、用户反馈处理不一定同步扩容。

所以，“从原型到生产更快”应该附带前提：评估体系、权限体系、观测体系、回滚机制已经就绪。

### 6.6 和 Addy Osmani《Agentic Autonomy Levels》的关系

这份 PDF 与 Addy 另一篇《Agentic Autonomy Levels》高度互补：

- 《Agentic Autonomy Levels》回答：一个任务应该给 Agent 多高自主性？
- 这份 PDF 回答：整个 SDLC 应该如何围绕 Agent 重构？

两者共同指向同一个结论：**未来的软件工程不是“人不用管，AI 自己写”，而是人类把目标、边界、上下文、工具、验证和反馈系统设计好，让 AI 在其中高效执行。**

## 7. 可落地的实践清单

### 7.1 个人开发者

- 为每个项目建立 `AGENTS.md`，包含技术栈、目录结构、代码风格、禁止事项、测试命令。
- 把常用流程写成 skills，例如“新增 API endpoint”“修复测试失败”“写代码 review 报告”。
- 对生产代码坚持先定义 acceptance criteria，再让 Agent 实现。
- 要求 Agent 输出证据包：diff、测试结果、lint/typecheck、风险与未覆盖点。
- 对将上线的 AI 代码逐行审查，特别关注依赖是否真实、错误处理是否完整、边界条件是否覆盖。

### 7.2 团队负责人

- 把 context engineering 作为正式工程实践，而不是个人技巧。
- eval 先于 demo：demo 证明一次成功，eval 证明可重复成功。
- 为 AI 生成代码设计专门 review checklist。
- 明确 prototype 与 production 的边界，避免原型误上生产。
- 把 harness 组件作为共享基础设施维护。

### 7.3 组织层面

- 把 AI-assisted development 当作工程投资，而不是简单生产力插件。
- 在规模化前先建设 production substrate：eval、trace、权限、sandbox、安全 review。
- 尽量采用 MCP、A2A 等开放协议，降低供应商锁定。
- 设计人类 + Agent 混合团队的交接协议。
- 招聘和培养重点从“写最多代码”转向“规格、评估、架构判断、review”。

## 8. 延伸阅读

- Addy Osmani: [Agentic Engineering](https://addyosmani.com/blog/agentic-engineering/)
- Addy Osmani: [The Factory Model](https://addyosmani.com/blog/factory-model/)
- Addy Osmani: [The 80% Problem in Agentic Coding](https://addyo.substack.com/p/the-80-problem-in-agentic-coding)
- Addy Osmani: [Agentic Autonomy Levels](https://addyosmani.com/blog/agentic-autonomy-levels/)
- Google Agents Whitepaper Series: Introduction to Agents, Agent Quality, Prototype to Production
- Model Context Protocol: <https://modelcontextprotocol.io/>
- Google Agent Development Kit: <https://google.github.io/adk-docs/>

## 9. 一句话总结

**这份白皮书的核心不是鼓励大家“随便 vibe 一下就上线”，而是提醒我们：AI 已经解决了大量生成问题，接下来真正稀缺的工程能力是定义意图、组织上下文、搭建 harness、验证输出，以及在正确边界内编排 Agent。**
