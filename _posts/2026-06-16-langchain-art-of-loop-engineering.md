---
layout: post
title: "LangChain《The Art of Loop Engineering》解读：Agent 的价值在循环，而不只在模型"
description: "LangChain 将 Agent 系统拆成四层循环：工具调用、验证反馈、事件触发和基于 traces 的持续改进。文章的关键启发是，生产级 Agent 的竞争力来自围绕模型构建的工程循环。"
date: 2026-06-16
created_at: 2026-07-09 07:50:00 +0800
categories: AI Agents
---

# LangChain《The Art of Loop Engineering》解读：Agent 的价值在循环，而不只在模型

> 来源: LangChain Blog | [原文链接](https://www.langchain.com/blog/the-art-of-loop-engineering) | 发布日期: 2026-06-16

## 1. 文章基本信息

- **标题**: The Art of Loop Engineering
- **来源**: LangChain Blog
- **链接**: https://www.langchain.com/blog/the-art-of-loop-engineering
- **作者**: Sydney Runkle
- **发布日期**: 2026-06-16
- **阅读时间**: 约 7 分钟
- **主题**: AI Agent、Loop Engineering、Agent Harness、验证循环、事件驱动、持续优化

## 2. 文章要解决的核心问题（论点）

这篇文章试图回答的问题是：**如果一个 Agent 要稳定地完成真实世界任务，除了“让模型调用工具”之外，还需要在它外面设计哪些循环？**

一句话概括：**可靠 Agent 不是一个单独的 LLM + Tools 调用，而是一组层层嵌套的工程循环：执行、验证、触发、学习改进。**

## 3. 问题所处的背景上下文

过去很多人谈 Agent，重点放在“模型是否足够聪明”“工具是否足够多”“Prompt 是否写得好”。LangChain 这篇文章的切入点不同：它把 Agent 看成一个运行在真实系统里的工程组件，而不是一次性的聊天交互。

这个背景下有几个现实问题：

1. **单次 Agent 运行不可靠**  
   LLM 可以规划、调用工具、生成结果，但第一版输出经常有遗漏、越界修改、格式错误、测试失败等问题。

2. **生产环境需要持续运行，而不是人工点击执行**  
   真实业务里的 Agent 往往应该由 Slack 消息、Webhook、定时任务、新文档、新 Issue 等事件触发，而不是每次都由人手动打开聊天窗口。

3. **Agent 的改进信号来自运行轨迹**  
   每次运行都会产生 traces：模型看到什么、调用了什么工具、哪里失败、grader 给了什么反馈。这些不是日志噪音，而是优化 Agent harness 的高价值数据。

4. **行业共识正在从 Prompt Engineering 走向 Loop Engineering**  
   文章呼应 swyx 的 “loopcraft: the art of stacking loops”：Agent 能力的关键不只是一次提示词，而是围绕模型堆叠怎样的循环。

## 4. 文章的核心观点

### 4.1 第一层：Agent Loop —— 模型调用工具直到任务完成

最内层循环就是经典 Agent：给模型上下文，让它反复调用工具，直到认为任务完成。LangChain 对应的实现是 `create_agent`：选择模型、接入工具，就得到一个基础执行循环。

文章用“内部文档 Agent”举例：用户提出文档改进请求，Agent 可以 clone repo、读文件、修改文档、开 PR。这一层的价值是**自动化工作本身**。

但这一层还不等于生产可用。因为“能做事”和“稳定正确地做事”之间还有距离。

### 4.2 第二层：Verification Loop —— 用评分器把失败反馈回去

第二层是在 Agent 外面包一个验证循环：Agent 产出结果后，由 grader 根据 rubric 检查；如果不合格，就把失败原因反馈给模型，让它重试。

grader 可以是确定性的，例如运行测试、检查链接、验证格式；也可以是 agentic 的，例如 LLM-as-a-judge。LangChain 对应的组件包括 `RubricMiddleware`，也可以用 `create_agent` 的 `after_agent` hook 自己接。

文档 Agent 的例子里，grader 会检查：

- 所有链接是否可访问；
- CI 是否通过；
- diff 是否只改了用户请求范围内的内容。

这一层的价值是**把质量要求从“人工事后检查”前移到系统循环中**。代价是延迟和成本增加，因此适合质量比速度更重要的生产场景。

### 4.3 第三层：Event-driven Loop —— 让 Agent 成为系统里的常驻组件

第三层是事件驱动循环：不是人手动调用 Agent，而是由外部事件触发它运行。事件可以是新文档、新消息、cron schedule、webhook 等。

LangChain 提到的基础设施包括：

- LangSmith Deployment 的 cron triggers / webhooks；
- Fleet 的 channels 和 schedules；
- openclaw 中类似 “heartbeats” 的常驻主动助手模式。

文档 Agent 的例子里，团队在 Slack 的 `#docs-plz` 频道发消息，Fleet channel 会触发 Agent 处理文档请求。

这一层的价值是**规模化自动化**：Agent 不再只是一个工具，而是业务系统里的后台 worker。

### 4.4 第四层：Hill Climbing Loop —— 用运行轨迹自动改进 Agent Harness

第四层是文章最重要的观点：前三层让 Agent 自动工作，第四层让 Agent **自动变好**。

每次 Agent 运行都会产生 trace，里面包含：

- 模型执行路径；
- 工具调用；
- 中间观察；
- grader 反馈；
- 成败结果。

Hill climbing loop 会用分析 Agent 读取这些 traces，发现重复问题，并修改 harness 配置，例如 prompt、工具描述、grader rubric、上下文策略等。LangChain 对应的产品是 LangSmith Engine。

关键点在于：外层循环不是简单“再跑一次 Agent”，而是**伸进内层，改写 Agent 的配置**。每次循环都让内层循环更有效。

文章还指出，未来可优化的不只是 prompt 和工具配置。对于 open-weight models，trace/eval 结果还可以作为 RL fine-tuning 的训练信号；memory、retrieved skills、辅助上下文也可以被同样的循环优化。

### 4.5 人类仍然在关键节点保留判断权

文章并不主张“完全无人值守”。相反，它强调每一层循环都有适合人类介入的位置：

- Agent loop：敏感工具调用前要求人工确认；
- Verification loop：敏感 workflow 中让人类充当 grader；
- Application loop：结果返回用户前由人审核；
- Hill climbing loop：harness 改进在部署前走人工 review。

这点很重要：自动化能检查链接是否失效，但“这篇文档的表达是否符合目标读者的认知方式”仍然需要人的经验、语境和品味。

## 5. 观点对应的论据

文章的论据主要来自一个贯穿全文的内部文档 Agent 例子，以及 LangChain 自身产品栈中的对应原语。

| 循环层级 | 做什么 | 带来的影响 | LangChain / LangSmith 原语 |
|---|---|---|---|
| Agent loop | 模型反复调用工具直到任务完成 | 自动化具体工作 | `create_agent`、任意 LangChain 支持的模型和工具 |
| Verification loop | 按 rubric 评分，不合格则反馈重试 | 提高正确性和一致性 | `RubricMiddleware`、`after_agent` hook |
| Event-driven loop | 由 cron、webhook、频道消息等触发运行 | 让 Agent 在系统中规模化运行 | LangSmith Deployment、Fleet channels/schedules |
| Hill climbing loop | 分析 traces 并改进 harness 配置 | 让 Agent 持续变好 | LangSmith Engine |

它的论证方式不是给出大规模 benchmark，而是提出一个工程框架：如果你要把 Agent 从 demo 推到生产，就要逐层补齐这些循环。

## 6. 我的评价和启发

### 6.1 我认同：Agent 的护城河越来越像“循环系统”，而不是单个模型调用

这篇文章最有价值的地方，是把 Agent 工程从“模型中心视角”拉回“系统中心视角”。很多 Agent demo 看起来失败，并不是因为模型完全不行，而是因为它缺少：

- 可验证的完成标准；
- 失败后的反馈机制；
- 和真实事件源的连接；
- 从运行数据中学习的闭环。

这也解释了为什么同一个模型，在不同 harness 里表现差异巨大。模型是发动机，但循环系统决定它能否持续、安全、可控地输出业务价值。

### 6.2 文章偏产品化，但框架本身很实用

需要注意的是，文章来自 LangChain，因此会自然地把每层循环映射到 LangChain / LangSmith 产品：`create_agent`、`RubricMiddleware`、Deployment、Fleet、Engine。这个映射有推广意味。

但即便不用 LangChain，这个四层框架仍然成立：

- Agent loop 可以用 Claude Code、Codex、OpenAI Assistants、自研 orchestrator；
- Verification loop 可以用 pytest、schema validation、lint、LLM judge、人工审查；
- Event-driven loop 可以用 cron、GitHub Actions、Slack bot、队列系统；
- Hill climbing loop 可以用日志分析、eval dashboard、自动 issue 生成、prompt 版本管理。

所以它的真正价值不在于“必须买某个产品”，而在于给 Agent 系统设计提供了一张清晰地图。

### 6.3 第四层 Hill Climbing 是最值得关注、也最容易被低估的部分

很多团队已经能做第一层和第二层：让 Agent 调工具、跑测试、失败重试。第三层也不难，用 cron/webhook 就能把它接入业务。

真正难的是第四层：如何把运行轨迹变成系统性改进？这里至少有三个挑战：

1. **信号提取**：trace 很多，但哪些失败代表 harness 问题，哪些只是偶发错误？
2. **改进归因**：失败到底该改 prompt、工具、检索上下文、权限边界，还是模型本身？
3. **安全部署**：自动生成的 harness 修改不能直接进生产，需要版本管理、eval gate 和人工 review。

这也是我认为未来 Agent 平台真正分化的地方：不是谁能发起一次 Agent run，而是谁能把成千上万次 run 变成可控的学习循环。

### 6.4 对个人和团队的实践启发

如果把这篇文章落到实践，我会建议按以下顺序建设 Agent：

1. **先定义任务边界和工具权限**：让 Agent 做少数高频、可验证的任务，而不是“什么都能做”。
2. **立刻加入验证循环**：哪怕只是运行测试、检查链接、检查输出 schema，也比纯靠模型自信结束要可靠得多。
3. **再接事件触发**：当单次运行质量可控后，再把它接到 cron、Slack、GitHub webhook 等入口。
4. **最后建设 trace → insight → harness change 的改进闭环**：先人工分析 traces，形成常见失败模式，再逐步半自动化。

也就是说，Loop Engineering 的成熟路径不是一上来追求“全自动”，而是先让每一层循环都有明确的输入、输出、评价标准和回滚机制。

## 7. 延伸阅读

- [The Art of Loop Engineering 原文](https://www.langchain.com/blog/the-art-of-loop-engineering)
- [swyx: loopcraft / stacking loops](https://www.latent.space/)
- [LangChain `create_agent` 文档](https://docs.langchain.com/)
- [LangGraph](https://www.langchain.com/langgraph) - 构建可控 Agent 状态机和工作流
- [LangSmith](https://www.langchain.com/langsmith) - Agent tracing、evaluation、deployment 平台
- [OpenClaw](https://github.com/langchain-ai/openclaw) - 文中提到的 heartbeats / proactive assistant 示例方向

## 8. 一句话总结

**Loop Engineering 的核心是：不要只优化 Agent 的一次回答，而要设计它如何执行、如何被验证、如何被触发、如何从失败中持续改进。**
