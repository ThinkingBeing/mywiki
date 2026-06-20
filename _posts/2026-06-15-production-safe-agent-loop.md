---
layout: post
title: "生产安全的 Agent Loop：退出条件、熔断器与审计轨迹"
description: "Daniel Nwaneri 文章指出，Agent 失败最危险的地方不是崩溃，而是持续正确执行一个没有停止条件的循环；生产级 Agent Loop 必须把完成定义、硬性预算、审计日志和人工签署做成系统边界。"
date: 2026-06-15
created_at: 2026-06-20 14:17:14 +0800
categories: AI Agents
source: "freeCodeCamp"
author: "Daniel Nwaneri"
original_url: "https://www.freecodecamp.org/news/how-to-build-a-production-safe-agent-loop-from-exit-conditions-to-audit-trails/"
---

# 生产安全的 Agent Loop：退出条件、熔断器与审计轨迹

> 来源: freeCodeCamp | [原文链接](https://www.freecodecamp.org/news/how-to-build-a-production-safe-agent-loop-from-exit-conditions-to-audit-trails/) | 发布日期: 2026-06-15

## 1. 文章基本信息

- **标题**: How to Build a Production-Safe Agent Loop: From Exit Conditions to Audit Trails
- **来源**: freeCodeCamp
- **链接**: https://www.freecodecamp.org/news/how-to-build-a-production-safe-agent-loop-from-exit-conditions-to-audit-trails/
- **作者**: Daniel Nwaneri
- **发布日期**: 2026-06-15
- **主题**: AI Agent、Agent Loop、退出条件、成本控制、审计轨迹、人类审查、生产安全

## 2. 文章要解决的核心问题

这篇文章要回答的问题是：当 AI Agent 从一次性问答变成多轮自主循环后，如何避免它在“没有报错、看似正常工作”的状态下无限运行、烧钱、污染下游系统，并且事后无法追责？

一句话概括：**生产级 Agent Loop 不能只依赖模型“自己知道何时完成”，而必须在循环外部定义完成条件、硬性预算、不可变审计日志和人工签署。**

## 3. 背景上下文

过去的聊天机器人通常是一次输入、一次输出，成本和风险相对可见。但 Agent 架构不同：它会规划、调用工具、读取上下文、重试、再验证，甚至让多个 Agent 互相协作。文章开头用两个事故说明问题：

1. 一个 Claude Code 递归循环在 5 小时内烧掉约 1.6 万到 5 万美元；
2. 一个四 Agent 的 LangChain 循环跑了 11 天，花费约 4.7 万美元，直到发票出现才被发现。

这两个案例的共同点不是“系统崩了”，而是**系统按指令持续运行**。传统软件遇到非法状态可能抛异常，LLM 遇到模糊状态则倾向于“继续帮忙”：重试、改格式、调用验证器、生成修正。这会让仪表盘看起来很健康：调用数在增长、任务在推进、没有 crash。但从业务角度看，它可能只是在不断消耗预算。

作者还强调了成本放大的机制：Agent 每次重试时通常会重新读取前面的上下文，所以失败不是线性成本，而会随着上下文增长而放大。文章引用的估算是：普通 chatbot 交互约 0.04 美元，而 orchestrated multi-agent workflow 可到 1.20 美元，复杂场景甚至是 30x 到 70x 的乘数。这里真正危险的不是“LLM 贵”，而是团队用 chatbot 的成本心智部署了 agentic workflow。

## 4. 文章核心观点

### 4.1 “完成定义”必须在循环开始前写下来

作者认为，Agent 开发里最贵的错误不是模型选错，也不是 retry limit 少配，而是开工前不能用一句话回答：**什么叫完成？**

文章设计的第一个 primitive 是 `SpecWriter`。它要求运行者在启动 loop 之前回答三个问题：

- 这个 Agent 做什么？
- 这个 Agent 不做什么？
- 完成状态长什么样，用一句话说清楚？

第三个问题最关键。比如“Agent audits the site”不是有效完成定义，因为它没有可判定边界；“Agent crawls the target URL, extracts all `<title>` and `<meta description>` tags, flags any missing or over-length, and stops”才是可执行的完成定义。

这个设计背后的隐含前提是：**Agent Loop 的停止条件不能藏在模型的主观判断里，而要成为外部系统可以检查的合同。** 如果完成定义是含糊的，后面的熔断器、审计和 review 都只能检查“有没有活动”，不能检查“有没有完成正确的事”。

### 4.2 熔断器要在每次 LLM 调用前检查，而不是事后统计

第二个 primitive 是 `CircuitBreaker`。它维护两个硬上限：

- `turn_limit`：最多允许多少轮 LLM 调用；
- `token_limit`：累计 token 预算上限。

任何一个超限就立刻抛出 `CircuitBreakerError`。作者特别强调：检查必须发生在每一次 LLM 调用之前。事后检查虽然能记录问题，但钱已经花出去了；预检才能真正阻止下一次支出。

这里的工程判断很明确：**预算边界必须是 hard stop，而不是 warning。** Warning 很容易被忽略，返回码可能没人处理；异常会强迫调用方面对这个状态。文章甚至让 `_trip()` 在抛异常前打印人类可读的 checkpoint banner，避免异常被吞掉后完全静默。

这个观点对生产 Agent 很重要。很多系统把“最多 N 次重试”当作配置项，但没有把 token、工具调用次数、运行时间、外部 API 成本统一纳入预算。作者的例子虽然简单，但抽象是对的：每次进入模型前，都应该有一个可计算的预算守门人。

### 4.3 Ledger 不是普通日志，而是治理证据

第三个 primitive 是 `Ledger`：一个 append-only 的 SQLite 审计表，每一轮写一行。字段包括：

- `session_id`
- `turn_count`
- `state_origin`
- `input_hash`
- `token_delta`
- `execution_time_ms`
- `pass_fail`
- `breach_reason`
- `created_at`

作者有几个细节值得注意：

1. **记录 input hash，而不是原始 input**  
   这样既能检测相同输入跨运行出现，又避免把 PII 或敏感内容直接写入审计库。

2. **append-only，不 update，不 delete**  
   审计轨迹的价值在于它能证明系统当时发生了什么。如果可以随意修改，它就退化成普通 debug notebook。

3. **使用 timezone-aware timestamp**  
   `datetime.now(timezone.utc).isoformat()` 避免跨时区系统里的时间解释问题，也避开 Python 3.12 对 `datetime.utcnow()` 的弃用。

这部分最重要的区分是：普通日志用于 debug，ledger 用于 governance。debug 关注“哪里错了”，governance 关注“这个系统是否遵守边界，以及是否有证据证明”。

### 4.4 Agent Loop 本体应该只是把 spec、breaker、ledger 串起来

第四个 primitive 是 `AgentLoop`。它本身不复杂，关键在执行顺序：

1. `circuit_breaker.check(turn, total_tokens)`：先检查预算；
2. `client.messages.create(...)`：再调用 LLM；
3. `ledger.write(...)`：记录这一轮；
4. 如果 `stop_reason == "end_turn"`，返回结果；否则继续下一轮；
5. 如果熔断器触发，写一条 `state_origin="circuit_breaker"`、`pass_fail=False` 的 ledger row，并返回 breach result。

这体现了一种很健康的架构：模型不是系统的控制核心，而只是循环里的一个被约束执行单元。真正的控制权在外部：spec 定义承诺，breaker 定义预算，ledger 定义可追踪证据。

文章还强调 `_system_prompt()` 不只放 `done_looks_like`，也要放 `what_it_does` 和 `what_it_does_not`。负向范围很重要，因为很多 Agent 出问题不是因为不知道要做什么，而是因为不知道不该做什么。

### 4.5 Review Surface 把“看起来不错”升级为“对照承诺签署”

第五个 primitive 是 `ReviewSurface`。作者指出，熔断器能保护钱包，ledger 能记录发生了什么，但它们都不能证明输出是否符合一开始的承诺。

所以 review surface 会组合一个五元素 frame：

1. 原始承诺：spec 里的做什么、不做什么、完成定义；
2. 验收标准：`done_looks_like`；
3. diff：第一轮输入 vs 最终输出、完成轮数、总 token、是否 breach；
4. evidence：该 session 的所有 ledger row；
5. unresolved assumptions：由 breach 或失败行推导出的未解决假设。

reviewer 满意后调用 `.attest()`，把签署写入 ledger，并生成 `frame_hash`。这个 hash 是对 canonical frame 的 SHA-256，相当于审计收据：证明 reviewer 看过的是这个 frame，而不是后来被改写的摘要。

这部分是文章最有治理意识的地方。作者区分了两个概念：approval 和 attestation。approval 只是“我同意结果”；attestation 是“我对照原始承诺、证据和未解决假设后签署”。在受监管或高风险场景里，这两者不是同一份法律文件。

## 5. 观点对应的论据与案例

### 5.1 成本事故说明 Agent Loop 的失败常常是“正常运行式失败”

文章用 Claude Code 和 LangChain 的高额成本事故说明：Agent Loop 的危险不一定来自 crash，而来自没有停止条件的持续执行。系统没有报错，Agent 也不是恶意行为，它们只是不断“完成被要求继续做的事”。

这个论据支撑了作者的核心判断：生产安全的重点不是让模型更聪明，而是让循环有外部边界。

### 5.2 成本曲线来自上下文重复读取

作者指出，Agent 失败后重试时不是从零开始，而是带着前面失败尝试的上下文继续。第 1 轮可能只花 100 token，第 2 轮 200，第 10 轮就可能成千上万。多 Agent 编排还会把这个乘数继续放大。

这解释了为什么“只多跑几轮”在 Agent 系统里不是小事。每一轮都可能带着越来越大的上下文、更多工具结果和更多中间状态。

### 5.3 五个 Python primitive 提供了最小可落地架构

文章的代码不是大型框架，而是五个可独立复用的小模块：

| 模块 | 作用 | 文章给出的规模 |
|---|---|---|
| `spec_writer.py` | 运行前强制定义 scope 和 done | 约 104 行 |
| `circuit_breaker.py` | 对轮数和 token 做硬停止 | 约 41 行 |
| `ledger.py` | 追加式 SQLite 审计轨迹 | 约 113 行 |
| `agent_loop.py` | 串起 spec、breaker、ledger、LLM client | 约 128 行 |
| `review_surface.py` | 生成 review frame，并记录人工签署 | 约 114 行 |

这种规模本身就是论据：生产安全不一定先从复杂平台开始，很多风险可以用非常小的外层 harness 先挡住。

### 5.4 SEO Audit Agent 说明“定时运行并停止”往往比 24/7 自主更诚实

文章用 SEO audit agent 做例子。SEO 审计天然是周期性任务：爬取页面、检查 title/meta/H1、输出问题、等待搜索引擎重新索引。让 Agent 24/7 一直跑并不会加快这个业务节奏，只会在空档里烧 token。

作者因此提出一个朴素但重要的判断：很多 Agent 不应该被设计成常驻自治系统，而应该是 cron job：按计划运行，产出可审查结果，然后停止。

### 5.5 Pluggable LLM Client 说明生产架构不应绑定单一模型供应商

文章用 `LLMClient` Protocol 把 Anthropic SDK 的 `client.messages.create(...)` 形状抽象出来，并说明 OpenAI adapter 可以通过约 20 行 shim 适配。这个点不是文章主线，但很实用：生产 loop 的安全边界应该围绕“调用模型前后发生什么”，而不是某个 provider 的 SDK 细节。

## 6. 我的评价和启发

### 6.1 这篇文章真正有价值的地方：把 Agent 安全讲成控制系统问题

很多 Agent 文章会聚焦模型能力：模型能不能规划、能不能调用工具、能不能自我修复。这篇文章更工程化：它不试图证明模型多聪明，而是问“如果模型一直工作，我们如何约束它？”

我认为这个视角非常正确。生产 Agent 的核心不是 autonomy，而是 bounded autonomy：

- 有明确任务边界；
- 有预算边界；
- 有时间和轮数边界；
- 有可追踪证据；
- 有人类签署点；
- 有下游发布闸门。

Agent 越强，越需要这些外部边界。否则增强模型能力只是在增强一个无限循环的执行力。

### 6.2 我特别认同：done condition 是第一等工程对象

文章反复强调 “What does done look like?”。这句话看似简单，但在 Agent 项目里经常被跳过。很多任务描述是愿望式的：改善性能、优化体验、修复问题、做一次审计。对人类团队来说，这些模糊表述可以通过会议和经验补齐；对 Agent Loop 来说，它们会变成无限空间。

更好的做法是把 done condition 写成验收函数：

- 输入是什么？
- 输出必须包含什么？
- 哪些事情明确不做？
- 哪些测试或检查必须通过？
- 什么时候必须停止并交给人？

这也适用于我们日常使用 Agent：不是“帮我研究一下”，而是“阅读这 5 个来源，输出 8 条证据化结论，列出不确定项，不访问付费墙，最多运行 10 分钟”。

### 6.3 文章略显简化的地方：token limit 不是唯一成本，工具副作用同样危险

文章的 circuit breaker 主要用 turn 和 token 做上限，这是一个很好的最小实现。但真实生产 Agent 还有更多预算维度：

- 外部 API 调用次数；
- 数据库写入/删除次数；
- 浏览器操作次数；
- 代码提交次数；
- 云资源创建数量；
- wall-clock 时间；
- 队列积压；
- 人类 review 负担。

尤其是具有写权限的 Agent，最危险的不只是烧 token，而是对外部系统产生不可逆副作用。因此我会把作者的 `CircuitBreaker` 扩展成多维 policy guard：不仅限制 token，也限制 tool call、权限范围、资源配额和变更类型。

### 6.4 Ledger 的方向正确，但合规场景需要更强不可篡改性

作者在下一步里提到 Ed25519 signing，这一点很关键。SQLite append-only 是很好的本地工程约束，但不是强不可篡改保证。数据库管理员仍然可以 UPDATE 或重写文件。对于内部工具，这可能足够；对于合规、金融、医疗或安全审计场景，需要更强机制：

- 每行日志签名；
- hash chain；
- WORM storage；
- 外部时间戳服务；
- 权限隔离；
- 审计库与执行环境分离。

所以这篇文章适合被理解为“最小可行治理架构”，而不是最终合规架构。

### 6.5 Human attestation 是对抗“自动化幻觉”的关键

我很喜欢 review surface 的设计。很多团队的 Agent workflow 只有两个状态：跑完了、没跑完。真正应该多一个状态：**是否有人对照原始承诺和证据签署过？**

这能避免一种常见问题：Agent 输出很漂亮，reviewer 只读最终报告，觉得合理，于是批准。但它可能没有完成一开始承诺的范围，或者中间 breach 后绕过了某些步骤。review surface 强迫 reviewer 同时看 promise、criteria、diff、evidence、assumptions，相当于把 review 从“审美判断”变成“证据判断”。

### 6.6 对个人和团队的落地建议

如果要把这篇文章的方法用到真实 Agent 项目，我会按这个顺序落地：

1. **先写 Agent spec 模板**  
   每个 Agent 任务都必须填写：做什么、不做什么、完成定义、输入、输出、权限、预算、失败时交给谁。

2. **给每个 loop 加硬停止**  
   至少包括最大轮数、最大 token、最大运行时间。写操作还要加最大变更文件数、最大删除行数、最大外部 API 调用数。

3. **把日志升级为 session ledger**  
   每一轮记录 session_id、输入 hash、工具调用摘要、token、耗时、结果、breach 原因。不要只靠 stdout。

4. **引入人工 review frame**  
   输出交给下游系统前，必须展示：原始承诺、验收条件、最终输出、审计轨迹、未解决假设。

5. **从只读 Agent 开始**  
   比如监控 CI、汇总 issue、检查 SEO、生成日报。等审计和 review 稳定后，再给写权限。

6. **把 cron 当成默认架构，而不是 24/7 常驻**  
   如果业务本身是周期性的，就让 Agent 周期性运行并停止。持续自治应该是例外，而不是默认。

## 7. 延伸阅读 / 相关链接

- 原文: [How to Build a Production-Safe Agent Loop: From Exit Conditions to Audit Trails](https://www.freecodecamp.org/news/how-to-build-a-production-safe-agent-loop-from-exit-conditions-to-audit-trails/)
- 示例仓库: [dannwaneri/production-safe-agent-loop](https://github.com/dannwaneri/production-safe-agent-loop)
- Anthropic Claude API 文档: https://docs.anthropic.com/
- Python SQLite 文档: https://docs.python.org/3/library/sqlite3.html
- OpenTelemetry: https://opentelemetry.io/
- OWASP LLM Top 10: https://owasp.org/www-project-top-10-for-large-language-model-applications/

## 8. 总结

这篇文章最值得带走的一句话是：**一个没有退出条件的 Agent Loop，不是自治系统，而是等待发生的账单事故。**

作者给出的五个 primitive——SpecWriter、CircuitBreaker、Ledger、AgentLoop、ReviewSurface——并不复杂，但抓住了生产 Agent 的关键边界：先定义完成，再限制预算，再记录证据，最后让人对照承诺签署。

对工程实践来说，这篇文章的启发是：不要把 Agent 安全寄托在模型“应该会停”上。模型只是循环里的执行器；停止条件、成本边界、审计轨迹和发布闸门，必须由外部系统负责。Agent 越自动化，外部控制系统就越重要。
