---
layout: post
title: "Anthropic：面向长时间应用开发的 Harness 设计"
description: "Anthropic 通过 Planner、Generator、Evaluator 三代理架构，解决长时间自主编码中的上下文衰退、自我评估失真和复杂应用验证问题；核心启发是：Agent 能力越强，Harness 越要围绕真实瓶颈动态简化与重构。"
date: 2026-04-10
categories: anthropic
author: Prithvi Rajasekaran
source: anthropic.com/engineering
---

# Anthropic：面向长时间应用开发的 Harness 设计

> 来源: Anthropic Engineering | [原文链接](https://www.anthropic.com/engineering/harness-design-long-running-apps) | 发布日期: 2026-03-24

## 1. 文章基本信息

- **标题**: Harness design for long-running application development
- **来源**: Anthropic Engineering
- **链接**: https://www.anthropic.com/engineering/harness-design-long-running-apps
- **作者**: Prithvi Rajasekaran（Anthropic Labs 团队）
- **发布日期**: 2026-03-24
- **主题**: 长时间自主编码、Agent Harness、前端设计评估、Planner/Generator/Evaluator 多代理架构、Claude Agent SDK、Playwright MCP

## 2. 文章要解决的核心问题（论点）

这篇文章要解决的问题是：**如何让 Claude 这样的编码 Agent 在数小时无人干预的情况下，持续构建完整应用，并且避免上下文失控、自我评价过度乐观、复杂功能“看起来完成但实际不可用”的问题？**

一句话概括：**Anthropic 的答案不是单纯换更强模型，而是设计一个围绕模型短板的 Harness：用 Planner 扩展需求，用 Generator 分阶段实现，用 Evaluator 像真实用户一样测试，并通过可调的结构把长任务变成可评估、可迭代的工程流程。**

## 3. 问题所处的背景上下文

AI 编程 Agent 的早期价值，主要体现在“帮人完成局部任务”：补代码、修 bug、写测试、解释错误。但要让 Agent 从一句自然语言需求出发，连续工作数小时，构建一个接近完整的应用，问题就会明显变复杂。

文章指出 Anthropic 之前已经做过两类相关尝试：

1. **前端设计 Skill**  
   通过 prompt engineering 和设计原则，让 Claude 的前端输出质量高于默认水平。

2. **长时间编码 Agent Harness**  
   通过 initializer agent 将产品需求拆解成任务列表，再由 coding agent 一项项实现，并用结构化 artifact 在会话之间传递状态。

这些方法有效，但都碰到了天花板。尤其是在更复杂的应用构建中，Agent 会出现两个反复发生的失败模式：

- **上下文问题**：长任务中上下文窗口逐渐被填满，模型失去连贯性，甚至在接近上下文上限前过早收尾，即所谓 “context anxiety”。
- **自我评估问题**：Agent 评价自己产出的东西时，常常过于宽容；哪怕人类明显能看出质量一般，它也会自信地说“很好”。

这篇文章的核心贡献，就是围绕这两个瓶颈设计新的多代理 Harness。

## 4. 文章的核心观点

### 4.1 Naive Agent 不适合长时间复杂应用开发

作者认为，直接让一个 Agent 长时间连续开发完整应用，会遇到两类根本性问题。

第一类是**上下文衰退**。随着任务推进，对话历史越来越长，里面混入旧推理、失败尝试、过期代码状态和大量中间细节。模型会逐渐偏离任务，也可能因为感知到上下文接近上限而提前结束。

第二类是**自我评估失真**。模型生成代码或设计之后，再让它自己判断质量，往往会过度正面。这个问题在主观任务上尤其明显，例如“这个设计是否好看”；但在可验证任务上也存在，例如某个功能其实没有真正实现，只是 UI 上看起来存在。

所以，长时间 Agent 不是“把 prompt 写长一点”就能解决，而需要外部 Harness 来承担分解、监督、验证和反馈。

### 4.2 Context Reset 与 Compaction 的区别很关键

文章区分了两种处理长上下文的方式：

- **Compaction**：把早期对话总结压缩，让同一个 Agent 在较短历史上继续。
- **Context Reset**：彻底清空上下文，启动新的 Agent，通过结构化 handoff artifact 传递状态和下一步。

作者认为，compaction 保留连续性，但不能完全消除 context anxiety；reset 则给 Agent 一个真正干净的上下文，但代价是 handoff artifact 必须足够完整，否则下一轮无法无缝接上。

在 Claude Sonnet 4.5 的早期实验中，context anxiety 足够明显，以至于 compaction 不够，context reset 成了 Harness 的关键设计。但后来 Opus 4.5/4.6 能力提升后，某些场景又可以减少甚至移除 reset。这说明 Harness 不是固定结构，而是对当前模型能力缺口的补偿。

### 4.3 Generator-Evaluator 分离是解决自评问题的核心杠杆

文章借鉴 GAN（生成对抗网络）的思想，把“生成”和“评估”拆成两个角色：

- **Generator**：负责产出设计或实现代码。
- **Evaluator**：负责独立评估输出质量，并给出具体反馈。

重要的是，Evaluator 仍然是 LLM，也会天然偏宽容；但相比让 Generator 批判自己，调教一个独立 Evaluator 变得更“挑剔”要容易得多。一旦外部反馈足够具体，Generator 就有了可迭代的方向。

这也是文章最值得借鉴的结构性观点：**不要让做事的 Agent 同时拥有最终验收权。**

### 4.4 主观质量可以通过标准变得“可评分”

作者先在前端设计任务上做实验，因为设计质量最能暴露自评问题。Claude 默认容易生成安全、规整但平庸的界面。为了让“好设计”可评估，作者设计了四个评分标准：

1. **Design quality**：颜色、排版、布局、图像等是否形成整体氛围和身份。
2. **Originality**：是否有定制化决策，而不是模板、库默认样式或典型 AI slop。
3. **Craft**：技术执行，包括层级、间距、色彩和对比度。
4. **Functionality**：用户是否能理解并完成任务。

作者刻意提高 design quality 和 originality 的权重，因为 Claude 默认在 craft 和 functionality 上已经相对不错，但在原创性和审美风险上偏保守。

这带来一个很重要的启发：**即使是审美这种主观任务，也可以通过明确原则、few-shot 校准和 evaluator 反馈，转化为可迭代优化的目标。**

### 4.5 三代理架构：Planner、Generator、Evaluator

在完整应用开发任务中，作者最终设计了三代理系统：

| Agent | 作用 |
|---|---|
| Planner | 将 1-4 句话的用户需求扩展成完整产品规格，偏产品上下文和高层技术设计，不写过细实现细节 |
| Generator | 按照规格实现应用，早期版本按 sprint 一项项开发，并在每个 sprint 后交给 QA |
| Evaluator | 使用 Playwright MCP 像真实用户一样操作应用，检查 UI、API、数据库状态，并按标准打分 |

这里 Planner 的设计很有意思：作者不希望 Planner 过早写出细粒度技术实现，因为一旦 spec 细节错了，错误会级联到后续开发。Planner 更应该定义交付物、产品范围和高层方向，把具体路径留给 Generator 在开发中决定。

### 4.6 Sprint Contract 是把高层需求变成可测试实现的桥梁

在早期版本里，每个 sprint 开始前，Generator 和 Evaluator 会协商一个 **sprint contract**：

- Generator 提出本轮要实现什么；
- 如何证明完成；
- Evaluator 检查这个定义是否覆盖用户故事和可测试行为；
- 双方迭代到达成一致后，Generator 才开始写代码。

这一步非常关键。因为 Planner 的 spec 有意保持高层，如果没有 sprint contract，Generator 可能“自以为理解了”但实际实现偏题。Contract 把模糊需求转成可验证条款，是长任务保持方向的锚点。

### 4.7 Harness 必须随着模型进步而简化

作者后来用 Opus 4.6 重新审视 Harness。因为新模型更擅长长任务、代码审查、调试和长上下文检索，原来为 Sonnet 4.5 / Opus 4.5 设计的一些脚手架可能不再必要。

例如，作者尝试移除 sprint 结构，让 Builder 连续工作更久，只在最后由 Evaluator 做几轮 QA。结果在 DAW（浏览器数字音频工作站）示例中，Builder 能连续运行两个多小时并保持相当连贯。

这说明：**Harness 的每个组件都代表一个关于“模型做不到什么”的假设。模型升级后，这些假设必须重新验证。**

## 5. 观点对应的论据

### 5.1 前端设计循环：Evaluator 用 Playwright MCP 实际访问页面

在前端设计实验中，Generator 先根据用户提示生成 HTML/CSS/JS 页面。Evaluator 获得 Playwright MCP，可以真实打开页面、浏览、截图、观察交互，然后对四个标准打分并写出批评意见。

作者运行 5 到 15 次迭代，每一轮反馈再给 Generator。结果是：

- 输出通常逐步变得更有辨识度；
- 中间某些版本有时比最后版本更好；
- 复杂度会随迭代增加；
- 评分标准本身的措辞会强烈塑造输出风格。

其中一个例子是“荷兰艺术博物馆网站”：第 9 轮还是一个精致但符合预期的暗色 landing page；第 10 轮突然转成 3D 房间式空间体验，用 CSS perspective 渲染展厅和墙上作品。这说明 evaluator feedback 有时能推动模型跳出单轮生成的保守模式。

### 5.2 游戏制作器对比：单 Agent 20 分钟 vs 完整 Harness 6 小时

作者用同一句 prompt 生成 “2D retro game maker”：

> Create a 2D retro game maker with features including a level editor, sprite editor, entity behaviors, and a playable test mode.

对比结果：

| 方案 | 时长 | 成本 | 结果 |
|---|---:|---:|---|
| 单 Agent | 20 分钟 | $9 | 界面看似符合预期，但核心 playable test mode 不工作 |
| 完整 Harness | 6 小时 | $200 | 规格扩展到 16 个功能、10 个 sprint，应用更完整，角色可移动并能玩，仍有物理和工作流问题 |

关键差异不是“界面更漂亮”，而是核心功能是否真的可用。单 Agent 做出了一个看起来像游戏编辑器的应用，但运行时实体不响应输入；完整 Harness 至少让 playable mode 工作起来。

### 5.3 Evaluator 能发现非常具体的实现 bug

文章列出 Evaluator 捕捉到的具体问题，例如：

- 矩形填充工具只在拖拽起点/终点放 tile，而不是填满区域；
- 删除实体 spawn point 的按键逻辑条件错误；
- FastAPI 路由顺序导致 `/frames/reorder` 被当成 `{frame_id}` 解析，返回 422。

这些不是泛泛而谈的“质量不好”，而是可以直接定位到文件、条件和路由匹配的问题。说明 Evaluator 通过 Playwright 和 contract criteria，确实能提供可执行反馈。

### 5.4 DAW 示例：简化后的 Harness 仍能构建复杂应用

在 Opus 4.6 版本中，作者移除 sprint construct，用更简化 Harness 构建浏览器 DAW：

> Build a fully featured DAW in the browser using the Web Audio API.

运行结果：

- 总时长约 3 小时 50 分钟；
- 成本约 $124.70；
- Planner 只花约 4.7 分钟；
- Builder 第一轮连续运行 2 小时 7 分钟；
- QA 多轮发现缺失功能，例如录音仍是 stub、clip resize/split 未实现、EQ 曲线等效果可视化不完整。

最终应用不是专业 DAW，但具备 arrangement view、mixer、transport 等核心组件，并且内置 Agent 能通过工具设置 tempo/key、生成 melody、鼓轨、混音和 reverb。

这证明模型进步后，Harness 可以更轻，但 Evaluator 对最后一公里的功能完整性仍然有价值。

## 6. 我的评价和启发

### 6.1 这篇文章最重要的不是“三代理”，而是“找到模型当前的真实瓶颈”

表面上看，文章在介绍 Planner、Generator、Evaluator 架构。但更深层的方法论是：**先观察模型在真实任务 trace 中怎么失败，再设计 Harness 补那个失败。**

- 如果模型会过早收尾，就做 context reset。
- 如果模型自评太宽，就拆出 evaluator。
- 如果模型 under-scope，就加 planner。
- 如果模型在高层 spec 到实现之间跑偏，就加 sprint contract。
- 如果新模型已经能长时间保持连贯，就移除过重的 sprint/reset。

这比固定套用某个“多代理框架”更重要。很多团队容易一上来就堆复杂 orchestrator，但如果不知道每个组件解决什么失败模式，Harness 很快会变成成本高、延迟大、难调试的黑箱。

### 6.2 我非常认同“Evaluator 需要调教”，不能默认相信 LLM QA

文章里有一句经验很真实：Claude 开箱作为 QA agent 并不好。它会发现问题，然后又说服自己“这不严重”，最后批准。它也容易只做表面测试，漏掉深层交互。

这对所有 Agent 系统都很关键：**Verifier / Evaluator 不是天然可靠的裁判，它也是一个需要数据、标准、few-shot 和失败案例校准的模型组件。**

如果我们要在自己的项目中用 evaluator，我会要求它：

- 必须先运行测试和实际操作，再发表判断；
- 输出 PASS/FAIL 必须附证据；
- 对每个验收条件逐条对应；
- 不允许用“overall good”覆盖具体失败；
- 对 stub、mock、display-only 功能保持高敏感。

否则 evaluator 可能只是把“自我感觉良好”换成“第二个模型感觉良好”。

### 6.3 Sprint Contract 是最值得借鉴到日常开发的机制

我认为文章里最实用、最可迁移的设计是 sprint contract。它解决的是 AI 编码里最常见的问题：用户说的是目标，Agent 写的是它自以为的实现，中间缺少验收协议。

在实际项目中，可以把每个 Agent 子任务都改成先生成 contract：

```markdown
## 本轮目标

## 不做什么

## 验收条件

## 需要运行的测试/检查

## 风险和依赖
```

然后让 verifier 先审 contract，再允许 builder 写代码。这比事后发现“你理解错了”成本低得多。

### 6.4 成本数字提醒我们：长时间 Agent 仍然是昂贵实验，不是免费劳动力

游戏制作器完整 Harness 花了 6 小时和 $200；DAW 花了近 4 小时和 $124.70。这些数字很重要，因为它们把“自主 Agent”从演示拉回工程现实。

长期无人值守编码有价值，但必须用于足够高价值的任务，并且需要：

- 成本上限；
- 最大运行时间；
- 中间检查点；
- 失败时停止策略；
- 产出可复用的 trace 和经验。

否则它可能只是昂贵地生成一堆你还要重写的代码。

### 6.5 文章也暴露了 Agent 应用构建的“表面完整性”问题

不管是 retro game maker 还是 DAW，Agent 很容易做出视觉上像产品的界面，但很多核心交互可能是 stub 或 display-only。比如 DAW 中 clip 不能拖动、录音按钮只是切换状态、效果器没有真实图形编辑。

这说明 Evaluator 不能只检查“页面上有没有按钮”，而要检查“按钮是否改变了真实状态、是否驱动真实后端、是否完成核心用户任务”。

对复杂应用来说，验证标准应该从 UI 元素存在，升级到端到端行为：

- 用户能否完成任务？
- 数据是否真的落库？
- 状态是否跨刷新保留？
- API 是否处理错误输入？
- 关键功能是否不是 stub？

### 6.6 对我们理解 Loop Engineering / Harness Engineering 的启发

这篇文章和前面提到的 Loop Engineering 其实是同一趋势的两个侧面：

- Loop Engineering 关注“系统如何持续驱动 Agent”；
- Harness Engineering 关注“外部结构如何弥补模型当前能力缺口”。

二者共同指向一个结论：未来 AI 工程师的核心能力，不只是会用模型，而是会设计模型外部的执行系统：状态、验证、权限、调度、反馈、成本、回滚。

## 7. 可执行落地建议

如果要把这篇文章的方法用于自己的项目，我建议从以下步骤开始：

1. **先收集失败 trace**  
   不要先设计复杂多代理系统。先让 Agent 做真实任务，观察它到底在哪里失败：under-scope、上下文混乱、自评过宽、不会测试、还是实现细节跑偏。

2. **为每类失败加一个最小 Harness 组件**  
   - under-scope：加 Planner；
   - 自评失真：加 Evaluator；
   - 长上下文衰退：加 context reset + handoff；
   - 需求到实现断层：加 sprint contract；
   - 测试不足：加 Playwright/pytest/CI 工具调用。

3. **Evaluator 要有硬规则**  
   让 Evaluator 按验收条件逐条判断，并要求引用证据。禁止只输出笼统评价。

4. **把主观标准显式化**  
   如果任务涉及设计、文案、产品体验，就把“好”的标准写出来，并用 few-shot 校准。

5. **用文件做 Agent 间通信**  
   让 Planner spec、sprint contract、QA report、bug list 都落盘。不要让关键状态只存在对话里。

6. **每次模型升级后重新压测 Harness**  
   删除不再 load-bearing 的组件，避免旧脚手架拖累新模型。

7. **限制成本和时长**  
   长任务要设预算、最大轮数和中间检查点。能 20 分钟暴露问题，就不要先跑 6 小时。

## 8. 延伸阅读

- Anthropic: Building effective agents
- Anthropic: Context engineering for agents
- Anthropic: Claude Agent SDK
- Anthropic: Frontend design skill
- Model Context Protocol / Playwright MCP
- Geoffrey Huntley: Ralph Wiggum method
- Addy Osmani: Loop Engineering / Agent Harness Engineering

## 9. 总结

这篇 Anthropic 工程文章的核心价值在于，它没有把长时间 Agent 编程包装成“模型自己就能做完一切”，而是非常工程化地展示了：**要让 Agent 连续数小时构建复杂应用，必须围绕模型失败模式设计外部 Harness。**

Planner 解决需求扩展和范围控制；Generator 负责实现；Evaluator 负责像真实用户一样测试；sprint contract 把高层需求转成可验收条款；context reset 和 handoff 处理长上下文问题；随着模型进步，Harness 又必须被重新简化。

真正的启发是：**Agent Harness 不是越复杂越好，而是要刚好补上当前模型做不到的那一块。模型变了，Harness 也要变。**
