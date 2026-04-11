---
layout: post
title: Claude Code 近半年更新历史分类解读
description: 对 Claude Code (Harness Agent) 近半年更新日志进行分类，重点解读 Agent 相关功能更新
date: 2026-04-11
categories: claude-code
---

# Claude Code (Harness Agent) 近半年更新历史分类解读

> 来源: https://code.claude.com/docs/en/changelog  
> 版本范围: 2.0.x → 4.0.0 (约半年内的更新)

## 一、版本概览

Claude Code 经历了从 2.0.x 到 4.0.0 的大版本迭代，主要版本包括:
- **2.0 系列**: 2.0.0 - 2.0.76
- **2.1 系列**: 2.1.0 - 2.1.101 (当前稳定版)
- **4.0 系列**: 4.0.0 (最新版本)

---

## 二、更新分类汇总

### 1. Agent/Subagent 核心功能 ⭐ 重点

| 功能 | 说明 |
|------|------|
| **background: true** | Agent 定义支持 `background: true` 让 Agent 始终在后台任务中运行 |
| **initialPrompt** | Agent 可以在 frontmatter 中声明 `initialPrompt` 来自动提交第一条消息 |
| **effort/maxTurns/disallowedTools** | 为插件提供的 Agent 添加了 effort、maxTurns、disallowedTools 的 frontmatter 支持 |
| **SendMessage 替代 resume** | Agent tool 不再接受 resume 参数，改用 `SendMessage({to: agentId})` 来继续之前生成的 Agent |
| **子Agent MCP 工具继承** | 修复了子Agent不从动态注入的MCP服务器继承MCP工具的问题 |
| **嵌套 Agent 防护** | 修复了队友意外通过 Agent tool 的 name 参数生成嵌套 Agent 的问题 |
| **子Agent 隔离工作树** | 修复了子Agent在隔离工作树中无法访问自己工作树内文件的权限问题 |
| **后台子Agent失败处理** | 修复了后台子Agent失败时不向父Agent报告部分进度的问题 |
| **子Agent工作树清理** | 修复了包含未跟踪文件的worktree被错误清理的问题 |

### 2. MCP (Model Context Protocol) 集成

| 功能 | 说明 |
|------|------|
| **MCP 工具搜索自动模式** | 当 MCP 工具描述超过上下文窗口的10%时，自动启用 MCPSearch 工具进行延迟加载 |
| **MCP OAuth 支持** | 支持 RFC 9728 Protected Resource Metadata 发现，以及 Client ID Metadata Document (CIMD/SEP-991) |
| **MCP 服务器去重** | 本地配置的 MCP 服务器优先，插件 MCP 服务器与手动配置的服务器重复时被跳过 |
| **MCP 工具结果持久化** | 添加 `_meta["anthropic/maxResultSizeChars"]` 注解来覆盖工具结果持久化限制（最高500K） |
| **MCP  elicitation** | MCP 服务器现在可以通过交互式对话框（表单字段或浏览器URL）请求结构化输入 |
| **claude.ai MCP 连接器** | 支持在 Claude Code 中使用 claude.ai MCP 连接器（如 Slack、Gmail 等） |
| **MCP 工具描述限制** | MCP 工具描述和服务器指令现在限制在 2KB 以内，防止 OpenAPI 生成的服务器膨胀上下文 |
| **HTTP/SSE 性能优化** | 减少了 HTTP/SSE 传输的连接请求，缓存认证失败避免重复连接尝试 |

### 3. 远程控制 (Remote Control)

| 功能 | 说明 |
|------|------|
| **交互式设置向导** | 添加了交互式 Google Vertex AI 和 AWS Bedrock 设置向导 |
| **会话标题优化** | AI 生成的会话标题现在在第一条消息后几秒内出现 |
| **容器重启后会话恢复** | 修复了远程会话在容器重启后忘记会话历史的问题 |
| **权限处理改进** | 被阻止时显示具体原因而非通用 "not yet enabled" 消息 |

### 4. 性能优化

| 功能 | 说明 |
|------|------|
| **提示缓存命中率** | 改进了 Bedrock、Vertex、Foundry 用户的提示缓存命中率 |
| **MCP 工具架构缓存** | 消除了每次turn对MCP工具架构的JSON.stringify，提升性能 |
| **SSE 传输优化** | 大型流式帧处理从二次时间复杂度优化为线性时间 |
| **长对话 SDK 性能** | 长对话的 SDK 会话不再在transcript写入时二次减速 |
| **滚动性能** | 用纯 TypeScript 实现替代 WASM yoga-layout 提升大型transcript的滚动性能 |
| **启动优化** | `-p` 模式启动更快（未认证HTTP/SSE MCP服务器节省约600ms） |

### 5. 开发者体验

| 功能 | 说明 |
|------|------|
| **Bash 工具安全** | 修复了反斜杠转义标志可能导致绕过权限的问题，改进了环境变量前缀和网络重定向检查 |
| **Hook 输出处理** | 超过50K字符的hook输出保存到磁盘而非直接注入上下文 |
| **Read 工具优化** | 使用紧凑的行号格式并去重未更改的重读，减少token使用 |
| **上下文低警告** | 上下文低警告显示为临时页脚通知而非持久行 |

### 6. 终端与 UI 修复

| 功能 | 说明 |
|------|------|
| **鼠标滚轮滚动** | 修复了 NO_FLICKER 模式下Windows Terminal的慢速鼠标滚轮滚动 |
| **Shift+Enter 处理** | 修复了 Windows Terminal Preview 1.25 上 Shift+Enter 提交而非插入换行的问题 |
| **CJK/Unicode 文本** | 修复了多字节文本（韩文、日文、希伯来文、阿拉伯文等）在 Windows 终端中的渲染问题 |
| **图形视图** | 修复了全屏模式下内容高度变化导致的重影线问题 |

### 7. 语音模式

| 功能 | 说明 |
|------|------|
| **按键泄漏修复** | 修复了释放语音按键后空格键被吞掉3秒的问题 |
| **WebSocket 处理** | 修复了语音模式音频在服务器静默断开WebSocket连接时无法恢复的问题 |
| **macOS 权限** | 修复了语音模式在 macOS Apple Silicon 上请求麦克风权限失败的问题 |

### 8. VS Code 集成

| 功能 | 说明 |
|------|------|
| **MCP 管理对话框** | 原生 MCP 服务器管理对话框 - 在聊天面板使用 `/mcp` 启用/禁用服务器、重连、管理 OAuth 认证 |
| **冷启动优化** | 减少启动会话时的冷启动子进程工作 |
| **速率限制横幅** | 添加了使用百分比和重置时间的速率限制警告横幅 |

---

## 三、与 Harness Agent 概念直接相关的核心更新

### 3.1 Agent 生命周期管理

```
1. Agent 定义增强
   - background: true → 后台持续运行
   - initialPrompt → 启动时自动执行
   - effort/maxTurns/disallowedTools → 精细控制

2. Agent 通信机制
   - SendMessage({to: agentId}) 替代 resume
   - 支持在插件 Agent 中传递消息

3. 子Agent 隔离
   - 独立的 tmux 工作树
   - 文件访问权限控制
   - MCP 工具继承
```

### 3.2 MCP 工具延迟加载 (Tool Search)

这是 Claude Code 最重要的性能优化之一：

- **问题**: 当配置大量 MCP 工具时，工具描述会占用大量上下文窗口
- **解决方案**: 
  - 当 MCP 工具描述超过上下文窗口的 10% 时，自动启用 MCPSearch
  - 工具被延迟加载而非全部预加载
  - 用户可通过将 MCPSearch 添加到 disallowedTools 来禁用

```python
# 自动阈值配置
auto:N  # N 是上下文窗口百分比 (0-100)
```

### 3.3 多平台认证集成

- **Bedrock/Vertex/Foundry**: 交互式设置向导
- **OAuth 增强**: 支持动态客户端注册、step-up 授权
- **claude.ai MCP**: Slack、Gmail 等连接器直接可用

---

## 四、版本演进时间线（近半年重点版本）

| 版本 | 关键更新 |
|------|---------|
| 2.1.0+ | Agent Teams 功能引入 |
| 2.1.50+ | MCP 工具搜索自动模式默认启用 |
| 2.1.70+ | 远程控制会话标题优化 |
| 2.1.80+ | VS Code MCP 管理对话框 |
| 2.1.90+ | background: true Agent 支持 |
| 2.1.100+ | 4.0.0 发布前的稳定性优化 |
| 4.0.0 | 最新大版本 |

---

## 五、总结与趋势分析

### 5.1 主要趋势

1. **Agent 能力增强**: 从简单的子进程演变为完整的后台任务系统
2. **MCP 生态成熟**: 从连接不稳定到智能延迟加载和去重
3. **远程协作**: Remote Control 从实验性功能到生产可用
4. **性能优先**: 大量优化聚焦于启动速度、滚动流畅度、上下文管理

### 5.2 对 Harness Agent 开发的启示

1. **MCP 是核心**: 几乎每个版本都在改进 MCP 集成
2. **后台任务**: `background: true` 和 `initialPrompt` 表明向持久化 Agent 演进
3. **工具搜索**: 自动延迟加载是处理大量工具的关键方案
4. **安全加固**: 大量 Bash 工具安全修复表明生产环境安全重视

---

## 六、参考资料

- [Claude Code Changelog](https://code.claude.com/docs/en/changelog)
- [GitHub CHANGELOG.md](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [Claude Code 官方文档](https://code.claude.com/docs/en/overview)