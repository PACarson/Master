# 待并入治理文件的内容

这份文件的两个区块分别对应两个插入位置：第一段插入 `UEF_ADR_Log.md`（生态级 ADR log），第二段插入 UEF 正文的 Candidate Patterns 表（Knowledge Base）。两段都不是这个窗口自行判定该不该收录——是照 §0.7 / D7 已经确立的格式如实记录，收不收、什么时候升级，仍然照 §0.9 的 Change Policy 走。

---

## 插入 UEF_ADR_Log.md

## UEF-ADR-XXX — Adopt Dual Bus Fact/Command Communication Architecture（pending code-level verification）

*（XXX：这个窗口看不到 `UEF_ADR_Log.md` 目前实际的编号进度，插入时接续下一个可用编号）*

**Status:** Accepted — 项目所有者直接授权记录，Evidence 栏位有明确保留条件
**Date:** 2026-07-31

**Context:** Domain OS 项目之间需要一种方式，既能通知彼此"发生了什么"（Fact），也能请求对方"做一件事"（Command），同时不违反 Domain Isolation（任何项目不能直接读写另一个项目的 Sheet），也不能陷入 mesh（所有项目互相直连、日后无法维护）或不必要的 hub（简单通知也要绕经 Execution OS）两个极端。这套设计是在另一个聊天窗口里做出来的，没有对照过这个生态实际的 EventBus / Execution 代码——详见 Evidence。

**Question:** 跨 Domain 通讯该怎么设计，这套设计跟 Blueprint 里已经是 Tier 1 的 Event / Execution 节点是否冲突或重复？

**Options Considered:**
(a) Mesh——所有 Domain 互相直接呼叫 ·
(b) Hub——所有跨 Domain 通讯都经过 Execution OS ·
(c) 单一 EventBus，Execution OS 直接呼叫其他项目编排多步骤流程 ·
(d) Dual Bus——EventBus 负责 Fact（异步广播），另立 CommandBus + Connector 负责 Command（同步 request/response），Connector 本身依 UCR7 的 Adapter 隔离原则设计

**Decision:** (d)。完整推导过程、被否决的方案、Envelope 与 Registry schema，记录在同一个窗口产出的 Communication Architecture ADR-019（Protocol）、ADR-020（Domain Communication Rules）、ADR-021（Communication Registry），与本条一并附上。

**Evidence:** 这个窗口实际验证过的：Google Apps Script 平台限制（执行时间与触发器配额、确认 installable trigger 不会因脚本或 API 写入而触发、Web App 没有原生路径路由——见 `Apps-Script-Platform-Constraints.md`）。**没有验证过的**：Personal AI Core / Rider OS / Reminder OS 现有、已经是 Blueprint Tier 1（三项目证据）的 EventBus-equivalent 与 TruthEngine/Execution 模式，是否已经有类似的 Fact/Command 区分、不同形状的区分、或完全没有区分。这个窗口产出的 Phase 1 代码（Rider OS publishEvent、Finance OS pollEventBus、Reminder OS Web App、ConnectorLib、Execution OS 测试骨架）**没有**对照过 UCR1（IIFE）、UCR2（前缀 `_`）、UCR3（AlertService.log）、UCR6（TruthEngine-equivalent，不能用裸 `appendRow()`）、UCR7（Adapter 隔离）检查——已确认与 UCR6 直接冲突（多处直接用了裸 `appendRow()`），文件后缀用的是 `.js`，不是 D8 定案的 `.js`。这一栏照 EP5 的精神如实写：没做过的检查，不能因为现在写进治理文件了就含糊带过。

**Impact:** Execution OS、Decision OS、Personal AI Core 现在有一份 Command 派发（Connector）与 Fact 传播（EventBus）的目标设计，加上一版 Phase 1 草稿实作。这条 ADR **不会**因为被记录，就让 Communication Architecture 自动升级成 Blueprint 的 Tier 1 或一条正式 UCR——升级仍然要走 §0.9 的两项目证据门槛。下方的 Candidate Patterns 条目，就是让这套设计的具体形状先保持"未定案"的状态，直到真的对照过。

**Next Steps:**
1. 读 Personal AI Core / Rider OS / Reminder OS 实际的 EventBus-equivalent 与 TruthEngine 代码，再决定是否继续往这个方向扩展。**注意版本**：项目所有者确认 UEF/Blueprint 本身也随各 OS 持续开发在更新，核对时以当下最新版本为准，不要假设仍是 v1.5 / v1.2。
2. 让 Phase 1 代码符合 UCR1/2/3/6/7 与 D8，或者写清楚为什么这里该是例外，再把它当作草稿以上的东西对待。
3. 如果真实的 EventBus-equivalent 已经有类似 Fact/Command 的区分，把两套设计合并成一套，用一条明确的 ADR 记录谁取代谁——不要让两套 Communication Architecture 并行存在。
4. 如果没有，这套设计就是未来 Blueprint 正式收录的候选，等第二个独立项目真的照这个形状建出来（§0.9）再升级。

**Related ADRs:** ADR-019 / ADR-020 / ADR-021（Communication Architecture 细节，同一个窗口产出）。下方 Candidate Patterns 条目（Knowledge Base）。

**Review Trigger:** 一旦读过 Personal AI Core、Rider OS、或 Reminder OS 至少其中一个项目实际的 EventBus / TruthEngine / Execution 源码，并且拿来跟这套设计对照过。

---

## 插入 UEF 正文的 Candidate Patterns 表

| Candidate | Source | What would confirm it |
|---|---|---|
| **Dual Bus Fact/Command separation** — 跨 Domain 通讯分成 Fact（EventBus，异步广播，Sheet 当 append-only log，订阅端游标存在自己的 Script Properties，不是共享列）与 Command（CommandBus，同步 HTTP request/response，经 Connector 路由，Connector 本身不含 business logic，只做 Routing/Validation/Permission/Retry/Idempotency-key 传递）两种，都带标准 Envelope（id、version、correlationId、causationId、chainDepth、payload）。完整推导、否决方案、schema 见 UEF-ADR-XXX 与 ADR-019/020/021。 | 另一个聊天窗口（讨论产出，未对照任何真实项目代码验证） | Personal AI Core / Rider OS / Reminder OS 现有、已经 Tier 1 的 EventBus-equivalent（Blueprint §2）本身是否已经有 Fact/Command 式的区分，或者是不同的形状——需要先读到真实代码，这条才有资格往 UCR 或 Blueprint Tier 1 移动，或者被取代 |
