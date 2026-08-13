# 待并入治理文件的内容——Knowledge Governance OS

跟另外两份 `UEF-ADR-*.md` 是同一种文件：待插入 `UEF_ADR_Log.md` 与 Candidate Patterns 表。这份的特殊之处是——本 session 已经用手动方式跑过一次真实 pilot（对象是 UEF 自己的 v1.5→v1.11 版本链），下面的 Recommendation Record 不是示范用的假资料，是刚刚验证出来的真实结果。

**范围说明，先讲清楚：** 「Data Management OS」这个名字这几轮下来同时指过两件不同的事——(a) 单一 Domain 自己的业务资料（Rider OS 的 Fuel/Daily/Maintenance，透过 Repository/StorageAdapter/Archive/Backup），(b) 整个生态的治理文件（UEF/ADR/Constitution/Universal Files，透过 Scan/Diff/Dependency/Version/Recommendation）。两者共用底层 Storage 能力，但业务职责不是同一件事，往后分开称呼：

```
Personal AI Core
                       │
             ┌─────────┴─────────┐
             │                   │
     Domain Data Mgmt      Knowledge Governance
             │                   │
       Repository              Advisor
             │                   │
       StorageAdapter      Recommendation
             │                   │
             └─────────┬─────────┘
                       ↓
                 Google Drive
```

`UEF-ADR-DataManagement-Architecture.md`（前一份）讲的是左边；这份从头到尾讲的都是右边。

---

## 插入 UEF_ADR_Log.md

## UEF-ADR-XXX — Knowledge Governance OS: Advisor, not Authority（pending implementation）

*（编号同样接续 `UEF_ADR_Log.md` 下一个可用号码）*

**Status:** Governance decision frozen (Accepted); implementation intentionally deferred. 用户原话「最后才做」——不是待定，是明确排序在 Repository/DataManagement/Archive 基础设施之后。
**Date:** 2026-08-07

**Context:** 本 session 反复出现同一类问题——v1.5 能不能删？哪些文件 coding 前必须读？D8/D9/D10 有没有被后续版本悄悄改写？每次都是手动一份一份文件去核对。这些是同一种能力被反复用手工重做，值得定义成一组正式能力，而不是每次重新发明分析方式。

**Question:** 怎么管理会持续增长的 Universal/治理文件集合（版本、归档、出处、依赖、pre-code 适用性），又不变成（a）自己判断就执行 delete/archive/merge 的黑箱，或（b）纯手工、文件一多就撑不住的流程？

**Options Considered:**
(a) 维持现状——每次有疑问都手动逐档核对 ·
(b) 全自动系统：扫描/diff/判断/直接执行（archive、delete、merge），不经人过目 ·
(c) Advisor-only：Scan/Diff/Dependency/Version/Archive-Candidate-Detection/Governance-Consistency-Check 只产生 Recommendation Record；delete/archive/supersede/merge/更改权威来源这类有后果的动作，一律要人明确批准才执行

**Decision:** (c)。(a) 不 scale；(b) 违反这整个生态一路坚持的原则——TruthEngine 不自己决定、Repository 不越权碰 Archive、human decision boundary 处处都是同一条线，这里没有理由例外。

**Evidence:** 本 session 手动跑了一次真实 pilot（见下方 Recommendation Records），对象是 UEF 自己的历史版本链：
- D8（origin: v1.5）vs 现行 v1.11 — diff 结果：逐字相同
- D9（origin: v1.6）vs 现行 v1.11 — diff 结果：逐字相同
- D10（origin: v1.7）vs 现行 v1.11 — diff 结果：逐字相同

过程中也验证出一件事，直接影响以后 Diff Engine 该怎么写：**第一次用 `sed` 按「从这个标题到下一个类似标题」的方式截取区段，D9 跟 D10 都产生了假的「有变更」信号**——不是内容真的变了，是截取边界没抓准（`sed` 按 regex 猜边界，v1.6/v1.7 当时还没有后续 Decision 可以当停止点，往下抓过头，把不相干的段落也算了进去）。改用真正按 markdown 标题结构截取（遇到下一个 `###` 或 `---` 才停）之后，三个都是干净的「逐字相同」。这代表 Diff Engine 不能只是「文字比对」，必须先做到「准确辨识区段边界」，不然会产生错误的警报，Recommendation 的可信度会被自己拖垮。

**没验证过的**——这整套还没有一行真正在跑的代码，Scan/Diff/Dependency/Version/Archive-Candidate/Consistency-Check 这些 Engine 都只存在于本文件的设计与这次手动跑过的一次 pilot 里。Tier 上是概念阶段，比一般 Tier 3 多了一次真实、可重现的验证案例。

**Impact:** 这条 ADR 定案的是「职责边界」（Advisor not Authority）跟「产出格式」（Recommendation Record），不是马上要有系统在跑。下方 Candidate Patterns 条目让它保持未定案，直到真的写成代码、且有第二个独立场景（不只是 UEF 自己的版本链）也用同样的模式。

**Next Steps:**
1. 目前维持手动——这次的三条 Recommendation（见下）状态是 `Pending`，等你批准才会真的把 v1.5/v1.6/v1.7 移进 `Historical/`。
2. 如果以后要写成真正的代码，第一个实作目标就是这次已经手动验证过的同一个场景（UEF 版本链的 Scan + Diff + Recommend），不用另外找 pilot——这次的三条记录就是验收基准：代码跑出来的结果要跟这次手动验证的结果一致。
3. Diff Engine 必须走结构感知 pipeline，不能是 regex 猜边界——这次的假阳性是第一手教训：

   ```
   Document
     ↓
   Structure Parser
     ↓
   Section Boundary Detection
     ↓
   Normalized Section
     ↓
   Diff
     ↓
   Evidence
     ↓
   Recommendation
   ```

   不是 `Document → Regex → Diff`。Diff correctness ≠ text comparison correctness——这次 v1.6/v1.7 都撞过一次假阳性，原因是当时没有下一个 Decision 可以当停止点，regex 往下抓过头，换成按标题结构截取才干净。
4. 明确排在这个顺序之后，不提前：**① Domain Repository → ② Data Management/Storage → ③ Archive/lifecycle 基础设施 → ④ Data/Knowledge Governance OS → ⑤ AI Advisor。** 这条顺序本身也是这个 OS 自己主张的原则的体现——不要为了未来可能的治理需求，提前把现在的系统做复杂。

**Related ADRs:** 跟 `UEF-ADR-DataManagement-Architecture.md` 是同一条线的延伸——那份处理「一个 Domain OS 自己的资料」，这份处理「整个生态的治理文件本身」，模式相同（Repository/Domain 不碰 Archive 细节 → Data Management OS 不碰业务逻辑 → 这里 → AI 不自己执行有后果的操作 → 人核准），层级不同。

**Review Trigger:** 真的写成代码、且有第二个独立场景采用同样模式時，回来更新 Evidence、推进 Tier 状态。

---

## 插入 UEF 正文的 Candidate Patterns 表

| Candidate | Source | What would confirm it |
|---|---|---|
| **Knowledge Governance OS——Advisor not Authority，管的是治理文件本身，不是 Domain 业务资料（见上方 Domain Data Mgmt / Knowledge Governance 范围图）。** Scan/Diff/Dependency Analysis/Version Analysis/Archive-Candidate-Detection/Governance-Consistency-Check 只产生结构化 Recommendation Record（Type/Target/Reason/Evidence/Risk/Recommendation/Human Decision，ID 前缀 `DMO-` 是 Recommendation 专用命名空间，非 Business Event），delete/archive/supersede/merge/更改权威来源一律需要人明确批准，AI 不自动执行。Diff 必须走 Structure Parser → Section Boundary Detection → Normalized Section → Diff 的结构感知 pipeline，regex 猜边界会产生假阳性。 | 本 session 手动 pilot，2026-08-07——对象是 UEF 自己的 v1.5/v1.6/v1.7，三条 Decision（D8/D9/D10）逐字比对相同；没有一行真正的代码 | 真的写成代码运作过；或第二个独立场景（不是 UEF 自己的版本链）也採用同样的 Advisor-only 模式 |

---

## 本次 Pilot 的三条 Recommendation Record（真实结果，非示范）

**ID 命名空间说明：** `DMO-YYYYMMDD-NNN` 是 Recommendation Record 的 ID 命名空间，不是 Business Event ID，也不是 Execution Log ID——以后 Domain Data Management 如果也有自己真正的业务记录/执行日志，两边的 ID 前缀不会撞在一起造成混淆。

```
Recommendation ID: DMO-20260807-001
Type: ArchiveCandidate
Target: Universal_Engineering_Framework_v1_5.md
Reason: No active dependency detected — D8 (v1.5's own Decision) is
        fully, verbatim represented in the current baseline (v1.11).
Evidence: diff(v1.5 §D8, v1.11 §D8) → identical, 0 lines changed.
Risk: Historical audit trail becomes one step less directly
      accessible (still retrievable from Historical/, not lost).
Recommendation: Archive to Historical/, do not delete.
Human Decision: Pending
```

```
Recommendation ID: DMO-20260807-002
Type: ArchiveCandidate
Target: Universal_Engineering_Framework_v1_6.md
Reason: No active dependency detected — D9 (v1.6's own Decision) is
        fully, verbatim represented in the current baseline (v1.11).
Evidence: diff(v1.6 §D9, v1.11 §D9) → identical, 0 lines changed.
          (First extraction attempt using naive sed-range boundaries
          produced a false "changed" signal — corrected with
          markdown-heading-aware extraction; see ADR Evidence above.)
Risk: same as DMO-20260807-001.
Recommendation: Archive to Historical/, do not delete.
Human Decision: Pending
```

```
Recommendation ID: DMO-20260807-003
Type: ArchiveCandidate
Target: Universal_Engineering_Framework_v1_7.md
Reason: No active dependency detected — D10 (v1.7's own Decision) is
        fully, verbatim represented in the current baseline (v1.11).
Evidence: diff(v1.7 §D10, v1.11 §D10) → identical, 0 lines changed.
          Same false-positive-then-corrected pattern as
          DMO-20260807-002.
Risk: same as DMO-20260807-001.
Recommendation: Archive to Historical/, do not delete.
Human Decision: Pending
```

**两种 Recommendation Type 的区别（原则先定，完整 Type System 等真的开始写 Engine 再展开）：**
- `ArchiveCandidate` = historical content remains represented in the active baseline（v1.5/v1.6/v1.7 属于这种——内容原封不动带到了 v1.11）
- `SupersededWithRationale` = historical content was explicitly replaced by a later decision, with the replacement rationale preserved（v1.10 属于这种——v1.11 明确取代了它，取代的理由保留在 v1.11 的 Status 里，不是内容原封不动带过去）

v1.8/v1.9/v1.10 没有包含在这次 pilot 的三条 Recommendation 里，原因就是它们该标 `SupersededWithRationale`，不是 `ArchiveCandidate`，这次没有展开第二种类型的完整判断逻辑，留给以后真的要做的时候再补。

三条都是 `Pending`——本文件跟这次 session 都没有移动或删除任何文件。要 Approve 哪一条，还是要全部先留着，等你决定。
