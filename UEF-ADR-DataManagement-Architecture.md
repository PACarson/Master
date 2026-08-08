# 待并入治理文件的内容——Domain Data Layer（Repository + DataManagement）

这份文件跟 `UEF-ADR-Communication-Architecture.md` 是同一种文件：两个区块分别对应两个插入位置，第一段插入 `UEF_ADR_Log.md`（生态级 ADR log），第二段插入 UEF 正文的 Candidate Patterns 表（Knowledge Base）。这份文件自己不判断该不该收录、什么时候升级——照 §0.7 / D7 已经确立的格式如实记录，收不收、什么时候升级仍然照 §0.9 的 Change Policy 走。

跟 Communication Architecture 那份不同的是：这份从头到尾都对照过 Rider OS 实际的 Constitution 与 `.xlsx` 资料，不是在没看过真实代码的情况下设计的——但 Evidence 栏位仍然诚实标出「看过资料」跟「还没实作」的差别，这是两件不同的事，不能因为前者成立就含糊带过后者。

---

## 插入 UEF_ADR_Log.md

## UEF-ADR-XXX — Adopt Repository + DataManagement Split for Domain Data Layer（pending implementation）

*（XXX：这次同样看不到 `UEF_ADR_Log.md` 目前实际的编号进度，插入时接续下一个可用编号）*

**Status:** Accepted — 设计经过多轮讨论定案，Evidence 栏位有明确保留条件（见下）
**Date:** 2026-08-07

**Context:** Rider OS 是这个生态里第一个、也是目前唯一有真实 Constitution（P1: Truth Layer is Sacred、CR13: Single Calculation Source）与真实代码的 Domain OS。它现有的 Current State/Projection 靠 on-demand 重算（不缓存），配合倒序扫描找「最新符合条件的记录」——这个模式目前在 `getLastClosedShift()`、`_getLastFuelRecord()`、`getTruthKM()` 等函数里各自独立实作。同一时间，用户在规划所有 Domain OS 共用的 Archive/归档策略，需要先确定：Archive 加进来之后，这些倒序扫描函数要怎么改、改动该集中在哪一层，才不会变成每个 Repository 各自维护一套 Archive fallback 逻辑。

**Question:** Domain OS 的 Data Layer（Blueprint 目前只写「这个 Domain 专属的 Google Sheet(s)，只有 Service Layer 能读写」，没有内部结构）该怎么分层，才能让 Archive、未来可能的 Search Index、甚至未来可能的储存后端迁移（Google Sheets → BigQuery/SQLite），都不需要改动 TruthEngine 或业务逻辑？

**Options Considered:**
(a) 维持现状——每个 scanner 函数各自写自己的倒序扫描逻辑，Archive fallback 需要时各自补 ·
(b) 所有写入统一经过一个 `logEvent()` 强制入口，TruthEngine 自己判断要不要连 Archive 一起查 ·
(c) 有 Repository 层，但直接呼叫裸 `SpreadsheetApp`（`getRows`/`appendRow`/`updateRow`） ·
(d) Repository（各 Domain 自己拥有，提供 `findLastFuel()` 这类业务语言查询）→ DataManagement（跨 Domain 共用，declarative `read`/`append`/`update`/`archive`/`backup`，StorageAdapter 藏在其内部）→ Google Sheets/Drive；`read()` 用 `where: {column, equals}`（非 predicate function）+ `mode`（`CURRENT`/`CURRENT_AND_ARCHIVE`/`ARCHIVE_ONLY`/`AUTO`）；DataManagement 一开始先做成 Rider OS 内部模块，API 稳定后才抽成 GAS Library

**Decision:** (d)。(a) 会让 Archive 一来要改的地方随 Repository 数量增加；(b) 让 TruthEngine 知道 Current/Archive 存不存在，跟 Domain 只管业务真值的定位冲突；(c) 的 predicate function 没办法翻译成未来 SQL 的 `WHERE` 子句，等于口头上说要能换储存后端、实际做不到。

**Evidence:** **验证过的**——Rider OS 实际 `00_Project_Constitution.js`（P1、CR13 原文）、实际函数实作（`getLastClosedShift()`、`_getLastFuelRecord()`、`getTruthKM()`）、实际 `.xlsx` 资料（20 个分页全部读过，含笔数与栏位；`Reminder_History` 确认是空的，`Reminder_Queue` 确认 SENT 纪录从未搬移；从各表首笔日期算出实际增速：`Event_Log` ~582 笔/月、`Alerts_Log` ~358 笔/月、`Daily` ~84 笔/月、`Fuel` ~57 笔/月、`Maintenance` ~14 笔/月）。Google Sheets/Apps Script 平台限制这次重新查证（非沿用旧资料）：单一 Spreadsheet 上限 1000 万个 cell、算全部分页加总非单一分页；超过约 10 万行开始有感变慢；Apps Script 单次执行硬上限 6 分钟（免费与 Workspace 帐号相同）；同一 script 最多 20 个 time-driven trigger。

**没验证过的**——DataManagement 模块与任何一个 Repository 都还没有实际写出来、没有跑过、没有对照 UCR1/2/3/6/7 检查过。这套分层只在 Rider OS 讨论过，没有第二个 Domain OS 独立验证——不满足 §0.9 / 本文件 D7 的二项目证据门槛，Tier 上还够不到 Tier 2（Tier 2 需要「a real implementation exists」），比一般 Tier 3 多了真实资料佐证，但仍是概念阶段。

**Impact:** Rider OS 现在有具体的 Data Layer 目标设计，可以直接照这个开始写模块。这条 ADR 被记录，**不会**让这套设计自动升级成 Blueprint Tier 1/2 或正式 UCR——升级仍要走两项目证据门槛。下方 Candidate Patterns 条目让这套形状保持「未定案」，直到真的实作、跑过，且有第二个 Domain OS 独立做出同样的分层。

**Next Steps:**
1. 在 Rider OS 专案内新增 DataManagement 模块（暂不发布成独立 GAS Library），先实作最小的 `read()`/`append()`/`update()`，`read()` 支援 `where`（declarative）、`orderBy`、`limit`、`mode`（先只做 `CURRENT`，其余 Archive 真的接上后再实作）。
2. 把 `_getLastFuelRecord()` 改写成 `FuelRepository.findLastFuel()`，内部呼叫 DataManagement 的 `read()`——验证这条链路在真实 GAS 环境下能跑。
3. 用 `Reminder_History`（目前是空的，栏位已设计好要接 Archive，且不影响 KM/Earnings 等核心真值）作为第一个真正接上 Archive 逻辑的试点。
4. `Event_Log` 重新命名（例如 `Execution_Log`），跟 `Alerts_Log` 一起在 Constitution 里明确归类成「System Log」，避免以后跟「Event Store」这个业务概念混淆——成本低，可以跟 Step 1 同时做。
5. API 经过 Rider OS 实际验证、跑稳定一段时间后，才考虑抽成独立 GAS Library 给其他 Domain 用；届时如果有第二个 Domain OS 独立采用同样分层，回来更新本条 Evidence，推进 Tier 状态。

**Related ADRs:** 无直接关联的既有 ADR——这是这个生态第一次针对「Domain 内部 Data Layer」（Blueprint 现有描述之下一层）的分层设计。跟 Communication Architecture（另一份 UEF-ADR-XXX）是同类但不同层的姊妹关切：一个处理 Domain 之间怎么通讯，这个处理 Domain 自己内部资料怎么存取，彼此不冲突、不重叠。

**Review Trigger:** Rider OS 的 DataManagement 模块与至少一个 Repository（例如 `FuelRepository`）真的写出来、在真实 GAS 环境跑过之后——尤其要看 `where`/`mode` 这套 declarative 查询形状够不够用，还是常常需要退回裸 predicate。一旦有第二个 Domain OS 独立采用类似分层，两者一起回来更新本条。

---

## 插入 UEF 正文的 Candidate Patterns 表

| Candidate | Source | What would confirm it |
|---|---|---|
| **Domain Data Layer 三层分离**——TruthEngine 只呼叫自己 Domain 的 Repository（业务语言查询，如 `findLastFuel(vehicleId)`）；Repository 呼叫跨 Domain 共用的 DataManagement（`read`/`append`/`update`/`archive`/`backup`，全部走 declarative `where: {column, equals}` 而非 predicate function，让 Google Sheets → BigQuery/SQLite 之类的储存后端迁移不用改上层；`mode` 参数决定查 Current／Archive／两者都查）；StorageAdapter（真正碰 Sheets/Drive 的最底层）藏在 DataManagement 内部，Repository 不直接接触。DataManagement 完全不知道 Fuel/KM 这类业务概念，也不读各 Domain 的 CONFIG（`spreadsheetId`/`sheetName` 一律当参数传入）。 | Rider OS 设计讨论，2026-08-07——对照过 Rider OS 实际 Constitution 与 `.xlsx` 资料，但 Repository/DataManagement 代码本身还没写 | 第二个 Domain OS（Property OS、Finance OS 等）独立采用同样的三层分离；或 Rider OS 实际实作、跑过，证明这套 declarative where/mode 形状在真实场景下站得住 |
| **Business Event 与 System Log 分表分名**——业务事件（Fuel/Maintenance/Daily/Booking 等）与系统执行日志（现有 `Event_Log`、`Alerts_Log`，记录 API 呼叫/耗时/成功与否，非业务事件）明确分开命名，避免「Event Store」这个词在 Domain 内部同时指两种不同的东西 | Rider OS 实际 `.xlsx`——发现 `Event_Log`（栏位：EventID/EventType/Duration_ms/Result）实际内容是执行遥测，不是业务事件流 | 第二个 Domain OS 独立需要（或已经有）同样的 Business Event / System Log 命名区分 |
