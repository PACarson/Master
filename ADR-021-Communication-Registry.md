# ADR-021：Communication Registry（通讯登记系统）

**Status**: Proposed — 内部一致，但尚未对照 Personal AI Core / Rider OS / Reminder OS 现有的 EventBus-equivalent 与 TruthEngine/Execution 代码验证（见 Session-Handoff-Brief.md）
**Date**: 2026-07-29
**适用范围**：所有 Domain OS 之间、Domain OS 与 Execution OS / Decision OS / Personal AI Core 之间的跨系统通讯

> 本文件是 Communication Architecture 系列 ADR 之一（共 3 份）：
> - ADR-019 — Communication Protocol（`ADR-019-Communication-Protocol.md`）
> - ADR-020 — Domain Communication Rules（`ADR-020-Domain-Communication-Rules.md`）
> - **ADR-021 — Communication Registry**（本文件）
>
> 共同背景、Envelope 定义、Freeze Boundary 总表、Rollout Phases 与设计演进 Appendix 都在 ADR-019；本文件聚焦 Event/Command/Service 三个 Registry 本身该怎么设计。冻结的是 **Communication Contract**，不是 **Implementation**。

## Context

Fact 和 Command 的具体定义（哪些 eventType/commandType 存在、谁能发布/谁能订阅、payload 长什么样、目前是不是还该被使用）需要一个所有人（包括 AI）都能查询的单一真实来源，同时这份资料本身也要能被 AI 当作 Context Builder 的资料源，不能是一份只有人看得懂、AI 每次都要整份塞进 Prompt 的文件。最初曾考虑做成 `.js` 文件，后来确定改用 Google Sheet。

## Decision

**Communication Registry** 作为总称，底下有三个子 Registry，全部实现为 Google Sheet 的 tab（不是代码文件）：

```
Communication Registry
├── Event Registry     — 每个 eventType 一行
├── Command Registry   — 每个 commandType 一行
└── Service Registry   — 每个 Domain 一行
```

**Event Registry** 栏位：`eventType │ version │ description │ publisher │ payload_schema │ stability │ replacedBy`

**Command Registry** 栏位：`commandType │ version │ owner (目标 Domain) │ required_permission │ payload_schema │ stability │ replacedBy`
（`required_permission` 对应 autonomy level，例如 `AI.AutoExecute` / `Human.ConfirmRequired`——读多写少、出错代价低的 Domain 命令可标记自动执行；涉及真实资产、法律文件的命令标记为需要人工确认。）

**Service Registry** 栏位：`domain │ baseUrl │ serviceVersion │ status │ capabilities`
- `capabilities` **不手动维护**，用 Sheet 公式（如 `QUERY`）从 Command Registry 依 owner 即时过滤生成，避免两份资料各自维护、彼此漂移。

**Version vs Stability**（两个不同的轴）
- `version`：这个 eventType/commandType 的 payload 契约版本号，改版要新增一个 version，不是直接编辑既有版本的内容。一旦某个 version 被标记 Stable，那个版本的 schema 本身也视为冻结，真正要改内容就开新版本。
- `stability`：这个契约现在还该不该被使用（Stable / Beta / Deprecated）。搭配 `replacedBy` 指出被什么取代，Connector/AI 遇到 Deprecated 目标时，正常执行、只写一条 audit warning，不硬拦（同 ADR-020 第 7 条的精神）。

**Service Version vs Schema Version**（另一组容易混的概念）
- Service Version：这个 Domain 整体部署的第几代（通常整包重写才会变动）。
- Schema Version：某个 eventType/commandType 的 payload 契约版本（前述 `version` 栏位）。两者变动频率、变动原因完全不同，不应共用同一个版号。

Service Registry 的 `status`（Active/Inactive）也跟 Event/Command Registry 的 `stability`（Stable/Beta/Deprecated）是两个不同的轴：前者回答"这个部署现在活着吗"，后者回答"这份契约还该不该被用"。

## Alternatives Considered

**A. Registry 做成 `EventRegistry.js` / `CommandRegistry.js` 代码文件**
被否决。每次改动都要在每个引用它的项目里重新部署、更新版本号，这层部署摩擦对现阶段不值得。改用 Sheet，改动就是编辑一行，不需要重新部署。

**B. Service Registry 的 `capabilities` 手动维护一列**
被否决。新增一个 commandType 时，Command Registry 加一行是必须动作，但如果忘了同步更新 capabilities 列表，两边就会不一致，而且没人会立刻发现——这跟更早发现的 Personal AI Core / Execution Layer 职责重复是同一类问题：两个地方存同一件事，迟早飘掉。改用公式从 Command Registry 即时推导，只有一份真正的数据来源。

**C. URL 直接存在 Command Registry 的每一行里**
被否决，改为独立的 Service Registry（一个 Domain 一行）。如果 URL 挂在 Command Registry 上，同一个 Domain 的多个 command 会重复写同一个 URL，Domain 重新部署、URL 改变时每一行都要改，容易漏改。独立成 Service Registry 后，一个 Domain 的 URL 只存在一个地方，改一次全部生效。

## Rationale

三个子 Registry 分开、但统称 Communication Registry，让"Fact 长什么样""Command 长什么样""去哪里找 Domain"三个问题互相独立又能一起被查询——这也直接对应"AI 不用每次读整份 Blueprint，只查 Registry 里跟当前事件/命令相关的那一行"的省 Token 设计。

## Consequences

- ✅ 新增 Command 只需要在 Command Registry 加一行，不需要改任何代码或重新部署。
- ✅ Capabilities 永远与 Command Registry 同步，不会有第二份手动维护的数据源飘掉。
- ⚠️ Sheet 公式（QUERY 等）本身的读取延迟、以及 Registry Sheet 变大以后的查询效能，未在讨论中验证过。

## Deferred Decisions

- 共享密钥应存放的位置（见 ADR-019 Deferred Decisions）。
- Registry Sheet 本身的存取权限（谁可以编辑、AI 是否有写入权限还是只读）未讨论。

## Frozen（本 ADR 冻结的内容）
- Communication Registry 作为总称，下辖 Event/Command/Service 三个子 Registry
- 三者皆为 Google Sheet，不用代码文件
- Version 与 Stability 是两个不同的轴；Service Version 与 Schema Version 是两个不同的轴
- capabilities 由公式推导，不手动维护
- Deprecated + replacedBy 机制

---

## 相关 Conflicts（完整列表见 ADR-019；以下为与本 ADR 直接相关者）

**capabilities 手动 → 推导**：Service Registry 的 `capabilities` 栏位最初被提议手动维护，随后被指出与 Command Registry 有重复维护、容易漂移的风险，与更早发现的 Personal AI Core / Execution Layer 职责重复是同一类问题。**最终版本：capabilities 由 Sheet 公式从 Command Registry 推导，见上方 Alternatives B。**

**URL 归属的正规化**：URL 最初被设想为存在 Command Registry 的每一行里，后被拆分为独立的 Service Registry。两者不冲突，后者是对前者的正规化改进。**最终版本以 Service Registry 为准，见上方 Alternatives C。**

## 相关 Open Questions

- **Registry Sheet 本身的存取权限规划**（谁能编辑、AI 是否有写入权限）未讨论。
- **Sheet 公式（QUERY 等）驱动 capabilities，在 Registry 数据量变大后的查询延迟**，未经实际验证，留待 Phase 1/2 观察。

---

*完整的 Freeze Boundary 总表、Rollout / Validation Phases 与设计演进 Appendix，请见 ADR-019。本文件记录至 2026-07-29；后续如需修改 Frozen 区块任何内容，请开新的 ADR 并注明取代关系，不要直接编辑本文件的 Frozen 部分。*
