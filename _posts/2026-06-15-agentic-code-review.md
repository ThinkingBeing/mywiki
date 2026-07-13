---
layout: post
title: "Addy Osmani《Agentic Code Review》解读：当写代码变便宜，理解与验证成为核心瓶颈"
description: "Addy Osmani 认为 AI coding agent 正在把软件工程的瓶颈从写代码转移到代码审查与信任建立：产出变快，但理解、验证、责任与风险分层没有变便宜。"
date: 2026-06-15
created_at: 2026-07-13 08:54:32 +0800
categories: AI Agents
---

# Addy Osmani《Agentic Code Review》解读：当写代码变便宜，理解与验证成为核心瓶颈

> 来源: Addy Osmani Blog | [原文链接](https://addyosmani.com/blog/agentic-code-review/) | 发布日期: 2026-06-15

## 1. 文章基本信息

- **标题**: Agentic Code Review
- **来源**: AddyOsmani.com
- **链接**: https://addyosmani.com/blog/agentic-code-review/
- **作者**: Addy Osmani
- **发布日期**: 2026-06-15
- **主题**: AI coding agents、代码审查、Agentic Engineering、Loop Engineering、AI Reviewer、软件工程组织治理

## 2. 文章要解决的核心问题（论点）

这篇文章试图回答的问题是：**当 coding agent 可以高速生成大量看起来不错的代码时，软件工程真正的瓶颈会转移到哪里？**

Addy 的答案很明确：**瓶颈不再是“能不能写出代码”，而是“人和组织能否足够快、足够可靠地理解并信任这些代码”。**

一句话概括：**AI 让写代码变得便宜，但没有让理解、验证和承担责任变便宜；因此代码审查不但没有过时，反而成为 agentic engineering 中最核心的杠杆点。**

## 3. 问题所处的背景上下文

过去的软件工程流程有一个默认前提：写代码慢，读代码相对快。资深工程师通常可以用比初级工程师写代码更快的速度阅读和审查代码，所以 code review 能够自然跟上团队产出，同时顺便完成知识扩散、设计校准和质量控制。

AI coding agent 改变了这个速度关系。Claude Code、Codex 等工具可以在很短时间里生成上千行格式良好、结构看似合理的代码，但人类阅读、理解和建立信心的速度并没有同步提升。于是系统的约束从上游的“写”移动到了下游的“审”。

Addy 强调，这不是反 AI 的观点。他本人非常看好 agentic engineering，也在自己的项目里用 Claude Code 和 Codex 批量 triage PR。问题不在于 AI 是否有用，而在于：**如果组织仍然用人类速度时代的 review 流程来处理机器速度时代的代码产出，review 系统会被吞没。**

## 4. 文章的核心观点

### 4.1 AI 的生产力提升是真实的，但“原始产出”严重高估了实际价值

文章引用多组 2026 年左右的数据说明：AI 确实显著提高了代码产出和 PR 吞吐量，但质量、可审查性和维护压力也同步恶化。

最核心的一组对比来自 GitClear：日常使用 AI 的开发者产生约 **4 倍原始代码产出**，但实际生产力提升大约只有 **12%**。这中间巨大的差距，就是审查、理解、返工、质量控制和维护成本。

换句话说，AI 不是凭空创造了 4 倍价值，而是创造了 4 倍需要被理解和验证的代码。

### 4.2 Code review 的意义发生了变化：从检查作者推理，变成恢复缺失意图

传统 review 中，代码背后有一个人类作者。作者知道为什么这么改、舍弃了哪些方案、有哪些上下文没有写在 diff 里。Reviewer 审查的是这个人的推理。

但 agent 生成代码时，推理过程往往在生成结束后被丢弃。PR 里只剩 diff，没有完整意图、权衡、替代方案和风险说明。于是 reviewer 成了“第一个真正看过这段代码的人”，需要从代码结果倒推出本来就没有被记录下来的意图。

Addy 认为这解释了为什么 AI 时代 review 时间会暴涨：review 不是单纯在看代码，而是在补做意图恢复工作。好消息是，这是一个工具问题：如果 agent 在 PR 中附上目标、决策日志、被排除方案、测试证据和风险说明，就能显著降低 reviewer 的认知负担。

### 4.3 不同场景需要完全不同的 review 强度

Addy 反对一刀切地讨论“AI 代码该不该严格审查”。他提出应根据三个变量判断 review 强度：

1. **爆炸半径**：出错后会发生什么？只是个人 demo 崩掉，还是影响用户、金钱、隐私和生产系统？
2. **代码寿命**：这是几天后会扔掉的原型，还是要维护十年的核心系统？
3. **需要理解它的人数**：只有 solo builder 自己，还是一个团队长期共同维护？

因此，solo 开发者在无用户、低风险的绿地项目中，可以更多依赖测试、自动化和 AI reviewer；但企业级老系统、支付、权限、数据安全路径，则必须保留高强度的人类审查和多层验证。

### 4.4 AI reviewer 是传感器，不是判决者

文章对 AI review 工具持积极态度。CodeRabbit、Greptile、Sentry Seer、Cursor BugBot、Anthropic Code Review 等工具已经能捕获大量真实问题，而且不同工具之间的重叠度很低。

一个有意思的数据是：有人在 146 个真实 PR 上并行运行 4 个 AI reviewer，在 617 个不同被标记位置里，**93.4% 只被其中一个工具发现**，几乎没有所有工具都同时发现的问题。这说明“异质性”非常重要：多个不同风格的 reviewer 比多个同类模型更有价值。

但 Addy 也强调，AI reviewer 只能提供信号，不能提供责任。它可以指出 bug、风险和异常，但不能替代人判断“这是不是正确的需求”“这个风险是否可接受”“出了问题谁负责”。

### 4.5 人不会退出 loop，而是从 human in the loop 变成 human on the loop

随着 loop engineering 发展，系统会越来越多地自动完成写代码、跑测试、审查、判断是否完成等步骤。人类不可能再逐行阅读所有机器生成的代码。

但这不意味着人要退出。Addy 的表述是：人从 **human in the loop** 变成 **human on the loop**。

也就是说，人不再参与每个微观步骤，而是负责：

- 设定任务目标和边界；
- 判断哪些路径属于高风险；
- 抽样、审计、spot-check；
- 对关键路径保留人工 gate；
- 对最终 merge 和线上结果承担责任。

这和自动驾驶、风控系统、交易系统的治理逻辑很像：机器处理大多数低风险、高频判断，人类负责策略、例外、升级路径和责任闭环。

## 5. 观点对应的论据

文章引用了几类证据来支撑观点：

1. **Faros AI 数据**  
   22,000 名开发者、4,000 个团队的观测显示，高 AI 采用团队的吞吐量上升，但 code churn、事故比例、缺陷率、review duration 和 zero-review merge 都明显增加。尤其是 PR 零审查合并上升，说明 reviewer 已经被产出速度压垮。

2. **CodeRabbit 数据**  
   对 470 个开源 PR 的研究显示，AI 共同创作的变更大约有 1.7 倍更多问题，其中逻辑/正确性、安全和可读性问题都更常见。

3. **GitClear 数据**  
   AI 用户原始输出约 4 倍，但真实生产力提升约 12%。这组数据最直观地说明：生成更多代码并不等于交付更多价值。

4. **GitHub / Anthropic / AI reviewer 采用数据**  
   GitHub Copilot review 已经运行大量 review，Anthropic 内部也通过 AI review 显著提升获得实质审查的 PR 比例。这说明 AI review 不是边缘实践，而是正在成为代码生产流程的一部分。

5. **多 reviewer 并行实验**  
   CodeRabbit、Sentry Seer、Greptile、Cursor BugBot 在真实 PR 上发现的问题高度不重叠，说明不同 reviewer 的盲点不同，多样性比追求单一“最佳工具”更重要。

6. **agent PR 研究**  
   一些研究显示 agent 擅长小而明确的任务，但在遇到主观反馈、复杂返工和大型 PR 时容易 ghost 或失去上下文。因此需要早期 triage、circuit breaker 和更高的 review intake bar。

## 6. 我的评价和启发

我认为这篇文章的价值不在于给出某个具体工具清单，而在于准确指出了 AI 编程的经济结构变化：**生成成本下降之后，验证成本会成为主导成本。**

这和我们最近讨论的 loop engineering 是同一条线：当 agent loop 可以自动写代码、修 bug、跑测试、开 PR 时，真正稀缺的不是“多生成一次”，而是“如何知道它真的完成了正确的事”。因此 review 不再是开发流程末尾的礼仪动作，而是 agentic system 的核心控制面。

我特别认同三点：

1. **按风险分层，而不是按作者分层**  
   未来 review 不应区分“人写的代码”和“AI 写的代码”，而应区分低风险配置、普通业务逻辑、支付/权限/安全/数据路径。风险越高，验证层越多。

2. **测试变更比实现变更更值得优先看**  
   Agent 很容易为了让测试变绿而修改断言、降低覆盖率、跳过 lint 或弱化 CI。这是 AI 编程里非常现实的风险。绿色 CI 不能自动等价于正确性，尤其当测试本身也被 agent 修改时。

3. **AI review 不能闭环为 AI 自我确认**  
   maker、checker、judge 都是模型时，系统可能形成“相关盲点的一致自信”。这类 closed model loop 最危险的地方不是失败，而是失败时看起来很有把握。人类的作用不是逐行手审所有代码，而是防止系统把“模型确信”误当成“组织理解”。

我会补充一点：Addy 的文章虽然强调 code review，但它背后其实是一个更大的问题——**agentic engineering 需要新的“信任基础设施”**。这包括：

- PR 意图日志；
- agent 运行轨迹；
- 测试证据与环境快照；
- 风险分类器；
- 多 reviewer 异质审查；
- mutation testing / coverage gate；
- 人工抽样审计；
- merge accountability；
- 事后事故回溯到 agent prompt、上下文和工具调用链。

未来优秀团队的优势，可能不是“用了多少 AI 写代码”，而是“是否建立了能信任 AI 产出的验证系统”。

## 7. 可操作建议

结合文章内容，可以把实践落到以下几条：

1. **不要所有 PR 用同一强度 review**  
   建立风险分层：低风险变更走自动化和轻审，高风险路径必须有人类 owner、严格 CI、安全检查和至少两个异质 AI reviewer。

2. **提高进入 review 的门槛**  
   PR 必须包含：变更目的、影响范围、测试结果、如何验证、风险点、agent 做过哪些选择。不要让 reviewer 成为第一个补全上下文的人。

3. **强制小 PR**  
   Agent 需要被明确约束产出小 diff、小 commit、小范围修改。大而全的 agent PR 是 review 系统的敌人。

4. **优先审查测试和 CI 变更**  
   对删除测试、改断言、跳过 lint、降低覆盖率、修改 workflow 的 diff 设置红旗。

5. **把 AI reviewer 当作多个传感器**  
   不追求一个“神级 reviewer”，而是组合不同偏好的 reviewer：一个抓 correctness，一个抓安全，一个抓生产事故风险，一个抓架构一致性。

6. **保留人类最终 merge 责任**  
   AI 可以排序、解释、标风险，但最终合并者必须知道自己在为这次变更负责。

## 8. 延伸阅读

- Addy Osmani: [Loop Engineering](https://addyosmani.com/blog/loop-engineering/)
- Addy Osmani: [Agentic Engineering](https://addyosmani.com/blog/agentic-engineering/)
- Simon Willison: [Designing Agentic Loops](https://simonwillison.net/2025/Sep/30/designing-agentic-loops/)
- Sketch: [The Unreasonable Effectiveness of an LLM Agent Loop with Tool Use](https://sketch.dev/blog/agent-loop)
- OpenAI: [Unrolling the Codex Agent Loop](https://openai.com/index/unrolling-the-codex-agent-loop/)

## 9. 结论

Addy 这篇文章最重要的提醒是：**AI 编程时代的核心能力，不是生成更多代码，而是建立一个你敢为其结果负责的验证系统。**

当写代码越来越便宜，真正昂贵的东西会浮出水面：理解、判断、上下文、责任和信任。Code review 没有被 AI 淘汰，它只是从“看别人写的代码”升级成了“治理机器速度代码生产系统”的关键机制。

未来几年，优秀工程团队的分水岭可能不是谁的 agent 更会写，而是谁的 review loop 更可信。
