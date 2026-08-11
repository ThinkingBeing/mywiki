---
layout: post
title: "Agent 工作中用户插入新指令：Claude Code、Codex、Deep Agents 的处理策略对比"
description: "基于 Claude Code、OpenAI Codex、LangChain Deep Agents 三个开源项目，研究 Agent 在工作过程中收到用户新指令时，应如何排队、中断、替换、转发或恢复。"
date: 2026-08-11
created_at: 2026-08-11 23:43:57 +0800
categories: AI Agents
source: 开源项目研究
---

# Agent 工作中用户插入新指令：Claude Code、Codex、Deep Agents 的处理策略对比

> 研究主题：Agent 正在工作过程中，用户输入了新的指令，Agent 应该如何处理？有哪些处理策略？为什么？
>
> 研究对象：
> - `tanbiralam/claude-code`，本地 commit：`6f6f12b`
> - `openai/codex`，本地 commit：`279b932`
> - `langchain-ai/deepagents`，本地 commit：`e12acba`
>
> 本文基于本地浅克隆仓库的 README、架构文档、源码和测试文件进行分析。需要说明：`tanbiralam/claude-code` 是 Claude Code 泄露源码的第三方整理仓库，不能等同于 Anthropic 官方开源项目；本文只把它作为可观察的 Agent CLI 设计样本。

---

## 1. 问题定义：什么是 mid-turn user steering？

在 Agent 系统里，用户并不总是在一个“回合”结束后才输入新指令。真实使用中经常出现这种情况：

- Agent 正在执行 shell 命令，用户说：“停一下，别继续了。”
- Agent 正在修改代码，用户补充：“顺便注意安全问题。”
- Agent 已经派出子 Agent，用户说：“让那个子任务只看测试失败部分。”
- Agent 正在长时间研究，用户发现方向错了，说：“不是这个仓库，是另一个。”
- Agent 正在等待审批，用户输入确认、拒绝或修改建议。

这类问题可以称为 **mid-turn user steering**：Agent 当前工作还没结束，用户在中途插入新的控制信号。

它看似只是“多收到一条消息”，但实际上涉及 Agent loop 的核心设计：

1. 新指令是否应该立即打断当前执行？
2. 如果当前正在执行工具或命令，是否允许中断？
3. 新指令是追加到当前上下文，还是排队到下一轮？
4. 如果有子 Agent 或后台任务，新指令应该给主 Agent，还是转发给子 Agent？
5. 如何避免把工具输出、系统通知、子任务结果误判成用户指令？
6. 如何保证中断后状态可恢复、历史可解释、审计可追踪？

这不是一个 UI 细节，而是 Agent 可靠性、可控性和安全性的基础问题。

---

## 2. 为什么不能简单地把新输入 append 到上下文？

最直觉的做法是：用户来了新消息，就直接 append 到 messages 里，让模型下一步看到。

但这在真实 Agent 系统中很危险。

### 2.1 当前可能正在工具执行中

如果 Agent 正在执行 shell 命令、写文件、调用 MCP 工具、跑测试，模型此刻并没有在采样。把新消息 append 到上下文不会立即生效，反而可能污染下一次状态转换。

### 2.2 工具执行需要原子性

有些操作不能随便中断，例如：

- 正在写文件；
- 正在执行数据库迁移；
- 正在部署；
- 正在应用 patch；
- 正在等待子进程输出。

如果用户新指令强行打断，可能留下半完成状态。

### 2.3 用户输入和系统事件必须类型隔离

Agent 系统里“看起来像消息”的东西很多：

- 用户真实输入；
- 工具输出；
- 子 Agent 汇报；
- 系统提醒；
- 权限审批请求；
- 定时任务通知；
- 后台任务状态变化。

如果这些都只是普通文本，模型很容易把系统事件当作用户指令，或者把工具输出当成新任务。

### 2.4 长任务需要可恢复性

生产级 Agent 不只是一次 prompt-response。它可能有 checkpoint、thread、run、subagent、remote execution。新指令如果改变任务方向，系统需要知道：

- 哪个 run 被中断；
- 为什么中断；
- 是否替换为新任务；
- 原来的历史如何保留；
- 是否可以 resume。

因此，成熟的 Agent 项目通常不会把中途输入简单当作普通聊天文本，而会把它建模成 **排队消息、控制操作、interrupt、approval answer、subagent update** 等更明确的结构。

---

## 3. Claude Code：中途输入排队，下一轮注入

在 `tanbiralam/claude-code` 中，能观察到一种很实用的策略：**用户在工具执行中途输入的内容，会作为 queued command 保存，并在下一轮被 drain 进上下文。**

相关证据包括：

- `src/components/VirtualMessageList.tsx` 的注释说明，用户在工具执行中途发送的 prompt 会以 `queued_command` attachment 形式进入消息列表，并在下一轮被 drain。
- `src/components/Messages.tsx` 的注释说明，mid-turn 输入是用户真实输入，需要保留；它不同于工具结果、系统 meta 消息或 task notification。
- `src/components/CoordinatorAgentStatus.tsx` 中展示了后台 agent task 的 `pendingMessages.length`，UI 会显示 “N queued”，说明对子 Agent 或后台任务也存在待处理消息队列。

这说明 Claude Code 的设计不是“新消息马上打断当前执行”，而是：

```text
用户中途输入
  ↓
保存为 queued_command / pending message
  ↓
当前工具或当前 turn 到达安全边界
  ↓
下一轮 drain 到上下文或转发给对应任务
```

### 3.1 为什么这种策略合理？

Coding Agent 的工作经常涉及文件修改、命令执行、测试运行。很多操作天然需要“完成一个安全边界”后再处理新指令。

例如 Agent 正在跑：

```bash
npm test
```

用户中途说：

> 如果失败，只看 auth 相关测试。

这时不一定要杀掉 `npm test`。更好的策略可能是：

1. 让测试跑完或到达可取消点；
2. 把用户补充作为 queued command 记录；
3. 下一轮模型看到测试结果和用户补充，再决定只分析 auth 相关失败。

这样既不丢用户输入，也不破坏当前工具执行。

### 3.2 Claude Code 策略的特点

| 特点 | 说明 |
|---|---|
| 默认不强制中断 | 更重视工具执行原子性和上下文完整性 |
| 中途输入结构化保存 | 使用 `queued_command` / pending messages，而不是普通文本混入 |
| UI 可见 | queued 数量会展示给用户，减少“我说的话是不是丢了”的不确定感 |
| 支持后台 Agent | 子任务也可以有 pending messages |

### 3.3 适合场景

这种策略适合：

- 用户补充约束；
- 用户追加说明；
- 当前命令不宜立即中断；
- 子 Agent 正在工作，需要稍后接收补充；
- 希望系统行为稳定、少出错。

### 3.4 局限

它的代价是：用户会感觉“不够即时”。如果用户说的是“停下”，排队到下一轮可能不够。此时需要显式 interrupt 机制配合。

---

## 4. OpenAI Codex：把中断建模为正式的 session operation

OpenAI Codex 的设计更偏协议化。它不是只把新输入存成队列，而是有明确的操作类型和事件类型。

在 `openai/codex` 中可以看到：

- 测试里使用 `Op::Interrupt`。
- `user_shell_cmd.rs` 中断后期望收到 `EventMsg::TurnAborted`。
- 中断原因可以是 `TurnAbortReason::Interrupted`。
- 另一个测试里出现 `TurnAbortReason::Replaced`，说明系统区分“被用户中断”和“被新 turn 替换”。
- `mcp_turn_metadata.rs` 中有 `RequestUserInput` 和 `UserInputAnswer`，说明 Agent 可以主动请求用户输入，用户再用结构化 answer 继续。
- `codex_delegate.rs` 关闭 delegate 时会先提交 `Op::Interrupt`，再提交 `Op::Shutdown`，说明 interrupt 是 session 控制面的正式操作。

Codex 的抽象大致是：

```text
用户/客户端操作
  ↓
Op::UserInput / Op::Interrupt / Op::UserInputAnswer / Op::Shutdown
  ↓
session runtime 处理 turn 生命周期
  ↓
EventMsg::TurnComplete / TurnAborted / RequestUserInput / ...
```

### 4.1 Interrupted 与 Replaced 的区别

这点非常值得学习。

- **Interrupted**：当前任务被中断，通常对应用户说“停一下”“取消当前操作”。
- **Replaced**：当前 turn 被新任务替换，表示旧任务不再是主线。

这两个语义不一样。

如果只是 Interrupted，系统可能需要保留中断边界，让用户之后恢复或查看。

如果是 Replaced，系统更明确地表达：“新任务取代旧任务，旧任务的后续不应继续。”

### 4.2 RequestUserInput / UserInputAnswer 的价值

很多 Agent 场景不是用户主动插入，而是 Agent 主动暂停请求用户决策，例如：

- 是否允许执行危险命令？
- 是否选择某个路径？
- 是否批准 patch？
- 是否继续部署？

如果把这些都当普通聊天，系统很难知道用户回答对应哪个请求。Codex 用 `RequestUserInput` 和 `UserInputAnswer` 这类结构化事件，就能把用户回答绑定到具体 turn 或 call id。

### 4.3 Codex 策略的特点

| 特点 | 说明 |
|---|---|
| 控制面明确 | 用户输入、中断、回答、关闭都是不同 Op |
| turn 生命周期清晰 | 有 TurnComplete、TurnAborted 等事件 |
| 中断原因结构化 | Interrupted、Replaced 等 reason 可用于历史和恢复 |
| 适合 SDK/TUI/远程客户端 | 客户端可以精确控制 session |

### 4.4 适合场景

这种策略适合：

- 用户明确要求停止；
- 用户新任务替换旧任务；
- shell 命令或模型采样需要取消；
- Agent 主动请求用户确认；
- 多客户端或 SDK 需要稳定协议；
- 需要审计和恢复的长任务。

### 4.5 局限

协议化设计更复杂。系统必须维护 turn id、event stream、abort reason、history marker、delegate shutdown 等状态。如果只是一个轻量脚本 Agent，成本可能过高。

---

## 5. Deep Agents：基于 LangGraph 的 interrupt、checkpoint 与 async subagent update

Deep Agents 的重点不在 TUI 层，而在生产级 Agent runtime。

它的 README 描述自己是 “batteries-included agent harness”，内置：

- sub-agents；
- filesystem；
- context management；
- shell access；
- persistent memory；
- human-in-the-loop；
- skills；
- tools。

更关键的是，`openwiki/architecture/overview.md` 明确说明：Deep Agents 建在 LangChain 和 LangGraph 之上，LangGraph 负责 state、checkpoint、streaming 和 interrupts；Deep Agents 通过 middleware、backend、profile、subagent 等组合改变 agent 看到什么、能做什么。

架构关系可以简化为：

```text
Deep Agents  -> opinionated harness: middleware, backends, profiles, subagents
LangChain    -> agent abstraction: model + tools + middleware
LangGraph    -> execution runtime: state, checkpoints, streaming, interrupts
```

### 5.1 Async subagent 的 update 策略

`libs/deepagents/deepagents/middleware/async_subagents.py` 描述了 async subagent：

- 它运行在远程 Agent Protocol / LangGraph server 上；
- 与同步 subagent 不同，它会立即返回 task ID；
- 主 Agent 可以 monitor progress；
- 主 Agent 可以 send updates while the subagent works。

在 `test_async_subagents.py` 中可以看到：`update_async_task` 给已有 thread 发送新消息时，会调用 remote runs.create，并设置：

```python
multitask_strategy="interrupt"
```

这说明 Deep Agents 对“给正在运行的子 Agent 追加新指令”的处理不是简单 append，而是：

```text
主 Agent / 用户产生补充指令
  ↓
定位 async task / thread_id
  ↓
创建新的 remote run
  ↓
使用 multitask_strategy="interrupt"
  ↓
中断旧 run，并以新输入继续该 thread
```

### 5.2 checkpoint/resume 的意义

Deep Agents 依赖 LangGraph 的 checkpoint 机制。这对 mid-turn steering 很重要：

- 用户中途打断后，系统知道当前状态在哪里；
- 新指令可以在已有 thread 上继续；
- 长任务不会因为进程退出或远程调用中断而完全丢失；
- 子 Agent 的状态可以独立于主 Agent 管理。

这比单进程内存队列更适合生产环境。

### 5.3 Deep Agents 策略的特点

| 特点 | 说明 |
|---|---|
| runtime-first | 把问题交给 LangGraph 的 state/checkpoint/interrupt 处理 |
| middleware-first | 通过 middleware 改变工具、提示、状态和权限 |
| subagent 可更新 | async task 有 task_id/thread_id/run_id，可 monitor/update/cancel |
| 适合生产 | 远程运行、持久化、部署、追踪和评估能力更强 |

### 5.4 适合场景

这种策略适合：

- 长时间任务；
- 后台 Agent；
- 远程 Agent 服务；
- 多子任务并行；
- human-in-the-loop；
- 需要 checkpoint/resume 的生产系统。

### 5.5 局限

Deep Agents 的策略依赖较重的运行时抽象。如果只是一个本地 CLI，直接引入 LangGraph 式 checkpoint/interrupt 可能显得复杂。但对于生产级 Agent，这种复杂性是必要基础设施。

---

## 6. 三个项目的横向比较

| 维度 | Claude Code | OpenAI Codex | Deep Agents |
|---|---|---|---|
| 核心形态 | 本地 coding CLI / TUI | 本地 coding agent + SDK/TUI 协议 | Agent harness / runtime SDK |
| 中途用户输入 | queued command / pending message | session operation，如 Interrupt、UserInputAnswer | graph interrupt / async task update |
| 默认倾向 | 不立即打断，排队到安全边界 | 明确控制 turn 生命周期 | 依赖 checkpoint + runtime interrupt |
| 子任务处理 | 后台 agent 有 pending messages | delegate 可 interrupt/shutdown | async subagent 可 update/cancel/check |
| 用户确认 | 有权限/工具确认体系 | RequestUserInput / UserInputAnswer | human-in-the-loop / interrupt |
| 状态管理 | CLI 消息队列与 transcript | session event + abort reason + history | LangGraph state/checkpoint |
| 设计重点 | 不丢输入，UI 可感知，工具执行安全 | 协议清晰，turn 生命周期可控 | 生产可恢复，远程任务可管理 |

---

## 7. 可抽象出的处理策略

### 策略一：排队到下一轮

**代表项目：Claude Code**

适用于：用户补充说明、当前工具不宜中断、当前 turn 即将结束。

流程：

```text
新输入 → queued_command → 当前操作结束 → 下一轮注入上下文
```

优点：

- 不破坏工具执行；
- 不丢用户输入；
- 实现相对简单；
- 用户可以看到 queued 状态。

缺点：

- 不够即时；
- 不适合“立刻停止”类指令。

### 策略二：显式中断当前 turn

**代表项目：OpenAI Codex**

适用于：用户明确要求停止、取消、改变方向。

流程：

```text
新控制信号 → Op::Interrupt → TurnAborted(Interrupted) → 等待下一步输入
```

优点：

- 响应及时；
- 状态明确；
- 历史可审计。

缺点：

- 需要可取消的执行模型；
- 需要设计中断边界；
- 可能留下半完成工作，需要恢复策略。

### 策略三：用新任务替换旧 turn

**代表项目：OpenAI Codex**

适用于：用户说“不是这个，改做那个”。

流程：

```text
新任务 → replace current turn → TurnAborted(Replaced) → 开始新 turn
```

优点：

- 避免旧任务继续消耗资源；
- 对用户意图表达更准确。

缺点：

- 旧任务的中间成果可能被废弃；
- 需要在历史中标记替换边界。

### 策略四：请求用户输入后继续

**代表项目：Codex、Deep Agents**

适用于：审批、确认、选择、补充参数。

流程：

```text
Agent 发出 RequestUserInput / interrupt
  ↓
用户回答 UserInputAnswer / resume command
  ↓
继续原任务
```

优点：

- 用户回答可以绑定具体请求；
- 安全性强；
- 适合高风险工具调用。

缺点：

- 交互协议复杂；
- UI/SDK 必须支持 request-answer 绑定。

### 策略五：更新后台子 Agent

**代表项目：Claude Code、Deep Agents**

适用于：Agent 已经把任务分给子 Agent，用户想补充方向。

流程：

```text
用户补充 → 定位 task_id/thread_id → pending message 或 update_async_task → 子 Agent 中断/继续
```

优点：

- 不影响主 Agent；
- 可以精确 steering 某个 worker；
- 适合多 Agent 协作。

缺点：

- 需要任务 ID 和任务状态管理；
- 用户可能不知道该指令应该给哪个 Agent；
- 需要处理主 Agent 与子 Agent 的上下文一致性。

### 策略六：checkpoint + resume

**代表项目：Deep Agents**

适用于：长任务、远程 Agent、生产环境。

流程：

```text
运行状态 checkpoint
  ↓
用户 interrupt/update
  ↓
从 checkpoint resume 或开启新 run
```

优点：

- 可恢复；
- 可审计；
- 适合生产部署和长任务。

缺点：

- 基础设施较重；
- 需要 runtime 层支持。

---

## 8. 推荐设计：Agent 应该如何处理用户中途新指令？

结合三个项目，可以得到一个比较实用的设计框架。

### 8.1 先分类用户新指令

用户中途输入不应一律处理成普通消息，而应先分类：

| 类型 | 示例 | 推荐策略 |
|---|---|---|
| 补充信息 | “顺便注意安全问题” | 排队到下一轮，或发送给当前子任务 |
| 纠偏 | “不是这个文件，是另一个” | 到安全边界后切换，必要时 interrupt |
| 停止 | “停下，别继续了” | 显式 interrupt |
| 替换任务 | “别做 A 了，改做 B” | replaced / abort old turn + start new turn |
| 审批回答 | “允许执行” / “拒绝” | UserInputAnswer / approval response |
| 子任务 steering | “让那个 worker 只看测试” | update subagent task |

### 8.2 再判断当前执行状态

同一条用户指令，在不同状态下处理方式不同：

| 当前状态 | 推荐行为 |
|---|---|
| 模型正在采样 | 可以取消采样并开始新 turn，或等待采样结束 |
| shell 命令正在运行 | 如果命令可安全终止，则 interrupt；否则排队 |
| 文件写入/patch 中 | 尽量等原子操作结束后处理 |
| 等待用户审批 | 把输入绑定到审批请求 |
| 子 Agent 正在运行 | 允许定向 update/cancel 子 Agent |
| 长任务远程运行 | 通过 checkpoint/resume 或 run interrupt 处理 |

### 8.3 最后选择控制策略

一个成熟 Agent 可以按优先级处理：

1. 如果用户明确说“停止/取消”：立即 interrupt。
2. 如果用户明确替换任务：abort/replaced 当前 turn，开启新 turn。
3. 如果用户只是补充约束：排队到下一轮。
4. 如果当前在等待审批：作为该审批的 answer。
5. 如果新指令指向某个子任务：转发或 update 子 Agent。
6. 如果不确定：先记录为 queued input，并向用户说明会在当前安全点后处理。

---

## 9. 实现上的关键设计点

### 9.1 消息必须有类型，不要只有 role/content

至少需要区分：

- real user prompt；
- queued command；
- tool result；
- system/meta message；
- task notification；
- approval answer；
- interrupt signal；
- subagent update。

Claude Code 在 UI 层显式排除 meta、tool result、task notification，只保留真实用户输入，这是非常重要的工程细节。

### 9.2 中断必须有原因

不要只有 `cancelled = true`，最好区分：

- interrupted by user；
- replaced by new task；
- timeout；
- failed；
- shutdown；
- permission denied。

Codex 的 `TurnAbortReason::Interrupted` 和 `TurnAbortReason::Replaced` 是很好的设计。

### 9.3 子 Agent 必须可定位

如果系统支持子 Agent，就需要：

- task_id；
- thread_id；
- run_id；
- status；
- pending messages；
- update/cancel/check/list 工具。

Deep Agents 的 async subagent state 和 Claude Code 的 pendingMessages 都体现了这一点。

### 9.4 工具执行需要安全边界

不是所有任务都能随时杀掉。Agent runtime 应该知道：

- 哪些工具可取消；
- 哪些工具必须等待结束；
- 哪些工具取消后需要 rollback；
- 哪些工具取消后需要提示用户检查状态。

### 9.5 UI 要告诉用户“新指令已收到”

如果新指令被排队，用户应该看到状态，例如：

```text
已收到，将在当前命令结束后处理。
```

或者显示 queued count。Claude Code 的 “N queued” 是一个很小但很重要的 UX 设计。

---

## 10. 一个推荐的通用架构

综合三个项目，一个 Agent 系统可以设计成以下结构：

```text
User Input Router
  ├─ classify: prompt / interrupt / replace / approval / subagent update
  ├─ bind: current turn / pending approval / child task / new turn
  ↓
Turn Manager
  ├─ active turn
  ├─ queued inputs
  ├─ abort reason
  ├─ safe interruption boundary
  ↓
Tool Runtime
  ├─ cancellable tools
  ├─ atomic tools
  ├─ approval-required tools
  ↓
Subagent Manager
  ├─ task_id / thread_id / run_id
  ├─ pending messages
  ├─ update / cancel / check / list
  ↓
Checkpoint / History
  ├─ completed turns
  ├─ interrupted boundaries
  ├─ replaced markers
  ├─ resumable state
```

这个架构对应了三个项目的经验：

- Claude Code 提醒我们：中途输入要排队、保留、显示，不能丢。
- Codex 提醒我们：中断和替换要协议化，turn 生命周期要明确。
- Deep Agents 提醒我们：生产级长任务要依赖 checkpoint、interrupt、subagent runtime。

---

## 11. 总结：Agent 的“听话”能力，本质是控制平面能力

用户中途输入新指令，看起来是一个聊天体验问题，其实是 Agent 控制平面问题。

一个真正可用的 Agent，不仅要会完成任务，还要能在任务进行中被用户安全地 steering。

从这三个项目可以学到：

1. **不要丢输入**：用户中途输入必须结构化保存。
2. **不要乱中断**：工具执行要有安全边界。
3. **不要混淆消息类型**：用户输入、工具输出、系统通知、子任务结果必须区分。
4. **中断要有语义**：Interrupted、Replaced、Timeout、Shutdown 不应混成一个 cancelled。
5. **子任务要可控**：后台 Agent 必须能 update、cancel、check。
6. **长任务要可恢复**：生产级 Agent 需要 checkpoint/resume。
7. **用户要看得见状态**：排队、中断、等待审批都应该有明确反馈。

如果用一句话概括：

**Agent 不是只要会执行用户指令，更要能在执行过程中正确理解和吸收用户的新指令。这个能力决定了 Agent 从 demo 走向真实生产的可靠性。**
