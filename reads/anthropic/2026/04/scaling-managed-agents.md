# Scaling Managed Agents: Decoupling the brain from the hands

> 来源: Anthropic Engineering Blog | https://www.anthropic.com/engineering/managed-agents | 2026年4月

## 1. 文章基本信息

- **标题**: Scaling Managed Agents: Decoupling the brain from the hands
- **来源**: Anthropic Engineering Blog
- **链接**: https://www.anthropic.com/engineering/managed-agents
- **作者**: Lance Martin, Gabe Cemaj, Michael Cohen
- **发布日期**: 2026年4月

## 2. 文章要解决的核心问题（论点）

如何设计一个能够适应未来模型演进的 Agent 系统，使得当模型能力提升时，Agent 的基础设施不需要频繁重构？

## 3. 问题所处的背景上下文

- **问题起源**: Agent 的 "harness"（套索/框架）编码了关于"模型不能做什么"的假设，但这些假设会随着模型能力提升而过时
- **具体案例**: 早期 Claude Sonnet 4.5 存在"上下文焦虑"问题，会在接近上下文限制时提前完成任务，Anthropic 通过在 harness 中添加上下文重置来解决。但当切换到 Claude Opus 4.5 时，这个行为已消失，上下文重置反而成为负担
- **业界现状**: 大多数 Agent 系统将"大脑"（模型+harness）和"手"（沙箱/工具）耦合在一起，导致难以扩展和维护

## 4. 文章的核心观点

**核心观点**: 借鉴操作系统虚拟化硬件的思路，将 Agent 的各个组件虚拟化为独立接口，实现解耦

### 关键观点一：虚拟化 Agent 组件

- 将 Agent 分解为三个核心接口：
  - **Session（会话）**: 事件日志，持久化记录所有交互
  - **Harness（套索）**: 调用模型并路由工具调用的循环
  - **Sandbox（沙箱）**: 执行环境，模型可以运行代码和编辑文件
- 每个组件可以独立更换而不影响其他组件

### 关键观点二：不要"养宠物"

- 原来的单体架构意味着整个系统像"宠物"——一个容器失败，整个会话就丢失
- 解耦后，组件变成" cattle（牲畜）"——可替换、无状态

### 关键观点三：安全边界的重新设计

- 原来：未信任代码与凭证在同一容器，提示注入可以轻易读取环境变量
- 解耦后：凭证永远不在模型生成的代码可访问的沙箱中，使用金库（vault）存储

### 关键观点四：Session 不是模型的上下文窗口

- 长周期任务往往超出模型上下文限制
- Session 作为外部上下文对象，通过 `getEvents()` 接口让"大脑"可以查询历史事件
- 可灵活选择位置切片、倒带重读、选择性检索

### 关键观点五：多脑多手架构

- **多脑**: 解耦后可以连接客户自己的 VPC 资源，harness 不需要部署在客户网络中
- **多手**: 一个大脑可以调用多个执行环境，让模型决定在哪里执行任务
- 性能提升：p50 TTFT 降低约 60%，p95 降低超过 90%

## 5. 观点对应的论据

| 观点 | 论据 |
|------|------|
| 解耦的必要性 | Claude Sonnet 4.5 的"上下文焦虑"在 Opus 4.5 中已消失，证明假设会过时 |
| 性能提升 | 实际部署数据：p50 TTFT 降低 60%，p95 降低 90%+ |
| 安全改进 | 通过 vault 存储凭证，模型生成的代码永远无法直接访问敏感信息 |
| 可扩展性 | harness 变成无状态的，可以启动多个实例，需要时再连接沙箱 |

## 6. 我的评价和启发

### 可信度评估

- 高可信。文章来自 Anthropic 官方工程博客，作者是核心开发团队成员
- 提供了真实的性能数据支撑观点
- 借鉴了操作系统 decades 积累的虚拟化经验，方法论成熟

### 认同与不认同

**认同**:
- 解耦是系统设计的正确方向，尤其在快速迭代的 AI 领域
- "不要养宠物"的比喻非常形象，符合云原生实践
- Session 与上下文窗口分离的设计很聪明，解决了长任务的核心痛点

**不认同/需观察**:
- 接口抽象是否足够通用？实际业务场景可能更复杂
- 多手架构对模型推理能力要求高，当前模型是否足够可靠？

### 对我的启发

1. **架构思维**: 借鉴成熟领域的抽象方法（操作系统虚拟化）来解决新问题
2. **前瞻性设计**: 设计系统时要假设"未来的模型会更强"，不要把当前模型的能力边界编码进基础设施
3. **安全永远不能假设**: 凭证必须与执行环境物理隔离

### 可以进一步探索的方向

- 如何评估不同抽象接口的优劣？
- Multi-agent 场景下的资源调度策略
- 开源替代方案（如 LangChain 的 Agent 架构）如何借鉴这个思路？

## 7. 延伸阅读

- [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Harness Design for Long-Running Apps](https://www.anthropic.com/engineering/harness-design-long-running-apps)
- [Claude Managed Agents 文档](https://platform.claude.com/docs/en/managed-agents/overview)