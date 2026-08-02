# Universal Domain Blueprint

**Status**: Draft v1 — 待审阅（不同于 ADR-019/020/021 的 Accepted 状态，这是本文件第一次成形，尚未经过讨论/修改回合）
**Date**: 2026-07-29
**依赖**：ADR-019（Communication Protocol）、ADR-020（Domain Communication Rules）、ADR-021（Communication Registry）——本文件假设三者已冻结，不重复其中的 Context/Rationale，只处理"引用它们之后，一个 Domain OS 内部该长什么样"。

## 这份文件解决什么问题

ADR-019/020/021 定义了 Domain 之间怎么说话（协议、信封、Registry）。这份文件定义的是**协议之内、每个 Domain 自己的盒子里该有什么**——不管是 Rider OS、Finance OS，还是未来任何一个新 Domain，打开它的 Apps Script Project，都应该看到同一种骨架。这样新增 Domain 时不用每次重新设计"我该怎么组织这个项目"，AI 协作时也能预期任何 Domain 的文件长什么样、业务逻辑在哪里找。

本文件不涉及任何具体 Domain 的业务规则（那是 Evolutionary，属于 Domain 自己），只定义结构本身。

---

## 1. Domain Anatomy（每个 Domain OS 内部的标准结构）

```
{Domain}OS  (一个独立 Apps Script Project)
│
├── Entry Layer        —— 接收 Command 的唯一入口（doPost），只做验证与分派，不含业务逻辑
├── Service Layer       —— 业务规则实际发生的地方，Entry Layer 唯一允许调用的对象
├── Data Layer          —— 这个 Domain 专属的 Google Sheet(s)，只有 Service Layer 能读写
├── Event Publisher      —— 业务发生后，把 Fact 写进 EventBus（可选，取决于此 Domain 是否产生别人关心的事实）
├── Event Subscriber     —— 轮询 EventBus，处理感兴趣的 Fact（可选，取决于此 Domain 是否关心别人的事实）
└── Local Audit          —— 本 Domain 自己的操作记录，与 EventBus/Registry 的 Audit 分开
```

呼叫方向永远是单向的：

```
Connector --HTTP--> Entry Layer --> Service Layer --> Data Layer
                                          │
                                          └──(发生了新事实)──> Event Publisher --> EventBus
```

Entry Layer 不直接碰 Data Layer；Service Layer 不直接处理 HTTP 细节（那是 Entry Layer 的事）；这条分工线是本文件唯一要求所有 Domain 遵守的内部边界，其余内部怎么再细分，Domain 自己决定。

---

## 2. 文件命名规范

同一个 Apps Script Project 内，建议固定使用这几个文件名，AI 协作时可以直接预期在哪个文件找到什么：

| 文件 | 内容 |
|---|---|
| `Entry.js` | `doPost`，验证共享密钥、幂等检查、依 commandType 分派、包装 CommandResult |
| `Service.js` | 业务逻辑，`COMMAND_HANDLERS` 映射表 |
| `EventPublisher.js` | 封装对 EventBus 的 `appendRow` 写入 |
| `EventSubscriber.js` | （可选）轮询 EventBus 的 trigger 函数 |
| `Idempotency.js` | commandId 去重记录 |
| `Config.js` | 常数：EventBus ID、Service Registry 相关 ID、本 Domain 名称 |
| `Audit.js` | 本地操作记录 |

Domain 内部当然可以再拆更多文件（比如把 Service.js 拆成多个业务模块），但这七个名字保留给上述固定用途，不要挪作他用——这样任何人或 AI 打开一个陌生的 Domain Project，找 `Entry.js` 就知道入口在哪，不用每次重新摸索。

---

## 3. Module Boundary 与 Data Ownership

直接继承 ADR-020 的 Domain Isolation 原则，落到本 Domain 内部就是：

- 只有 `Service.js` 里的函数可以呼叫 `SpreadsheetApp` 操作本 Domain 自己的 Sheet。
- `Entry.js`、`EventSubscriber.js` 都不直接操作 Sheet，只能呼叫 `Service.js` 暴露的函数。
- 没有任何文件可以用 `SpreadsheetApp.openById()` 打开别的 Domain 的 Sheet——需要别的 Domain 的资料，只能透过 Dispatch Command 或订阅它发布的 Fact 拿到。

---

## 4. 接入 Communication Contract 的四个动作

一个 Domain 最多需要实现以下四种能力，具体需要几种取决于这个 Domain 的角色（纯粹的资料终点，例如 News OS，可能一个都不需要主动做，只需要 Entry Layer）：

**发布 Fact**（引用 ADR-019 Event Envelope、ADR-020 §7 警告优先原则）
```javascript
// EventPublisher.js
function publishEvent(eventType, payload, correlationId, causationId, chainDepth) {
  const sheet = SpreadsheetApp.openById(EVENTBUS_ID).getSheetByName('events');
  sheet.appendRow([
    Utilities.getUuid(), eventType, 1, DOMAIN_NAME, new Date().toISOString(),
    correlationId, causationId || '', chainDepth || 0, JSON.stringify(payload)
  ]);
  // 发布前建议查一次 Event Registry，没登记只记 warning，不阻挡发布（ADR-020 §7）
}
```

**订阅 Fact**（引用 ADR-020 alternatives D：游标存在自己的 Script Properties，不用共享列）
```javascript
// EventSubscriber.js
const INTERESTED_TYPES = []; // 本 Domain 关心的 eventType 列表

function pollEventBus() {
  const props = PropertiesService.getScriptProperties();
  let cursor = Number(props.getProperty('eventbus_cursor') || 1);
  const sheet = SpreadsheetApp.openById(EVENTBUS_ID).getSheetByName('events');
  const lastRow = sheet.getLastRow();
  if (lastRow <= cursor) return;

  sheet.getRange(cursor + 1, 1, lastRow - cursor, 9).getValues().forEach(row => {
    const [, eventType, , , , correlationId, causationId, chainDepth, payloadJson] = row;
    if (INTERESTED_TYPES.includes(eventType) && chainDepth < CHAIN_DEPTH_LIMIT) {
      handleFact(eventType, JSON.parse(payloadJson), correlationId, causationId, chainDepth);
    }
  });
  props.setProperty('eventbus_cursor', String(lastRow));
}
```

**接收 Command**（引用 ADR-019 Command Envelope/CommandResult，见下方 §6 参考骨架的 `Entry.js`）

**发送 Command**（呼叫 ADR-020 定义的共用 ConnectorLib）
```javascript
ConnectorLib.execute('CreateReminder', payload, correlationId, causationId, chainDepth);
```

---

## 5. Registry 义务

一个 Domain 存在，就必须在 Communication Registry（ADR-021）留下对应记录，这不是可选项：

- **Service Registry**：必须有且只有一行——domain、baseUrl、serviceVersion、status。
- **Event Registry**：为每一个会发布的 eventType 各登记一行，payload_schema 写清楚。
- **Command Registry**：为每一个会接受的 commandType 各登记一行，标注 owner=本 Domain、required_permission（autonomy level）。

新增/修改一个 eventType 或 commandType 时，Registry 那一行永远跟代码同一次改动一起提交，不要让 Registry 落后于实际代码——这是 Registry 能被 AI 信任当作查询依据的前提。

---

## 6. 参考实作骨架

以下是通用模板，`{DomainName}` 替换成实际 Domain 名称即可，业务逻辑填进 `Service.js` 的 `COMMAND_HANDLERS`。

```javascript
// Entry.js
const SHARED_SECRET = PropertiesService.getScriptProperties().getProperty('CONNECTOR_SECRET');
const DOMAIN_NAME = '{DomainName}OS';

function doPost(e) {
  const body = JSON.parse(e.postData.contents);
  const t0 = Date.now();
  const base = { commandId: body.commandId, correlationId: body.correlationId, targetDomain: DOMAIN_NAME, timestamp: new Date().toISOString() };

  if (body.secret !== SHARED_SECRET) {
    return jsonResponse({ ...base, status: 'REJECTED', errorCode: 'AUTH' });
  }
  if (isAlreadyProcessed(body.commandId)) {
    return jsonResponse(getCachedResult(body.commandId));
  }
  const handler = COMMAND_HANDLERS[body.commandType];
  if (!handler) {
    return jsonResponse({ ...base, status: 'REJECTED', errorCode: 'UNKNOWN_COMMAND' });
  }

  try {
    const result = handler(body.payload);
    const response = { ...base, status: 'SUCCESS', result, duration: Date.now() - t0 };
    recordProcessed(body.commandId, response);
    return jsonResponse(response);
  } catch (err) {
    return jsonResponse({ ...base, status: 'FAILED', errorCode: 'EXECUTION_ERROR', errorMessage: err.message, duration: Date.now() - t0 });
  }
}

function jsonResponse(obj) {
  return ContentService.createTextOutput(JSON.stringify(obj)).setMimeType(ContentService.MimeType.JSON);
}
```

```javascript
// Service.js —— 业务规则写在这里，Entry.js 永远不直接碰 Sheet
const COMMAND_HANDLERS = {
  // 'CreateXxx': (payload) => { ...操作本 Domain 自己的 Sheet...; return result; }
};
```

```javascript
// Idempotency.js —— 建议起点，非 Frozen；Command 量大的 Domain 建议改用 Sheet tab 而非 Script Properties
function isAlreadyProcessed(commandId) {
  return PropertiesService.getScriptProperties().getProperty('processed_' + commandId) !== null;
}
function getCachedResult(commandId) {
  return JSON.parse(PropertiesService.getScriptProperties().getProperty('processed_' + commandId));
}
function recordProcessed(commandId, response) {
  PropertiesService.getScriptProperties().setProperty('processed_' + commandId, JSON.stringify(response));
  // 注意：Script Properties 总容量 500KB，量大需要定期清理或改存 Sheet
}
```

---

## 7. AI Integration

Domain 本身不需要为了"给 AI 用"而写任何特殊代码——只要正确实现上面的 Entry/Publisher/Subscriber 模式，Personal AI Core 就能透过标准的 Command/Fact 机制与它互动，不需要针对每个 Domain 客制。AI 发出的 Command 一样要通过 Connector、一样受 Command Registry 的 `required_permission` 约束（读多写少、出错代价低的命令可标 `AI.AutoExecute`；涉及真实资产的命令标 `Human.ConfirmRequired`，见 ADR-021）。Domain 不需要，也不应该自己判断"这是不是 AI 发的"再做不同处理——权限分级发生在 Registry 层，不是 Domain 内部。

---

## 8. Governance（本文件自己怎么改）

- **Domain 内部业务逻辑、Service.js 里怎么写**：完全 Evolutionary，Domain 自己决定，不需要改动本文件。
- **改动第 1 节的 Domain Anatomy 本身**（比如要求所有 Domain 多一层强制的什么 Layer）：影响所有现有与未来 Domain，需要走 ADR，不能私下改一个 Domain 就算数。
- **改动第 2 节的文件命名规范**：同上，属于全体约定，需要 ADR。
- 更大范围的工程规范、AI 协作规则、跨 Blueprint 的变更流程，属于 Universal Engineering Framework（UEF）的范畴，不在本文件定义。**修正**：写这段时以为 UEF 还没成文，实际上真实的 UEF v1.5 已经存在（另一个更严谨、以真实代码为依据的过程产出，与本文件、ADR-019/020/021 一样都未对照过），本文件第 1、2 节的 Domain Anatomy 与文件命名规范，跟 UEF 里已有的 UCR1/2/3/6/7 等规则是否一致，同样需要核对，不能假设两者天然兼容。

---

## 9. 新增 Domain OS 清单

1. 建一个新的 Apps Script Project，命名 `{Name}OS`。
2. 建立专属 Google Sheet 作为 Data Layer，只有这个 Project 能读写。
3. 依 §2 建立 `Entry.js` / `Service.js` / `EventPublisher.js` / （需要的话）`EventSubscriber.js` / `Idempotency.js` / `Config.js` / `Audit.js`。
4. 部署 Web App，套用 §6 骨架。
5. Service Registry 新增一行。
6. 会发布 Fact 的话，Event Registry 逐一登记 eventType。
7. 会接受 Command 的话，Command Registry 逐一登记 commandType、required_permission。
8. 会订阅其他 Domain 的 Fact 的话，实现 `EventSubscriber.js`，游标存自己的 Script Properties。
9. 手动跑一次最小验证：发一个测试 Command、（如适用）发一个测试 Event，确认 CommandResult、Audit log 正常。
10. 验证无误，Service Registry 的 status 改成 Active。

---

## 待你审阅的地方

跟 ADR-019/020/021 不同，以下是本文件第一版里我自己做的新判断，还没被你验证过，标出来方便你决定要不要改：

- **第 1 节六层 Domain Anatomy 的切法**（Entry / Service / Data / Publisher / Subscriber / Audit）是我这次新提出的综合，此前讨论没有明确定案到这个颗粒度。
- **第 2 节的七个固定文件名**同样是新提议，如果你已经有自己的命名习惯，改这里比改所有 Domain 容易。
- **Idempotency 用 Script Properties 起步、量大再换 Sheet**这个建议是延续 EventBus 那次"先用最简单方式、真的遇到瓶颈再升级"的原则做的类推，但没有专门针对 Command 量验证过。

---

*本文件依赖 ADR-019/020/021，任何修改若牵动 Communication Contract 本身（信封、Registry 结构），请回到对应 ADR，不要在本文件里另外定义。*
