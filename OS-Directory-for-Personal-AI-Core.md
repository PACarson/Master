# OS 名录 — Personal AI Core Knowledge Base 条目

供 Personal AI Core 查询"某个请求该找哪个 Domain"用。判断路由或需要了解某个 Domain 大致职责时，先查这份名录，不必读整份 Universal Blueprint 或 Communication Registry。

---

## Universal Persistence / Checkpoint Governance — 采用状态

`Universal_Engineering_Framework_v1_12.md` §0.6 item 3-4（Atomic Persistence / Checkpoint Rule）新增于 2026-08-16。下表区分四种状态，不把「规则存在」直接等同「已采用」——**Adopted 是 UEF 的 governing-authority 关系（D1）自动成立的**，不需要该 OS 自己的文件才生效；**Verified 则需要真的打开该 OS 自己的治理文件核对**。本 session 上传的 `Master-main.zip` 只包含 Universal/UEF 层，不含任何单一 OS 自己的 repo，所以下表目前没有一格是 Verified——这不是遗漏，是如实反映本 session 能核对到的范围。

| OS | Rule Defined | Adopted (inherited) | Verified | Checkpoint System Active |
|---|---|---|---|---|
| Personal AI Core | ✅ | ✅ → **Locally Adopted** | ✅ **Confirmed**（见下方详细状态） | ⏳ Pending |
| Rider OS | ✅ | ✅ → **Locally Adopted** | ✅ **Confirmed**（见下方详细状态） | ⏳ Pending |
| Finance OS | ✅ | ✅ | ⏳ Pending | ⏳ Pending |
| Investment OS | ✅ | ✅ → **Locally Adopted** | ✅ **Confirmed**（见下方详细状态） | ⏳ Pending |
| Property OS | ✅ | ✅ | ⏳ Pending | ⏳ Pending |
| Reminder OS | ✅ | ✅ → **Locally Adopted** | ✅ **Confirmed**（见下方详细状态） | ⏳ Pending |
| Personal Life OS | ✅ | ✅ → **Locally Adopted** | ✅ **Confirmed**（见下方详细状态） | ⏳ Pending |
| News OS | ✅ | ✅ → **Locally Adopted** | ✅ **Confirmed**（见下方详细状态） | ⏳ Pending |
| Inventory & Procurement OS | ✅ | ✅ | ⏳ Pending | ⏳ Pending |
| Execution OS | ✅ | ✅ | ⏳ Pending | ⏳ Pending |
| Content / Video OS | ✅ | N/A（尚未设计架构） | N/A | N/A |
| Research / Knowledge OS | ✅ | N/A（尚未设计架构） | N/A | N/A |
| Decision OS | ✅ | N/A（尚未开工） | N/A | N/A |
| Health OS | ✅ | N/A（仅构想阶段） | N/A | N/A |

**Personal AI Core — 详细状态（2026-08-17 完成本地采纳后更新）**

上一版这里曾写「Personal AI Core 连自己最初 adopted UEF 的 ADR log 都没有」——这是错的，予以订正：Personal AI Core 其实有一份内容扎实的 ADR 记录（ADL-001~005），文件叫 `00_Architecture_Decision_Log.js`，只是不叫 UEF §0.2 表格期待的通用命名 `00_ADR_Log.js`。这只是命名差异，不重命名这个文件，也不因此新增任何治理要求。

- Universal Defined：✅（`Universal_Engineering_Framework_v1_12.md` §0.6 item 3-4）
- Inherited by Governance Authority：✅（D1，自动继承，从规则存在那一刻起就成立）
- Locally Adopted：✅——`00_Architecture_Decision_Log.js` 新增 ADL-005（决策记录，未解锁 P8，未改写 P6.1），Constitution「四、Coding Rules」新增 C13，两者均已完成
- File-level Verification：✅，但**仅限本次任务实际改过的三个文件**（`00_Architecture_Decision_Log.js`／`00_Project_Constitution.js`／`00_Project_State.js`，各自都走完 persist + 独立核验 + 记录 checkpoint）——不代表 Personal AI Core 全部文件都已核对过，也不代表其它未改动的文件有任何问题
- Overall Personal AI Core Governance Verification：✅ **Confirmed** — Phase C Final Acceptance 已于 2026-08-17 完成（read-only 逐项核验全数通过，无 blocking discrepancy，见 `Universal-Recovery-Manifest.md` 对应 checkpoint）。以上四点依然分开记录，不合并成一个笼统的"已 Verified"——这一条是最后到位的，不是从一开始就跟其它三点同时成立

**已知缺口（本次任务发现，不在此次处理范围内，仅记录，不擅自决定）**
- registry 命名跟 `Universal_Engineering_Framework_v1_12.md` 自己的 Scope 行、以及这次任务需求里的清单，原本三边对不齐；其中 "Investment OS 是否等同于 Finance OS" 这一条现已解开——实际审计 Investment OS 真实 repo 后确认两者是完全不同的独立项目（Investment OS 是投资组合/估值/股息追踪系统，有自己的 Constitution／ADR-1／30 个引擎文件；Finance OS 是订阅 Rider OS 支出事件的自动记账系统），Investment OS 现已单独成行，不再跟 Finance OS 混为一谈。仍未解开的：Scope 行提到的 Productivity OS（当现有项目）、Shopping/Vehicle OS（当未来项目）这份名录都没有；"Compliance OS" 整个仓库只在 `domain-os-roadmap.md` 的举例文字出现过，没有正式条目；需求里的 "Procurement OS"／"Inventory OS" 在这份名录里是合并的同一个 Inventory & Procurement OS
- Rider OS 的 `00_Project_Constitution.js` 头部"Governed by: UEF v1.1"这一行从未随后续同步更新过（本地仓库另外留有 v1.2/v1.3 快照）——ADR-009 里记录了这个发现，没有修复
- Rider OS 自己的 `UEF_ADR_Log.md`（ecosystem-level ADR，含 UEF-ADR-001）在这次任务实际处理的 Master repo 快照里没有找到对应/延续的文件——是命名/结构已经变动，还是这份 ecosystem-level 记录本身失落了，待确认

**Rider OS — 详细状态（2026-08-17）**

Universal Defined ✅ ／ Inherited by Governance Authority ✅ ／ Locally Adopted ✅（`00_ADR_Log.js` 新增 ADR-009，`00_Project_Constitution.js` 新增 CR14）／ File-level Verification ✅（三个改过的文件——ADR Log／Constitution／Project State——都做了 diff+md5+独立 view，另外对整个仓库做了一次 `diff -rq`，确认只有这三个文件不同，其余 30+ 个应用代码/配置/历史文档文件逐一 identical）。

跟 Personal AI Core 不同的地方：这一轮 audit（Phase A）与实作（Phase B/C-B 等价物）是在同一个回合里连续完成的，没有走 Personal AI Core 那种逐文件、逐步等你批准的节奏——这是因为你已经明确授权把整套流程当 proven pattern 直接套用，加上这一轮你提到马上要开新窗口。文件级的核验强度跟 Personal AI Core 是同一个标准（并不因为节奏压缩就降低），但没有经过你对每一步的实时确认，所以这里如实标成"完成，但未经你逐项审阅"，跟 Personal AI Core 那种经过你多轮明确批准的 Verified 不完全同一性质。

**News OS — 详细状态（2026-08-19）**

Universal Defined ✅／Inherited by Governance Authority ✅／Locally Adopted ✅（`00_Project_Constitution.txt` 新增独立 §9「Engineering / AI-Assisted Development Governance」，`00_Project_State.txt` §1 新增 Governance 条目）／File-level Verification ✅，**仅限本次实际改过的两个文件**（`00_Project_Constitution.txt`／`00_Project_State.txt`，各自都走完 persist + 独立核验 + 记录 checkpoint）／Overall News OS Governance Verification ✅ **Confirmed**——本轮 audit（OS-N Read-Only Audit）与实作是你预先明确授权的单轮完成，核验强度与 Rider OS 同一标准。

跟 Personal AI Core／Rider OS 不同的地方：News OS 没有既有的 ADR/ADL 机制——OS-N audit 已确认全仓库搜索 "ADR"／"ADL"／"Architecture Decision" 零命中，所以这次没有新增第三份"决策记录"文件，只改了两个文件（Constitution + Project State）；规则改用 News OS 自己既有的 §1 版本历史机制记录，没有套用 Personal AI Core／Rider OS 的三文件模式。新增的 §9 明确跟 News OS 自己既有的「persistent state」（P2，运行期 Sheets 资料）、「checkpoint」（File_Map §6，单次执行内的 sysLog 诊断标记）、「source of truth」（P7，哪个 Sheet 对某笔资料有权威性）、「verified」（Project_State 里对技术细节的确认）四种既有用法做了区分，避免术语冲突。

Checkpoint System Active 仍标 ⏳ Pending——这轮只建立了规则，还没有后续实际开发工作可以证明规则真的被日常遵守，跟 Personal AI Core／Rider OS 目前的状态一致，不因为这轮完成就一并标记。

News OS 现有的 L4 Knowledge Engine 7 天试运行、L5-L7（TrendEngine／InsightEngine／DecisionEngine）开发限制、3 项已知 WATCH 项目，本轮均未触碰。

**Investment OS — 详细状态（2026-08-19）**

Universal Defined ✅／Inherited by Governance Authority ✅／Locally Adopted ✅（`00_Project_Constitution.txt` 新增独立章节「ENGINEERING ARTIFACT PERSISTENCE & DEPLOYMENT GOVERNANCE」G1-G4，`00_Project_State.txt` 用既有的 ARCHITECTURE DECISIONS 日志追加条目）／File-level Verification ✅，**仅限本次实际改过的两个文件**／Overall Investment OS Governance Verification ✅ **Confirmed**——audit（含一轮持久化目的地专项排查）与实作均在你逐步授权下完成。

这个 registry 原本没有 Investment OS 这一行——三份任务需求文本里都点名过"Investment OS"，但这份名录一直没有对应条目，且已知缺口清单曾把它误判为"可能等同 Finance OS"。实际审计 Investment OS 真实 repo（804K，自己的 Constitution／ADR-1／30 个引擎文件）后确认这是一个完全独立的投资组合追踪系统，不是 Finance OS——已新增这一行，不是覆盖既有条目。

跟 Personal AI Core／Rider OS／News OS 都不同的地方：Investment OS 仓库里嵌了一份完整但停在 v1.1 的 UEF 快照（`99_00`~`99_13`、`99_AR_00`~`99_AR_08`，仍被 ADR-1 等重大决策流程实际引用，但不含 §0.6 persistence 规则）——这次新规则只引用 Master UEF v1.12，完全没有修改或依赖这份嵌入快照，两者是独立议题。此外，Investment OS 的实际持久化基础设施是 GitHub + `clasp push` 部署到 live GAS（你在对话里直接告知，仓库自己的治理文件原本完全没提过）；新规则因此没有像其他 OS 一样考虑过 `/mnt/user-data/outputs/` 这类 AI 工具路径，而是直接写 Investment OS 自己的真实链路：GitHub commit（file-level）→ `clasp push`（engine-level，明确不要求逐文件 push）→ 独立 live 核验（方法不预设，但明确要求"push 命令没报错"不等于"已核验"）。G3 引用了 2025-06-13 真实发生过的 TruthEngine v1/v2 drift 事故作为这条规则存在的实证，不是假设性风险。

Checkpoint System Active 仍标 ⏳ Pending——跟其它三个 OS 一致，规则刚建立，还没有后续 commit→deploy→verify 的实际循环可以证明。Investment OS 自己的业务逻辑、财务记录、市场数据逻辑、ADR-1 正在进行的 Phase 6 部署、L5-L7 等价的开发限制、30 个应用/引擎代码文件，本轮均未触碰。

**Reminder OS — 详细状态（2026-08-22）**

Universal Defined ✅／Inherited by Governance Authority ✅／Locally Adopted ✅（Constitution 新增 P10，新增 `00_ADR_008_AI_Development_Persistence_Governance.gs`，两者均引用 UEF v1.12 §0.6 而非重复摘录）／File-level Verification ✅，**仅限本次实际改过的四个文件**（`00_Project_Constitution.js`／`00_ADR_008_AI_Development_Persistence_Governance.js`／`00_Project_State.js`／`00_File_Map.js`，各自都走完 persist + 独立核验 + 记录 checkpoint）／Overall Reminder OS Governance Verification ✅ **Confirmed**——四档 file-level checkpoint 之上，另有一次 READ-ONLY FINAL VERIFY（重新 md5sum 四份输出+对全新解压的 pristine 副本做 full-repo `diff -rq`，确认零 drift）加一条独立的 GOVERNANCE milestone checkpoint（见 `Universal-Recovery-Manifest.md`）。

跟其它四个 OS 不同的地方：这是目前节奏最细的一轮——不是像 Rider OS／News OS／Investment OS 那样单轮完成，也不是像 Personal AI Core 那样多个文件放在同一回合，而是 Constitution／ADR／Project State／File Map 四个文件各自单独一个回合，逐档 modify→persist→独立核验→checkpoint→STOP，每一档都经你实时审阅、明确指令后才进下一档；draft 阶段你还给了两处具体措辞修正（P10 最后一段的适用范围表述、ADR-008 不自维护跨 OS 清单改指向本 Manifest），均已套用。Reminder OS 本来就有既有的 ADR 机制（7 个 ADR，005 号刻意保留给未来 Reminder Scheduler，未挪用），新规则记在 `00_ADR_008`，延续既有编号序列，不是新建一种文件类型；P10 放进 Reminder OS 自己既有的 P-series，不是模仿 News OS 的独立 §9 或 Investment OS 的独立 G-block。

Checkpoint System Active 仍标 ⏳ Pending——跟其它四个 OS 一致，规则刚建立，还没有后续实际开发活动可以证明规则真的被日常遵守。Reminder OS 自己的业务逻辑（ReminderEngine／TemporalEngine／SheetUtils／QueryEngine／EventBus／Output／Setup）、全部测试文件、`README.md`（已确认 stale，非权威文件）、以及 File Map 里早于这次任务就存在的既有缺口（内容停在 2026-07-15，仍详细描述已被删除的 `25_ReminderEngine.gs`／`26_ReminderOffsetEngine.gs`，缺 ADR-006/007 登记），本轮均未触碰。Source-code persistence 是否为 GitHub（`.clasp.json`／`.claspignore` 证据强烈但本仓库文字从未明确确认，跟 Investment OS 那次你直接告知不同）刻意未在这次治理采纳里下判断，留待另行确认。

**Personal Life OS — 详细状态（2026-08-25）**

Universal Defined ✅／Inherited by Governance Authority ✅／Locally Adopted ✅（`00_ADR.js` 新增 ADR-2026-07-24-024，`00_Project_Constitution.js` 零之七(四) 追加简短引用，两者均引用 UEF v1.12 §0.6 items 3-4 而非重复摘录）／File-level Verification ✅，**仅限本次实际改过的三个文件**（`00_ADR.js`／`00_Project_Constitution.js`／`00_Project_State.js`，各自都走完 persist + 独立核验 + 记录 checkpoint）／Overall Personal Life OS Governance Verification ✅ **Confirmed**——三档 file-level checkpoint 之上，另有一次跨文件 13 项 GOVERNANCE CHECKPOINT read-only 核验（全过）加一次 8 节 STATUS CHECKPOINT read-only 综述（全过），两者均在这次 Universal 层同步之前完成，见 `Universal-Recovery-Manifest.md` 对应 GOVERNANCE checkpoint。

跟其它五个 OS 不同的地方：Personal Life OS 是唯一一个在 Universal Governance 采纳之前，**先走了一整轮独立的 Constitution reconciliation**——最初上传的 `00_Project_Constitution.js` 核对时发现是历史性的 Productivity OS v4.7 文本（跟仓库其余已演进到 v5.1/v5.2 的治理文件不同步；期间还先发现过一次上传插曲，第一份档案误含了 News OS 已完成的治理内容），先经过一轮独立授权的 reconciliation（874 行历史文本原样保留，新增「零之七、v5.2 现状桥接」145 行）并独立核验通过，这次 ADR-024 才在这份已核验过的 Constitution 之上进行。OS-N audit 阶段还发现一项直接的本地证据：2026-08-14 Sprint 4 开发中途真实发生过一次容器文件系统重置事故（`00_ADR.js` ADR-2026-07-24-021／`Sprint4_Recovery_Audit.md`），两个文件的工作确认永久遗失——ADR-024 引用这次事故当本地证据，但没有夸大：事故当时的回应是补救（Recovery Audit），不是预防，预防规则是这次才正式建立。

Checkpoint System Active 仍标 ⏳ Pending——跟其它五个 OS 一致，规则刚建立，还没有后续实际开发活动可以证明规则真的被日常遵守。Personal Life OS 自己既有的 Sprint Acceptance Gate 机制（ADR-2026-07-24-019，明确保持独立、不被本次规则取代）、22 条既有 ADR（001-022）、业务逻辑与全部应用代码、以及 Constitution reconciliation 阶段已发现并如实记录、明确排除在这轮范围外的四份仍停在 Productivity OS v4.7 的治理文件（`00_Known_Limitations.js`／`00_Command_Reference.js`／`00_Roadmap.js`／`00_Architecture_Review.js`），本轮均未触碰。

*本节由 Universal Governance Propagation 任务（2026-08-16）建立。之后每次实际核对某个 OS 的真实 repo 后，回来更新对应格子，不要整批批量改——每次只改刚核对过的那一个 OS。*

---

## Personal AI Core
生态的 AI 中枢。负责 Router、AI Orchestrator、Prompt Manager、Memory、RAG、Context Builder、Knowledge Base、Audit、AI Capability Registry、Tool Calling，协调本地 AI 与云端 AI（Claude、GPT、Gemini、Qwen 等）之间的分工。
**状态**：已连接 Personal Life OS、Reminder OS，核心能力持续开发中。

## Rider OS
外送工作与车辆管理。负责开工/收工记录、收入、油耗、BUDI95 补贴、保养、车况、提醒、统计报表、事件日志。
**状态**：每天实际使用，完成度最高。Phase 1 已验证作为 Fact 发布端（ExpenseCreated）。

## Finance OS
资产、投资、预算与现金流管理。负责银行、股票、REIT、ETF、股息、现金流、预算、支出、收入、净值、储蓄目标、投资组合分析。
**状态**：Phase 1 已验证订阅 Rider OS 支出事件并自动记账，其余功能仍在设计中。

## Property OS
房地产管理。负责房屋资料、房贷、贷款、缺陷记录、维修、水电账单、Sinking Fund、门牌税、租客、租金、文件、估价。
**状态**：架构与 Sheet 已建好，尚无真实数据流入，尚未接入 Communication Layer。

## Reminder OS
提醒事项的执行端，对外提供 CreateReminder 等 Command，由其他 Domain 或 AI 触发建立提醒。
**状态**：已连接 Personal AI Core，正常运作。Phase 1 已验证作为 Command 接收端（含幂等与认证校验）。
*注：与 Personal Life OS 早期功能清单中的"Reminder"存在重叠，建议统一由 Reminder OS 负责，Personal Life OS 改为透过 Command 呼叫它，不重复实现。*

## Personal Life OS
日常生活与个人事务中心。负责 Life Inbox、待办、个人记忆、目标、习惯、日记、购物清单、文件、日历、家庭事务、个人知识库，是生活资料的入口。
**状态**：已连接 Personal AI Core，持续升级中。

## News OS
资讯收集与分析。负责 RSS 订阅、过滤、排序、AI 摘要、Telegram 推送，分类涵盖 Malaysia、AI、Economy、Property、Finance、Stock、Technology。
**状态**：每天实际使用，存在已知但尚未修复的 bug。先修 bug，不建议现在接入 Communication Layer。
**后续建议**：News OS 本质上更像信息的终点（读多、几乎不产生别人需要的 Fact），有可能根本不需要成为 EventBus 的发布者或订阅者，除非未来想让其他 Domain 订阅"NewsDigestReady"之类的事件。

## Content / Video OS
内容创作与自媒体管理。负责拍片计划、脚本、分镜、内容日历，YouTube/Facebook/TikTok 发布，AI 辅助脚本、缩图、SEO 与成效分析。
**状态**：尚未设计架构。
**后续建议**：写架构时就把"订阅 Finance / Property 等 Domain 的大事件（例如触发一支投资主题的影片企划）"设计进第一版，不要等架构定型后再回来改。

## Research / Knowledge OS
知识库与研究支持。负责资料收集、长文研究、RAG、知识整理、引用来源管理。
**状态**：尚未设计架构。
**后续建议**：跟 Content / Video OS 一样，写架构时就把 Communication Layer 的订阅设计进第一版。

## Inventory & Procurement OS
库存与采购管理。负责比价、历史价格、供应商资料、成本分析、采购记录，以及工具、零件、消耗品库存与提醒。
**状态**：架构已完成，尚未撰写 runtime。

## Execution OS
跨 Domain 的命令编排者。消费 Fact，判断是否需要触发一个或多个 Command，透过 Connector 派发给对应 Domain 执行；不直接拥有任何业务资料。
**状态**：架构已完成，已与 Personal AI Core 职责切分完毕。Phase 1 已验证作为 Command 派发端（测试骨架，非真正编排逻辑）。
**后续建议**：先留空壳，不要提前把完整的多步骤编排逻辑建出来，等第一个真实的多步骤流程出现（例如真的处理一笔房产交易）再填入真正的 workflow 追踪与编排规则。

## Decision OS
跨 Domain 的判断者。消费 Fact 进行分析（例如预算是否超支），必要时透过 Command 让其他系统采取行动；本身不直接修改任何 Domain 的资料。未来承载 Decision Record、Alternatives、Evidence、AI Debate 等能力。
**状态**：尚未开工。

## Health OS（构想阶段）
健康管理，预计涵盖体重、运动、病历、饮水、睡眠、热量、用药提醒。
**状态**：仅在早期讨论中提及，尚未纳入正式架构。

---

## OS 之间的关系

**结构性关系（稳定，不常变，安全手动维护）**
- Execution OS、Decision OS、Personal AI Core 三者都能消费 EventBus 上的任何 Fact，也都能透过 Connector 对任何已注册 Service 的 Domain 发送 Command——这是架构层级的能力，不是某条具体连线。
- 任何 Domain 都可以发布 Fact；是否公开 Command，由该 Domain 自己决定（写不写 Service、要不要在 Command Registry 登记）。
- Personal AI Core 另外维护一份私有的 `AI_INTERESTED_EVENTS` 订阅清单，用于批量更新 Memory、生成 Insight，不是逐事件即时反应。

**目前已知的具体连线（快照，不是唯一真实来源）**

| Fact / Command | 来源 | 目标 | 状态 |
|---|---|---|---|
| ExpenseCreated（Fact） | Rider OS | Finance OS | 已验证（Phase 1） |
| ExpenseCreated，category=fuel（Fact） | Rider OS | Inventory & Procurement OS | 规划中（Phase 2） |
| CreateReminder（Command） | Execution OS | Reminder OS | 已验证（Phase 1） |

这张表容易随时间跟实际情况脱节，真正的来源永远是 CommunicationRegistry 的 Event Registry（publisher 栏）与 Command Registry（owner 栏）。这里只用来快速浏览，需要确认最新关系时请查 Registry 本身。

---

*维护提醒*：这份名录是手动维护的文字，跟 ADR-021 里"capabilities 由 Registry 公式推导、不手动维护"的原则不完全一致——Communication Registry 的 ServiceRegistry 只记录有对外开放 Command 的能力，不包含每个 Domain 内部的完整职责描述（例如 Rider OS 记油耗这类没有开 Command 的功能），所以这份名录暂时没办法完全自动生成。更新 OS 状态或新增 Domain 时，记得回来同步这份文件，避免跟实际情况脱节。
