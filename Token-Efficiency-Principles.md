# Token 效率原则

整个讨论过程反复出现的目标：Personal AI Core 不需要每次都反复理解整个系统。这份文件把具体做法集中起来，分散在各个 ADR 里的相关设计（Registry 结构、批量处理等）都呼应这几条。

## 1. EventBus / CommandBus 的日常流转不该经过 AI

Rider 发布事件、Finance 轮询处理、Connector 派发命令，这些都是纯代码，零 token。AI 应该只在真正需要"理解、总结、生成、判断"的地方介入，例如每天跑一次的 AI Daily Summary，读当天累积的事件产出一份摘要，而不是每个事件都单独调一次 AI。这一条比任何 Prompt 技巧都省，也是 Personal AI Core 订阅私有的 `AI_INTERESTED_EVENTS` 清单时"只做 Memory / Insight Queue，真正生成放到每天批量处理"这个设计的理由。

## 2. Payload 用引用，不用整段描述

Event 或 Command 的 payload 只放金额、日期、来源 ID 这类关键字段，不要把完整的自然语言描述塞进去。AI 真的需要理解某笔资料的完整上下文时，再用 source_ref／source_id 去对应 Domain 查，大部分时候根本用不到完整描述。

## 3. Registry 当作 AI 的查表工具，不要每次读整份 Blueprint

Context Builder 需要理解某个 Event 或 Command 时，只查 Registry 里对应的那一行，不需要把整份 Universal Blueprint 塞进 Prompt。这也是 Event Registry／Command Registry 做成 Sheet、而不是 .js 文件的附带好处——Sheet 天生就是"查一行"，不是"读整份文件"。

## 4. Prompt Caching（透过 Claude API 呼叫时）

Anthropic API 支持把重复出现、不常变的内容（例如 Blueprint 摘要、Domain 规则）标记为可缓存。重复调用同样内容时，最多能省下九成输入 token 成本，延迟也能降到一半以下；默认缓存 5 分钟，可以设定延长到 1 小时。如果 Router 也会调用 GPT / Gemini / Qwen，值得查一下它们各自有没有类似机制。**这条原则讨论中提出过但还没有落进任何 ADR**，实作 Personal AI Core 呼叫外部 AI API 的那一段时值得回来用上。

## 5. AI 调用尽量批量，不要逐条

需要 AI 判断或归类的场景，累积一批再一次调用，比每个事件单独调用省下大量重复的 system prompt 开销。跟第 1 条是同一个精神的不同应用场景。
