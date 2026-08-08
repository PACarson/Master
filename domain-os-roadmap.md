# Domain OS 架构决策记录

*个人参考文件，之后随时可以回来查 · 最后更新：2026-08-05*

## 现状

十几个 Domain OS（Rider / Property / Finance / Personal Life / Compliance…持续增加中），共用 Domain Logic、Event Bus、Truth Engine、Scheduler、AI、Connector 这套架构，目前全部跑在 Google Apps Script + Google Sheets 上。

目标：把这套生态发展成可以用 10–20 年、AI 能直接查询的个人「大数据库」（Second Brain + 十几个 Domain OS + AI Core）。

## 一句话结论

**现在**：留在 GAS，把每个 OS 的操作介面从 Sheet 原生控件换成 HtmlService 独立 Web App，其他先不动。

**以后**（碰到明确的触发条件才启动）：迁移到 Next.js + PostgreSQL/pgvector + MCP，而且用「渐进式、扁平」的方式做，不要一次到位的企业级分层。

---

## Part A · 现在就可以开始（不用等任何触发条件）

按优先顺序：

1. **Sheet 读写全部改成批次操作。** 一次 `getValues()` 拿整个 range、在内存里处理、一次 `setValues()` 写回去，不要一格一格呼叫。Google 自己的案例：同样逻辑没批次要跑 70 秒，批次之后 1 秒——这是所有 Domain Engine 现在就能拿到的效能提升，跟要不要换 UI、要不要换后端完全无关。

2. **挑一个 OS，把介面从 sidebar/dialog 换成独立发布的 HtmlService Web App。** 目前投资报酬率最高的一步——不用碰 Engine，也不用做任何"要不要离开 GAS"的架构决定。技术选型：Tailwind CDN 管样式，Vue 3 CDN 管互动（不要 React，没有打包流程会很痛苦），`google.script.run` 包一层 Promise。建议从最简单的一个 OS 开始练手，抓到手感后再套用到其他 OS。

3. **建立共用的 Shared UI（设计系统）。** 把共用的 CSS/JS（配色、字体、卡片样式、导航组件）写成一份文件，放到 GitHub Pages（免费），每个 OS 的 HtmlService 页面用 `<link>` / `<script src>` 引入——跟引入 Tailwind CDN 是同样做法。可以从第二个 OS 开始时正式抽出来，不用一开始就做。

4. **前端行为改成"一次拿整页资料"。** 页面载入时一次性把需要的资料都拿回来，Tab 切换、排序、筛选都在前端处理，不要每次互动都呼叫一次 `google.script.run`——这个改动能大幅降低"慢半拍"的体感。

5. **确认不用 AppSheet。** AppSheet 写回 Sheet 不会触发 onEdit trigger，会直接绕过 Event Bus，让 Truth Layer 出现不同步的资料流——这是架构层面的不兼容，不只是美学问题，可以直接排除这个选项。

6. **定期检查 Apps Script 配额用量**（后台 Executions 页面）。配额按 Google 帐号算、不是按项目算，十几个 OS 如果都挂了 time-driven trigger 跑 Scheduler，执行时间会加总。建议每季看一次，避免哪天所有 Scheduler 集体撞墙却不知道为什么。

---

## Part B · 备用计划：触发条件

出现以下任一情况，才需要认真考虑启动 Greenfield 迁移，不需要提前准备：

| 触发条件 | 说明 |
|---|---|
| 某个 OS 的资料量逼近数万〜数十万笔 | Rider OS 因为每天有订单，最有可能第一个碰到；Property / Compliance 这类资料量本来就有限的 OS，大概率永远不会碰到。**2026-08-07 实测**：照 Rider OS 现有 `.xlsx` 的实际增速推算，长最快的 `Event_Log`（系统执行日志，非业务事件）要 ~14 年才到 10 万行，`Daily`/`Fuel`/`Maintenance` 等业务表要 ~99–595 年——目前离这个触发条件非常远，可以放心留在 Part A |
| AI 需要对所有 Domain OS 做大量即时分析 | 会撞上 GAS 单次执行 6 分钟的上限 |
| 系统开始有多个使用者同时操作 | 家人、员工一起用，GAS 的并发写入能力会开始吃力 |
| 想要真正的离线体验、推播通知、原生手机功能 | GAS 的 sandbox iframe 环境下，localStorage 持久化本身就不稳定，类似离线快取的功能大概率也靠不住 |
| 做了骨架屏、一次拿整页资料之后，还是每天因为 0.5–1.5 秒的往返延迟而烦躁 | 这是 GAS 架构里无法根除的摩擦，只能靠换架构解决 |

在这些条件出现之前，Part A 的路线成本最低、投资报酬率最高，不需要因为"以后可能要换"而现在就焦虑。

---

## Part C · 备用计划：触发后的目标架构

### 技术栈

| 层 | 选择 | 备注 |
|---|---|---|
| 前端 | Next.js + React + TypeScript + Tailwind + shadcn/ui | 有 Vue 习惯的话 Nuxt 也站得住脚，生态小一点但心智负担更低 |
| Application 层 | Next.js API Route / Server Action | 不另架 Express/NestJS，先不要 Service/Repository/Factory |
| Database | PostgreSQL + pgvector（用 Supabase 托管） | 结构化资料跟语义搜索同一个 DB 完成，不用同步两套系统；不要额外包 DatabaseAdapter——会挡住 Postgres/pgvector 独有的 hybrid search 能力 |
| ORM | Drizzle 或 Prisma 都可以 | Prisma 7 已经把过去 bundle size / 冷启动的劣势补掉大半，2026 年这更接近个人习惯选择，不是硬指标差距 |
| AI Core | MCP + Vercel AI SDK + Tool Calling | 每个 Domain OS 把自己的函数包成 MCP tool；不要自己写 Planner/Router——给模型一份写清楚的 tool 清单，模型本身就是 Planner 和 Router |
| Auth | Google Login | 个人/家庭使用足够，不用自己写帐号系统 |
| Mobile | PWA 优先 | 真的需要原生功能再考虑 Flutter/React Native |
| 部署 | Vercel + Supabase 免费额度起步 | 之后如果要自架（Mini PC / NAS / VPS）省月费，一定要配自动化、加密、异地备份（如 restic/borgbackup → Backblaze B2），不然"近乎零成本"容易变成一次性全部归零，对 Finance/Compliance 这类敏感资料尤其重要 |
| 代码组织 | pnpm workspace + Turborepo monorepo | `apps/web` + `packages/{各 domain}-os` + `packages/ai-core` + `packages/ui` + `packages/db` |

**不建议**：Cloudflare D1 当主资料库（底层是 SQLite，不是 Postgres，并发写入弱）；单独的 vector DB（Pinecone/Qdrant/Weaviate）；一次到位的 Hexagonal/Clean Architecture 六层结构；AI Core 拆成 Planner/Router/Memory/Workflow/Audit 等一堆具名子系统。

### 每个 Domain OS 的内部结构（渐进式，不是一次到位的六层）

```
property-os/
├── index.ts       # 对外 export 的函数：logMaintenance(), getSummary()...
├── schema.ts       # 类型定义 + ORM schema
├── queries.ts       # 读取相关逻辑
├── commands.ts      # 写入/异动相关逻辑
├── tools.ts        # 包成 MCP tool，直接调用上面的函数
└── test/
```

Web UI 的 API route 跟 AI Core 的 MCP tool 呼叫同一批函数，逻辑不重复。等某个 OS 真的复杂到出现明显痛点（例如 Finance OS 规则多到自己都理不清）再对那一个 OS 局部加分层，不要现在就为全部 OS 预先付这个成本。

### 迁移方式

不要一次性切换。挑一个最简单的 OS 当 pilot，把 Web App、Postgres、AI Core/MCP 整条链路走通、跑稳，再照同一个 pattern 复制到其他 OS；GAS 那边保持运作，直到每个 OS 都迁移验证完成。业务逻辑本身（GAS 里已经想清楚的规则）从 JavaScript 搬到 TypeScript，大部分是语法转译，不是重新设计，风险比"重写"这个词听起来的要低。

---

## 不论现在还是以后都要守住的原则

1. **Domain First** — 业务规则永远属于各自的 Domain OS，不重复。
2. **AI Through Tools** — AI 只能透过 Tool 调用 Domain 的函数，永远不直接碰资料库；守住这条线，AI 就不会破坏 Truth Layer。
3. **Simple by Default** — 默认选最简单能动的做法，只有真的感觉到痛才加抽象层；架构复杂度要匹配团队规模（现在是你一个人）和现在真实的痛点，不是想像中十年后的规模。
4. **Shared Only When Proven** — 至少两个 Domain 都重复需要的东西，才放进 Shared。
5. **不要为了灵活性抽象掉一个技术的独有优势** — 别为了"以后可能换资料库"就把 Postgres/pgvector 的 hybrid search 包进一个通用接口里；今天选它就是为了用它的独有能力。
6. **Optimize When It Hurts** — 效能、维护、扩展性的问题，等真正出现了再处理，不要预先优化。

---

## 参考数字（GAS 配额速查）

| 项目 | 上限 |
|---|---|
| 单次执行时间 | 6 分钟（个人版、Workspace 版现在一致） |
| 每日 trigger 总执行时间 | 个人版 90 分钟／Workspace 版 6 小时 |
| Sheets 读写速率 | 每分钟 300 次请求 |
| UrlFetch 呼叫次数 | 个人版每天 20,000 次／Workspace 版 100,000 次 |
| google.script.run 单次往返延迟 | 一般 400–1500 毫秒，写法差的可到 3 秒 |

---

*这份文件是这一系列讨论的整理结论。之后如果 GAS 或某个 OS 开始出现 Part B 列出的状况，可以直接回来这份文件的 Part C 当作起点。*
