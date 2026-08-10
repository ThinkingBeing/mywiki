---
layout: post
title: "Palantir Ontology 系统介绍：企业 AI 的语义操作层"
description: "基于 Palantir 官方文档，系统介绍 Ontology 如何把企业数据、业务对象、关系、逻辑、权限、动作和 AI Agent 连接成可运行的数字操作层。"
date: 2026-08-11
created_at: 2026-08-11 07:52:21 +0800
categories: AI Agents
source: Palantir 官方文档
---

# Palantir Ontology 系统介绍：企业 AI 的语义操作层

> 本文优先基于 Palantir 官方文档与官网信息，试图用“业务人也能理解”的方式，系统化解释 Palantir 所说的 **Ontology / 本体**。

---

## 一句话理解

在 Palantir 语境里，**Ontology 不是一个普通的数据模型，也不只是知识图谱**。它更像是企业的“数字操作系统”：把企业里的数据、业务对象、关系、规则、模型、权限、操作动作和应用工作流连接成一个可运行的业务世界。

Palantir 官方把 Ontology 描述为组织的 **operational layer / 运营层**：它位于数据集、虚拟表、模型等数字资产之上，把这些资产连接到现实世界中的工厂、设备、产品、客户订单、财务交易等实体或概念。在 Foundry 中，Ontology 也被称为组织的 **digital twin / 数字孪生**，它把数据集和模型映射为对象类型、属性、关系类型和动作类型，从而形成组织世界的完整图景。

---

## 1. 为什么企业需要 Ontology？

传统企业数据系统通常有几个痛点：

- **数据分散**：ERP、MES、WMS、CRM、数据仓库、IoT、文档库各自为政。
- **语义缺失**：表里有 `customer_id`、`plant_code`、`material_no`，但系统不真正理解“客户”“工厂”“物料”在业务里的含义。
- **分析与操作割裂**：BI 报表能告诉你发生了什么，但不能直接帮助你改变现实业务。
- **AI 无法可靠行动**：大模型能生成建议，但不知道哪些数据可信、哪些动作允许执行、执行后如何写回业务系统。
- **权限与治理复杂**：谁能看什么、改什么、执行什么动作，常常分散在多个系统里。

Palantir 的观点是：企业真正要优化的不是“数据本身”，而是 **决策**。官方文档明确说，Ontology 代表企业中的决策，而不只是数据；它把人类和 AI Agent 直接连接到运营系统，以应对组织中最困难的问题。

Palantir 把一个运营决策拆成四个组成部分：

1. **Data / 数据**：做决策需要的信息。
2. **Logic / 逻辑**：评估决策的规则、启发式方法、计算过程、预测模型、优化模型等。
3. **Action / 动作**：把选择的决策编排并执行。
4. **Security / 安全**：确保决策符合组织政策和权限控制。

所以，Ontology 的核心目的不是“建一个漂亮的数据目录”，而是把数据、逻辑、动作、安全统一起来，让企业能够在真实业务中做更好、更快、更安全的决策。

---

## 2. Ontology 到底是什么？

可以从三层理解。

### 2.1 现实世界的业务地图

Ontology 首先是一张企业现实世界的业务地图。Palantir 官方说，Ontology 是“对世界的分类”；在 Foundry 中，它是组织的数字孪生，把组织的数字资产整合成一个连贯整体。

例如一家制造企业里，Ontology 可能包含：

- 工厂 Plant
- 生产线 Production Line
- 设备 Machine
- 物料 Material
- 供应商 Supplier
- 客户订单 Customer Order
- 生产工单 Work Order
- 库存 Inventory
- 运输 Shipment
- 质量事件 Quality Incident
- 维护任务 Maintenance Task

这些不是孤立表，而是现实业务对象。

### 2.2 对象、属性、关系、动作

Palantir 官方把 Ontology 的基础组成部分定义为：

- **Object Type / 对象类型**：组织中的实体或事件类型，比如“飞机”“订单”“病人”“机器”“交易”。
- **Object / 对象实例**：某个具体实体或事件，比如“订单 #12345”。
- **Property / 属性**：对象的特征，比如订单金额、设备状态、交付日期。
- **Link Type / 关系类型**：两个对象类型之间的关系，比如“订单属于客户”“设备位于工厂”“工单消耗物料”。
- **Action Type / 动作类型**：用户或系统可以对对象执行的一组变更，比如更新订单状态、创建维护任务、调整库存。
- **Function / 函数**：可复用的代码逻辑，可以读取对象、对象集合和属性值，也可以用于 Action 和应用中。
- **Role / 角色**：Ontology 的核心权限模型，用来控制谁能访问哪些本体资源。
- **Interface / 接口**：描述对象类型形状和能力的本体类型，支持类似“多态”的建模，让具有相同形态的对象类型可以被一致地交互。

### 2.3 可运行的企业操作层

Palantir 特别强调 Ontology 同时包含两类元素：

- **Semantic elements / 语义元素**：对象、属性、关系。
- **Kinetic elements / 动态/行动元素**：动作、函数、动态安全等。

这点非常关键。

很多知识图谱或数据模型只解决“世界是什么样”的问题；Palantir Ontology 还要解决“人和 AI 可以如何安全地改变这个世界”的问题。

也就是说，它不仅描述业务，还承载业务操作。

---

## 3. 用一个例子理解：航空公司 Ontology

假设一家航空公司要提升航班准点率。

传统数据方式：

- 航班表在一个系统。
- 机组排班在另一个系统。
- 飞机维修记录在另一个系统。
- 天气数据在外部接口。
- 机场地勤状态在另一个系统。
- BI 报表只能展示延误率、历史趋势。

Ontology 方式：

它会把这些整合成一个业务世界：

- **Flight / 航班**：航班号、起飞时间、预计到达时间、当前状态、延误风险。
- **Aircraft / 飞机**：机型、当前位置、维护状态、可用性。
- **Crew / 机组**：资质、排班、剩余工作时长。
- **Airport / 机场**：天气、跑道状态、拥堵程度。
- **Maintenance Event / 维修事件**：故障类型、预计完成时间、影响范围。
- **Links / 关系**：航班使用某架飞机；航班需要某组机组；飞机当前位于某机场；航班受某个维修事件影响。
- **Actions / 动作**：更换飞机、调整登机口、重新安排机组、通知乘客、创建维修工单。
- **Logic / 逻辑**：延误预测模型、机组合法性规则、飞机可用性规则、成本优化模型。
- **Security / 安全**：调度员能改航班计划；维修人员能更新维修状态；AI Agent 只能提出建议，关键动作需要人工确认。

这样，当 AI Agent 被问：“如何减少这个航班延误？”它不是凭空聊天，而是能在 Ontology 中理解真实对象、关系、约束、权限和可执行动作。

这就是 Palantir 所说的：Ontology 把 AI 锚定在企业运营真相中，从而降低幻觉风险，并建立可用于决策的信任。

---

## 4. Ontology 和数据库有什么区别？

Palantir 文档专门用数据库/数据集类比来解释 Ontology：

| 传统数据集 | Ontology |
|---|---|
| Dataset / 数据集 | Object Type / 对象类型 |
| Row / 行 | Object / 对象 |
| Column / 列 | Property / 属性 |
| Field / 单元格值 | Property Value / 属性值 |
| Join / 表连接 | Link Type / 关系类型 |

这个类比有助于入门，但 Ontology 远超数据库表。

核心区别：

### 4.1 数据库关注“存储”，Ontology 关注“现实业务对象”

数据库里是一张张表；Ontology 里是“客户”“订单”“设备”“交易”“患者”“供应链节点”等业务对象。

### 4.2 数据库关系通常是技术性的，Ontology 关系是业务语义

数据库的 join 是字段连接；Ontology 的 link 是现实世界关系，比如“供应商供应物料”“设备属于产线”“订单由客户创建”。

### 4.3 数据库通常不包含业务动作

数据库可以 `UPDATE` 一行，但它不知道“批准采购订单”“重新调度生产计划”“提交维修请求”这些业务动作的语义和治理。Ontology 的 Action Type 则明确描述用户可以对对象执行哪些变更。

### 4.4 数据库不天然连接 AI、应用、权限和操作流

Ontology 是 Palantir Foundry/AIP 应用、AI、权限、操作动作之间的统一层。它不是一个只给工程师看的 schema，而是给业务用户、开发者、应用和 AI Agent 共同使用的业务运行层。

---

## 5. Ontology 和知识图谱有什么区别？

Ontology 与知识图谱有相似之处：都有对象、属性、关系，都强调语义。

但 Palantir 的 Ontology 更偏 **operational ontology / 运营本体**，不是单纯的知识表达系统。

| 维度 | 知识图谱 | Palantir Ontology |
|---|---|---|
| 主要目标 | 表达实体与关系 | 支撑企业运营决策和行动 |
| 数据来源 | 文本、数据库、三元组等 | 企业系统、数据集、模型、流数据、用户操作 |
| 是否可执行动作 | 通常不是核心 | Action Type 是核心组成 |
| 是否承载权限治理 | 不一定 | 角色、权限、动态安全是核心 |
| 是否连接业务应用 | 需要额外开发 | Foundry/AIP 应用原生基于 Ontology |
| 是否连接 AI Agent | 可连接 | Palantir 明确把 Ontology 作为 AIP 的核心决策系统 |

简单说：**知识图谱告诉你“世界如何连接”；Palantir Ontology 还告诉你“在这个世界里，谁可以基于什么规则执行什么动作”。**

---

## 6. Palantir Ontology 的核心组件

### 6.1 Object Type / 对象类型

对象类型是现实世界实体或事件的 schema。

例子：Customer、Product、Factory、Machine、Order、Invoice、Shipment、Patient、Mission。

对象类型不是简单表名，而是业务中的“名词”。

### 6.2 Object / 对象实例

对象是对象类型的一个具体实例。

例如：客户 Tesla、订单 Order #2026-0001、机器 Machine A-17、航班 Flight UA123、病人 Patient 789。

### 6.3 Property / 属性

属性描述对象的特征。

例如订单的属性：订单号、金额、创建时间、承诺交付日期、当前状态、延误风险。

属性不仅是字段，还可以带有格式、元数据、治理规则等。

### 6.4 Link Type / 关系类型

关系类型描述两个对象类型之间的关系。

例如：Customer 下了 Order；Order 包含 Product；Machine 位于 Factory；Shipment 运送 Order；Maintenance Task 影响 Machine。

这使得用户和 AI 可以沿着业务关系探索问题，而不是手写复杂 SQL join。

### 6.5 Action Type / 动作类型

Action Type 定义用户可以一次性对对象、属性值、关系做哪些变更，以及动作提交后会产生哪些副作用。

例如：创建采购请求、批准订单、关闭质量事件、调整生产计划、派发维修任务、触发客户通知。

这是 Palantir Ontology 与普通数据建模最大的不同之一：它把“改变业务状态”的动作纳入模型。

### 6.6 Function / 函数

Function 是代码逻辑，可以接收对象或对象集作为输入，读取对象属性，并被 Action 和应用调用。

例如：计算订单延误风险、根据库存与需求生成补货建议、预测设备故障概率、检查一个动作是否违反业务规则、为调度方案计算成本。

Palantir 官方强调，在 agentic orchestration 时代，AI 的非确定性推理需要与确定性函数、算法、统计过程、优化模型结合。

### 6.7 Roles / 角色与权限

Roles 是 Ontology 的中心权限模型，可在 Ontology 层或资源层授权。

这意味着 Ontology 不是一个人人都能随便看的大图谱，而是可治理的企业操作层。

### 6.8 Object Views / 对象视图

Object Views 是围绕某个对象聚合信息和工作流的中心，比如对象的关键资料、关联对象、相关指标、分析、仪表盘和应用。

比如你打开一个“机器”对象视图，可以看到：机器当前状态、历史故障、所属产线、相关工单、预测维护风险，以及可执行动作：创建维修任务、标记停机、更新状态。

---

## 7. Ontology 如何支持 AI / AIP？

Palantir AIP 的关键不是“接入一个大模型”这么简单，而是让 AI 能在企业真实上下文中安全行动。

Palantir 官网对 AIP 中 Ontology 的描述包括：

- Ontology 是 AIP 核心的 decision-centric system，把 AI 与企业数据、逻辑和动作结合起来。
- Ontology 把实时数据整合进业务的语义模型，使 AI 锚定在企业运营真相中，降低模型幻觉风险，并建立决策信任。
- Ontology 把 AI 连接到传统业务逻辑、ML 模型、优化器和其他企业计算资产；被许可的逻辑资产可以成为补充 AI 推理的确定性工具。
- Ontology 使 AI 能够安全地把决策同步回运营数据库、边缘平台和其他行动系统；AI 生成的提案可以经过人工验证，场景可以被生成并检查影响，所有动作都可以审计。

这说明 Palantir 的 AI 架构不是：

```text
LLM + 数据库查询
```

而更像是：

```text
LLM / Agent + Ontology + 权限 + 工具函数 + 操作动作 + 审计 + 人工验证
```

这也是为什么 Palantir 特别强调 Ontology 对企业 AI 的重要性。

---

## 8. Ontology 的“决策中心”思想

Palantir 的一个重要表述是：Ontology 代表的是企业决策，不只是数据。

这句话很关键。

传统数据平台的中心是：

```text
数据 → 报表 → 人看 → 人决策 → 人去另一个系统执行
```

Ontology 想变成：

```text
数据 + 逻辑 + 权限 + 动作 + 应用 + AI → 在同一个业务语义层中完成决策和执行
```

例如供应链补货：

传统方式：

1. BI 显示库存不足。
2. 计划员下载 Excel。
3. 查 ERP。
4. 问供应商。
5. 手动创建采购单。
6. 再回系统更新状态。

Ontology 方式：

1. 库存、需求、供应商、订单、物流、风险都建模成对象。
2. 预测模型计算缺货风险。
3. 优化函数生成补货建议。
4. AI Agent 解释原因并生成方案。
5. 用户在应用里审查。
6. Action Type 提交采购请求。
7. 权限、审计、写回 ERP 自动处理。

这就是“从分析到操作”的转变。

---

## 9. Ontology 的应用层：用户如何使用它？

Palantir 官方文档说，Ontology 深度集成到 Palantir 面向用户的分析和运营工具中：用户可以创建 Object Views，在 Object Explorer 中搜索对象，在 Quiver 中做复杂分析，在 Workshop 中构建高质量应用等。

官方还把 Ontology-aware applications 分为不同风格：

### 9.1 探索型应用

这类应用不需要 builder 预先配置复杂流程，用户可以在数据进入 Ontology 后直接探索。官方举例包括 Object Explorer 和 Quiver。

适合查找对象、追踪关系、临时分析、发现异常、回答开放式业务问题。

### 9.2 工作流型应用

这类应用通常需要 builder 预先配置，面向具体业务流程。例如 Workshop 或 Slate 应用。

适合供应链控制塔、维修调度台、风险审批系统、生产排程系统、客服处置工作台、医疗资源协调系统。

也就是说，Ontology 既能支持探索分析，也能支持正式运营流程。

---

## 10. Ontology 与模型 / ML 的关系

Palantir 文档中特别谈到 “Models in the Ontology”。官方认为，模型不仅是孤立的预测服务，而应该成为 Ontology 的组成部分。

其价值包括：

- 模型可以直接读写或作用于业务对象。
- 一个模型输出可以被后续用例复用。
- 预测、优化、仿真不再是一次性项目，而能成为企业操作层的一部分。
- 当 ML 模型纳入 Ontology 后，Ontology 不仅是数据的单一事实源，也成为逻辑的单一事实源。
- 模型编码了组织对未来变化的预期，使 Ontology 能够作为企业级数字孪生，支持跨组织仿真。

举例：需求预测模型、供应链优化模型、设备故障模型、价格模型、风险模型。

如果这些模型只是散落在 notebook 或某个 API 里，很难进入业务流程；进入 Ontology 后，它们能被应用、Action、AI Agent 和用户工作流调用。

---

## 11. Ontology 的价值总结

### 11.1 把数据变成业务对象

不再让用户面对表、列、join，而是面对客户、订单、设备、航班、工单、任务等真实业务对象。

### 11.2 统一企业语义

不同系统中的同一个业务概念可以被统一建模，避免“每个部门一套定义”。

### 11.3 连接分析与操作

不仅能看数据，还能在同一语义层中执行动作。

### 11.4 让 AI 有上下文

AI 不再只读取碎片数据，而是理解对象、关系、权限、业务逻辑和可执行动作。

### 11.5 降低 AI 幻觉与越权风险

Ontology 把 AI 锚定在企业运营真相中，并通过权限、人工验证、审计和确定性工具约束 AI 行为。

### 11.6 复用业务逻辑和模型

函数、优化器、ML 模型可以成为 Ontology 中可复用的逻辑资产。

### 11.7 支持企业级数字孪生

当数据、模型、关系、动作和决策历史都进入 Ontology，企业可以进行仿真、场景分析和运营优化。

---

## 12. 用“人话”类比

### 12.1 Ontology 是企业的“游戏引擎”

游戏引擎里有角色、地图、物品、状态、规则、动作、权限、物理逻辑。

玩家不是直接操作数据库，而是在游戏世界里行动。

企业 Ontology 类似：

- 角色：客户、员工、设备、供应商。
- 地图：工厂、仓库、航线、供应链网络。
- 状态：库存、订单状态、设备健康度。
- 规则：审批规则、排程规则、风险规则。
- 动作：批准、调度、派单、采购、通知。
- AI Agent：像游戏里的智能 NPC，但必须遵守规则和权限。

### 12.2 Ontology 是企业的“操作系统内核”

数据仓库像硬盘，存数据；BI 像显示器，展示数据；AI 像智能助手。

但如果没有操作系统，它们无法安全协同。

Ontology 就像企业 AI 和运营应用的 OS kernel：管理对象、权限、动作、业务逻辑、状态变更，并给应用和 AI 提供统一接口。

### 12.3 Ontology 是“业务 API 层”

过去应用直接对接数据库、ERP、CRM，各系统接口混乱。

Ontology 把企业能力封装成：查询对象、沿关系导航、调用函数、执行动作、遵守权限、记录审计。

于是应用和 AI 都可以基于统一业务 API 工作。

---

## 13. 和传统企业架构的关系

可以把 Palantir Ontology 放在企业架构中间：

```text
上层：
  业务应用 / 工作流 / Dashboard / AI Agent / AIP / Workshop

中间：
  Palantir Ontology
  - 对象
  - 属性
  - 关系
  - 动作
  - 函数
  - 模型
  - 权限
  - 审计

下层：
  ERP / CRM / MES / WMS / 数据湖 / 数据仓库 / IoT / 文档 / 外部数据
```

传统架构通常是“上层应用各自连接下层系统”。

Palantir 的思路是：先把下层系统映射到统一 Ontology，再让应用和 AI 面向 Ontology 工作。

---

## 14. 什么时候 Ontology 特别有价值？

Ontology 最适合复杂、动态、跨系统、需要行动闭环的场景。

例如：

- 供应链控制塔
- 制造排程
- 设备预测性维护
- 国防任务规划
- 航空调度
- 医院资源协调
- 金融风险处置
- 客服运营
- 能源网络优化
- 物流与库存管理
- 政府应急响应

这些场景共同特点是：数据来自多个系统；决策需要实时或近实时；决策涉及复杂规则；决策后必须执行动作；执行动作需要权限和审计；AI 不能只“建议”，还要安全嵌入流程。

---

## 15. 可能的误解

### 15.1 Ontology 就是数据目录

不是。数据目录告诉你有哪些数据；Ontology 把数据映射到业务对象、关系、动作和应用流程。

### 15.2 Ontology 就是知识图谱

不完全是。它有知识图谱的语义关系，但更强调运营、动作、权限和决策闭环。

### 15.3 Ontology 是一次性建模项目

不是。企业业务变化，Ontology 也要演化。对象、关系、Action、函数、应用都会随着业务迭代。

### 15.4 Ontology 只是给 AI 用的

不是。Ontology 早于当前生成式 AI 热潮，是 Foundry 的核心能力之一；但在 AIP 时代，它成为 AI Agent 安全落地的关键基础。

### 15.5 有了 Ontology 就不需要数据工程

不是。Ontology 依赖底层数据集、虚拟表、模型、数据同步和治理。它是在数据工程之上建立业务语义和操作层。

---

## 16. 我的归纳：Palantir Ontology 的本质

如果用一句更抽象的话说：

**Palantir Ontology 是一个把企业现实世界形式化、可计算化、可治理化、可操作化的语义操作层。**

它完成四件事：

1. **把现实世界对象化**：客户、订单、设备、任务、事件都变成可计算对象。
2. **把业务关系显式化**：谁影响谁，什么依赖什么，可以被系统、用户和 AI 理解。
3. **把业务逻辑工具化**：规则、函数、模型、优化器成为可调用资产。
4. **把业务行动治理化**：谁能做什么、AI 能否建议或执行、动作如何审计，都被纳入系统。

这也是为什么 Palantir 会把 Ontology 放在 AIP 的核心位置：没有 Ontology，AI 只是聊天或分析；有了 Ontology，AI 才可能进入企业真实运营流程。

---

## 参考来源

- [Ontology building Overview - Palantir Docs](https://palantir.com/docs/foundry/ontology/overview/)
- [Why create an Ontology? - Palantir Docs](https://palantir.com/docs/foundry/ontology/why-ontology/)
- [Core concepts - Palantir Docs](https://palantir.com/docs/foundry/ontology/core-concepts/)
- [Ontology-aware applications - Palantir Docs](https://palantir.com/docs/foundry/ontology/applications/)
- [Models in the Ontology - Palantir Docs](https://palantir.com/docs/foundry/ontology/models/)
- [Palantir AIP](https://www.palantir.com/platforms/aip/)
