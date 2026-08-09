---
layout: post
title: "Superpowers 解读：面向 Coding Agents 的软件工程纪律"
description: "对 obra/superpowers 项目及其 14 个 skills 的结构化解读，重点分析从 brainstorm、worktree、plan、TDD、subagent 到 code review 的 Agentic Software Engineering 流程与约束。"
date: 2026-08-08
created_at: 2026-08-10 06:54:28 +0800
categories: AI Agents
---

# Superpowers 解读：面向 Coding Agents 的软件工程纪律

本文是对 **obra/superpowers** 项目及其全部 14 个 skills 的结构化解读。

> 仓库：`obra/superpowers`
> 本地读取版本：`package.json` 显示 `6.2.0`
> 最新读取 commit：`44c9b2d docs: remove the "We're Hiring" section from the README`
> 项目定位：**面向 coding agents 的完整软件开发方法论 + 可组合 skills 库 + 多 Agent harness 插件适配层**

---

## 1. 项目一句话定位

**Superpowers 是一套给 AI coding agents 使用的软件开发方法论。**

它不是单纯的 prompt collection，也不是单个编码工具，而是：

- 一组强制触发的工程 skills；
- 一套从想法到交付的软件开发流程；
- 一套适配 Claude Code、Codex、Cursor、Gemini CLI、OpenCode、Kimi、Pi 等 harness 的插件系统；
- 一套明确反对“Agent 随手写代码”的工程纪律。

它的核心目标是让 AI coding agent：

1. 不要一上来就写代码；
2. 先澄清需求；
3. 写设计文档；
4. 写实施计划；
5. 使用 TDD；
6. 使用子 Agent 分任务实现；
7. 每个任务后做 code review；
8. 完成前用证据验证；
9. 最后再合并、PR 或保留分支。

---

## 2. 项目方法论总览

README 中给出的基础工作流是：

```text
brainstorming
  ↓
using-git-worktrees
  ↓
writing-plans
  ↓
subagent-driven-development / executing-plans
  ↓
test-driven-development
  ↓
requesting-code-review
  ↓
finishing-a-development-branch
```

可以把它理解成一条强约束的软件工程流水线：

1. **Brainstorming**
   先把想法打磨成设计，不能直接实现。

2. **Git Worktree**
   在隔离工作区里开发，避免污染当前分支。

3. **Writing Plans**
   把设计拆成小任务，每个任务都可测试、可审查。

4. **Subagent-Driven Development / Executing Plans**
   用子 Agent 或当前 session 执行计划。

5. **TDD**
   先写失败测试，再写最小实现。

6. **Code Review**
   每个任务后审查，防止偏离计划和代码质量下降。

7. **Verification Before Completion**
   完成前必须有新鲜验证证据。

8. **Finishing Branch**
   所有测试通过后，让用户决定 merge、PR、保留分支等。

---

## 3. 版本 v6.2.0 的重要变化

从 `RELEASE-NOTES.md` 看，v6.2.0 重点在两个方向：

### 3.1 Subagent-Driven Development 改进

v6.2.0 修正了 SDD 的任务进度与 review 修复循环：

- `.superpowers/sdd/` 改成 **plan-scoped workspace**：
  - 以前不同 plan 可能共享同一个 ledger；
  - 现在变成 `.superpowers/sdd/<plan-basename>/`；
  - 避免不同计划的状态污染。

- Review-fix loop 改成 **resume implementer**：
  - 修复 review 问题时，不再总是新派一个 agent；
  - 而是恢复原 implementer 的上下文语义；
  - 同时增加 re-review prompt；
  - 设置五轮 circuit breaker，避免无限修复循环。

### 3.2 Skills 压缩

v6.2.0 对多个 skills 做了压缩：

- 删除 recap、selling prose、过度说服性文字；
- 把必要论证移动到 rationalization table；
- 把 `testing-anti-patterns.md` 改为正向的 `writing-good-tests.md`；
- 强化 TDD 中对“测试必须可证伪”的要求；
- 删除 `finishing-a-development-branch` 中“丢弃工作”的默认选项，只保留显式请求路径。

这说明项目很重视 **skills 的 token 成本与行为塑形效果**，不是越长越好。

---

## 4. 全部 skills 总览

仓库当前有 14 个 skills：

| Skill | 核心用途 |
|---|---|
| `using-superpowers` | 启动规则：任何任务前必须检查并调用相关 skill |
| `brainstorming` | 实现前先澄清想法并形成设计 |
| `writing-plans` | 把设计写成可执行实施计划 |
| `using-git-worktrees` | 创建或确认隔离工作区 |
| `executing-plans` | 在无子 Agent 情况下按计划执行 |
| `subagent-driven-development` | 用子 Agent 按任务实现、审查、修复 |
| `test-driven-development` | 强制 RED-GREEN-REFACTOR |
| `requesting-code-review` | 完成任务后请求代码审查 |
| `receiving-code-review` | 严谨处理审查反馈 |
| `verification-before-completion` | 声称完成前必须运行验证 |
| `finishing-a-development-branch` | 完成分支、合并/PR/保留/清理 |
| `systematic-debugging` | Bug 前先根因调查，不猜修 |
| `dispatching-parallel-agents` | 多个独立问题并行派 Agent |
| `writing-skills` | 用 TDD 方法编写和测试 skills |

---

# 5. Skill 逐个解读

---

## 5.1 `using-superpowers`

**定位：**
这是整个项目的 bootstrap skill。它定义了最根本的规则：**只要有 1% 可能某个 skill 适用，就必须先调用 skill。**

### 核心规则

> Invoke relevant or requested skills BEFORE any response or action.

也就是说，在做任何事情之前，包括：

- 回答问题；
- 提澄清问题；
- 读文件；
- 查代码；
- 改代码；
- 执行命令；

都要先检查是否有相关 skill。

### 关键约束

它列出一堆常见“逃避 skill”的想法：

| 想法 | 实际问题 |
|---|---|
| “只是简单问题” | 问题也是任务，也要检查 skill |
| “我先看下代码” | skill 决定你该怎么探索 |
| “我需要更多上下文” | skill 检查在澄清问题之前 |
| “这个 skill 太重了” | 简单事常常变复杂 |
| “我记得这个 skill” | skill 会变，必须读当前版本 |

### 特殊规则

如果当前 Agent 是被派发出来的 subagent，且只是执行具体任务，就忽略这个 bootstrap，避免子 Agent 递归启动整套流程。

### 我的理解

`using-superpowers` 是整个系统的“宪法”。它的作用是阻止 Agent 最常见的坏习惯：凭感觉直接行动。

---

## 5.2 `brainstorming`

**定位：**
任何创造性工作、功能开发、组件构建、行为修改之前都必须使用。

### 硬门槛

在用户批准设计前，不能：

- 写代码；
- scaffold 项目；
- 调用实现 skill；
- 做任何 implementation action。

它特别反对一句话：

> “这个太简单，不需要设计。”

哪怕是 todo list、单函数 utility、配置改动，也必须经过设计，只是设计可以很短。

### 流程

必须按顺序完成：

1. 探索项目上下文：
   - 文件；
   - 文档；
   - 最近 commit。

2. 适时提供 visual companion：
   - 不是一开始就提供；
   - 只有当问题用视觉表达更清楚时才单独发起；
   - 用户接受后打开浏览器 companion。

3. 一次问一个澄清问题：
   - 理解目的；
   - 约束；
   - 成功标准。

4. 提出 2–3 个方案：
   - 说明 trade-off；
   - 给推荐方案。

5. 分段展示设计：
   - 每段复杂度适配；
   - 用户逐段批准。

6. 写设计文档：
   - 保存到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`；
   - commit。

7. Spec self-review：
   - 检查 placeholder；
   - 矛盾；
   - 模糊点；
   - scope。

8. 用户 review written spec。

9. 进入 `writing-plans`。

### 我的理解

`brainstorming` 的核心不是“聊天 brainstorm”，而是 **实现前的设计冻结机制**。它强迫 Agent 把隐性需求显性化，并通过用户批准建立边界。

---

## 5.3 `writing-plans`

**定位：**
当已有 spec 或需求后，在动代码前写实施计划。

### 计划的目标读者

它假设执行计划的人：

- 是熟练开发者；
- 但完全不了解当前代码库；
- 工程品味一般；
- 不擅长测试；
- 容易偷懒。

因此计划必须写得非常具体。

### 默认保存位置

```text
docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md
```

用户偏好可以覆盖。

### 核心内容

计划必须包括：

1. 文件结构映射：
   - 哪些文件创建；
   - 哪些文件修改；
   - 每个文件负责什么。

2. Global Constraints：
   - 所有任务都必须遵守的规则；
   - 版本限制；
   - 依赖限制；
   - 命名；
   - 精确值；
   - copy。

3. 任务结构：
   - 每个任务都 bite-sized；
   - 每个任务有独立可测试交付物；
   - 每个任务有接口说明；
   - 每个任务有测试步骤；
   - 每个任务能被 reviewer 单独判断。

4. 无占位符：
   - 不能写 “implement X”；
   - 不能写 “add appropriate tests”；
   - 要给足上下文和验证方式。

### 重要思想

一个任务的边界不是“工作量最小”，而是：

> 这个任务是否值得一个独立测试循环和独立 reviewer gate。

### 我的理解

`writing-plans` 是把“设计”转换成“可以交给低上下文 Agent 执行的任务说明”。它非常适合多 Agent 协作，因为每个任务都要自包含。

---

## 5.4 `using-git-worktrees`

**定位：**
开始 feature work 或执行计划前，确保工作发生在隔离 workspace 中。

### 核心原则

> Detect existing isolation first. Then use native tools. Then fall back to git.

流程：

1. 先检查是否已经在 worktree 中：

```bash
git rev-parse --git-dir
git rev-parse --git-common-dir
git branch --show-current
```

2. 防止把 submodule 误判为 worktree：

```bash
git rev-parse --show-superproject-working-tree
```

3. 如果已经在 worktree 中：
   - 不再创建新的；
   - 直接进入项目 setup。

4. 如果不在 worktree：
   - 先询问用户是否要创建；
   - 用户拒绝则在原地工作。

5. 创建方式优先级：
   - harness 原生 worktree tool；
   - 手动 `git worktree add`。

### 项目 setup

根据项目类型运行安装：

- Node.js；
- Rust；
- Python；
- Go。

### clean baseline

创建后要运行项目测试或合适验证，确认新工作区基线是干净的。

### 我的理解

这个 skill 的重点不是 git 命令，而是 **不要污染当前工作区**，同时尊重 harness 自己的 worktree 管理能力。

---

## 5.5 `executing-plans`

**定位：**
当有书面 implementation plan，但当前环境没有合适 subagent 能力，或者要在单 session 中执行时使用。

### 流程

1. 加载并审查 plan：
   - 先确保隔离 workspace；
   - 读取 plan；
   - 批判性审查；
   - 有问题先问用户。

2. 执行任务：
   - 每个 task 标记 in_progress；
   - 严格按 plan 步骤执行；
   - 按计划运行 verification；
   - 完成后标记 completed。

3. 完成开发：
   - 调用 `finishing-a-development-branch`。

### 停止条件

必须停止并问用户：

- 缺依赖；
- 测试失败；
- 指令不清；
- 反复验证失败；
- plan 有关键缺口。

### 我的理解

这是没有 subagent 时的 fallback。Superpowers 明确说，如果有子 Agent 能力，优先用 `subagent-driven-development`。

---

## 5.6 `subagent-driven-development`

**定位：**
这是 Superpowers 的核心高级流程：用 fresh subagent 按计划逐任务实现，并在每个任务后 review。

### 核心原则

> Fresh subagent per task + task review + final broad review.

它强调：

- 每个任务一个新 subagent；
- subagent 不继承当前 session 历史；
- controller 精确构造任务上下文；
- 每个任务后有 review；
- 最后做整个分支的 broad review。

### 连续执行规则

它明确要求：

> Do not pause to check in with your human partner between tasks.

除非：

- BLOCKED；
- 有阻止继续的真实歧义；
- 所有任务完成。

不要反复问“要继续吗”。

### 工作区

v6.2.0 后，SDD workspace 是 plan-scoped：

```text
.superpowers/sdd/<plan-basename>/
```

里面存：

- task brief；
- implementer report；
- review package；
- progress ledger。

### 每任务流程

1. Dispatch implementer：
   - 给子 Agent 一个任务；
   - 明确 plan；
   - 明确 task；
   - 明确验证要求。

2. Handle report：
   - implementer 报告完成、阻塞或失败；
   - controller 读取报告和 diff。

3. Review task：
   - review 同时检查 spec compliance 和 code quality；
   - reviewer 只读，不改工作树；
   - reviewer 必须给文件/行证据。

4. Fix loop：
   - 有 load-bearing finding 时，恢复 implementer 修复；
   - re-review 只检查修复；
   - 五轮 circuit breaker；
   - 超过则 controller adjudication。

5. Complete task：
   - ledger 记录任务完成；
   - 继续下一个任务。

6. Final review：
   - 所有任务后做整个分支 review。

### 我的理解

这是 Superpowers 最像“工程管理系统”的 skill。controller 像 tech lead / EM，implementer 像开发者，reviewer 像 code reviewer。它通过文件而不是粘贴上下文传递任务，降低 token 成本和上下文污染。

---

## 5.7 `test-driven-development`

**定位：**
任何 feature、bugfix、refactoring、行为变更前必须用。

### 铁律

```text
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

如果先写了生产代码：

> Delete it. Start over.

并且不能把先写的代码当参考、不能边看边改、不能保留。

### RED-GREEN-REFACTOR

1. RED：
   - 先写 failing test；
   - 这个 test 必须验证真实行为。

2. Verify RED：
   - 运行测试；
   - 确认它因预期原因失败；
   - 如果没失败，说明测试无效。

3. GREEN：
   - 写最小实现；
   - 不加未来功能；
   - 不做 speculative design。

4. Verify GREEN：
   - 测试通过；
   - 相关 suite 通过。

5. REFACTOR：
   - 只在测试保护下清理；
   - 每次重构后重跑测试。

### 例外

需要问用户的例外：

- throwaway prototypes；
- generated code；
- configuration files。

### 关键测试标准

好的测试必须：

- 可证伪；
- 测行为，不测实现；
- 期望值独立于代码；
- 能因真实生产变更而失败。

### 我的理解

这个 TDD skill 非常强硬。它不是“建议先写测试”，而是把先写代码视为违反流程，必须删除重来。这是为了防止 Agent 先实现再补无效测试。

---

## 5.8 `systematic-debugging`

**定位：**
遇到 bug、测试失败、异常行为、性能问题、构建失败时，在修复前必须使用。

### 铁律

```text
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

没有完成 Phase 1，就不能提出修复。

### 四个阶段

#### Phase 1：Root Cause Investigation

先调查根因：

- 仔细读错误；
- 读完整 stack trace；
- 复现问题；
- 找最近改动；
- 理解系统行为；
- 不要猜。

#### Phase 2：Pattern Analysis

找模式：

- 是否多个失败共享根因；
- 是否和环境有关；
- 是否和数据有关；
- 是否和时序有关；
- 是否出现过类似问题。

#### Phase 3：Hypothesis and Testing

提出假设并测试：

- 假设要可证伪；
- 逐个测试；
- 不要一次改多个变量；
- 不要为验证假设引入新问题。

#### Phase 4：Implementation

只有根因明确后才修复：

- 修复根因，不修症状；
- 加回归测试；
- 验证原问题消失；
- 检查是否有类似位置也需要修。

### Red flags

如果出现这些想法，要停止：

- “我知道问题在哪”
- “先试这个”
- “改一下应该行”
- “没时间查根因”
- “这是简单 bug”

### 我的理解

这个 skill 和 TDD 一样强硬。它反对 Agent 的“猜修文化”。相比 Matt Pocock skills 的 `diagnosing-bugs`，Superpowers 版本更强调 root cause investigation 和 rationalization 防御。

---

## 5.9 `verification-before-completion`

**定位：**
在声称工作完成、修复成功、测试通过、准备 commit / PR 前必须使用。

### 铁律

```text
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

如果没有在当前消息中运行验证命令，就不能说它通过。

### Gate function

在任何完成性声明前：

1. IDENTIFY：什么命令证明这个声明？
2. RUN：运行完整命令；
3. READ：读取完整输出、exit code、失败数量；
4. VERIFY：输出是否支持声明？
5. ONLY THEN：带证据陈述。

### 常见要求

| 声明 | 需要证据 |
|---|---|
| Tests pass | 测试命令输出 0 failures |
| Linter clean | linter 输出 0 errors |
| Build succeeds | build exit 0 |
| Bug fixed | 原始症状测试通过 |
| Regression test works | red-green cycle 证据 |
| Agent completed | VCS diff 显示变更 |
| Requirements met | 逐项需求 checklist |

### Red flags

- “should”
- “probably”
- “seems to”
- “done”
- “perfect”
- 没验证就 commit / push / PR

### 我的理解

这是防止 Agent 编造完成状态的核心 skill。它要求完成声明必须绑定新鲜工具输出。

---

## 5.10 `requesting-code-review`

**定位：**
完成任务、重大 feature、merge 前，派 code reviewer subagent 审查。

### 使用时机

强制：

- SDD 中每个任务后；
- major feature 完成后；
- merge main 前。

可选：

- 卡住时；
- 重构前；
- 复杂 bug 修复后。

### 流程

1. 获取 git SHAs：

```bash
BASE_SHA=$(git rev-parse HEAD~1)
HEAD_SHA=$(git rev-parse HEAD)
```

2. 派 `general-purpose` subagent，使用 `code-reviewer.md` 模板。

3. 根据反馈行动：
   - Critical：立即修；
   - Important：继续前修；
   - Minor：记录；
   - reviewer 错时要技术性 push back。

### 我的理解

这个 skill 强调 review 早做、常做。reviewer 不继承当前 session 历史，而是得到精确上下文，减少 bias。

---

## 5.11 `receiving-code-review`

**定位：**
处理 code review 反馈时使用，防止 Agent 盲目迎合 reviewer 或用户。

### 核心原则

> Verify before implementing. Ask before assuming. Technical correctness over social comfort.

### 响应模式

1. READ：完整读反馈；
2. UNDERSTAND：用自己的话复述要求；
3. VERIFY：对照代码库现实；
4. EVALUATE：判断是否适合当前代码库；
5. RESPOND：技术性确认或反驳；
6. IMPLEMENT：一项一项实现并测试。

### 禁止回应

不能说：

- “You're absolutely right!”
- “Great point!”
- “Excellent feedback!”
- “Let me implement that now”

这些被认为是 performative agreement。

### 反馈不清时

如果任何 item 不清楚：

- 停止；
- 不实现；
- 先问清楚。

原因是多个反馈项可能有关联，部分理解会导致错误实现。

### YAGNI 检查

如果 reviewer 建议加入“更专业”的能力，例如：

- complex abstraction；
- extra extensibility；
- generalized framework；
- 未被需求证明的配置；

必须做 YAGNI 检查。

### 我的理解

这个 skill 很有意思：它不是教 Agent “听话”，而是教 Agent 不要表演性认同。review 反馈也要经过技术验证。

---

## 5.12 `finishing-a-development-branch`

**定位：**
实现完成、测试通过后，决定如何整合工作。

### 流程

1. Verify tests：
   - 运行完整测试套件；
   - 测试失败则停止，不展示完成菜单。

2. Detect environment：
   - 判断普通 repo、worktree、detached HEAD；
   - 记录 `WORKTREE_PATH`。

3. Determine base branch：
   - 确认 work 从哪个 base 分出来；
   - 不确定就问用户；
   - 防止 merge 到错误分支。

4. Present options：
   - merge locally；
   - push and create PR；
   - keep as-is。

5. Execute choice。

6. Cleanup workspace：
   - 只有 Superpowers 创建的 worktree 才清理；
   - 外部管理的 worktree 不乱删。

### 特别变化

v6.2.0 移除了默认菜单中的 “Discard this work”。
如果用户明确要求丢弃，才走带 typed confirmation 的路径。

### 我的理解

这是交付出口 skill。它把“代码完成”之后的选择变成受控流程，并特别防止误删 worktree 或误合并。

---

## 5.13 `dispatching-parallel-agents`

**定位：**
当有多个独立任务 / 独立失败 / 独立子系统问题时，用多个 Agent 并行调查。

### 使用条件

适合：

- 3+ test files 失败且根因不同；
- 多个子系统独立损坏；
- 每个问题不依赖其他问题上下文；
- 无共享状态干扰。

不适合：

- 失败相关；
- 修一个可能影响多个；
- 需要理解全局系统状态；
- Agent 会互相干扰。

### 模式

1. Identify independent domains；
2. Create focused agent tasks；
3. Dispatch in parallel；
4. Review and integrate。

### Agent prompt 要求

每个 Agent 都应拿到：

- 具体问题；
- 相关文件；
- 需要运行的命令；
- 输出格式；
- 不该做什么；
- 完成标准。

### 我的理解

这是并行调查模式，不是为了炫技。只有问题真的独立时，并行才有效，否则会制造更多噪声。

---

## 5.14 `writing-skills`

**定位：**
创建、修改、验证 skills 时使用。它把 skill 写作当成 TDD。

### 核心原则

> If you didn't watch an agent fail without the skill, you don't know if the skill teaches the right thing.

### TDD 映射

| TDD 概念 | Skill 创建 |
|---|---|
| Test case | pressure scenario with subagent |
| Production code | SKILL.md |
| RED | 没有 skill 时 Agent 违规 |
| GREEN | 有 skill 后 Agent 遵守 |
| Refactor | 关闭 loopholes |
| Write test first | 先运行 baseline scenario |
| Watch it fail | 记录 Agent rationalization |
| Minimal code | 只写针对违规的 skill |
| Watch it pass | 验证 Agent compliance |

### Skill 不是

- 一次性解决记录；
- 项目特定配置；
- 个人工作日志；
- 成功故事 narrative。

### Skill 类型

1. Technique
   具体操作方法。

2. Pattern
   思维模型。

3. Reference
   工具/API/文档参考。

### Skill Discovery Optimization

描述字段很关键：

- description 是“什么时候用”，不是“技能做什么”；
- 不要在 description 里总结流程；
- 应覆盖触发关键词；
- 用 `Use when...`；
- 不要第一人称；
- 不要太抽象。

### Token efficiency

它强调：

- 不要复制 `--help`；
- 不要重复其他 skill；
- 大参考下沉到 reference 文件；
- frequently-loaded skill 要非常短；
- 只保留会改变 Agent 行为的内容。

### Rationalization table

技能必须防御 Agent 的常见借口，例如：

- “这个例外可以跳过”
- “我已经理解了”
- “这只是小改动”
- “精神比文字重要”

### 我的理解

`writing-skills` 是 Superpowers 的元方法论。它把 prompt writing 转成可测试工程，不靠感觉写 skill，而是先看 Agent 怎么失败，再写最小文本修复失败。

---

# 6. Superpowers 的核心工程哲学

我把它提炼成 10 条。

## 6.1 Skill 不是建议，而是强制流程

Superpowers 通过 `using-superpowers` 把 skill 使用前置到所有行动之前。
只要 skill 可能相关，就必须读并使用。

## 6.2 先设计，再实现

`brainstorming` 禁止在设计批准前写代码。
它认为“简单需求不需要设计”是危险借口。

## 6.3 TDD 是硬约束

`test-driven-development` 要求先写失败测试。
如果先写了生产代码，必须删除重来。

## 6.4 Debug 先找根因

`systematic-debugging` 禁止没根因就修。
它把“先试一下”视为失败模式。

## 6.5 证据先于完成声明

`verification-before-completion` 要求完成声明必须有当前运行的验证输出。

## 6.6 子 Agent 要隔离上下文

`subagent-driven-development` 和 `dispatching-parallel-agents` 都强调：
子 Agent 不继承你的 session 历史，只拿任务所需上下文。

## 6.7 Review 是流水线的一部分

每个任务后 review，最后 broad review。
不是最后临时看一眼。

## 6.8 计划要足够低上下文

`writing-plans` 假设执行者不了解项目、品味一般、不擅长测试。
所以 plan 必须自包含、可验证、可审查。

## 6.9 反 rationalization 是核心设计

几乎所有强约束 skills 都有：

- Red Flags；
- Common Rationalizations；
- Iron Law。

这是为了防止 Agent 为跳过流程找借口。

## 6.10 Skill 本身也要测试

`writing-skills` 要求用 RED-GREEN-REFACTOR 写 skill。
这说明 Superpowers 不把 prompt 当玄学，而当可验证工件。

---

# 7. 与 mattpocock-skills 的对比

你刚才看的 Matt Pocock skills 和 Superpowers 很相似，但侧重点不同。

## 共同点

两者都强调：

- 先澄清再实现；
- TDD；
- 代码审查；
- 任务拆分；
- 子 Agent；
- 上下文边界；
- 文档化；
- 防止 Agent 乱写代码。

## Superpowers 更强的地方

1. **更强的强制性**
   Superpowers 有 `using-superpowers` 作为全局 bootstrap，要求任何行动前先检查 skill。

2. **更重视防 rationalization**
   大量 skills 都有 Iron Law、Red Flags、Common Rationalizations。

3. **更强调 TDD 的硬性执行**
   写了生产代码再补测试必须删除重来。

4. **更成熟的 subagent-driven-development**
   有 plan-scoped ledger、task brief、review package、re-review loop。

5. **更多 harness 适配**
   支持 Claude Code、Codex、Cursor、Gemini、OpenCode、Kimi、Pi、Antigravity 等。

## Matt Pocock skills 更强的地方

1. **领域建模和架构语言更细**
   Matt Pocock skills 中的 `domain-modeling`、`codebase-design`、`wayfinder` 对领域语言、deep module、decision map 的设计更精细。

2. **大型模糊项目规划更有概念工具**
   `wayfinder` 的 fog of war、frontier、decision tickets 很适合超大项目。

3. **工程语言更偏架构设计**
   Matt Pocock skills 更像“架构师 + 产品工程师”的流程，Superpowers 更像“严格工程主管 + TDD coach”。

---

# 8. 我对 Superpowers 的总体评价

Superpowers 的最大价值是：**它系统性地约束 AI coding agent 的冲动。**

普通 Agent 容易这样工作：

```text
用户说需求
  ↓
Agent 直接写代码
  ↓
补一些测试
  ↓
说完成
  ↓
用户发现偏了
```

Superpowers 把它改成：

```text
用户说需求
  ↓
必须 brainstorming
  ↓
用户批准设计
  ↓
写计划
  ↓
隔离 worktree
  ↓
TDD 实现
  ↓
每任务 review
  ↓
完成前验证
  ↓
用户选择合并方式
```

这套方法非常适合：

- 多步骤 feature；
- 复杂 bugfix；
- 长时间 autonomous coding；
- 有多个 coding agents 的工作流；
- 对质量、可审查性、可恢复性要求高的项目。

它不适合：

- 极小的一次性脚本；
- 用户明确只想快速探索；
- 没有测试环境的临时代码；
- 纯问答或轻量编辑。

不过 Superpowers 自己会说：即使任务很小，也要经过相应流程，只是流程可以短。它宁愿多一点纪律，也不愿让 Agent 开始滑坡。

---

## 9. 最值得深入研究的 5 个 skills

如果你想继续深挖，我建议优先看这 5 个：

1. **`using-superpowers`**
   理解它如何把 skills 变成强制流程。

2. **`brainstorming`**
   理解它如何阻止 premature implementation。

3. **`test-driven-development`**
   理解它如何防止 Agent 先写代码再补假测试。

4. **`subagent-driven-development`**
   理解它如何组织多 Agent 开发和审查。

5. **`writing-skills`**
   理解它如何把 skill 写作变成 TDD。

如果要和 `mattpocock-skills` 做对照分析，重点可以放在：

- Superpowers 的 `brainstorming` vs Matt 的 `grilling`
- Superpowers 的 `writing-plans` vs Matt 的 `to-spec` / `to-tickets`
- Superpowers 的 `subagent-driven-development` vs Matt 的 `implement`
- Superpowers 的 `systematic-debugging` vs Matt 的 `diagnosing-bugs`
- Superpowers 的 `writing-skills` vs Matt 的 `writing-for-agents`
