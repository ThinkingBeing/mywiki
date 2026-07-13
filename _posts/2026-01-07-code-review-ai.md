---
layout: post
title: "Addy Osmani《AI writes code faster》解读：AI 没有杀死代码审查，而是让证明责任显性化"
description: "Addy Osmani 认为 AI 让代码生成速度大幅提升，但工程师的职责仍是证明代码真的可用：PR 必须携带测试、手动验证、风险说明和人类责任。"
date: 2026-01-07
created_at: 2026-07-13 09:47:57 +0800
categories: AI Agents
---

# Addy Osmani《AI writes code faster》解读：AI 没有杀死代码审查，而是让证明责任显性化

> 来源: Addy Osmani Blog | [原文链接](https://addyosmani.com/blog/code-review-ai/) | 发布日期: 2026-01-07

## 1. 文章基本信息

- **标题**: AI writes code faster. Your job is still to prove it works.
- **来源**: AddyOsmani.com
- **链接**: https://addyosmani.com/blog/code-review-ai/
- **作者**: Addy Osmani
- **发布日期**: 2026-01-07
- **主题**: AI 编程、代码审查、PR 证据、测试、团队协作、安全、Human Accountability

## 2. 文章要解决的核心问题（论点）

这篇文章试图回答的问题是：**当 AI 能够更快地生成代码时，code review 的职责是否会被削弱，甚至被替代？**

Addy 的回答是：**不会。AI 没有杀死代码审查，而是让“证明代码真的能工作”的责任变得更加显性。**

一句话概括：**AI 可以加速写代码，但不能替你承担证明、理解和负责；如果 PR 里没有证据说明它真的工作，你并不是更快交付，只是把工作转嫁给 reviewer 或未来的事故处理者。**

## 3. 问题所处的背景上下文

到 2026 年初，AI 辅助编程已经从“偶尔补全代码”进入“生成大块功能、重构、测试和 PR”的阶段。文章提到，超过 30% 的资深开发者报告自己正在交付主要由 AI 生成的代码。

这带来了一个新的流程张力：

- AI 很擅长快速起草 feature、重构和测试；
- 但它在逻辑、边界条件、安全和真实业务语义上仍然容易犯错；
- 代码产出速度提升后，review、验证和知识传递反而成为新的瓶颈。

Addy 的核心背景判断是：过去我们常说“代码能跑才算数”，AI 时代这条规则更重要，而不是更不重要。因为 AI 让“看起来完成”的成本大幅降低，也让没有证据的完成感更危险。

## 4. 文章的核心观点

### 4.1 AI 改变了 review 的形态，但没有改变 review 的基本责任

文章开头就给出判断：AI 没有消灭 code review，而是把 proof burden 显性化。

过去 reviewer 常常根据作者的经验、代码 diff 和测试结果来判断变更是否可信。AI 参与之后，作者更容易提交自己并未完全理解的代码。因此 reviewer 不能只看“代码写得像不像”，而要看：

- 这段代码要解决什么问题？
- 作者是否理解它为什么这样做？
- 有没有测试或手动验证证明它工作？
- 风险在哪里？
- 哪些部分需要人类重点审查？

也就是说，review 从“读代码”变成“审查证据、意图和责任”。

### 4.2 Solo 开发者可以跑得更快，但必须有验证系统兜底

Addy 区分了 solo developer 和团队开发者的不同处境。

Solo 开发者没有团队知识传递和长期协作成本，因此可以更激进地使用 AI：让 agent 写代码、重构、生成测试、跑自动化循环，然后只人工查看关键部分。文章引用 Peter Steinberger 的实践：他不再阅读大量代码，而是观察 AI 输出流，只检查关键位置。

但 Addy 明确指出，这种“inference speed”开发方式只有在强测试体系存在时才成立。成功的 solo AI 开发者不是盲信 AI，而是建立了能快速捕捉问题的验证系统：

- 先写或让 AI 起草 `spec.md`；
- 人类确认规格；
- agent 按“写 → 测 → 修”的循环执行；
- 使用语言无关、数据驱动的测试；
- 对最终产品进行手动点击、运行和关键路径验证。

因此，即使是最激进的 solo 工作流，底线仍是：**你交付的是你已经证明可工作的代码。**

### 4.3 团队环境中，AI 会把 review 瓶颈放大

在团队中，代码不是一个人写完就结束。它要被别人维护、调试、on-call、扩展和审计。因此 AI 生成代码的主要风险不只是 bug，而是破坏团队的共享理解。

文章指出，随着 AI 采用增加：

- PR 变得更大；
- 事故和 change failure rate 上升；
- 人类 reviewer 面临更高负担；
- 如果没人真正读懂代码，团队会积累理解债务。

这也是为什么 Addy 认为团队不能把 AI reviewer 当成人类签核的替代品。AI 可以做初筛、找低级问题、提示风险，但最终仍需要人类 sign-off，因为团队需要有人能解释、维护并为代码负责。

### 4.4 安全是 AI 代码审查中最不能省略人工判断的领域

文章特别强调 security。AI 生成代码中安全缺陷比例较高，常见问题包括：

- 逻辑错误；
- XSS；
- 权限和认证问题；
- secret 处理不当；
- prompt injection；
- 数据外泄；
- agentic tooling / AI IDE 带来的新攻击面。

Addy 给出的规则很实用：**任何触及 auth、payments、secrets、untrusted input 的代码，都要把 AI 当作高速实习生，而不是可信专家；必须有人类 threat model review 和安全工具检查。**

这条原则很适合作为团队 AI 编程规范的一部分。

### 4.5 PR Contract：AI 时代作者欠 reviewer 的东西

文章最可操作的部分，是提出一个简单的 **PR Contract**。无论代码是否由 AI 生成，作者都应该在 PR 中提供：

1. **What / why**：1-2 句话说明变更意图；
2. **Proof it works**：测试通过、手动验证步骤、截图、日志或运行结果；
3. **Risk + AI role**：风险等级，以及哪些部分由 AI 生成或修改；
4. **Review focus**：希望人类 reviewer 重点看的 1-2 个问题，例如架构、安全、边界条件。

Addy 强调，这不是官僚流程，而是尊重 reviewer 时间，也是作者责任的强制函数。如果你无法填写这份 contract，说明你还没有足够理解自己的变更，不该要求别人批准。

### 4.6 AI reviewer 应该是 first-pass reviewer，不是 final arbiter

文章对 AI review 工具持肯定态度：可以用 Claude、Gemini、GPT 做 ad-hoc diff 检查，也可以用 Cursor、Claude Code、Gemini CLI、GitHub Copilot、Snyk 等工具进行 IDE 内检查、PR bot 扫描、安全分析和测试生成。

但这些工具的定位应该是：

- spellcheck，而不是 editor；
- sensor，而不是 verdict；
- first pass，而不是 final sign-off。

AI reviewer 可以帮人类节省低级检查的时间，把 reviewer 的注意力释放到架构、业务逻辑、安全、可维护性和组织上下文上。

## 5. 观点对应的论据

文章的论据主要来自行业实践和若干数据点：

1. **资深开发者 AI 采用率**  
   文章提到，到 2026 年初，超过 30% 的资深开发者报告正在交付主要由 AI 生成的代码。这说明问题已经不是少数人的实验，而是正在进入主流工作流。

2. **AI 在逻辑和安全上的弱点**  
   Addy 提到 AI 在逻辑错误、安全问题、边界条件上更容易失误，例如逻辑错误更常见，XSS 风险更高。这支撑了“AI 适合加速，但不能无审查合并”的判断。

3. **团队 PR 体积与故障风险上升**  
   随着 AI 采用增加，PR additions 变多、incident per PR 和 change failure rate 上升。这里说明 AI 的问题不是单个 diff 是否漂亮，而是整体系统吞吐上升后，人类 review capacity 被压迫。

4. **OCaml 维护者拒绝 13,000 行 AI PR 的案例**  
   这个案例很好地说明：代码不一定“明显错”，但如果规模大到没人有能力审查，它对维护者就是负担。AI 能生成大量代码，不代表社区或团队有能力吸收它。

5. **AI review 工具的混合反馈**  
   有团队报告 AI review 能抓到大量 bug，也有人认为 AI review 评论是噪声。因此工具要配置：调敏感度、关闭无价值 comment、明确 opt-in/opt-out，而不能默认把所有 AI 评论当有效信号。

## 6. 我的评价和启发

我认为这篇文章可以看作 Addy 后来《Agentic Code Review》的前置版本：这里先提出“proof over vibes”，后者进一步讨论当 agent 产出规模暴涨后，review 如何变成系统级瓶颈。

这篇文章最有价值的地方，是把 AI 编程的责任边界说清楚了：**AI 负责加速候选解的生成，人负责证明候选解满足需求。**

我尤其认同三点：

1. **PR 不是代码包裹，而是证据包裹**  
   AI 时代，一个好的 PR 不该只是 diff，而应该包含意图、验证、风险和审查焦点。否则 reviewer 就被迫从零恢复上下文，这会极大浪费高价值人类注意力。

2. **“我没读代码，但测试过了”只有在测试可信时才成立**  
   如果测试覆盖关键行为、不能被轻易篡改，并且包含真实用户路径，那么 AI 可以大幅提高 solo 开发速度。反过来，如果测试薄弱，AI 只是更快地产生未经验证的复杂性。

3. **团队 review 的核心功能是共享理解**  
   很多人把 review 简化成找 bug，但在团队里，review 还承担知识转移、架构一致性、责任分配和长期维护能力建设。AI 可以辅助这些过程，但不能自动替代“有人真正懂这段代码”。

我会补充一点：未来团队需要的不只是 AI review bot，而是 **PR Evidence Pipeline**。也就是在 PR 产生时自动收集和展示：

- agent prompt / plan；
- 修改范围；
- 测试命令与结果；
- 手动验证截图或日志；
- 安全扫描结果；
- 风险标签；
- AI 参与范围；
- reviewer 应重点关注的问题。

这样 code review 才能从“人工补上下文”变成“基于证据做判断”。

## 7. 可操作建议

结合文章内容，可以落地成以下实践：

1. **为所有 AI 参与的 PR 增加 PR Contract 模板**  
   必填：what/why、proof it works、risk + AI role、review focus。

2. **把“无证据 PR”挡在 review 之前**  
   没有测试、日志、截图、手动验证说明的 PR，不进入正式 review。

3. **建立风险分层**  
   auth、payments、secrets、untrusted input、数据迁移、权限模型等变更自动标为高风险，要求人工安全审查。

4. **强制小 PR 和 stackable changes**  
   Agent 可以一次做很多事，但提交给人类 review 的单元必须小、清晰、可解释。

5. **AI 先审，人类后判**  
   让 AI 做 first-pass：格式、低级 bug、测试建议、潜在重复代码、简单安全问题；人类重点看架构、业务意图、安全边界和可维护性。

6. **对测试变更提高警惕**  
   AI 很可能为了通过测试而修改测试本身。任何大规模测试重写、断言放松、覆盖率下降、CI 规则变化都应触发重点审查。

7. **要求作者能解释 AI 生成代码**  
   “AI 写的”不能成为不了解实现的理由。作者点击 merge 前，必须能解释关键路径如何工作、为什么这样设计、失败时如何回滚。

## 8. 延伸阅读

- Addy Osmani: [Agentic Code Review](https://addyosmani.com/blog/agentic-code-review/)
- Addy Osmani: [Loop Engineering](https://addyosmani.com/blog/loop-engineering/)
- Addy Osmani: [Agentic Engineering](https://addyosmani.com/blog/agentic-engineering/)
- Simon Willison: [Designing Agentic Loops](https://simonwillison.net/2025/Sep/30/designing-agentic-loops/)
- Sketch: [The Unreasonable Effectiveness of an LLM Agent Loop with Tool Use](https://sketch.dev/blog/agent-loop)

## 9. 结论

这篇文章的核心警句是：**AI 写代码更快，但你的工作仍然是证明它能工作。**

AI 让工程师从大量机械性编码中解放出来，但并没有让需求判断、安全边界、测试可信度、团队理解和最终责任消失。相反，因为产出速度变快，这些环节更容易被跳过，也更需要被制度化。

未来好的 code review 不会是“逐行手读所有代码”，而会是“让自动化处理低级检查，让 AI 做初筛，让人类基于证据审查风险、意图和责任”。

换句话说：**AI 时代最重要的工程文化不是 trust the vibe，而是 proof over vibes。**
