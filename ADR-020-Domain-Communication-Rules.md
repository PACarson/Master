# ADR-020：Domain Communication Rules（跨 Domain 通讯规则）

**Status**: Proposed — 内部一致，但尚未对照 Personal AI Core / Rider OS / Reminder OS 现有的 EventBus-equivalent 与 TruthEngine/Execution 代码验证（见 Session-Handoff-Brief.md）
**Date**: 2026-07-29
**适用范围**：所有 Domain OS 之间、Domain OS 与 Execution OS / Decision OS / Personal AI Core 之间的跨系统通讯

> 本文件是 Communication Architecture 系列 ADR 之一（共 3 份）：
> - ADR-019 — Communication Protocol（`ADR-019-Communication-Protocol.md`）
> - **ADR-020 — Domain Communication Rules**（本文件）
> - ADR-021 — Communication Registry（`ADR-021-Communication-Registry.md`）
>
> 共同背景、Envelope 定义、Freeze Boundary 总表、Rollout Phases 与设计演进 Appendix 都在 ADR-019；本文件聚焦 Domain 之间该遵守的通讯规则本身。冻结的是 **Communication Contract**，不是 **Implementation**。

## Context

生态包含 10+ 个 Domain OS，全部是各自独立的 Apps Script Project。需要一套规则确保新增 Domain 不需要修改既有系统，也不会让系统在几年后变成互相直接调用、无法维护的网状结构。

## Decision

**1. Dual Bus Architecture**
- **EventBus**：广播 Fact（已经发生的事实）。特性：Async、Broadcast、Immutable。发布者不知道谁在订阅，订阅者不知道发布者是否存在。
- **CommandBus + Connector**：分发 Command（希望发生的动作）。特性：Request/Response、同步、明确指定目标。

**2. Fact 与 Command 是唯一的两种跨 Domain 通讯**，不存在第三种类型（包括曾讨论过的 "workflow_trigger"，见下方否决方案）。判断准则：已经发生的、任何人都能独立反应的 → Fact；希望某件事发生、需要知道成不成功的 → Command。

**3. Domain Isolation（固定原则）**
- Reminder OS、Finance OS、Property OS、News OS、Personal Life OS 等全部是独立 Apps Script Project。
- 任何 Domain 不允许直接修改其他 Domain 的 Sheet，不允许直接存取其他 Domain 的 Repository。
- 只能透过该 Domain 对外公开的 Service（经由 Connector 呼叫）互动。

**4. Connector 的职责边界**
- 只负责：Routing、Validation、Permission、Retry、Idempotency-key 传递、CommandResult 包装。
- 永远不能拥有 Business Logic——业务规则永远留在 Domain 自己的 Service 里。
- Connector 实际上分两层：
  - 共用机制（重试、权限检查、路由查找、结果包装）——真正的 Universal Infrastructure，被三个 dispatcher（Execution / Decision / Personal AI Core）共用。
  - 每个 Domain 自己的 `doPost` 入口——属于 Domain 自己的部署与边界，不是"Connector 本体"，只是 Domain 对外暴露 Service 的方式。

**5. Idempotency 由 Domain 的 Service 负责，不是 Connector**。Connector 只负责每次重试都带同一个 commandId；真正判断"这个 commandId 是否已处理过"，必须发生在 Domain 自己的 Service 内（例如维护一份最近处理过的 commandId 记录，重复送达时直接回传上次的结果）。

**6. chainDepth 用于防止级联**。任何因反应另一个 Event/Command 而产生的新 Event/Command，chainDepth = 上一个的 chainDepth + 1。处理到 chainDepth 超过阈值时，只记录 warning、不再继续反应——阈值本身是可调的运行参数（Evolutionary），机制存在与否才是 Frozen 的部分。

**7. Registry 校验失败：警告优先，不做硬拦截——但要分清楚在哪一层**
- **发布 Fact 前**做 Registry 校验（eventType/publisher 是否登记）：若没通过，只写一条 warning 到 audit，事件照常发布。理由：事实本身是真的，不该被文件层级的登记问题卡住。
- **Domain 收到不认识的 commandType**：必须明确回传 REJECTED，不能静默成功或无反应。理由：这不是"文件登记问题"，是"没人知道该做什么"——静默处理等于让呼叫方误以为动作已完成。
- **Connector 找不到 commandType 对应的路由**：同样回 REJECTED（errorCode: NO_ROUTE），而非警告后照样发送——因为跟 Fact 不同，Command 没有一个固定、独立于登记与否的送达位置；没有路由，Connector 根本不知道要送去哪。
- 三者的区别本质上是：**已经为真的事实，不该被文件问题拦下；尚未执行、且无法理解或无处可送的动作，不该被静默当作已完成。**

**8. REJECTED 不重试，FAILED 才重试**。Connector 重试逻辑只对 FAILED（送达了但 Domain 执行出错）生效；REJECTED（认证失败、找不到路由、不认识的 commandType）重试也不会有不同结果，不重试。

## Alternatives Considered

**A. CommandBus 做成第二个 Sheet 轮询队列** — 详见 ADR-019 Alternatives D，因 Command 需要同步结果而否决。

**B. Connector 挂在 Execution Layer 底下**
被否决。因为 Decision OS 和 Personal AI Core 也会 Dispatch Command，如果 Connector 属于 Execution OS，另外两者要嘛各自重写一份一样的路由/重试/权限逻辑，要嘛绕道透过 Execution OS 才能发命令——后者正是 Dual Bus 架构原本要消灭的那种"暗地里还是要经过谁"的耦合。

**C. `kind: fact | workflow_trigger` 作为 Event Registry 的一个栏位**
最初被提出用来解决"怎么知道一个多步骤流程卡在哪一步"，后来被否决并修正：workflow_trigger 不是事件本身的属性，是 Execution OS 这一个订阅者对该事件的反应方式（同一个 PropertyPurchased，对 Finance 只是记一笔账，对 Execution 才值得追踪成一个 workflow）。最终改为：Execution OS 自己维护一份私有的"我把哪些 eventType 当 workflow 起点"名单，在自己的 `workflows` tab 里追踪进度，不写进公开的 Event Registry。

**D. EventBus 用"每个订阅者一个 processed 列"追踪订阅状态**
被否决，改用"订阅者自己在 Script Properties 存游标"。前者需要 EventBus 的 Sheet 结构随订阅者增加而修改；后者让 EventBus 完全不需要知道有谁在订阅，新增订阅者零成本。

**E. 手动计算行号再写入 Sheet**（`getLastRow()+1` 再 `setValues()`）
被否决，改用 `appendRow()`。手动算行号在并发写入下有竞态风险，`appendRow()` 把这段逻辑交给 Sheets 后端处理。

## Rationale

Fact/Command 分离对应真实分布式系统里常见的 CQRS 思路：查询/事实一条路，命令一条路。这个分离让"谁拥有资料"（永远是 Domain 自己）与"谁负责通讯基础设施"（EventBus/CommandBus/Connector，谁都不拥有）彻底分开，是这套设计能撑过"未来有 10+ 个 Domain、多语言 Runtime"的关键。

## Consequences

- ✅ 新增 Domain 完全不需要修改既有 Domain 或 Communication Layer 本身。
- ✅ Execution / Decision / AI Core 三者可以平等地发命令，不互相依赖。
- ⚠️ Idempotency 的实际实现分散在每个 Domain 自己的 Service 里，需要每个 Domain 各自记得实现，这是刻意的设计取舍，不是遗漏。

## Deferred Decisions

- Execution OS、Decision OS、Personal AI Core 彼此之间是否也是独立 Apps Script Project？本窗口从未明确回答，见下方 Open Questions。
- chainDepth 的具体阈值数字，留给实作阶段依实际情况调整。

## Frozen（本 ADR 冻结的内容）
- Dual Bus 模型（EventBus 广播 Fact，CommandBus+Connector 分发 Command）
- Domain Isolation 规则
- Connector 职责边界（不含业务逻辑）
- Idempotency 判断权在 Domain Service
- chainDepth 机制存在（阈值本身是 Evolutionary）
- Registry 校验的"警告 vs 拒绝"分层原则
- REJECTED 不重试

---

## 相关 Conflicts（完整列表见 ADR-019；以下为与本 ADR 直接相关者）

**原始 Hybrid 架构的执行机制缺口**：最初设想 Execution OS 直接"建 Mortgage、建 Reminder、更新 Finance"，但没有说明这些动作具体怎么跨 Domain 执行，也没说明如何在不违反 Domain Isolation 的前提下做到。此空缺在设计 Dual Bus Architecture 时被填补：Execution OS 不直接做任何事，只 Dispatch Command，由 Connector 转交给目标 Domain 自己的 Service 执行。

**workflow_trigger 提出后又被否决**：见上方 Alternatives C。最终版本：Event Registry 不含 kind 栏位。

**"警告 vs 拒绝"表面上的不一致**：Registry 校验失败，在 Fact 与 Command 两种情境下表面上给出了不同答案。这不是真正的矛盾，而是两种情境本质不同：Fact 有固定的发布位置（不受登记与否影响），Command 的送达位置完全依赖 Registry 查找结果。两者并存，见上方第 7 条的完整说明。

## 相关 Open Questions

- **Execution OS、Decision OS、Personal AI Core 彼此之间，是否也是各自独立的 Apps Script Project？** 目前只确认了 Domain 层（Reminder/Finance/Property/News/Personal Life OS）全部独立，三个 dispatcher 之间的关系从未被明确问过或回答。这会影响它们共用的 Connector Library 该怎么部署（同项目内直接 import，还是也需要走 Apps Script Library 跨项目共用）。

---

*完整的 Freeze Boundary 总表、Rollout / Validation Phases 与设计演进 Appendix，请见 ADR-019。本文件记录至 2026-07-29；后续如需修改 Frozen 区块任何内容，请开新的 ADR 并注明取代关系，不要直接编辑本文件的 Frozen 部分。*
