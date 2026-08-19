# Personal Digital OS — 文件索引

INDEX 负责定位（orientation）——第一次进来，知道这个世界怎么组成、每样东西该去哪找。要执行（写代码前该读什么），去 `00-Pre-Code-Documentation-Index.md`，不在这里找细节。每一项只回答「是什么」+「去哪看」，其余留给各自文件。

## Universal Engineering Framework

生态级工程治理——lifecycle、Engineering Principles、AI Engineering Protocol（含 export 纪律、File/Engine/Sprint checkpoint 层级）、编码规则、版本/ADR 治理。**只认当前最新版**：`Universal_Engineering_Framework_v1_12.md`。v1.5–v1.11 是历史版本，见下方 Historical / Archive。

## Recovery / Continuity

跨 project/OS 或跨 AI session 的 checkpoint 记录 → `Universal-Recovery-Manifest.md`（`Universal_Engineering_Framework_v1_12.md` §0.6 item 4 要求的 ecosystem-level artifact）。单一 project 内自己的 file/Engine/Sprint checkpoint 记在该 project 自己的 `00_Project_State.js`，不在这里查。

## Domain OS

生态里有哪些 Domain OS、各自负责什么、目前进度 → `OS-Directory-for-Personal-AI-Core.md`。跨 Domain 该怎么通讯（EventBus/CommandBus/Registry）→ `ADR-019/020/021`，**设计中，尚未对照真实代码验证**，当前状态与下一步见 `Session-Handoff-Brief.md`。

## Core Infrastructure

Domain OS「能包含什么」的架构蓝图，Tier 1/2/3 分级 → `Universal_Domain_OS_Blueprint_v1.2.md`。写代码前必读的文件总表 + 判断流程 + 文档审计报告 → `00-Pre-Code-Documentation-Index.md`。

## Data Management

单一 Domain 自己的资料怎么存取——Repository（业务查询）/ DataManagement（归档、索引、备份）/ StorageAdapter（Sheets/Drive）→ `UEF-ADR-DataManagement-Architecture.md`。目前只在 Rider OS 设计验证过，代码还没写。

## Knowledge Governance

整个生态治理文件本身怎么管——版本、归档、出处、依赖。Advisor only，delete/archive/merge 一律要人批准 → `UEF-ADR-KnowledgeGovernanceOS.md`。设计阶段，implementation 排最后。

## Historical / Archive

`Universal_Engineering_Framework_v1_5.md` ~ `v1_11.md`——v1.12 的 provenance，只在要查某条规则当初怎么来的、有没有被后续版本改写时才打开，不当现行规则依据。`Communication-Architecture-ADR-019-021.md` 已过时，被拆分后的 `ADR-019/020/021` 取代，不要读。

---

细节、开放问题、完整背景故事都在各自文件里，不在这份 INDEX 重复——尤其是 `Session-Handoff-Brief.md`，Communication Architecture 那条线现在的状态与已知问题都以它为准。
