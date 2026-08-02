# ADR-019：Communication Protocol（通讯协议）

**Status**: Proposed — 内部一致，但尚未对照 Personal AI Core / Rider OS / Reminder OS 现有的 EventBus-equivalent 与 TruthEngine/Execution 代码验证（见 Session-Handoff-Brief.md）
**Date**: 2026-07-29
**适用范围**：所有 Domain OS 之间、Domain OS 与 Execution OS / Decision OS / Personal AI Core 之间的跨系统通讯

> 本文件是 Communication Architecture 系列 ADR 之一（共 3 份）：
> - **ADR-019 — Communication Protocol**（本文件）
> - ADR-020 — Domain Communication Rules（`ADR-020-Domain-Communication-Rules.md`）
> - ADR-021 — Communication Registry（`ADR-021-Communication-Registry.md`）
>
> 三份合起来是 Universal Blueprint 的正式设计文档。本文件是系列的入口，收录共同背景、三份 ADR 共用的 Freeze Boundary 总表、Rollout Phases 与设计演进 Appendix；ADR-020、021 各自聚焦自己的决定，遇到共同内容会回指到这里。任何 AI（Claude、ChatGPT、本地 AI）即使没有原始讨论记录，也应该能仅凭这三份文件完整理解这套架构与它背后的理由。

## 如何阅读本文件

本文件冻结的是 **Communication Contract**（格式、协议、规则），不是 **Implementation**（具体某个 Connector 怎么写、某个 Domain 的业务逻辑）。JSON 区块中出现的 `"字段 | null"`、`"uuid"` 等写法是文档记号，不是字面上合法的 JSON，代表类型或格式说明。

---

## 背景：这三份 ADR 从何而来

最早的讨论从三种候选通讯架构的比较开始：

- **Mesh**（所有 Domain 互相直接呼叫）——被否决。Domain 数量一旦到两位数，N² 级别的直接连结会变成难以维护的网状结构，也容易出现循环依赖。
- **Hub**（所有跨 Domain 通讯都经过 Execution OS）——被否决。多数跨 Domain 通讯其实只是"某件事发生了，另一个 Domain 需要知道"这种简单通知，如果都要绕经 Execution OS，会不必要地增加延迟与耦合。
- **Hybrid**（事实经 EventBus 广播，多步骤流程走 Orchestrator，Domain 永远拥有自己的资料）——被采用，成为后续所有讨论的起点。

在具体设计"Execution OS 到底怎么让别的 Domain 做一件事"时，发现光有 EventBus 不够用——这催生了 Fact 与 Command 的正式区分，以及 EventBus / CommandBus 双总线设计，也就是这三份 ADR 要正式定案的内容（见 ADR-020）。

另外，早期讨论也发现 Personal AI Core（Universal Layer）与 Execution Layer（跨 Domain 引擎）两份清单里，Memory、RAG、Context Builder、Prompt Manager、Knowledge Base、Audit、AI Capability Registry 几乎是同一批能力被重复列了两次。这个发现直接影响了后续"Connector 该归属哪一层"的判断逻辑（见 ADR-020）：既然 Execution OS 不该独占原本属于共用层的能力，Connector 也不该独占，而该归属大家共用的 Universal Infrastructure。

现阶段所有实现都在 Google Apps Script + Google Sheets 上，但长期目标从来不是"做 Apps Script"，而是建一个与技术栈无关的 AI Operating System，未来要接本地 AI、Docker、REST API、多语言 Runtime。这个前提贯穿了以下所有决定。

---

## Context

生态由多个 Domain OS 组成（Rider、Finance、Property、News、Personal Life、Reminder 等），每个 Domain 都是独立的 Apps Script Project（见 ADR-020）。这些 Domain 之间必须能互相传递信息，但不能互相直接修改对方的资料。确定 Hybrid 架构之后，需要一个具体协议来实现它，且这个协议必须与 Apps Script 无关，Apps Script 只能是第一种实现，未来能无痛换成任何语言、任何 Runtime。

## Decision

1. **采用 HTTP + JSON 作为唯一的跨 Domain 通讯协议**，不采用 Apps Script Library。
2. **Event（Fact）与 Command 使用统一的信封（Envelope）格式**：

**Event Envelope**
```json
{
  "eventId": "uuid",
  "eventType": "RideCompleted",
  "version": 1,
  "sourceDomain": "RiderOS",
  "timestamp": "ISO 8601",
  "correlationId": "uuid",
  "causationId": "uuid | null",
  "chainDepth": 0,
  "payload": {}
}
```

**Command Envelope**
```json
{
  "commandId": "uuid",
  "commandType": "CreateReminder",
  "version": 1,
  "correlationId": "uuid",
  "causationId": "uuid | null",
  "chainDepth": 0,
  "payload": {}
}
```

**CommandResult**
```json
{
  "commandId": "uuid",
  "correlationId": "uuid",
  "status": "SUCCESS | FAILED | REJECTED",
  "errorCode": "string | null",
  "errorMessage": "string | null",
  "duration": 0,
  "targetDomain": "ReminderOS",
  "timestamp": "ISO 8601"
}
```

3. **每个 Domain 只有一个统一的服务端点**（现阶段是它自己 Apps Script Web App 的部署 URL），端点内部统一接受上述 Command Envelope，由 Domain 自己依 `commandType` 分派到对应 Service，**不采用按操作切分路径的 REST 端点**（如 `POST /reminder/create`）。
4. **不引入 traceId**。系统统一使用 correlationId（整条业务流程）+ causationId（直接前因）+ chainDepth（级联深度）三个字段。
5. **Connector 归属 Universal Infrastructure**，具体是 Communication Layer 里 Command 的一侧，与 Event Registry 所在的 Fact 一侧对称，被 Execution OS、Decision OS、Personal AI Core 三者共用，不归属任一方（细节见 ADR-020）。
6. 现阶段认证机制为**共享密钥（Shared Secret）**，每个 Domain 各自持有一把（不是全系统共用一把），由呼叫方在请求体中带上；未来可升级为 JWT。

## Alternatives Considered

**A. Apps Script Library**（供 Execution/Decision/AI Core 直接呼叫每个 Domain）
被否决。原因：(1) Library 是 Apps-Script-to-Apps-Script 专属机制，Docker、Python、本地 AI 完全无法呼叫；(2) 根据 Google 官方文档，使用 Library 的脚本执行速度会比整段代码写在同一个项目里慢，因为 Library 调用是在呼叫方那一次执行里完成的，等于共用呼叫方的执行时间与配额——两个"独立"的 Domain，执行预算却悄悄绑在一起，这跟 Domain Isolation 的精神冲突。

**B. Google `scripts.run`（API Executable + OAuth）**
被否决。此机制要求呼叫方与目标脚本挂在同一个 GCP 项目下，还需要处理 OAuth token 的取得与刷新，对个人系统而言比"HTTP POST 一个 JSON + 共享密钥"复杂得多，且没有对应的额外好处。

**C. 每个 Domain 按操作切分 REST 路径**（`POST /reminder/create`、`POST /finance/expense/create`）
被否决。原因有两层：(1) 实务上，Apps Script Web App 的部署 URL 固定是 `/macros/s/{id}/exec`，本身不支持路径式路由，Google 官方范例的做法就是在同一个端点内用一个字段做条件分派，一个非官方的路径路由功能存在，但不论存取设置如何都要额外处理使用者 access token，为一个 Google 随时可能移除的功能增加这层复杂度不划算；(2) 即使未来换到真正支持路径路由的平台（Node/Python），统一信封仍然更简单——Connector 永远只需要知道"这个 Domain 的一个 URL"，不需要为每个操作记住不同路径，减少一整类"路径拼错"的 bug，也让 Command Registry 不需要为每个 commandType 各自存一条 URL。

**D. CommandBus 做成第二个 Sheet 轮询队列**（与 EventBus 同样的机制）
被否决。Fact 天生异步（RideCompleted 发布后，Finance 十分钟后处理没有人在等），轮询合适；但 Command 通常需要立即知道成功或失败（比如 AI 想知道 CreateReminder 到底建成功没有），如果做成轮询队列，会把 Command 最需要的"立即拿到结果"这个特性做丢。改采用同步 HTTP 请求，直接拿到 CommandResult 回应。

**E. traceId 作为第三个追踪 ID**
被否决。不是因为它与 correlationId 概念重复，而是因为它要解决的具体问题——单次执行内部的耗时拆解——已经被 `duration` + `timestamp` + `causationId` 覆盖：`duration`/`timestamp` 能还原时间线，`causationId` 能还原"这几个 Command 是不是同一个反应的兄弟节点"。多加一个概念只会增加以后不同 AI（Claude、ChatGPT、本地 AI）各自发明不同名字的风险。
*前提*：这个结论成立的前提是 `duration` 与 `timestamp` 必须被如实记录，不能大略估算。

## Rationale

选 HTTP 而非 Library，根本原因不是"Apps Script 做不到"，而是长期目标从来不是"做 Apps Script"，是建一个与技术栈无关的 AI Operating System。HTTP 是唯一一个从今天（Apps Script 互相呼叫）到未来（Docker、本地 AI、Node、Python、Rust 互相呼叫）都不需要更换协议的选项——不是"为了未来牺牲现在"，而是现在能拿到的、跟未来完全同形的最简单方案。围绕 HTTP 的失败模式（超时、5xx、网络中断）现在就写清楚重试/错误处理，以后任何 Domain 换语言、换 Runtime，这段逻辑完全不用重写；如果先照 Library 的失败模型（只会抛异常，没有真正的超时/网络失败）写，以后必然要重做一次。

统一信封而非按操作切路径同理：Connector 的呼叫方代码永远只认 `execute(commandType, payload)`，不需要因为 Domain 内部新增了一个操作，就跟着改自己的路由逻辑。

## Consequences

- ✅ Domain 换语言/换平台时，Connector 呼叫方代码完全不用改，只需要更新 Service Registry 里的 URL。
- ✅ 重试/超时/错误处理只需要写一次，覆盖所有 Domain、所有未来 Runtime。
- ✅ Command Registry 不需要为每个 commandType 各自存 URL，统一查 Service Registry。
- ⚠️ HTTP 比 Library 多一次网络往返延迟（对个人系统体量而言可忽略）。
- ⚠️ 需要自行处理部署纪律（Apps Script Web App 要用「管理部署」更新既有部署，不能每次建新部署，否则 URL 改变）与访问控制（Web App URL 本身不是安全凭证，需要应用层的共享密钥校验）。

## Deferred Decisions

- Shared Secret 何时升级为 JWT，没有明确的时间点或触发条件，留待未来需要更严谨的存取控制时再决定。
- 共享密钥应存放在哪里（Service Registry 的一个栏位，还是各 Domain 自己的 Script Properties）——本窗口并未明确讨论。**建议**：不要存在 Registry Sheet 里（Registry 是要给 AI 读取做 Context Building 用的，明文密钥不该出现在这类文件里），改存在 Connector 与各 Domain 自己的 Script Properties，Registry 只存 URL、Version、Status、Capabilities 等非敏感栏位。此为整理时补充的建议，非讨论逐轮定案内容，落地前请再次确认。

## Frozen（本 ADR 冻结的内容）
- Event Envelope / Command Envelope / CommandResult 的栏位结构
- 协议：HTTP + JSON
- 单一端点 + 信封内 commandType 分派（不用 REST 路径切分）
- 不引入 traceId 的决定
- Connector 归属 Universal Infrastructure（Command 侧）

---

## 相关 Conflicts（完整列表见本文件「Conflicts」总节；以下为与本 ADR 直接相关者）

**同一项目假设被推翻**：Connector 实现的早期示例代码曾假设"如果 Execution OS 与 Reminder OS 在同一个 Apps Script 项目"，这个分支后来被确认不成立（所有 Domain 确认为独立项目）。**最终版本：一律采用跨项目 HTTP 呼叫。**

**correlationId / causationId 曾遗漏 Command 一侧**：最初只被列在 Event Envelope 上，Command Envelope 一度没有这两个字段，导致 Fact → Command → Fact 的追踪链会在"Fact 变成 Command"这一步断掉。**最终版本：Command Envelope 与 CommandResult 都带上 correlationId（Command 也带 causationId），见本文件信封定义。**

**chainDepth 曾被遗漏**：在重新罗列 Event Envelope 栏位时被遗漏，几乎导致这个早先谈妥的级联防护机制被静默拿掉。**最终版本：chainDepth 恢复为 Event/Command Envelope 的固定栏位。**

**原始 Hybrid 架构的执行机制缺口**：最初设想 Execution OS 直接"建 Mortgage、建 Reminder、更新 Finance"，但没有说明这些动作具体怎么跨 Domain 执行，也没说明如何在不违反 Domain Isolation 的前提下做到。此空缺在设计 Dual Bus Architecture 时被填补：Execution OS 不直接做任何事，只 Dispatch Command，由 Connector 转交给目标 Domain 自己的 Service 执行（详见 ADR-020）。

## 相关 Open Questions

- **共享密钥应该存放在 Registry Sheet 里还是各自的 Script Properties？** 见上方 Deferred Decisions，本文件给出建议但非讨论定案。

---

## 跨 ADR 核心原则速查

| 原则 | 定义在 | 一句话 |
|---|---|---|
| Fact vs Command | ADR-020 | 已发生的事实 vs 希望发生的动作 |
| EventBus / CommandBus | ADR-019/020 | 双总线，职责完全分离 |
| Domain Isolation | ADR-020 | 各自独立部署，只能透过公开 Service 互动 |
| Connector 双层 | ADR-020 | 共用机制是 Infrastructure；每个 Domain 的入口属于 Domain 自己 |
| Idempotency 归属 | ADR-020 | Domain Service 判断，Connector 只传 commandId |
| Registry 驱动 | ADR-021 | Connector/Context Builder 永远查 Registry，不查硬编码 |
| 警告优先 | ADR-020 | 已发生的事实不该被文件问题拦下；无法理解/无处可送的动作不能静默视为完成 |
| chainDepth | ADR-019/020 | 字段冻结，阈值可调 |
| Version vs Stability | ADR-021 | 前者是契约版本号，后者是这份契约还该不该被用 |

---

## Architecture Freeze Boundary（三份 ADR 共同的总表）

**Frozen（不得随意修改，改动需要新的 ADR）**
- Event Envelope / Command Envelope / CommandResult 栏位结构（ADR-019）
- HTTP + JSON 协议、单一端点 + 信封内分派（ADR-019）
- EventBus / CommandBus 职责分离规则（ADR-020）
- Registry 结构（Event / Command / Service 三个子 Registry 的栏位）（ADR-021）
- Domain Communication Contract（Domain Isolation、Connector 职责边界、Idempotency 归属）（ADR-020）
- 已标记 Stable 的具体 eventType/commandType payload schema 版本内容（要改内容开新 version，不能编辑既有 Stable 版本）（ADR-021）

**Evolutionary（允许持续扩展，不需要新 ADR）**
- 新增 Event Type / Command Type（在 Registry 加一行）
- 新增 Domain（自己部署 Web App + Service Registry 加一行）
- 新增 Connector 对某个 Domain 的具体实现
- Domain Service 内部业务逻辑
- Workflow 编排逻辑（Execution OS 内部决定哪些 Fact 触发哪些 Command）
- chainDepth 的具体阈值数字、轮询频率、触发器分配等运行参数
- Registry 里某个契约从 Beta 升级到 Stable，或从 Stable 转为 Deprecated（状态变动，不是结构变动）

---

## Rollout / Validation Phases

「Phase 1」不是要一次做完整个系统，而是先验证 Communication Contract 本身是否可行——这也是本系列讨论一路坚持的原则：先验证架构，再扩展生态。三个 Phase 依序进行，前一个没跑通，不进入下一个；Phase 2 之后的具体细节属于 Evolutionary，不受 Freeze Boundary 约束。

**Phase 1 — Communication Validation**：验证 Rider → Finance（Fact）与 Execution → Reminder（Command）两条最小链路，确认 Communication Contract 可行。

*Fact Flow*：Rider OS --publish ExpenseCreated--> EventBus --poll--> Finance OS --> 建立 Expense Record
验证项：EventBus 发布是否正常、Finance OS 订阅是否正常、correlationId 是否保留、chainDepth 是否正确递增、Event Registry 查询是否正常、Audit 是否有记录。

*Command Flow*：Execution OS --CreateReminder--> Connector --HTTP--> Reminder OS --> ReminderService.create() --> CommandResult
验证项：Connector 能否从 Service Registry 找到目标、HTTP 请求是否成功、commandId 是否一致、幂等是否生效、CommandResult 是否正确回传、失败时是否正确产生对应的 REJECTED/FAILED 状态。

两条都跑通，代表 EventBus、CommandBus、Connector、三个 Registry 全部验证可行，才算 Phase 1 完成。

**Phase 2 — Domain Expansion**：让 Property OS、Personal Life OS、Inventory & Procurement OS 等其他 Domain 接入既有 Contract，不修改 Communication Layer 本身。

**Phase 3 — Decision & AI Integration**：接入 Decision OS、AI Core、本地 AI 等高级能力，包括：Decision OS 消费 Fact、产生判断、Dispatch Command；Personal AI Core 批量处理 Insight；AI Router 与 Context Builder 让多个 AI（本地 AI、Claude、GPT、Gemini 等）走同一套 Communication Contract；以及各 Domain 依需要逐步从 Apps Script 迁移到其他语言与 Runtime（Connector 呼叫方式不变，只更新 Service Registry 的 baseUrl）。

---

## Appendix：设计演进记录

**V1 — 单一 EventBus**：最初设想只有一个 EventBus，Execution OS 直接编排多步骤流程，跨 Domain 执行细节未定义。

**V2 — 引入 Connector 概念**：意识到"Execution 想让某件事发生"跟"广播一个已发生的事实"是两回事，开始讨论 Command 与 Connector。

**V3 — Execution / Decision / Personal AI Core 角色分离**：三者都被定义为可以消费 Fact、必要时 Dispatch Command 的平行角色，而不是 Execution 独大。

**V4 — Dual Bus Architecture 定案**：EventBus（Fact）与 CommandBus（Command）正式分离为两条总线，workflow_trigger 被否决，correlationId/causationId 引入。

**V5 — Communication Registry 定案**：Event Registry、Command Registry、Service Registry 三个子 Registry 明确划分，capabilities 改为推导而非手动维护。

**V6 — Envelope Freeze**：Event/Command Envelope、CommandResult 栏位定案，chainDepth 补回，Architecture Freeze Boundary 正式区分 Frozen 与 Evolutionary，三份 ADR 文件成形。

---

*本文件记录至 2026-07-29。后续如需修改 Frozen 区块任何内容，请开新的 ADR 并在此处注明取代关系，不要直接编辑本文件的 Frozen 部分。*
