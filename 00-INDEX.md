# Personal Digital OS — 文件索引

## 先看这个：状态在 2026-07-31 有一次重大修正

这个目录里 ADR-019/020/021 与 Phase 1 代码，是在没有对照任何真实项目代码的情况下设计出来的。同一天确认了真实存在的 Universal Engineering Framework v1.5 / Universal Domain OS Blueprint v1.2（另一个更严谨、以真实代码为依据的过程产出），发现 Blueprint 里 Event、Execution 早就是 Tier 1（三个真实项目验证过），这个目录的设计完全没有对照过它们。

**补充背景（用户确认）**：UEF / Blueprint 这两份文件本身是真实、权威的，但因为很多 OS 项目还在持续增强、写代码，这两份治理文件也还没来得及完全跟上——不是两边一个对一个错，是两边都在动。这代表真的去对照 Rider OS / Personal AI Core / Reminder OS 源码的时候，UEF 很可能已经不是 v1.5、Blueprint 也很可能已经不是 v1.2 了。核对时找当下最新版本，不要假设这次上传的版本号届时仍然是最新的。

**开新窗口时先读 `Session-Handoff-Brief.md`**，第一件事是去读 Personal AI Core / Rider OS / Reminder OS 实际的 EventBus-equivalent 与 TruthEngine 代码，再决定这里的设计该继续、该让位、还是该合并。这次上传的两份真实文件原始版本也复制进了这个目录（`Universal_Engineering_Framework_v1_5.md`、`Universal_Domain_OS_Blueprint_v1_2.md`），不用回头找上传记录——但要记得上面那条：核对时先确认它们是不是还是当下最新版本。

这是这份索引，帮助任何人（或任何 AI）第一次进来时知道该先看哪份、每份在讲什么。

## 建议阅读顺序

0. **Session-Handoff-Brief.md** — 新窗口从这份开始，讲清楚现在的状态和第一步该做什么。
0.5. **`Universal_Engineering_Framework_v1_5.md`、`Universal_Domain_OS_Blueprint_v1_2.md`** — 真实、权威的原始文件，本目录其余内容都是相对这两份来定位的，读之前的判断都以这两份实际内容为准，不要以这份索引的转述为准。
1. **OS-Directory-for-Personal-AI-Core.md** — 了解生态里有哪些 OS、各自负责什么、目前进度到哪、彼此关系。
2. **ADR-019-Communication-Protocol.md** — Communication Architecture 系列的入口，包含整体背景、Envelope 定义、Freeze Boundary 总表、Rollout Phases、设计演进 Appendix。
3. **ADR-020-Domain-Communication-Rules.md** — Dual Bus、Domain Isolation、Connector 职责边界。
4. **ADR-021-Communication-Registry.md** — Event / Command / Service 三个 Registry 的设计。
5. **Engineering-Principles.md** — 通用工程原则（先验证再扩展、抽象先合并、AI 自主权分级），不限于 Communication Architecture。
6. **Token-Efficiency-Principles.md** — AI 调用的省 Token 具体做法，包含尚未落进任何 ADR 的 Prompt Caching 建议。
7. **Apps-Script-Platform-Constraints.md** — 支撑以上所有决定的 Apps Script 平台限制查证结果，技术参考用。
8. **Phase1-Setup-Guide.md** + `01`–`05` 五个 `.js` 文件 — Phase 1 的实际落地代码与操作步骤（Fact Flow: Rider→Finance；Command Flow: Execution→Reminder）。**注意：目前违反 UCR1/2/3/6 与 D8，未经真实代码对照，见下方未决事项。**
9. **Universal-Domain-Blueprint.md** — 在 ADR-019/020/021 之上，"Domain 内部该长什么样"的第一版草稿（Entry/Service/Data Layer 切法、七个固定文件名、新增 Domain 清单）。同样未对照真实代码，且文末原本误写"UEF 尚未成文"——已确认 UEF v1.5 真实存在，该处已修正。
10. **UEF-ADR-Communication-Architecture.md** — 准备好可以贴进真实 `UEF_ADR_Log.md` 与 UEF 正文 Candidate Patterns 表的内容，Evidence 栏位已诚实标注验证与未验证部分。

## 目前状态

Communication Architecture 的格式已经设计完（见 ADR-019 Freeze Boundary），Phase 1 代码已写出，**但尚未确认实际跑通，也尚未对照真实的 UEF/Blueprint 代码库**。Phase 2（其他 Domain 接入既有 Contract）还没开始动手；已有初步顺序建议（见 ADR-019 Phase 2、OS Directory 各 Domain 的"后续建议"），但这些建议同样建立在未经对照的设计之上。

## 已知的未决事项

- **最优先**：Personal AI Core / Rider OS / Reminder OS 实际的 EventBus-equivalent 与 TruthEngine/Execution 代码，还没有读过、还没有跟这里的设计对照
- Phase 1 代码仍违反 UCR6（裸 `appendRow()`）、UCR1（无 IIFE）、UCR2（前缀应为 `_` 开头）、UCR3（无 AlertService.log）——这几条需要先看到真实的 TruthEngine/AlertService 代码长什么样才能正确改，不是单纯改名就能解决，先不动。D8（文件后缀）已经是唯一一条纯格式、不涉及逻辑的项目，已改成 `.js`
- Execution OS、Decision OS、Personal AI Core 彼此之间是否也是独立 Apps Script Project（影响 ConnectorLib 该怎么部署）
- 共享密钥该放在哪里更安全——目前建议存 Script Properties、不进 Registry Sheet，已落实在 Phase 1 代码里，但这个决定本身没有正式讨论定案
- Registry Sheet 本身的存取权限规划
- Sheet 公式（QUERY 等）驱动 capabilities，在 Registry 数据量变大后的查询延迟，未经实际验证

## 另一条独立线：Domain Data Layer（Repository + DataManagement）

跟上面 0–10 的 Communication Architecture 是不同 session、不同关切——那边处理 Domain 之间怎么通讯，这里处理「一个 Domain 自己内部怎么存取资料」（Blueprint 现有 Data Layer 节点之下一层：Repository 业务查询、DataManagement 共用归档/索引/备份、StorageAdapter 存取 Sheets/Drive）。全程对照过 Rider OS 实际的 Constitution 与 `.xlsx` 资料，但 Repository/DataManagement 代码本身还没写。内容见 `UEF-ADR-DataManagement-Architecture.md`，格式跟 `UEF-ADR-Communication-Architecture.md` 一样——两段分别待插入 `UEF_ADR_Log.md` 与 Candidate Patterns 表。`domain-os-roadmap.md` 的资料量触发条件表也补了这次从 `.xlsx` 实测出的增速数字。

## 还没写的部分（范围提醒，不是遗漏）

最早的构想里，Universal Engineering Framework 还包含 Governance、Blueprint Change Policy、AI 修改规则；Universal Domain Blueprint 还包含 Folder Structure、Module Boundary、Data Ownership 等。这个目录只处理了 Communication Architecture 这一层，而且这一层本身还没跟真实的 UEF v1.5 / Blueprint v1.2 对齐——这是刻意分阶段，不代表已经决定不需要，只是还没轮到，而且顺序可能因为这次的发现而调整。
