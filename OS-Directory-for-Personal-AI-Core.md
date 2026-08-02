# OS 名录 — Personal AI Core Knowledge Base 条目

供 Personal AI Core 查询"某个请求该找哪个 Domain"用。判断路由或需要了解某个 Domain 大致职责时，先查这份名录，不必读整份 Universal Blueprint 或 Communication Registry。

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
