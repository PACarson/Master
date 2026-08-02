# Engineering Principles — 通用工程原则

对应最早提到、但一直没有正式写出来的 Universal Engineering Framework（UEF）。这份文件里的原则不是 Communication Architecture 专属，是未来任何 Domain OS 开发都该遵守的通用规则。

## 先验证，再扩展（Walking Skeleton）

不要在还没被真实使用验证过的系统上，提前设计一整套覆盖所有想象中未来场景的架构。做法是：挑一个真实存在的最小场景，把最简化版本的每一层都跑通（哪怕 Execution OS 一开始只是"按顺序调用 + 写 log"，EventBus 一开始只是一张 Sheet），用真实数据观察问题，让复杂度从真实的重复劳动和真实的 bug 里长出来，而不是预先设计出来。

这条原则在讨论过程中具体应用了两次：
- 决定不要一次把 10 个 Domain OS、20 个 Execution Layer 引擎都建出来，先验证 Rider → Finance 这一条真实场景（后来变成 Phase 1 的 Fact Flow）
- Communication Architecture 定案后，Phase 1 只验证两条最小链路，确认可行才进 Phase 2，不是要一次做完整个系统

## 抽象要先合并，等真的分裂了再拆

最早的 Execution Layer 列了 20 个"引擎"（EventBus、Truth Engine、Scheduler、Reminder Engine、Temporal Engine、State Engine、Replay Engine、Knowledge Engine、AI Contract、Output Engine、Audit Engine、Notification Engine、Obligation Engine、Context Engine、RAG Engine、Prompt Engine、Memory Engine、Review Engine、Issue Signature、Change Log、AI Capability Layer）。这个数量对当时的实际建成进度（只有 Rider OS 真正成熟）明显超前。建议的合并方式：

- 时间类（Scheduler、Reminder Engine、Temporal Engine）→ 先合并成一个 Time Engine
- 状态与追溯类（Truth Engine、State Engine、Replay Engine、Change Log、Issue Signature）→ 本质上都是"发生过什么、现在状态是什么、能不能重放"，EventBus 配一张事件日志表就已经满足大部分需求
- AI 相关（Knowledge Engine、AI Contract、Context Engine、RAG Engine、Prompt Engine、Memory Engine、AI Capability Layer）→ 直接并入 Personal AI Core，不在 Execution Layer 重复列一次（这条后来在 Personal AI Core 与 Execution Layer 的职责重叠讨论中被正式确认，也是 ADR-020 判断 Connector 该归属 Universal Infrastructure 而非 Execution Layer 的同一个逻辑）
- Output Engine + Notification Engine → 视实际使用状况决定要不要合并
- Review Engine、Obligation Engine → 具体要解决的场景还不清楚，先不建，等真的遇到需求再回来设计

这份清单本身没有被否决，只是被要求先想清楚一个真实用例再动手，避免为了假设的未来需求预先建一堆用不上的抽象。

## AI 自主权按风险分级，不要一刀切

Personal AI Core 未来会有自动 Review Code、自动修改、自动测试、自动部署的能力，方向本身没问题，但不同 Domain 的风险不一样：

- 读多写少、出错代价低的领域（News OS、Content OS）：可以给 AI 更大自主权，自动执行
- 涉及真实资产、法律文件的领域（Finance OS、Property OS）：出错代价高，而且这类 bug（比如净值算错）经常要几个月后才被发现，"自动修改 + 自动部署"应该保留一个人工确认的关卡

这个原则后来具体落地成 Command Registry 里的 `required_permission` 栏位（`AI.AutoExecute` / `Human.ConfirmRequired`），但原则本身比这一个栏位更通用——未来 Personal AI Core 自己的自动化能力（不只是透过 Command 触发别人）也该套用同一套风险分级逻辑，不是只在 Communication Registry 里才生效。

## Registry 校验失败，警告优先于硬拦截——但要看情境

已经为真的事实，不该被一个文件层级的登记问题拦下；但尚未执行、且无法理解或无处可送的动作，不该被静默当作已完成。完整推导过程记在 ADR-020 第 7 条，这里只标注它其实是一条更通用的工程直觉：越接近"记录已发生的事"，越该倾向不拦截；越接近"即将执行、可能造成后果的事"，越该倾向明确拒绝而不是猜测。
