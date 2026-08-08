---
layout: post
title: "Matt Pocock Skills 体系解读：面向 AI Agent 的工程纪律"
description: "对 mattpocock-skills 中 engineering 与 productivity 技能的结构化解读，重点分析流程、约束、协作边界与 Agentic Software Engineering 方法论。"
date: 2026-08-08
created_at: 2026-08-08 10:27:29 +0800
categories: AI
---

# Matt Pocock Skills 体系解读：面向 AI Agent 的工程纪律

> 解读对象：`mattpocock-skills` 项目中的 `engineering` 与 `productivity` 技能  
> 解读重点：流程、约束、协作边界、适用场景  
> 解读时间：2026-08-08

## 1. 总体判断

这套 skills 不是一组普通 prompt，而是一套面向 AI Agent 的工程操作系统。它的目标不是让 AI “更快写代码”，而是给 AI 加上工程纪律：什么时候澄清、什么时候研究、什么时候原型验证、什么时候拆票、什么时候实现、什么时候审查、什么时候交接。

如果用一句话概括：

> 这套技能把 AI 从“代码生成器”约束成一个可以参与长期工程协作的执行者。

它最核心的价值有三点：

1. **阶段边界清楚**：想法、规格、票据、实现、审查、交接，各阶段职责明确。
2. **模糊性显式建模**：用 frontier、fog of war、blocking edges、decision tickets 管理未知。
3. **人类判断保留在关键节点**：Agent 负责查事实、构造选项、验证证据；用户负责真正的决策。

---

## 2. 技能体系总览

这套 skills 大致分为两组：

### Engineering 技能

面向软件工程流程，包括：

- 技能路由：`ask-matt`
- 仓库初始化：`setup-matt-pocock-skills`
- 需求澄清：`grill-with-docs`、`grilling`
- 领域建模：`domain-modeling`
- 架构设计：`codebase-design`、`improve-codebase-architecture`
- Debug：`diagnosing-bugs`
- TDD 与实现：`tdd`、`implement`
- 代码审查：`code-review`
- 规格与票据：`to-spec`、`to-tickets`
- Issue 管理：`triage`
- 大型模糊项目规划：`wayfinder`
- 原型验证：`prototype`
- 研究：`research`
- 合并冲突：`resolving-merge-conflicts`
- 人类操作向导：`wizard`

### Productivity 技能

面向协作、学习、交接和文档，包括：

- 无状态访谈：`grill-me`
- 上下文交接：`handoff`
- 教学工作区：`teach`
- 问卷生成：`to-questionnaire`
- 重新解释：`wait-what`
- Agent 文档写作：`writing-for-agents`

---

## 3. 总流程：idea → ship

`ask-matt` 是整个体系的路由器。它定义了从想法到交付的主路径：

```text
idea
  ↓
grill-with-docs / grill-me
  ↓
如果需要可运行验证 → prototype
  ↓
如果是多 session 任务 → to-spec → to-tickets
  ↓
implement
  ↓
tdd
  ↓
code-review
  ↓
commit / ship
```

这个流程背后的核心判断是：

- 模糊想法不能直接实现；
- 需要先通过访谈澄清；
- 如果纸面讨论不够，就用 prototype 回答问题；
- 多 session 任务要先规格化、再拆票；
- 实现阶段必须有测试和审查。

这和很多 AI coding workflow 的区别在于：它不是“用户说一句，Agent 写一堆代码”，而是先建立清晰的认知路径。

---

## 4. 关键技能解读

## 4.1 `ask-matt`：技能路由器

`ask-matt` 不是解决具体问题的技能，而是回答“现在该走哪条流程”。

它把任务分成几种入口：

- 普通想法：进入 `grill-with-docs` 或 `grill-me`；
- Bug：进入 `diagnosing-bugs`；
- 外部 issue / PR：进入 `triage`；
- 巨大模糊项目：进入 `wayfinder`；
- 代码健康维护：进入 `improve-codebase-architecture`；
- 人类必须操作的步骤：进入 `wizard`。

关键约束：

- `to-tickets` 之前尽量保持一个完整上下文，不要过早清空或压缩；
- `/to-tickets` 产生的票据已经是 agent-ready，不要再 triage；
- `wayfinder` 用于巨大且模糊的工作，不要滥用；
- `implement` 后要运行 `code-review`。

`ask-matt` 的价值是建立一张“技能地图”，避免 Agent 在错误阶段做正确动作。

---

## 4.2 `setup-matt-pocock-skills`：仓库级初始化

很多工程技能都依赖仓库里有统一配置，例如：

- issue tracker 在哪里；
- triage 标签叫什么；
- domain docs 放哪里；
- ADR 放哪里；
- `AGENTS.md` 或 `CLAUDE.md` 如何指向这些约定。

`setup-matt-pocock-skills` 就是把一个普通 repo 初始化成“适合 Agent 协作的 repo”。

它会先探索：

- git remote；
- `AGENTS.md` / `CLAUDE.md`；
- `CONTEXT.md` / `CONTEXT-MAP.md`；
- `docs/adr/`；
- `.scratch/`；
- 是否 monorepo；
- 是否已有 `docs/agents/`。

然后再让用户确认：

- 使用 GitHub、GitLab、本地 Markdown，还是其他 issue tracker；
- 是否使用默认 triage labels；
- 使用单上下文还是多上下文 domain docs。

关键约束：

- 先探索，不要假设；
- 已有 `CLAUDE.md` 就编辑它，否则才考虑 `AGENTS.md`；
- 不要重复添加 `## Agent skills`；
- 写入前展示草案并让用户确认。

---

## 4.3 `grilling` / `grill-me` / `grill-with-docs`：需求澄清机制

`grilling` 是底层访谈原语。它不是随意提问，而是把计划或设计展开成一棵 decision tree。

核心概念是 **frontier**：当前可以问、且不依赖未解决前置问题的问题集合。

每一轮：

1. 只问当前 frontier 上的问题；
2. 每个问题给推荐答案；
3. 等用户回答；
4. 根据回答重新计算 frontier；
5. frontier 为空时，说明共享理解已经建立。

关键约束：

- 事实由 Agent 查询，不要问用户可查事实；
- 决策由用户做，Agent 不替用户拍板；
- 依赖未解决问题的后续问题要留到下一轮；
- 用户确认 shared understanding 前，不要执行。

`grill-me` 是无状态包装器，适合没有工作目录的对话。  
`grill-with-docs` 是状态化版本，适合 repo 中的项目，会把讨论结果沉淀到 `CONTEXT.md`、ADR 等文档。

---

## 4.4 `domain-modeling`：统一语言与领域模型

`domain-modeling` 维护项目中的统一语言。它不是简单读取 `CONTEXT.md`，而是在设计过程中主动挑战和修正术语。

典型动作包括：

- 用户使用和 glossary 冲突的词时立即指出；
- 用户使用模糊词时要求精确定义；
- 用具体边界场景压力测试概念；
- 将用户说法和代码行为交叉验证；
- 术语确定后立即更新 `CONTEXT.md`；
- 只有必要时才创建 ADR。

`CONTEXT.md` 的边界非常严格：

- 只放 glossary；
- 不放实现细节；
- 不当作 spec；
- 不当作 scratchpad；
- 不记录实现决策。

ADR 只有同时满足三个条件才创建：

1. 难以反转；
2. 没有上下文会显得意外；
3. 是真实 trade-off 的结果。

这体现了一个重要原则：术语不清，架构必乱。

---

## 4.5 `codebase-design`：deep module 设计语言

`codebase-design` 提供一套架构词汇，核心是 **deep module**：小接口、深实现、清晰 seam、可测试。

关键概念：

- **Module**：任何有 interface 和 implementation 的东西；
- **Interface**：调用者必须知道的一切，不只是类型签名；
- **Implementation**：模块内部实现；
- **Depth**：调用者理解少量 interface 后能获得多少行为；
- **Seam**：可以替换行为而不修改当前位置的地方；
- **Adapter**：在 seam 上满足 interface 的具体实现；
- **Leverage**：调用者得到的能力增益；
- **Locality**：维护者得到的变更集中性。

重要原则：

- Depth 是 interface 的属性，不是实现行数；
- 删除模块后复杂度扩散，说明模块有价值；
- interface 是测试面；
- 一个 adapter 只是 hypothetical seam，两个 adapter 才说明 seam 真实存在；
- 不要滥用 boundary，优先说 seam 或 interface。

这套语言反对浅层封装、无意义 service、过早抽象和为了测试而拆碎代码。

---

## 4.6 `improve-codebase-architecture`：架构健康巡检

这个技能用于扫描代码库，寻找可以把浅模块变深的机会。

流程：

1. 先限定扫描范围；
2. 如果用户没有指定范围，就看 git history 找热点；
3. 读取 `CONTEXT.md` 和相关 ADR；
4. 使用 `codebase-design` 的词汇；
5. 派 sub-agent 探索代码库；
6. 找理解摩擦、浅模块、测试困难、seam 泄漏等问题；
7. 输出 HTML 报告；
8. 用户选中候选后，再进入 `grilling` 讨论。

报告要求：

- 写到临时目录，不放进 repo；
- 使用 Tailwind 和 Mermaid；
- 每个候选有 before / after 可视化；
- 包含 files、problem、solution、benefits、recommendation strength；
- 最后给 top recommendation。

关键约束：

- 先 scope 再 scan，避免 YAGNI；
- 不要一开始就设计 interface；
- 如果候选违反 ADR，只有 friction 足够真实时才提出。

---

## 4.7 `diagnosing-bugs`：严格调试闭环

这是整套 skills 中工程纪律最强的技能之一。

核心原则：

> No red-capable command, no hypothesis.

也就是说，没有能捕捉用户具体 bug 的反馈循环，就不能开始猜原因。

完整流程：

1. **建立 feedback loop**  
   可以是 failing test、curl 脚本、CLI fixture、headless browser、captured trace replay、throwaway harness、fuzz loop、bisect harness 等。

2. **收紧 loop**  
   让它更快、更确定、更精确。

3. **复现并最小化**  
   确认是用户描述的 bug，并缩小到每个剩余元素都 load-bearing。

4. **提出 3–5 个可证伪假设**  
   每个假设都要有预测。

5. **插桩验证**  
   每个 probe 对应一个假设；一次只改一个变量；debug log 必须带唯一前缀。

6. **先写 regression test，再修复**  
   但测试必须在正确 seam 上。

7. **清理与复盘**  
   重跑原始 loop，删除 debug instrumentation，记录真正原因，并思考架构上如何预防。

这个技能专门防止 AI 常见失败：看到错误后直接读代码、猜原因、改一处、祈祷通过。

---

## 4.8 `tdd` 与 `implement`：测试驱动实现

`tdd` 强调测试行为，而不是实现细节。测试要写在 seam 上，也就是可观察行为的公共边界。

关键约束：

- 写测试前先确认 seam；
- 没有确认 seam，不写测试；
- 一个 cycle 只做一个 seam、一个 test、一个最小实现；
- red before green；
- 不提前添加 speculative feature；
- 重构不属于红绿循环，而属于 review 阶段。

反模式：

- 测私有方法或内部协作者；
- 用实现逻辑重复计算 expected value；
- 水平切片：先写所有测试，再写所有实现。

`implement` 是实现包装器：

1. 根据 spec 或 ticket 实现；
2. 尽量使用 `tdd`；
3. 定期运行 typecheck 和单测；
4. 最后跑完整测试；
5. 用 `code-review` 审查；
6. commit 到当前分支。

它定义了“完成”的标准：不是代码写了，而是测试、审查、提交都完成。

---

## 4.9 `code-review`：双轴审查

`code-review` 把审查分成两个互不污染的轴：

1. **Standards**：代码是否符合 repo 标准和基本 code smell 判断；
2. **Spec**：代码是否实现了原始 issue / spec。

流程：

1. 用户指定 fixed point，例如 `main`、commit、tag、`HEAD~5`；
2. 使用 `git diff <fixed-point>...HEAD`；
3. 验证 ref 可解析且 diff 非空；
4. 查找 spec 来源；
5. 查找 coding standards 来源；
6. 派两个 sub-agent 并行审查；
7. 分别输出 `## Standards` 和 `## Spec`；
8. 不合并、不重排。

内置 smell baseline 包括：

- Mysterious Name
- Duplicated Code
- Feature Envy
- Data Clumps
- Primitive Obsession
- Repeated Switches
- Shotgun Surgery
- Divergent Change
- Speculative Generality
- Message Chains
- Middle Man
- Refused Bequest

关键点是 Standards 和 Spec 分离：

- 代码干净但做错需求，是 Spec fail；
- 需求做对但代码质量差，是 Standards fail。

这避免两个维度互相遮蔽。

---

## 4.10 `to-spec` 与 `to-tickets`：从讨论到可执行工作图

`to-spec` 把已经讨论清楚的内容整理成 spec。它明确要求：

> Do NOT interview the user — just synthesize what you already know.

也就是说，它不负责澄清需求，而是把已经澄清的内容固化。

Spec 结构包括：

- Problem Statement
- Solution
- User Stories
- Implementation Decisions
- Testing Decisions
- Out of Scope
- Further Notes

关键约束：

- 不要写容易过时的具体 file path；
- prototype 中高度决策化的 snippet 可以例外内嵌；
- 需要确认测试 seam。

`to-tickets` 把 spec / plan 拆成 tracer-bullet tickets。

每个 ticket 应该：

- 是 vertical slice；
- 打通一条窄而完整的端到端路径；
- 完成后可独立验证；
- 适合一个新上下文完成；
- blockers 明确。

对于 wide refactor，不能强行 vertical slicing，而要使用 expand–contract：

1. expand：新旧形式并存；
2. migrate：分批迁移调用点；
3. contract：删除旧形式。

---

## 4.11 `triage`：issue / PR 状态机

`triage` 用于处理外部进入的 issue 和 PR。

类别角色：

- `bug`
- `enhancement`

状态角色：

- `needs-triage`
- `needs-info`
- `ready-for-agent`
- `ready-for-human`
- `wontfix`

每个 issue 应该有且只有一个 category role 和一个 state role。

处理流程：

1. 展示 unlabeled、needs-triage、needs-info 且 reporter 有新回复的项目；
2. 读取完整 issue / PR，包括 comments、labels、author、dates、PR diff；
3. 查 prior triage notes，避免重复问；
4. 探索代码库；
5. 检查是否已经实现；
6. 检查 `.out-of-scope/` 是否有相似拒绝记录；
7. 推荐 category 和 state；
8. 对 bug 复现，对 PR checkout 并运行测试；
9. 必要时进入 `grilling` 和 `domain-modeling`；
10. 写 agent brief、needs-info notes 或 wontfix 说明。

硬约束：所有 triage 评论必须带免责声明：

```markdown
> *This was generated by AI during triage.*
```

这个技能的价值是把 issue 处理变成可审计状态机，而不是随意贴标签。

---

## 4.12 `wayfinder`：大型模糊项目的决策地图

`wayfinder` 用于一个 session 容纳不了、目标模糊、路径未知的大型工作。

它默认是 planning tool，不是 implementation tool。

核心对象：

- **Destination**：地图要抵达的目标；
- **Map**：一个 `wayfinder:map` issue；
- **Decision tickets**：每个 ticket 解决一个决策；
- **Frontier**：当前未阻塞、未认领、可处理的 tickets；
- **Fog of war**：知道未来可能有问题，但还无法精确表达；
- **Out of scope**：明确不属于当前 destination 的工作。

地图结构：

```markdown
## Destination
## Notes
## Decisions so far
## Not yet specified
## Out of scope
```

Ticket 类型：

- Research：AFK，一手来源研究；
- Prototype：HITL，用 artifact 提高讨论保真度；
- Grilling：HITL，默认决策访谈；
- Task：HITL 或 AFK，用行动解锁一个决策。

关键约束：

- 默认只计划，不实现；
- 每个 session 不解决超过一个 ticket，research 例外；
- HITL ticket 必须和人类一起解决，Agent 不能自问自答；
- Map 是索引，不是细节仓库；
- 决策详情存在 ticket resolution 中；
- 不要把 fog 过早切成 tickets；只有问题能精确表述时才 ticket。

`wayfinder` 的核心价值是避免两种失败：过早执行，以及过早拆票。

---

## 4.13 `prototype`：用一次性代码回答设计问题

`prototype` 不是 MVP，而是为了回答一个明确问题。

两种分支：

1. **Logic / state model**  
   用单个 HTML 文件模拟状态机或逻辑，帮助人类判断“这个模型是否合理”。

2. **UI**  
   在一个 route 上生成多个 UI variations，通过 URL 参数或浮动栏切换。

共同约束：

- 从第一天就是 throwaway；
- 位置靠近真实使用区域，但命名明确；
- 启动必须简单；
- 默认无持久化；
- 不写测试；
- 不追求 polish；
- 必须展示完整相关状态；
- 验证后的决策折回真实代码；
- prototype 本身保存在 throwaway branch，作为 primary source。

这个技能允许快速试错，但要求把试错结果沉淀，避免知识丢失。

---

## 4.14 `research`：后台研究与一手来源

`research` 用于把阅读工作交给 background agent。

要求：

- 查官方文档、源码、标准、first-party APIs、specs；
- 每个 claim 回到一手来源；
- 输出一个 Markdown 文件；
- 保存到 repo 既有 notes 位置，没有惯例则放到合理位置并说明。

它把研究从“聊天里的回答”变成 repo artifact。

---

## 4.15 `resolving-merge-conflicts`：按意图解决冲突

这个技能用于已经处于 merge / rebase conflict 的状态。

流程：

1. 查看当前 merge / rebase 状态；
2. 查看冲突文件和 git history；
3. 为每个冲突找到 primary sources：commit、PR、issue；
4. 理解双方修改意图；
5. 逐个 hunk 解决，尽量保留双方意图；
6. 不兼容时按 merge 目标选择并记录 trade-off；
7. 运行 typecheck、tests、format；
8. 完成 merge / rebase。

硬约束：永远不要 `--abort`，也不要简单选择 ours / theirs。

它强调 merge conflict 本质上是意图冲突，不是文本冲突。

---

## 4.16 `wizard`：人类步骤脚本化

`wizard` 用于 Agent 无法替人完成的步骤，例如：

- 创建账号；
- 点击第三方 dashboard；
- 获取 API key；
- 设置 CI secrets；
- 执行 cutover。

它生成一个交互式 bash wizard，指导人类一步步操作。

流程：

1. 探索 `.env`、README、docker-compose、workflow；
2. 找出需要哪些值；
3. 明确每个值从哪里获得、写到哪里、是否 secret；
4. 为每个 stage 写清楚 URL、点击路径、复制内容；
5. 基于 `template.sh` 编写 wizard；
6. 使用 `ask_secret`、`write_env`、`set_secret` 等 helper；
7. 用 `bash -n` 和 shellcheck 验证；
8. 不由 Agent 端到端运行，因为它需要人类输入。

关键原则：如果 Agent 能自己做，就不要用 wizard。wizard 专门用于 human-in-the-loop 步骤。

---

## 5. Productivity 技能解读

## 5.1 `handoff`：上下文交接文档

`handoff` 把当前对话压缩成一个文档，让新 session、另一个 agent 或同事接手。

要求：

- 保存到 OS 临时目录；
- 包含 suggested skills；
- 不重复已有 artifact；
- 对 spec、plan、ADR、issue、commit、diff 只引用路径或 URL；
- redacts secrets / PII；
- 根据用户参数定制下一 session 的焦点。

它适合上下文边界：新目录、新 harness、prototype 往返、同事接手、side task。

---

## 5.2 `teach`：状态化教学工作区

`teach` 把当前目录变成长期学习空间。

工作区结构：

- `MISSION.md`：为什么学；
- `RESOURCES.md`：高质量资源；
- `reference/*.html`：参考材料；
- `learning-records/*.md`：学习记录；
- `lessons/*.html`：每节课；
- `assets/*`：可复用组件；
- `NOTES.md`：偏好和草稿。

教学理念：

- 不只讲知识，还要训练技能和积累智慧；
- 不依赖模型参数知识，要查高质量资源；
- 每节课围绕 mission；
- 每节课在用户的 zone of proximal development；
- 用 retrieval practice、spacing、interleaving 增强长期记忆。

这个技能把 Agent 从“一次性讲解者”变成长期课程设计者。

---

## 5.3 `to-questionnaire`：把未知交给知道的人

当用户不能独自回答某个问题，而答案在另一个人那里时，`to-questionnaire` 生成一份 Markdown 问卷。

核心原则：

> Grill the send, not the subject.

也就是说，不问用户问题本身的答案，而问：

- 问卷发给谁；
- 对方知道什么；
- 用户需要拿回哪些事实或决策。

问卷结构包括：

- Purpose；
- From / To；
- How answers will be used；
- Context；
- How to answer；
- 分主题问题；
- Anything else。

它适合异步协作，尤其是需要外部专家输入时。

---

## 5.4 `wait-what`：重新解释没讲清楚的内容

`wait-what` 用于用户表示“刚才没听懂”时。

要求：

- 停止推进；
- 重新解释上一条；
- 给更多上下文；
- 使用更简单、更明确的语言；
- 使用 `CONTEXT.md` 中的统一语言。

这是一个对话质量修复器。

---

## 5.5 `writing-for-agents`：Agent 文档写作方法论

这是整套 skills 背后的元技能，专门讲如何写给 Agent 消费的文档，例如 skill、`AGENTS.md`、`CLAUDE.md`。

核心概念：

### Context pointer

Skill description 或 `AGENTS.md` 中的一行，本质上都是 context pointer。它的 wording 决定 Agent 是否会加载目标材料。

原则：

- front-load leading word；
- 一个 branch 一个 trigger；
- 删除 body 已经承载的 identity。

### 两种负载

- **Context load**：总在上下文中的 token 和注意力成本；
- **Cognitive load**：人类知道哪个文档何时使用的成本。

### 信息层级

文档内容分三层：

1. In-file step；
2. In-file reference；
3. Disclosed reference。

必须执行的步骤应放主文件，分支参考可以下沉。

### Completion criteria

每个步骤都应有清晰、可检查、足够 demanding 的完成标准。模糊标准会导致 premature completion。

### Leading words

使用模型预训练中已有的概念来锚定行为，例如：

- tight loop；
- red；
- tracer bullet；
- fog of war。

### 避免否定式指令

否定会激活被禁止的行为。应该尽量写正向目标。

### Pruning

删掉重复、过时、可从环境查询的缓存、不改变模型行为的 no-op，以及沉积层。

这个技能把 prompt / skill 写作当成一种软件架构问题。

---

## 6. 这套体系解决的 Agent 失败模式

| Agent 常见失败 | 对应技能约束 |
|---|---|
| 需求没问清就写代码 | `grilling`, `grill-with-docs` |
| 大项目直接开干 | `wayfinder` |
| Issue 太模糊 | `triage`, `to-spec` |
| 拆票按层拆，无法独立验证 | `to-tickets` |
| 测试实现细节 | `tdd`, `codebase-design` |
| Debug 靠猜 | `diagnosing-bugs` |
| Review 只看代码，不看需求 | `code-review` |
| 架构抽象过浅或过早 | `codebase-design`, `improve-codebase-architecture` |
| 领域词混乱 | `domain-modeling` |
| 人类操作步骤反复解释 | `wizard` |
| 上下文断裂 | `handoff` |
| Skills 越写越臃肿 | `writing-for-agents` |

---

## 7. 可以借鉴到自己的 Agent 工作流中的原则

我认为最值得迁移的原则有 12 条：

1. **先澄清，再实现。**  
   模糊需求不能直接 coding。

2. **事实 Agent 查，决策用户定。**  
   不把可查事实丢给用户，也不替用户拍板。

3. **长期项目要留下纸面记录。**  
   `CONTEXT.md`、ADR、issue、spec、ticket、research note 都是工程资产。

4. **领域语言优先。**  
   术语不清，架构必乱。

5. **大任务先地图化。**  
   对巨大模糊任务，先处理 fog，不要直接拆实现票。

6. **实现要 vertical slice。**  
   每个 ticket 要有可验证的 end-to-end 行为。

7. **测试必须在正确 seam 上。**  
   不测内部实现，不写假安全感测试。

8. **Debug 先建立 red-capable loop。**  
   没有可复现反馈循环，不准猜原因。

9. **Review 分 Standards 和 Spec 两轴。**  
   不让代码质量和需求符合度互相遮蔽。

10. **Prototype 是回答问题，不是生产雏形。**  
    产物可丢，决策要沉淀。

11. **人类步骤要 wizard 化。**  
    不可自动化步骤也可以流程化、脚本化。

12. **Agent 文档要像代码一样维护。**  
    去重、分层、明确触发、删除 sediment。

---

## 8. 最终评价

这套 skills 的高级之处在于，它不是“让 AI 更聪明”的技巧，而是通过流程、文档、测试、票据和审查，让 AI 的行为更可控、更可验证、更可交接。

它把软件工程里的几个重要原则迁移到了 Agent 协作中：

- 明确阶段；
- 管理未知；
- 记录决策；
- 用测试验证行为；
- 用 review 防止偏差；
- 用 issue 和 handoff 管理上下文边界；
- 用领域语言降低沟通成本。

因此，这套技能更像是一种 **Agentic Software Engineering Process**，而不是 prompt collection。

对个人开发者，它可以减少 AI coding 的随机性。  
对团队，它可以提高 Agent 参与工程协作的可审计性。  
对长期项目，它可以把一次性对话沉淀成持续演化的工程资产。

一句话总结：

> Matt Pocock Skills 的核心价值，是把 AI Agent 放进一套有边界、有证据、有验证、有交接的工程系统里，而不是让它在一个无限上下文窗口里自由发挥。
