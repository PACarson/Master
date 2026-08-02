# Universal Domain OS Blueprint
## v1.2 — Stable, 2026-07-23

**Status:** Stable. Future changes governed by UEF §0.9 (Blueprint Change Policy, new in UEF v1.3): evidence from at least two independent projects, or a demonstrated ecosystem-wide benefit — not a single project's need, however reasonable. Ratified through Rider OS's complete Review + Production Readiness Audit, its own Architecture-Layers-to-Blueprint mapping, and the Cross-Cutting Capabilities addition (BP-7) — no longer a paper architecture.
**Relationship to UEF:** parallel, never merged. UEF v1.3 defines HOW software is engineered; this Blueprint defines **what** a Domain OS can contain.

## The governing principle

> Blueprint describes **"What a Domain OS can contain."**
> Not **"How it must be implemented."**
> *How* is UEF's job. *What* is this document's job. The two don't cross.

Every node below is written to that standard: a name, what it is, and how mature it is — never a spec for building it.

**BP-6 — Blueprint defines capabilities and architectural building blocks, never implementation details.** Blueprint can say a Planner Layer, a Projection Layer, a Knowledge Layer exists as a concept. It never mandates function naming, Sheet naming, GAS module splitting, Event format, or API structure — those are each project's own Architecture, and UEF Review (§8) is what checks whether a project's specific choices are sound, not this document. Every citation below is an *example of the capability existing*, not a template to copy exactly.

---

## 0. Governance

**Purpose:** every Domain OS project is governed.
**Contains:** nothing of its own — see UEF.
**Reference:** [Universal Engineering Framework v1.1](Universal_Engineering_Framework_v1.1.md), §0.2 (mandatory documents), §0.7 (ADR structure), §0.5 (Definition of Ready/Done/Production-Ready).

That's the whole node. Constitution, State, File Map, ADR, and Review History are UEF's single source of truth for what they're for and how they're structured — copying that content here would just be a second copy to keep in sync (EP2). If UEF's document set changes, this node doesn't need an edit — the reference still points at the current version.

---

## Tier definitions (revised)

Tier is not a maturity or quality score. It's **whether the ecosystem has an evidence-based, stable pattern for this node yet**:

- **Tier 1** — verified in **more than one** project independently
- **Tier 2** — a real implementation exists, but only in one project — not yet a cross-project pattern
- **Tier 3** — a concept only; no stable practice anywhere

Promotion is UEF EP5 (empirical verification) and EP6 (no decision by opinion alone) applied to this document specifically: a node moves up only when a second independent project actually builds it that way — never in anticipation of one. Nothing here gets designed ahead of that evidence (EP3).

**BP-5 — Tier is evidence-based and may move in either direction as ecosystem evidence changes.** A new project's implementation can promote a node; a pattern that turns out to have no real cross-project reuse value can be demoted just as legitimately. Tier is not a rank once earned and kept — it's a current snapshot of what's actually shared practice right now.

Re-checked against this stricter bar (multi-project, not just well-evidenced-in-one), a few nodes move from the v0.1 draft. Shown below with the reasoning, not silently changed.

---

## 1. Foundation

| Node | Tier | Evidence |
|---|---|---|
| **Configuration** | 1 | A single file/module holding cross-cutting constants — confirmed in Rider OS (`01_Config.txt`) and implied ecosystem-wide by the shared numbered-file-prefix convention and shared-Sheet architecture (Personal AI Core, Reminder OS). Rider OS's exact `CONFIG.SHEETS`/`CONFIG.COL` sub-structure is its own instantiation, not claimed as identical elsewhere |
| **Schema** | 1 | Shared Google Sheets backend, "all multi-file communication routes through the shared Spreadsheet" — stated explicitly for Personal AI Core *and* Reminder OS, confirmed directly for Rider OS |
| **Identity** | 2 *(was 1 in v0.1)* | Constitution's `SYSTEM IDENTITY` block confirmed only in Rider OS this session. Likely shared given common origin, but not independently verified in Personal AI Core / Reminder OS — downgraded rather than assumed |
| **Event Definitions** | 1 | `CONFIG.EVENT`-style enum (Rider OS) alongside the ecosystem-wide EventBus/event-sourcing discipline (explicit for Personal AI Core) |
| **Permissions** | 3 | No change — zero evidence anywhere |
| **Versioning** | 1 | Entailed by the already-universal `00_Project_State` document (UEF §0.2, confirmed present in all three projects) carrying a version field — not a separate claim |

## 2. Runtime

| Node | Tier | Evidence |
|---|---|---|
| **Request** | 1 | Telegram webhook → Router, universal entry point across every project |
| **Planner** | 2 | Confirmed by Steven: stays here until Maintenance, Investment, or Reminder independently grow their own planner — currently only Reward domain (`getTodayPlan()`) |
| **Decision** | 2 *(was 1 in v0.1)* | `DecisionEngine._buildActions()` is real, but Rider-OS-only — no named equivalent in Personal AI Core or Reminder OS this session. One project, not yet a pattern |
| **User Confirmation** | 3 | No change |
| **Execution** | 1 | Service layer routing through a Truth-equivalent (Rider OS's TruthEngine; Personal AI Core's LockService-protected EventBus writes; Reminder OS's lock-contention self-retry) — same safety property, three independent implementations |
| **Event** | 1 | EventBus / append-to-Sheet-as-event, explicit for Personal AI Core, structurally implied for Reminder OS's Tasks-sheet polling, confirmed for Rider OS |
| **Projection** | 2 | Confirmed by Steven: stays here until at least two Domain OS projects independently form a dedicated Projection layer rather than ad hoc read functions |
| **Query** | 1 | Read functions against the shared Sheet — the general pattern is multi-project (same shared-Sheet-as-truth evidence as Schema); Rider OS's specific `dashboard()`/`getSummary()` naming is its own |

## 3. Intelligence

| Node | Tier | Evidence |
|---|---|---|
| **Knowledge** | 3 | No change |
| **Analytics** | 2 | No change — MaintenanceIntelligence's cost aggregation, Rider-OS-only |
| **Prediction** | 2 *(was 1* in v0.1)* | `getForecast()`/`getTodayPlan()` are real and shipped, but Rider-OS-only, and rule-based arithmetic rather than anything resembling a cross-project pattern yet |
| **Suggestions** | 3 | No change |
| **Insights** | 3 | No change |
| **Learning** | 3 | No change — still nothing resembling ML/adaptive learning anywhere in the ecosystem |

## 4. Integration

| Node | Tier | Evidence |
|---|---|---|
| **Bridge** | 2 | No change — the shared Sheet as de facto bridge, informal |
| **Connectors** | 1 | Telegram Bot API — universal across every project |
| **APIs** | 3 | No change |
| **Import / Export** | 3 | No change |
| **External Systems** | 2 | No change — Telegram is real; Grab is referenced constantly but not integrated with directly |

## 5. Testing

Structure only — UEF §5 owns testing philosophy, not restated here.

| Node | Tier | Evidence |
|---|---|---|
| **Unit Tests** | 1 | `NN_Tests_<FeatureId>` + `runAllXTests()`, confirmed multi-project: Rider OS (5 files, 53 tests, run live) and Reminder OS ("a 39-test suite") |
| **Integration Tests** | — | Deliberately deferred per UEF §5, not a Blueprint gap |
| **Migration Tests** | 2 | Verified via the Migration Plan's own steps (P17), Rider-OS-only so far |
| **Validation** | 2 *(was 1 in v0.1)* | The manual-checklist-in-the-same-test-file convention is confirmed for Rider OS specifically; Reminder OS's 39 tests aren't confirmed to include the same manual-checklist half |

---

## Cross-Cutting Capabilities *(new in v1.2)*

Not a sixth layer. 0-5 above is the Domain Runtime lifecycle; these apply *across* every stage of it rather than belonging to any one, which is exactly why they don't get a numbered slot in that tree. Adding "6. Observability" would open the same door for Security, Configuration, Caching, and anything else that turns out to be widely used — the tree would grow a new top-level branch every time a second common capability got noticed. Kept separate on purpose, so 0-5 stays stable and this list is where cross-cutting capabilities accumulate instead.

```
Governance · Foundation · Runtime · Intelligence · Integration · Testing
────────────────────────────────────────────────────────────────
Cross-Cutting Capabilities: Observability · Security · Diagnostics ·
                             Telemetry
```

Each project adopts whichever it needs — adopting one doesn't require adopting all.

| Capability | Tier | Evidence |
|---|---|---|
| **Observability** | 2 | Rider OS's Observability Layer — EventLogger, HealthMonitor, MetricsEngine, AnomalyEngine, five real files, mapped from its Constitution this session. Real, single-project so far |
| **Diagnostics** | 2 | Rider OS's DiagnosticService (`/diag`) is real and concrete, kept as its own line per your list rather than folded into Observability — though it's worth a look later at whether "on-demand health check" (Diagnostics) and "continuous instrumentation" (Observability) are different enough to justify staying separate long-term, or whether they're the same capability under two names |
| **Security** | 3 | No evidence anywhere — reserved |
| **Telemetry** | 3 | No evidence anywhere — reserved |

**Resolved — Configuration stays out of this list (BP-8).** Foundation's Configuration node (§1) already owns domain configuration (constants, parameters). A cross-cutting "Configuration" would mean something genuinely different — operational/runtime settings like feature flags, log level, experiment switches, rollout control — but no project in the ecosystem has a real, cross-project instance of that yet. Designing it now, even just to give it a slot, would be exactly the speculative architecture UEF's EP3 exists to prevent. When it does show up with real evidence, it gets its own clearly distinct name (e.g. "Operational Configuration" or "Runtime Configuration") rather than reusing "Configuration" — two same-named nodes describing different things would cost every future reader (human or AI) more than waiting costs now.

---

## What changed from v0.1, and why

Applying the stricter "verified in more than one project" bar: **Identity, Decision, Prediction, and Validation move from Tier 1 to Tier 2.** Each of these had real, solid evidence — just from one project. That's exactly the distinction Tier is supposed to capture, so the honest move is downgrading them, not keeping a T1 label that the new definition no longer supports. Nothing here changes what's actually true about Rider OS — only how confidently the *ecosystem-wide pattern* claim is made.

## Decisions

**BP-1 through BP-6: ratified.** BP-1/2/3 per the tables above; BP-5/BP-6 stated inline where they apply (Tier definitions, governing principle).

**BP-4** — explicit promotion trigger for the two nodes Steven called out by name:
- **Projection** → Tier 1 when a second Domain OS project independently organizes read-side functions into a named Projection layer (not just "has read functions" — that's already true everywhere)
- **Planner** → Tier 1 when a second independent domain (Maintenance, Investment, Reminder, ...) builds its own planner alongside Reward's

Neither is close right now — named here so the bar is written down before anyone's tempted to round up early.

**BP-7** *(new)* — cross-cutting capabilities (Observability, Diagnostics, Security, Telemetry) live in their own section, not as a sixth layer in the 0-5 tree — see above. Rationale: a numbered layer per newly-noticed common capability would mean the tree never stops growing; a separate, open-ended list is where those accumulate instead, keeping 0-5 stable long-term.

**BP-8** *(new)* — Configuration does not get added to Cross-Cutting Capabilities. Resolved by reasoning alone, not deferred waiting for a second project's evidence (unlike BP-4's Projection/Planner triggers) — see the resolved note above.

---

## What this version does not do

- Does map Rider OS's `ARCHITECTURE LAYERS` onto this tree now (in `00_Project_Constitution.txt` itself, per BP-6 — project-specific mapping stays in the project, not here) — that exercise is what surfaced Observability in the first place.
- Does not touch UEF v1.2 beyond the ADR recording this decision.
- Does not add design content to any Tier 2 or Tier 3 node, including the two new cross-cutting ones — evidence first, in every direction.
- Does resolve the Configuration question (BP-8) — deliberately excluded, not left speculative.
