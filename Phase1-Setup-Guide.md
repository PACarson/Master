# Phase 1 落地指南：Fact Flow + Command Flow

对应 ADR-019 的 **Phase 1 — Communication Validation**。跑完这份指南，代表 EventBus、CommandBus、Connector、三个 Registry 全部验证可行，才能进入 ADR-019 定义的 Phase 2。

## 需要准备的东西

- 2 个新的独立 Google Sheet（不属于任何 Domain）：**EventBus**、**CommunicationRegistry**
- 在既有的 4 个 Apps Script 项目里各加一段代码：**Rider OS**、**Finance OS**、**Reminder OS**、**Execution OS**（如果 Execution OS 还没有独立项目，先建一个空的）
- 1 个新的 Apps Script 项目：**ConnectorLib**（发布成 Library，被 Execution / Decision / Personal AI Core 共用）

对应文件：
- `01-RiderOS-publishEvent.gs` → 贴进 Rider OS 项目
- `02-FinanceOS-pollEventBus.gs` → 贴进 Finance OS 项目
- `03-ReminderOS-WebApp.gs` → 贴进 Reminder OS 项目
- `04-ConnectorLib.gs` → 贴进新建的 ConnectorLib 项目
- `05-ExecutionOS-TestDispatch.gs` → 贴进 Execution OS 项目

---

## 第一步：建 EventBus Sheet

新建一个 Google Sheet，命名 `EventBus`。新增一个 tab 叫 `events`，第一行填表头（对应 ADR-019 的 Event Envelope）：

```
eventId | eventType | version | sourceDomain | timestamp | correlationId | causationId | chainDepth | payload
```

记下这个 Sheet 的 ID（网址 `/d/` 后面那一串）。

## 第二步：建 CommunicationRegistry Sheet

新建另一个 Google Sheet，命名 `CommunicationRegistry`。新增三个 tab：

**EventRegistry**，表头：
```
eventType | version | description | publisher | payload_schema | stability | replacedBy
```
填入第一行数据：
```
ExpenseCreated | 1 | Rider 完成一笔支出 | RiderOS | {amount,category,date,source_ref} | Stable |
```

**CommandRegistry**，表头：
```
commandType | version | owner | required_permission | payload_schema | stability | replacedBy
```
填入第一行数据：
```
CreateReminder | 1 | ReminderOS | AI.AutoExecute | {title,dueDate,notes} | Stable |
```

**ServiceRegistry**，表头：
```
domain | baseUrl | serviceVersion | status | capabilities
```
先填 domain 和 status，baseUrl 等第三步部署完 Reminder OS 再回来填。capabilities 那一格用公式从 CommandRegistry 推导（对应 ADR-021：不手动维护）：
```
domain: ReminderOS
baseUrl: （稍后填）
serviceVersion: v1
status: Active
capabilities: =TEXTJOIN(", ",TRUE,QUERY(CommandRegistry!A:C,"select A where C='"&A2&"'"))
```

**注意**：密钥不放进这份 Registry（ADR-019 的建议：Registry 是给 AI 读 context 用的，不该有明文密钥）。密钥分别存在 Reminder OS 和 ConnectorLib 各自的 Script Properties，见下方第三、四步。

记下 CommunicationRegistry 这个 Sheet 的 ID。

---

## 第三步：Reminder OS——部署成 Web App

1. 打开 Reminder OS 的 Apps Script 项目，贴上 `03-ReminderOS-WebApp.gs` 的内容。
2. 项目设置 → Script Properties，新增 `CONNECTOR_SECRET`，值填一串自己生成的随机字符串（例如 32 位随机码）。记下这串密钥，等下第四步要用。
3. Deploy → New deployment → 类型选 Web app，Execute as: Me，Who has access: Anyone。部署后会拿到一个 URL（`https://script.google.com/macros/s/.../exec`）。
4. 回到 CommunicationRegistry 的 ServiceRegistry，把这个 URL 填进 ReminderOS 那一行的 baseUrl。

## 第四步：ConnectorLib

1. 新建一个独立的 Apps Script 项目，贴上 `04-ConnectorLib.gs` 的内容。
2. Script Properties 新增两个：
   - `REGISTRY_SHEET_ID`：第二步 CommunicationRegistry 的 Sheet ID
   - `SECRET_ReminderOS`：跟第三步 Reminder OS 里填的 `CONNECTOR_SECRET` **完全一样**的字符串
3. Deploy → New deployment → 类型选 Library，记下 Script ID（Execution OS 那边要用）。

## 第五步：Rider OS

1. 打开 Rider OS 的 Apps Script 项目，贴上 `01-RiderOS-publishEvent.gs` 的内容。
2. Script Properties 新增 `EVENTBUS_SHEET_ID`：第一步 EventBus 的 Sheet ID。

## 第六步：Finance OS

1. 打开 Finance OS 的 Apps Script 项目，贴上 `02-FinanceOS-pollEventBus.gs` 的内容。
2. Script Properties 新增 `EVENTBUS_SHEET_ID`：同上。
3. Phase 1 验证阶段先不急着设置触发器，手动执行就够；跑通以后再到「触发器」页面加一个时间驱动触发器（例如每 10 分钟）执行 `pollEventBus`。

## 第七步：Execution OS

1. 打开（或新建）Execution OS 的 Apps Script 项目，贴上 `05-ExecutionOS-TestDispatch.gs` 的内容。
2. 左侧「程式库 / Libraries」→ 加入第四步的 ConnectorLib（用 Script ID），Identifier 填 `ConnectorLib`。

---

## 跑一次 Fact Flow

1. 在 Rider OS 项目里执行 `test_publishExpenseCreated`。
2. 打开 EventBus 的 `events` tab，应该多一行 `ExpenseCreated`，记下这行的 correlationId。
3. 在 Finance OS 项目里手动执行一次 `pollEventBus`。
4. 打开 Finance OS 自己 Sheet 里的 `Expenses` tab（脚本会自动建），应该出现刚才那笔 45.2 的记录，correlationId 应该跟 EventBus 那一行一致。

对照 ADR-019 验证项：
- EventBus 发布是否正常 —— events tab 多了一行
- Finance OS 订阅是否正常 —— Expenses tab 多了一行
- correlationId 是否保留 —— 两边比对一致
- chainDepth 是否正确递增 —— 这是原始 Fact，应该是 0
- Event Registry 查询是否正常 —— 人工核对 EventRegistry 里 ExpenseCreated 那一行存在、stability 是 Stable
- Audit 是否有记录 —— 如果 pollEventBus 过程出现不认识的 eventType 或解析失败，会写进 Finance OS 的 Audit tab（脚本自动建）

## 跑一次 Command Flow

1. 在 Execution OS 项目里执行 `test_dispatchCreateReminder`。
2. 查看执行记录，应该印出一个 `status: SUCCESS` 的 CommandResult，带着 commandId 和一个 reminderId。
3. 打开 Reminder OS 自己 Sheet 里的 `Reminders` tab，应该出现刚才那笔提醒。
4. 执行 `test_idempotency`——同一个 commandId 送两次，第二次应该拿回跟第一次完全相同的结果（同一个 reminderId），Reminders tab 只会多一行、不是两行。
5. （可选）故意把 Reminder OS 的 `CONNECTOR_SECRET` 改错一个字符，再跑一次 `test_dispatchCreateReminder`，应该收到 `status: REJECTED, errorCode: AUTH`，改回来后恢复正常。

对照 ADR-019 验证项：
- Connector 能否从 Service Registry 找到目标 —— 能正确解析出 Reminder OS 的 URL
- HTTP 请求是否成功、commandId 是否一致 —— 请求与回应的 commandId 相同
- CommandResult 是否正确回传 —— 结构符合 ADR-019 定义的 CommandResult
- 幂等是否生效 —— test_idempotency 的两次结果一致
- CommandFailed / REJECTED 是否正确产生 —— 故意改错密钥那步

---

全部跑完，Phase 1 完成，可以进入 Phase 2（其他 Domain 依同一套 Contract 接入，不用再改 EventBus 或 Connector 本身）。
