---
layout: post
title: "Query Loop：代理系统的心跳 - Claude Code 架构深度解读"
description: "Harness Engineering 系列文章深度解读 Claude Code 的核心架构。代理系统的成熟不在于模型多强，而在于能否维持可恢复的执行循环。Query Loop 正是这种「心跳」机制的核心实现。"
date: 2026-04-14
categories: claude-code
---

> 文章来源：Harness Engineering - Claude Code 设计指南第 3 章
> 原文链接：https://harness-books.agentway.dev/book1-claude-code/chapter-03-query-loop-heartbeat.html

## 核心观点

**代理系统的成熟标志，不是模型会不会说话，而是它能否在多轮之后仍然知道自己在做什么。**

Claude Code 的 Query Loop（查询循环）正是这种「心跳」机制的核心实现。它不是简单地把模型调用包在一个 try/catch 里就结束，而是维护了一套完整的跨迭代状态治理体系。

---

## 3.1 循环：代理系统的分水岭

很多人在设计 AI 系统时，会把它想象成一个「加强版问答接口」——用户发来一句话，模型输出一个结果，事情就算办完。

这种理解在**单轮问答**场景下没问题，但一旦系统开始：
- 调用工具
- 跨轮执行
- 处理中断
- 保存状态
- 重试失败
- 压缩上下文

「一问一答」的思维就会迅速失效。

Claude Code 从结构上明确承认：**代理依赖一段持续的、有状态的执行过程**。这种承认体现在 `query()` 和 `queryLoop()` 的分离——前者只是壳，后者才是核心。

---

## 3.2 状态属于主业务

很多系统倾向于把状态看成「包袱」，认为无状态更优雅。但对代理系统来说，只要进入真实工作流，状态就会自然出现。

Claude Code 的态度很直接——它明确定义了 Query Loop 的可变状态：

```typescript
messages
toolUseContext
autoCompactTracking
maxOutputTokensRecoveryCount
hasAttemptedReactiveCompact
pendingToolSummary
stopHookActive
turnCount
transition
```

这些状态在每次 Query Loop 启动时被整体装配成一个 `State` 对象，并在后续各个 continue 分支里整体更新。

**关键洞察**：Claude Code 没有把恢复、压缩、预算、hook、turn 计数散落在局部变量和布尔开关里，而是承认它们共同构成了「本轮结束后下一轮如何继续」的基础。它把状态当作心跳的一部分。

> 这就是成熟代理系统和一次性脚本的区别：脚本只关心这一步有没有跑完，代理系统还要关心这一步失败之后，下一步能不能继续承接前面留下的状态。

---

## 3.3 输入治理 > 模型推理

从外部看代理系统，很多人会以为什么核心动作是「调用模型」。但在工程上，真正重要的往往是**模型调用之前**那一长串整理工作。

Claude Code 在正式进入模型流之前，会依次执行：

1. **Memory 预取** (query.ts:297)
2. **Skill Discovery 预取** (query.ts:323)
3. **截取 compact boundary 后的有效消息** (query.ts:365)
4. **应用 tool result budget** (query.ts:369)
5. **History Snip** (query.ts:396)
6. **Microcompact** (query.ts:412)
7. **Context Collapse** (query.ts:428)
8. **尝试 Autocompact** (query.ts:453)

这串顺序本身就是一种**架构声明**：把「上下文治理」放在「模型推理」之前。

很多系统恰恰相反——先把大量上下文塞进去，再寄希望于模型自己判断什么重要。看似省事，实际上是把运行时应承担的责任转嫁给概率分布。

Claude Code 的做法更接近传统工程流程：**先整理现场，再开始执行**。不追求潇洒，但通常更稳妥。

---

## 3.4 模型调用只是循环的一段

等前面的治理工作都做完，Claude Code 才进入模型调用阶段。这里有个关键细节：系统使用 `for await` 流式消费模型输出，而不是同步拿一个完整结果。

这意味着模型输出是一串**事件流**，而不仅仅是「最终答案」。事件里可能包含：
- Assistant 文本
- Tool use 块
- Usage 更新
- Stop reason
- API 错误

**从工程角度看，这是一种根本性的变化**。一旦把模型输出当成事件流，系统架构就不再只是「请求-响应」，而更像「驱动-调度-反馈」的过程。

流式输出的意义不只是更早看到几个字，而是允许运行时在模型尚未完全结束之前，就开始安排下一步执行。

> 这也是为什么说 Query Loop 才是代理系统的心跳，而不是模型调用本身。模型调用只是心跳中的一次收缩，真正维持系统运行的是整套循环。

---

## 3.5 必须处理中断

一个真正的心跳，不只是能持续跳动，还必须能在必要时停下来。停不下来，系统就只剩惯性。

Claude Code 对中断的处理很实在：系统会优先处理 streaming abort。如果启用了 `streamingToolExecutor`，就必须先消费剩余结果，生成 synthetic tool_result，避免已发出的 tool_use 没有配套结果。

**背后有一个很基础的工程原则**：只要系统向外承诺了一段执行，就要在中断时把账补平。不能因为用户打断了，就假装前面的 tool_use 从未发生。

> 代理系统一旦进入多工具、多轮次状态，外部世界对它的要求就不只是「有没有最终答案」，而是「它留下的轨迹能不能被解释」。不能解释的执行轨迹，迟早会变成运维问题、审计问题。

---

## 3.6 必须处理恢复

如果说中断是外部世界打进来的意外，那么恢复就是系统内部预留的余量。

Claude Code 对恢复的处理是**层层递进**的，而不是简单重试。以 prompt-too-long 为例：

1. 先判断最后一条 assistant message 是否是被 withheld 的 prompt-too-long
2. 再试图让 context collapse 把积压的 collapse 提交出去
3. 如果还不够，进入 reactive compact

系统按**成本和破坏性从低到高**，逐层尝试恢复。

对 max_output_tokens 的处理也一样：先尝试提升 token cap；如果还不行，再生成 meta message，让模型从被截断处继续往下做。

> 这很能说明 Claude Code 的设计态度：它把恢复看成运行时主路径的一部分，而不是模型失败后的礼貌动作。

---

## 3.7 停止条件不能只有一个

普通问答系统的停止条件简单：有回答就结束。代理系统不能这么偷懒。

因为一个会话里，「当前轮结束」不等于「任务完成」，更不等于「系统成功」。

Claude Code 的 Query Loop 至少区分了这些情况：

- Streaming 正常完成但有 tool_use，需要 follow-up
- 没有 tool_use，进入 stop hooks 和后续判定
- 被用户中断
- 触发 prompt-too-long 恢复
- 触发 max-output-tokens 恢复
- Stop hook 阻塞导致重进循环
- API 错误直接返回

> 许多系统只有一种朴素想法：失败了就重试。Claude Code 则承认，**重试本身也是一种需要被管理的行为**。系统必须知道为什么重试、已经试过什么、哪些保护状态不能被重置、哪些情况会导致无限循环。

---

## 3.8 QueryEngine 的职责

如果 QueryLoop 还不足以说明问题，那么 QueryEngine 的存在就更直接了。

源码明确写着：

> QueryEngine owns the query lifecycle and session state for a conversation.

这句话已经把重点说得很明确：QueryEngine 管理的是一个 conversation 的 query lifecycle，而不是某一次调用。

---

## 总结：第三个原则

这一章可以收敛成一句话：

> **代理系统的核心能力，是维持可恢复的执行循环。**

一个成熟 agent 的「心跳」至少满足：

1. **有明确的跨轮状态** —— 用显式 State 管理，而不是局部变量
2. **能治理输入** —— 运行时先于推理，不把责任推给模型
3. **能流式承接模型输出** —— 把输出当事件流，而非最终文案
4. **能补齐中断后的执行账本** —— 关心因果闭环
5. **能区分完成、失败、恢复和继续** —— 不把重试当成万能药

> 缺少这些结构的系统，也许仍然能做出漂亮 demo，但它们更接近**一次性表演**，而不是**运行时**。表演当然有价值，只是不能替代秩序。

---

## 延伸阅读

- [第 2 章：Prompt 不是人格，Prompt 是控制平面](https://harness-books.agentway.dev/book1-claude-code/chapter-02-prompt-is-control-plane.html)
- [第 4 章：工具、权限与中断](https://harness-books.agentway.dev/book1-claude-code/chapter-04-tools-permissions-interrupts.html)