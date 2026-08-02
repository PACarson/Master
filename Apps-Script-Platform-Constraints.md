# Apps Script 平台约束 — 技术参考

这份文件记录讨论过程中实际查证过的 Google Apps Script 平台限制，是很多架构决定（尤其是 ADR-019、020）背后没有明写的技术依据。以后要新增 Domain、调整触发器策略、或评估要不要迁移平台，回来查这份文件，不用重新搜一次。

## 执行与配额限制

| 项目 | 数值 | 影响到的决定 |
|---|---|---|
| 单次执行时间上限 | 6 分钟（Consumer 与 Workspace 帐号统一，2026 年起不再有 30 分钟例外） | EventBus 轮询函数、Connector 呼叫都要在这个时间内完成 |
| 每日触发器总运行时间 | Consumer 90 分钟／天，Workspace 6 小时／天 | 轮询频率不能设太密；能合并的触发器就合并，不要每个用途都开一个新触发器 |
| 每个脚本的触发器数量上限 | 20 个／使用者／脚本 | 新增 Domain 时，优先把 EventBus 轮询塞进既有触发器 |
| 时间驱动触发器最短间隔 | 最快每分钟一次（排程本身不保证精准） | 技术上可以设很密，但个人系统场景下没必要，几分钟延迟无感 |
| UrlFetchApp 每日呼叫上限 | Consumer 20,000 次／天，Workspace 100,000 次／天 | Connector 大量派发 Command、或 AI Core 频繁呼叫外部 AI API 时要注意 |
| Properties Service 容量 | 单一值 9KB，单一 store 总量 500KB | 幂等快取、EventBus 游标现在用这个存；量大之后（尤其幂等快取）建议改成 Sheet tab |

## 决定性发现：Installable Trigger 不会因为脚本或 API 写入而触发

这是 EventBus 设计从"可能用 push"转向"一定用 poll"的关键依据。Google 官方文档明确说明：脚本执行和 API 请求都不会触发已安装的触发器（onEdit、onChange 等），不管是同一个脚本项目内部的写入，还是完全不同项目发起的调用，都不算数。也就是说在纯 Apps Script + Sheets 环境下，EventBus 不可能做成推送式通知，轮询是唯一可靠的做法——不是保守选择，是平台限制。

## Apps Script Library 的隐藏成本

呼叫 Library 里的函数，是在呼叫方那一次执行里完成的，不是独立的执行，这代表：
- 用了 Library 的脚本执行速度会比整段代码写在同一个项目里慢（Google 官方文档确认）
- 两个"独立"的 Domain，执行时间预算却共用了一份——这是 ADR-019 否决 Library 作为跨 Domain 通讯手段的核心理由
- Library 是 Apps-Script-to-Apps-Script 专属机制，Docker、Python、本地 AI 无法呼叫

## Web App 没有原生的路径路由

Web App 部署出来的 URL 固定是 `/exec`（或 `/dev`），不支持像 `/reminder/create` 这样的路径式路由。Google 官方范例的做法是在同一个端点内用一个字段（本项目用 commandType）做条件分派。这是 ADR-019 决定"统一信封、单一端点"而不是"按操作切路径"的技术依据之一——不只是风格选择，是这个平台上路径路由本来就要绕道才做得到（有一个非官方功能，但不论存取设置如何都要额外处理 OAuth access token，不划算）。

## Web App 回应不能细控 HTTP status code

ContentService 建立的回应，没有像一般 web framework 那样能自由设定 HTTP status code 的机制。这是 CommandResult 把 status（SUCCESS/FAILED/REJECTED）放进 JSON body、而不是依赖 HTTP status code 的原因之一——不管命令成功与否，HTTP 层级通常都是 200，真正的结果要看 body 里的 status 栏位。

## `scripts.run`（API Executable）评估后放弃

这个机制要求呼叫方与目标脚本挂在同一个 GCP 项目下，还需要处理 OAuth 2.0 token 的取得与刷新。对个人系统而言，比"HTTP POST 一个 JSON + 共享密钥"复杂得多，没有对应的额外好处（详见 ADR-019 Alternatives B）。
