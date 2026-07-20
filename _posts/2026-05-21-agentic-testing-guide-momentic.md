---
layout: post
title: "Agentic Testing for Engineering Teams：AI 编码时代的软件测试新范式"
description: "Momentic 这篇文章把 Agentic Testing 定义为由 AI Agent 直接规划、执行、观察并自适应调整的软件测试方式，核心判断是：AI coding 的普及会把工程瓶颈从写代码转移到验证代码，因此 AI verification 会成为下一阶段软件工程基础设施。"
date: 2026-05-21
created_at: 2026-07-21 07:51:50 +0800
categories: AI-Agent Testing Software-Engineering
source: Momentic Blog
author: Wei-Wei Wu
---

# Agentic Testing for Engineering Teams：AI 编码时代的软件测试新范式

> 来源: Momentic Blog | [原文链接](https://momentic.ai/blog/agentic-testing-guide) | 发布日期: 2026-05-21

## 1. 文章基本信息

- **标题**: Agentic Testing for Engineering Teams: A Definitive Guide (Part 1 of 2)
- **来源**: Momentic Blog
- **链接**: <https://momentic.ai/blog/agentic-testing-guide>
- **作者**: Wei-Wei Wu
- **发布日期**: 2026-05-21
- **主题**: Agentic Testing、AI 软件测试、端到端测试、AI Coding Agent、软件质量保障

## 2. 文章要解决的核心问题（论点）

这篇文章试图回答的问题是：**当 AI coding agents 大规模进入软件开发流程、代码和 UI 变化速度显著提高后，传统基于脚本和 selector 的自动化测试是否还能承担主要质量保障职责？如果不能，新的测试范式应该是什么？**

作者的核心论点可以概括为：**AI coding 需要 AI verification。** 当 AI 帮助团队更快地产生代码，软件工程的瓶颈会从“写代码”转移到“验证代码”。因此，测试也需要从人工编写固定脚本，演进到由 AI agent 根据自然语言目标自主规划、执行、观察并调整的 Agentic Testing。

## 3. 问题所处的背景上下文

### 3.1 AI 正在提高代码变化速度

文章的出发点是：AI coding tools 已经不再是边缘工具，而是工程团队日常开发的一部分。作者引用的趋势包括：

- 大量开发者已经在工作中使用 AI 工具。
- GitHub Copilot 用户规模快速增长。
- 相当比例的代码开始由 AI 辅助生成。
- 更多团队计划在 2026 年内采用 AI coding agents。

这意味着软件系统的变更频率会继续提高。组件结构、CSS class、DOM 层级、页面流程、表单字段和交互细节都可能因为 AI 辅助重构而更频繁变化。

### 3.2 传统自动化测试的脆弱性被放大

传统 E2E 测试通常依赖确定性的脚本和 selector，例如 Selenium、Cypress、Playwright 等工具链。它们的优势是可复现、可调试、适合 CI；但它们的弱点也很明显：

- UI 结构变化会导致 selector 失效。
- 页面文案、按钮位置、表单字段变化会造成脚本失败。
- 测试维护成本会随着产品迭代速度上升。
- Flaky tests 会降低团队对自动化测试的信任。

文章认为，在 AI coding agents 加速代码变化后，这类维护成本会被进一步放大。很多所谓 flakiness 并不是随机失败，而是 UI churn 造成的确定性断裂：测试脚本依赖的实现细节已经变了，但用户目标本身并没有变。

### 3.3 验证会成为 AI 软件工程的关键瓶颈

AI 让“生成代码”变得更便宜，但没有自动让“代码正确”变得更便宜。相反，AI 生成代码经常处在一种危险状态：**almost right, but not quite**。

这就要求工程系统拥有更强的反馈机制。没有验证，agent 可能继续基于错误假设行动；有高质量验证，agent 才能知道自己的修改是否真的满足用户需求。

因此，测试不再只是质量保障部门的后置环节，而会成为 AI agent 工作循环中的核心 reality check。

## 4. 文章的核心观点

### 4.1 什么是 Agentic Testing

文章将 Agentic Testing 定义为：**由 AI agent 执行的软件测试。** 用户不再需要为每一步写死 selector 和操作路径，而是用自然语言描述测试目标；agent 自己规划路径、操作应用、读取运行时信号，并根据页面状态变化调整下一步动作。

一个典型目标可能是：

> 验证一个新用户可以完成注册并进入 dashboard。

传统测试会写成一组确定性脚本：打开页面、点击某个 selector、输入固定字段、断言某个 DOM 节点存在。Agentic Testing 则更接近人类测试员：理解目标、观察页面、判断下一步该做什么，并在遇到变化时重新规划。

### 4.2 Agentic Testing 的四步循环

文章将 Agentic Testing 的工作方式概括为四个环节：

1. **Plan：规划**  
   Agent 将自然语言测试目标拆解为可执行步骤。例如从“验证新用户注册成功”拆成访问注册页、填写表单、提交、确认进入 dashboard 等动作。

2. **Run：执行**  
   Agent 在真实浏览器或应用环境中执行动作。它不是简单执行预先写死的 selector，而是基于页面语义识别按钮、输入框、导航链接等目标元素。

3. **Observe：观察**  
   每一步之后，agent 会读取多种运行时信号，包括 DOM、Accessibility tree、截图、网络请求、console logs、browser state 等。

4. **Adapt：调整**  
   如果页面状态和预期不同，agent 不会立刻因为某个 selector 失效而失败，而是重新观察并尝试调整路径。例如注册表单新增了 “Full name” 必填字段，agent 可以发现字段缺失并补填，而不是直接报错。

这四步让测试从“静态脚本执行”变成“目标导向的闭环验证”。

### 4.3 Agentic Testing 不是什么

文章花了不少篇幅区分 Agentic Testing 和其他相近概念，这一点很重要。作者强调，Agentic Testing 不是：

- 传统 Selenium / Cypress / Playwright 这类基于手写脚本和 selector 的自动化测试。
- 让 AI 生成 Playwright / Cypress 脚本。
- Self-healing selectors。
- 视觉回归测试。
- 单纯让 LLM 写一段测试代码，再由确定性 runner 执行。

这些工具或方法仍然围绕“脚本”展开，只是在脚本生成或 selector 修复上加了一层 AI。而 Agentic Testing 的核心区别在于：**测试执行本身由 agent 进行，agent 在执行过程中持续观察和决策。**

### 4.4 AI coding 需要 AI verification

全文最重要的判断是：**AI coding 需要 AI verification。**

如果代码越来越多由 AI agent 生成，那么验证机制也必须进入 agent 的运行循环。否则，团队可能得到更多代码、更快 PR、更频繁变更，但质量保障能力没有同步提升，最终只会把问题转移到 review、QA 和线上事故中。

这和大模型训练中的可验证奖励机制有相似之处：模型生成候选解，编译器、测试用例、运行结果提供 reward 或 feedback。测试信号越清晰、越可执行，coding agent 越容易知道自己是否走在正确方向上。

## 5. 观点对应的论据

### 5.1 AI 编码普及带来 UI 和代码结构的高频变化

文章引用行业数据说明 AI coding 工具采用率正在迅速提高，包括开发者使用率、Copilot 用户规模、AI 生成代码比例，以及团队计划采用 coding agents 的比例。作者用这些趋势支撑一个判断：未来软件系统的变化速度会更快，测试系统也必须适应这种变化速度。

这里的关键不是某个具体数字，而是趋势本身：当 AI 降低代码生产成本，工程组织的瓶颈自然会转移到验证、集成、调试和治理。

### 5.2 Flaky tests 与维护成本已经是现实问题

文章也引用了 flaky tests 相关研究和行业报告，说明测试不稳定已经消耗团队大量时间。虽然不同研究给出的比例不同，但共同指向一个问题：当测试失败不再被团队信任，自动化测试就会失去作为质量门禁的价值。

作者进一步把这个问题放到 AI coding 背景下：如果 UI churn 更频繁，传统测试脚本的脆弱性会更突出，测试维护会成为制约 AI 编码效率释放的瓶颈。

### 5.3 多信号观察比单一 selector 更接近真实测试

传统脚本通常依赖 DOM selector 和固定断言；Agentic Testing 则强调读取多种信号：页面文本、可访问性树、截图、网络请求、console 错误、浏览器状态等。

这让测试判断更接近真实用户行为。例如用户并不关心按钮的 CSS class 是否叫 `.submit-btn`，只关心是否能完成注册、是否看到 dashboard、是否出现错误提示。Agentic Testing 试图让测试目标从实现细节回到用户行为。

## 6. 我的评价和启发

### 6.1 我认同的部分

我认同文章最核心的判断：**AI 编码时代，验证会成为更重要的瓶颈。**

过去软件工程的主要约束常常是人类开发者写代码的速度；AI coding tools 出现后，代码生成速度提高，但正确性、可维护性、系统理解和质量保障并不会自动提高。甚至可以说，AI 让“产出更多待验证代码”变得更容易了。

因此，未来工程团队不能只投资 coding agent，也要投资 verification agent。测试、静态分析、运行时观测、代码审查、类型系统、CI/CD 这些反馈机制，都会成为 AI 软件工程的基础设施。

### 6.2 文章的营销成分

也需要注意，Momentic 本身提供 AI-powered automated testing platform，所以这篇文章有明显的品类定义和市场叙事成分。它将 Agentic Testing 描述为 generational shift，这个说法有合理性，但也带有供应商推动新类别的色彩。

因此，阅读时不能只接受“Agentic Testing 会替代传统测试”的叙事，而应该追问更具体的工程问题：

- Agentic Testing 是否真的比 Playwright + 良好测试设计更可靠？
- Agent 判断行为正确性的依据是否稳定？
- 测试结果是否可复现？
- Agent 出错时是否容易 debug？
- 成本和速度能否适应 CI？
- 自然语言测试目标是否会引入歧义？
- 对安全、权限、支付等高风险流程，agent 自主调整是否可接受？

这些问题决定了 Agentic Testing 是成为主流测试基础设施，还是主要停留在 smoke test / exploratory testing / demo 场景。

### 6.3 更可能的未来：分层互补，而不是完全替代

我不认为 Agentic Testing 会完全替代传统测试。更现实的形态是分层互补：

适合 Agentic Testing 的场景包括：

- 端到端用户流程验证。
- UI 经常变化的产品。
- Smoke test。
- Exploratory testing。
- AI agent 生成代码后的快速验证。
- 多信号综合判断的业务行为测试。
- 自然语言描述更清晰的验收测试。

不适合完全交给 agent 的场景包括：

- 单元测试。
- 精确协议测试。
- 性能测试。
- 安全测试。
- 确定性强、成本敏感的核心回归测试。
- 需要严格可复现和可审计的测试场景。

换句话说，Agentic Testing 更像是在传统测试金字塔之外新增一层“目标导向的智能验证层”，而不是替代所有已有测试工具。

### 6.4 对工程团队的启发

这篇文章对工程团队最直接的启发有三点：

1. **不要只衡量 AI 写了多少代码，也要衡量 AI 代码如何被验证。**  
   如果没有验证闭环，AI coding 的收益可能被 review 成本、调试成本和线上风险抵消。

2. **测试用例应该从实现细节转向行为目标。**  
   无论是否使用 Agentic Testing，团队都应该减少对脆弱 selector、DOM 结构和样式细节的依赖，更多表达用户可观察行为。

3. **Accessibility tree 会变得更重要。**  
   AI agent 理解页面往往依赖语义结构。Semantic HTML、form label、aria-label、heading hierarchy 等不仅影响无障碍体验，也会影响 AI 测试的稳定性。

## 7. 延伸阅读

- [Momentic 原文：Agentic Testing for Engineering Teams](https://momentic.ai/blog/agentic-testing-guide)
- [Playwright](https://playwright.dev/)
- [Cypress](https://www.cypress.io/)
- [Selenium](https://www.selenium.dev/)
- [GitHub Copilot](https://github.com/features/copilot)
- [The Rise of Agentic Testing: Multi-Agent Systems for Robust Software Quality Assurance](https://arxiv.org/abs/2601.02454)

## 8. 总结

这篇文章真正值得关注的地方，不是它提出了一个新名词，而是它指出了 AI 软件工程的一个结构性变化：**当 AI 让代码生产更快之后，验证系统会成为新的瓶颈。**

从这个角度看，Agentic Testing 是一个自然演进方向：测试从“人写脚本、机器执行”走向“人描述目标、agent 执行并反馈”。但它不会消灭传统测试，而会和单元测试、集成测试、类型检查、静态分析、人工 review 一起构成新的验证栈。

如果说 AI coding 的目标是加速实现，那么 Agentic Testing 的目标就是让这种加速不以牺牲质量为代价。未来更成熟的软件工程流程，很可能会围绕以下闭环展开：

> Specification → Implementation → Verification → Repair

其中，Implementation 越来越 agent 化，Verification 也必须同步 agent 化。