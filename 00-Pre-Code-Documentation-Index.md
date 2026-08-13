# 00 — Pre-Code Documentation Index

任何 coding task 开始前，第一步是读这份文件，不是凭记忆猜「大概要看哪些文档」。这份文件本身不判断该不该改任何既有文档——纯粹是盘点 + 分类 + 一套「写代码前先查这里」的决策流程。不删除、不改名、不合并任何现有文件。

---

## 一、文档总表

| # | 文档 | 用途 | 类别 | 适用范围 | 何时必读 | 何时可跳过 | 权威性 / 现行状态 | 相对优先级 |
|---|---|---|---|---|---|---|---|---|
| 1 | `Universal_Engineering_Framework_v1_11.md` | 生态级工程治理——lifecycle、Engineering Principles (EP1-6)、AI Engineering Protocol (§0.6，含 per-file export 规则)、UCR 编码规则、ADR/版本治理、Candidate Patterns | Engineering Governance | ALL code | 任何 coding task，无例外 | 从不可跳过 | **权威，现行**（最新版） | 1（最高，跟其他文档冲突时以此为准） |
| 2 | `Universal_Engineering_Framework_v1_5~v1_10.md`（历史链） | D8/D9/D10 等既有 Decision 的出生纪录与 provenance | Engineering Governance（历史） | 不直接适用 coding | 要查证某条 Decision 有没有被后续版本悄悄改写意思时（§0.8 Fidelity check） | 其余情况一律跳过 | **非权威，历史** | 不覆盖 v1.11 |
| 3 | `Universal_Domain_OS_Blueprint_v1.2.md` | Domain OS「能包含什么」——Tier 1/2/3 分级追踪哪些架构能力已被几个真实专案验证；明确只讲 What 不讲 How（BP-6） | Architecture | Architecture、Domain-specific | 设计/新增任何 Domain 能力、建新 Domain OS 之前 | 纯 bug fix、已有 pattern 内的小改动 | **权威，现行** | 2 |
| 4 | `Universal-Domain-Blueprint.md` | 较早、非正式的草稿，同样讲 Entry/Service/Data Layer 切法——**跟 #3 不是同一份文件，名字很像** | Architecture | Architecture | v1.2 Blueprint 没覆盖到的具体结构问题 | v1.2 已有答案时 | 参考性，跟 #3 有重叠、权威关系未定案（见下方 Audit） | 低于 #3，冲突时不能当仲裁依据 |
| 5-7 | `ADR-019-Communication-Protocol.md` / `ADR-020-Domain-Communication-Rules.md` / `ADR-021-Communication-Registry.md` | 跨 Domain 通讯协议——EventBus/CommandBus 双总线、Domain 间规则、Event/Command/Service Registry | Architecture / Communication | Domain-specific（跨 Domain 整合）、API | 新增或修改任何跨 Domain 的 Fact 发布、Command 派发、或 Registry 本身之前 | 完全在单一 Domain 内部、不涉及跨 Domain 通讯的工作 | **Status: Proposed——尚未对照 Personal AI Core / Rider OS / Reminder OS 真实代码验证**（见 Session-Handoff-Brief.md），当方向参考，不当已定案的硬约束 | 有条件的高（仅限适用范围内），每次引用都要附带「未验证」提醒 |
| 8 | `Communication-Architecture-ADR-019-021.md` | 拆分前的合并版 | — | — | **不要读** | 一律跳过 | **已过时，文件自己声明作废**，改读 #5-7 | 无 |
| 9 | `UEF-ADR-Communication-Architecture.md` | 待并入 `UEF_ADR_Log.md` 与 Candidate Patterns 表的 staging 内容，对应 #5-7 | Engineering Governance（staging） | Other（治理本身） | 处理治理文件合并工作时 | 一般 coding task | Staging，尚未真正并入任何现行文件 | 不适用于 coding task |
| 10 | `UEF-ADR-DataManagement-Architecture.md` | 待并入的 Repository + DataManagement + StorageAdapter Data Layer pattern（本 session 产出） | Architecture（Data Layer） | Data-persistence、Domain-specific（目前只有 Rider OS） | 建立/修改任何 Domain 的 Data Layer（Repository、DataManagement 模块、Archive）之前，尤其 Rider OS | 不涉及资料存取层的工作 | **Candidate Pattern**——对照过真实 Rider OS 代码与资料，但代码本身还没写，不是已验证的 Blueprint Tier 1/2 | 适用范围内高，但每次引用要附带「未实作」提醒 |
| 11 | `Engineering-Principles.md` | 通用原则——Walking Skeleton（先验证再扩展）、抽象先合并、AI 自主权按风险分级、Registry 校验警告优先——**文件自己说这是 UEF 的前身草稿，还没正式并入** | Engineering Governance（与 UEF EP1-6 重叠） | ALL code（若正式并入） | UEF 现有 EP1-6 没覆盖到的具体判断时参考 | UEF EP1-6 已经够用时 | 参考性，跟 UEF EP1-6 重叠、尚未整合（见 Audit） | 低于 UEF；跟 UEF 冲突时 UEF 赢 |
| 12 | `Token-Efficiency-Principles.md` | AI/token 用量控制原则——EventBus 日常流转不经过 AI、Registry 当查表工具、Prompt Caching、AI 呼叫尽量批量 | AI Engineering | AI | 建立任何重复呼叫 AI API 的功能、或设计 Personal AI Core 编排逻辑时 | 完全不涉及 AI API 呼叫的工作 | 参考性，提出过但还没落进任何 ADR | 中，仅限 AI 呼叫相关范围 |
| 13 | `Apps-Script-Platform-Constraints.md` | 已查证的 GAS/Sheets 平台限制——执行时间、触发器配额、Library 呼叫成本、Web App 路由/status code 限制，是多个 ADR 决定背后的技术依据 | Platform | GAS / Google 平台 | 任何涉及 Apps Script 触发器、配额、Library、或 Web App 端点的工作 | 纯业务逻辑、零 GAS 平台顾虑的工作 | **权威，已查证**（本 session 另外独立搜索过一次同样几个数字，结果一致，见 #10 的 Evidence） | GAS 平台范围内高 |
| 14 | `OS-Directory-for-Personal-AI-Core.md` | 每个 Domain OS 一段的路由参考 + 目前完成度 | Domain 路由 / 参考 | Domain-specific（路由判断）、Other | 判断一个新需求属于哪个 Domain、或要确认某 Domain 实际现状时 | 已经明确知道要改哪个 Domain 时 | 参考性，需要定期跟真实状态核对，不会自动同步 | 中 |
| 15 | `Session-Handoff-Brief.md` | Communication Architecture 那条线目前进度、新窗口第一件该做的事 | Session 交接，非编码规则本身 | Other | 专门要接续 Communication Architecture 对照真实代码那件事时 | 其他任何任务 | 时间点快照，会过期，用前先核对日期 | 仅限该任务 |
| 16 | `domain-os-roadmap.md` | GAS 内的 UI 现代化路径（Part A）+ 什么条件下才该考虑整个迁出 GAS（Part B/C） | Platform / 基础设施规划 | GAS 平台、Other | 评估要不要把某个 Domain 迁出 GAS、或做 UI 层决定时 | 纯后端/资料层工作 | 规划性，现行（2026-08-05 更新，本 session 的实测增速数字进一步确认 Part B 门槛还很远） | 中 |
| 17 | `00-INDEX.md` | 整个仓库的地图与阅读顺序 | Meta / 导航 | Other | 第一次进这个仓库、或任何新文档加入之后 | — | **权威（关于「有什么、在哪里」），现行**——本文件应该被加进它 | 导航层 |

---

## 二、写代码前的决策流程

**Step 1 — 判断任务类型。** 这是新 Engine？改既有逻辑？跨 Domain 整合？UI？纯 GAS 平台问题？

**Step 2 — 对照上表「适用范围」栏位，圈出所有适用类别。** 不要只看文件名像不像相关——`Apps-Script-Platform-Constraints.md` 名字看起来只跟 GAS 有关，但任何用到 Trigger 或 Library 的工作都该读。

**Step 3 — 载入该类别下所有「何时必读」命中的文档。** 不是「大概看一下」，是真的读完。

**Step 4 — 核对每份权威文档目前的版本。** UEF 现在是 v1.11，不是任何一份历史版本；Blueprint 现在是 v1.2。用错版本等于没读。

**Step 5 — 找出适用的规则与限制，包含上表标注「未验证」「Candidate Pattern」「staging」的项目——这些不是不用管，是要在开始写之前明确告诉使用者「这部分还没定案」。**

**Step 6 — 确认没有冲突、没有模糊地带之后，才开始实作。**

---

## 三、执行规则

- 不能只靠记住之前对话讲过什么来判断工程规则——每次都要真的查这份 Index 跟对应文档，不是凭印象。
- 不能因为文件名看起来跟当前任务无关，就假设它不相关（见 Step 2 的例子）。
- 规则要用某份权威文档「现在」的版本，不是历史版本（例如 UEF 用 v1.11，不是任何 v1.5-v1.10）。
- 历史版本只用于 provenance / audit，绝不当作现行实作的依据。
- 两份权威文档的规则如果看起来互相冲突，先停下来把冲突摊开讲清楚，不要自己选一个开始写。
- 不确定某份文档适不适用，先把这个不确定性讲出来，不要用猜的。

---

## 四、审计报告

**重复 / 重叠的规则**
- `Engineering-Principles.md` 的「先验证，再扩展（Walking Skeleton）」跟 UEF 自己的 EP3（反过早工程化）内容高度重叠，两者目前没有互相引用或整合。
- `Universal-Domain-Blueprint.md`（草稿）跟 `Universal_Domain_OS_Blueprint_v1.2.md`（权威版）都在讲「Domain 内部长什么样」，文件名又很像，容易读错本。

**已过时的文档**
- `Communication-Architecture-ADR-019-021.md`——文件自己声明作废，Status 栏位不可信，内容已被拆分的 ADR-019/020/021 取代。

**建议整合的文档**
- `Engineering-Principles.md` 的四条原则应该正式并入 UEF 的 EP 清单或作为附录标注对应关系，而不是维持一份孤立、未被引用的旁支文件。
- `Token-Efficiency-Principles.md` 目前提出的原则「还没落进任何 ADR」（文件自己承认），应该在 ADR-019/020/021 完成代码对照验证时一并正式收编，不要继续游离在外。

**权威性目前不清楚的文档**
- `ADR-019/020/021`——Status 明写 Proposed、未对照真实代码，这三份现在能不能当「必须遵守」还是只能当「设计方向参考」，没有明确定论，本表暂定为「有条件的高优先级」，建议这次代码对照完成后回来更新这一格。
- `UEF-ADR-Communication-Architecture.md`、`UEF-ADR-DataManagement-Architecture.md`——两份都是 staging 文件，什么时候、由谁、按什么标准正式并入 `UEF_ADR_Log.md`，目前没有写清楚的流程，只知道「等条件满足」。
- `Universal-Domain-Blueprint.md` 相对 `Universal_Domain_OS_Blueprint_v1.2.md` 到底是「已被取代」还是「更细节的补充」，没有明文说清楚。

**文档间的隐藏依赖（顺手发现，跟本次审计目标相关）**
- `Apps-Script-Platform-Constraints.md` 记录的 Library 呼叫成本（呼叫方与被呼叫方共用同一份执行时间预算）——跟 `UEF-ADR-DataManagement-Architecture.md` 提议「DataManagement 抽成共用 GAS Library」直接相关，但两份文件目前没有互相引用。等 DataManagement 真的要抽成 Library 时，这一条会是要重新核对的地方，不是新问题，只是现在两份文件还没连起来。

---

## 五、结构建议（尚未执行）

以后文档变多，`00-INDEX.md` 值得照下面这样分区导航，而不是维持一份扁平清单——这只是建议的分类视角，不代表文件本身要被搬动、改名：

```
00-INDEX.md
├── 00-Pre-Code-Documentation-Index.md（本文件）
├── Universal Engineering — UEF、Blueprint、ADR
├── Implementation — Library/Engine/API 规则
├── Platform — GAS、Google Sheets
├── Quality — Testing、Validation
└── Domain — 各 Domain OS 自己的文档
```

新增一份 Universal File 时，第一件事是把它加进本文件的总表，而不是指望以后凭记忆想起来它存在。
