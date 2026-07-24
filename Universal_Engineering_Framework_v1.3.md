# Universal Engineering Framework (UEF)
## v1.3 — Stable, 2026-07-23

**Status:** Ratified. MINOR bump from v1.2 (§0.9 — Blueprint Change Policy — added; sets the two-independent-projects evidence bar for future Blueprint changes, deliberately stricter than UEF's own MINOR bar). UEF itself is now marked **Stable**: v1.3 changes again only if two or more new projects independently show something needs adjusting — not for polish, and not based on Rider OS alone a second time.
**Scope:** Personal AI Core, Rider OS, Reminder OS, Productivity OS, and every future Domain OS project (Investment OS, News OS, Finance OS, Property OS, Shopping OS, Vehicle OS, Health OS, Inventory OS, ...).
**Rollout:** Rider OS was the pilot (D5) — first project to complete a full Review + Production Readiness Audit under UEF. Personal AI Core, then Reminder OS, retrofit next.

```
Personal AI Ecosystem
├── Universal Engineering Framework    (this document — governs HOW)
├── Universal Domain OS Blueprint      (existing — governs WHAT/runtime)
├── Personal AI Core
├── Rider OS
├── Reminder OS
├── Productivity OS
└── (future) Investment OS, News OS, Finance OS, Property OS,
    Shopping OS, Vehicle OS, Health OS, Inventory OS, ...
```
UEF **governs** every Domain OS project — an authority relationship, not a code dependency. No project "imports" UEF the way it imports a library; UEF sits alongside the Blueprint as a parallel, ecosystem-level layer that no single project's folder contains.

---

## How this was built, and what's verified vs. recalled

Rider OS's actual current files (`Rider-main.zip`) were read directly this session — `00_Project_Constitution.txt` v2.5, `00_Business_Rules.txt`, `00_Project_State.txt`, `00_File_Map.txt`, `P17_Schema_Migration_Plan.txt`. Everything attributed to Rider OS below is checked against that.

Everything attributed to **Personal AI Core** or **Reminder OS** comes from prior session records, not a fresh read this session. Per EP5 below, treat those claims (Reminder OS's ADR sequence, Personal AI Core's P7) as likely accurate but unverified today — worth confirming against their actual repos before they're retrofitted.

v1.0 also incorporates a second, independent review of the v0.1 draft. Evaluated on its merits rather than adopted outright (per standing practice): most of it held up and is folded in below; the one place it made an unverified claim (which specific projects currently need `00_Business_Rules`) is flagged rather than settled, since the conditional trigger in §0.3 resolves that empirically when each project is actually reviewed — it doesn't need to be argued in the abstract now.

### Headline finding (unchanged from v0.1)

You already had roughly 70% of a UEF before this document existed — informal, inconsistent between projects, not written down as a standalone standard. This document's job is mostly naming, standardizing, and closing gaps, not inventing a new methodology from nothing.

### Three real gaps found by reading the actual files

1. **Rider OS has no ADR file.** Closed by this rollout — see `00_ADR_Log.txt`.
2. **No project has a dedicated Review History file.** Still open — created at Rider OS's first UEF Review (next step, not yet done).
3. **Risk tiers disagreed** between established practice (3 tiers) and the original brief (4 tiers). Resolved below (D2).

### The one meta-decision everything below follows

Prevent over-engineering, premature optimization, and speculative design; prefer evolutionary architecture. Applied to the framework itself: every section is split into **Tier 1** (specified now, has real precedent), **Tier 2** (conditional, triggers on a defined condition), and **Tier 3** (named, deferred until something actually needs it). This keeps the framework from being the first thing that violates its own anti-speculative-design rule.

---

## Validation Status *(new in v1.2)*

Every section below is written the same regardless of whether it's been used yet. This table is the honest difference — what Rider OS's completed Review + Production Readiness Audit actually exercised, versus what's still exactly as untested as the day it was drafted.

| Section | Status | Basis |
|---|---|---|
| §0.1 Engineering Principles (EP1-6) | **VALIDATED** | Applied throughout — EP5 specifically caught the ADR-003/ADR-004 bugs before they shipped |
| §0.2 Mandatory Documents | **VALIDATED** | All five exist and were actively read/written for Rider OS this whole cycle |
| §0.3 Conditional Documents | PILOT | Business_Rules confirmed necessary in practice; the Schema/Migration template wasn't freshly exercised (P17's plan predates UEF) |
| §0.5 Definitions (Ready/Done/Production-Ready) | **VALIDATED** | The exact vocabulary organized the whole audit — "pending manual verification" → "Production-Ready" was the tracking model end to end |
| §0.6 AI Engineering Protocol | **VALIDATED** | Followed every session this cycle |
| §0.7 ADR Record Structure | **VALIDATED** | Five real ADRs written against it (UEF-ADR-001, Rider ADR-001-004); no field proved missing or unused |
| §0.8 Version Governance | **VALIDATED** | Actually exercised three times (v1.0→v1.1→v1.2), MINOR-bump-no-ADR rule applied correctly each time |
| §0.9 Blueprint Change Policy | DRAFT — unexercised | Written this version; no actual Blueprint change request has tested the two-project bar yet |
| §1 Initiation | DRAFT — unexercised | No new project or new feature was initiated under UEF this cycle; every P-item predates it |
| §2 Architecture | DRAFT — unexercised | Architecture Gates were never triggered — no layer, ownership, or cross-project dependency changed |
| §3 Contract Design | PILOT | Matches P17's actual pre-UEF pattern; no fresh contract change was made under UEF's own gate this cycle |
| §4 Implementation (UCR1-6) | PILOT | Already-established codebase conventions, continued through this cycle's edits — not exercised on a brand-new feature |
| §5 Validation | **VALIDATED, heavily** | The two-tier split is exactly what the whole audit ran on: 53 automated + 30 manual checks |
| §6 Release (versioning) | **VALIDATED** | v2.5.7→.8→.9 incremented correctly under the stated PATCH rule; Migration/Rollback template not freshly exercised |
| §7 Evolution | PARTIALLY VALIDATED | BUG-07's deletion is a real instance of "delete only once zero-callers is confirmed"; module split/merge unexercised |
| §8 Review Framework | **VALIDATED, heavily** | Full Audit Review ran twice for real; Risk Matrix classified real findings (F1-F7) |
| §9 Audit Framework | **VALIDATED, heavily** | Source of this version's own additions — the Audit Workflow was written FROM this cycle, not before it |
| Knowledge Base | **VALIDATED, growing** | Three new Failure Catalog entries this cycle, each from a real, fixed bug |

Nothing here changes what any section *says* — PILOT/DRAFT tags mean "not yet tested against reality," not "wrong." The tags should update as Personal AI Core and Reminder OS get their own retrofits and initiation/architecture-gate scenarios actually happen.

---

## The 10-part lifecycle

Unchanged from the original brief.

```
0. Engineering Governance   — the constitution: principles, mandatory docs, drift rules
     ↓
1. Initiation               — why a project/feature exists before design starts
     ↓
2. Architecture             — process for architecture work (not the architecture itself)
     ↓
3. Contract Design          — schemas, commands, events as versioned contracts
     ↓
4. Implementation           — coding standards, file/test conventions, AI coding rules
     ↓
5. Validation                — proving correctness (broader than "tests")
     ↓
6. Release                   — versioning, migration, rollback, changelog
     ↓
7. Evolution                 — deprecation, tech debt, module split/merge
     ↓
8. Review Framework           — reusable review workflow + decision matrix
     ↓
9. Audit Framework             — periodic evidence-based health checks
     ↓
Engineering Knowledge Base      — failure catalog, reference architecture, glossary
```

---

## 0. Engineering Governance

### 0.1 — Engineering Principles

- **EP1 — Code → Verify → Governance, never the reverse.** A suggestion, an external AI's proposal, an old doc, or last week's own governance file is a reason to re-check the live code — never a reason to update a governance doc directly.
- **EP2 — Documentation Drift is a defect, not a formality.** Any change to Architecture, Dependencies, Contracts, Interfaces, Modules, Layers, Data Ownership, or Responsibilities MUST trigger a check of whether Constitution / State / File_Map / Business_Rules / ADR / Review History need updating — same session as the code change.
- **EP3 — Anti-premature-engineering.** Build for the requirement in front of you; "might need this later" isn't justification on its own — it earns its own evaluation when that future arrives.
- **EP4 — Single source of truth for every derived calculation.** Two engines independently computing the same number is a defect waiting to happen, not two implementations to keep in sync.
- **EP5 — Empirical verification over inherited authority.** Applies to Claude as much as any external source — a claim's origin doesn't grant it truth. Re-derive from the live sheet/live code before accepting. (Applied to this very document's own Personal-AI-Core/Reminder-OS claims, and to the second review folded in above.)
- **EP6 — No engineering decision by opinion alone.** Non-trivial decisions use the Decision Matrix (§8). "I think X is better" is a hypothesis, not a decision.

### 0.2 — Mandatory Project Documents (Tier 1 — every project, no exceptions)

| Document | Purpose | Currently exists? |
|---|---|---|
| `00_Project_Constitution.txt` | Architecture + coding rules only | ✅ all 3 projects |
| `00_Project_State.txt` | Living changelog: version, in-progress, bugs, next steps | ✅ all 3 projects |
| `00_File_Map.txt` | Module index + common-edit-scenario runbook | ✅ all 3 projects |
| `00_ADR_Log.txt` | One record per architecture-affecting decision | ✅ Reminder OS · 🆕 Rider OS (this rollout) · ❌ Personal AI Core |
| `00_Review_History.txt` | Log of review/audit sessions, separate from the routine changelog | ❌ missing everywhere — created at each project's first UEF Review |

All project-level governance files use **`.txt`**, matching the ecosystem's existing convention — never `.gs`, even for content structured like a module.

### 0.3 — Conditional Documents (Tier 2 — required only when the trigger condition is met)

| Document | Required when... | Precedent |
|---|---|---|
| `00_Business_Rules.txt` | Project has non-trivial domain/business logic distinct from architecture | Rider OS split this out 2026-07-14 |
| Schema/Migration/Rollback/Compatibility Plan | A change touches a live Sheet schema already holding production data | `P17_Schema_Migration_Plan.txt` — the universal template (§3, §6) |

Whether any *specific* project (e.g. Reminder OS) currently meets the `00_Business_Rules` trigger isn't decided here — it's checked against that project's actual current code when it's reviewed, not argued about in the abstract.

### 0.4 — Documentation Drift Trigger List
**Architecture · Dependencies · Contracts · Interfaces · Modules · Layers · Data Ownership · Responsibilities.**

### 0.5 — Definitions

- **Definition of Ready** (before implementation starts): governing Constitution/Business_Rules principle exists, or is being written alongside this change · if a live schema is touched, a signed-off Schema/Migration Plan exists first.
- **Definition of Done** (implementation complete): code implemented · pure-logic unit tests pass · governance docs updated same session (EP2) · Change Impact Analysis reported (§0.6).
- **Definition of Production-Ready** (safe to close): Done, plus the manual verification checklist run once against the live deployment. Until then, status stays `pending`.

### 0.6 — AI Engineering Protocol

1. **Session bootstrap** — before proposing any change, read the project's full Constitution + State + File_Map (+ Business_Rules if present).
2. **Before implementing** — report a Change Impact Analysis: modules affected, contracts affected, tests affected, which governance docs need sync, technical debt introduced, architecture drift, ADR needed?, backward compatibility, migration needed?
3. **Process** — implement → self-check → surface what was found → fix → resubmit.
4. **After implementing** — sync governance docs the same session, never a deferred pass.

### 0.7 — ADR Record Structure *(refined in v1.1 after review — see UEF-ADR-001 / ADR-001 for the applied example)*

Every ADR entry, at either level, uses this shape:

| Field | Notes |
|---|---|
| ID | Sequential per log — `UEF-ADR-001` (ecosystem log) or `ADR-001` (project-local log) |
| Title | One line |
| Status | Proposed / Accepted / Superseded (by ADR-NNN) / Rejected |
| Date | |
| Context | Why this decision was needed. States the problem — doesn't restate detailed history already in State/Review History (EP4, one source of truth) |
| Question | The specific question being decided |
| Options Considered | Listed briefly — full Advantages/Disadvantages/Trade-offs reasoning (§8's Decision Matrix) is supporting material for non-trivial decisions, not boilerplate in every ADR |
| Decision | What was chosen |
| Evidence | What was verified before deciding, and at what confidence — directly verified this session vs. recalled from prior ones (EP5). Specific enough to be falsifiable later; an ADR that goes vague to "age well" loses the thing that makes it trustworthy — that job belongs to Context, not Evidence |
| Impact | What changes as a direct result of the decision. For a foundational/high-stakes entry this can include a short positive-effects/trade-offs note — not a separate mandatory section, since most ADRs are too small to need one (EP3) |
| Next Steps | What happens after, that this decision doesn't itself resolve |
| Related ADRs | Cross-references to ADRs this one follows from, supersedes, or is followed by. Often thin or empty on early entries — becomes valuable as the log grows |
| Review Trigger | The *condition* that should prompt re-evaluating this decision, not a calendar date — e.g. "after the pilot completes," "if UEF governance changes materially" |

Two ADR levels exist: an **ecosystem-level log** (lives in UEF's own directory — decisions about UEF itself, or cross-project matters) and a **project-level log** (`00_ADR_Log.txt` in each project — that project's own architectural decisions, including its own "we adopted UEF" entry).

### 0.8 — UEF Version Governance *(new in v1.0 — Decision D6)*

UEF versions itself the same way §6 versions project releases:
- **PATCH** — wording/clarity fixes, no semantic change
- **MINOR** — new checklist item, a Tier-3 Review Profile or Validation tier activated into Tier 1, a new template — no ADR required, just a note in the ecosystem changelog
- **MAJOR** — a change to the lifecycle itself, a core Engineering Principle, or a Mandatory/Conditional document tier — requires an ecosystem-level ADR

This v0.1 → v1.0 transition is itself the first Version Governance event: several additions (full Risk Matrix, ADR structure, this section) plus D1-D6, but no lifecycle change and no principle removed — so it's the initial ratification, covered by `UEF-ADR-001`, rather than a MAJOR change needing a separate ADR beyond that.

### 0.9 — Blueprint Change Policy *(new in v1.3)*

The Universal Domain OS Blueprint is Stable as of v1.2, ratified through Rider OS's complete Review + Production Readiness Audit and its own architecture mapping. Its own §0.8-equivalent:

- Changes are not made based on a single project.
- A Blueprint modification requires evidence from at least two independent projects, or a demonstrated ecosystem-wide benefit — not a design preference, however reasonable it sounds in the abstract.
- Blueprint evolution stays conservative and evidence-driven, same as this document's own EP5/EP3.

This is stricter than UEF's own MINOR bar (§0.8), deliberately — UEF changes based on lessons from running the process; Blueprint changes reshape what every current and future Domain OS project is built out of, which is a higher bar to clear before touching again.

---

## 1. Initiation

**1a. New Domain OS project**
- Vision/Purpose (one paragraph — matches the existing "SYSTEM IDENTITY" block style)
- Requirements, Constraints, Assumptions, Non-functional Requirements, Success Criteria, Out of Scope
- **ADR-000**: why this is a separate GAS project rather than a module inside an existing one (Domain Ownership, P7-style)
- A lightweight roadmap — bullet points, not a Gantt chart

**1b. New feature inside an existing project (the common case)**
- Maps onto the P-numbering convention proven in Rider OS's `00_Business_Rules.txt` (P15-P25): a new feature gets a principle number, a one-line rule, a status tag (§0.5, §8)
- No separate Initiation document — the P-entry itself, tagged `[PROPOSED]` before acceptance, *is* the initiation artifact

---

## 2. Architecture

Process for architecture work — not the architecture itself. That's the Blueprint's job; each project's Constitution "ARCHITECTURE LAYERS" section is where the Blueprint gets instantiated per-project. CQRS/Event Sourcing/EventBus/layer-naming stay out of this document deliberately.

- **Architecture Gates** — adding/removing a layer, moving module ownership, or changing cross-project dependency direction requires an ADR *before* implementation.
- **Dependency Rules** — since these are independent GAS deployments sharing only a Sheet, a "dependency" means *Project A assumes Project B populates Sheet X*. Every such assumption gets one line in both projects' File_Maps.
- **Ownership check** — before writing a new module, confirm which project owns that domain (Domain Ownership, Personal AI Core's P7). Default: new capability = new file in the project that already owns that domain, unless Initiation says otherwise.

---

## 3. Contract Design

The real "public API" here is **Google Sheet schemas**, **Telegram command syntax**, and **`CONFIG.EVENT`** values.

**Schema Contracts**
- Append-only by default. New fields go at the end; a column position, once assigned, is never reused.
- Any schema change requires: **Schema Proposal → Migration Plan → Rollback Plan → Backward Compatibility Review** — the shape already proven for P17, now the required universal template whenever §0.3's trigger fires.
- Blank/undefined in a new column always means "no data for this record," never silently coerced to 0.

**Command Contracts**
- New commands are additive; new *arguments* to existing commands prefer optional `key:value` tokens over new required positional params (P17's `shop:`/`parts:`/`labour:`/`total:` pattern).
- Command routing order (longer prefixes before shorter) is a Contract-Design-level rule for any project with a text-command router.

**Event Contracts**
- `CONFIG.EVENT` values are append-only; a removed event is marked `[REMOVED]` in File_Map rather than deleted from the historical record.

**Cross-Project Contracts**
- Where two independently-deployed projects read/write the same sheet (Reminder OS polling a shared Tasks sheet), that sheet's schema gets the *strictest* version of this section.

---

## 4. Implementation

**Universal Coding Rules (UCR)** — the subset of Rider OS's CR1-CR13 that isn't Rider-OS-specific. Each project keeps its own locally-numbered CR series for domain-specific rules (RON95-only fuel, per-bike KM isolation stay in Rider OS's own Constitution).

- **UCR1** — IIFE module pattern for every file
- **UCR2** — Private functions prefixed `_`
- **UCR3** — Every public function wraps in try/catch → `AlertService.log(level, module, func, input, msg)` → user-facing error message
- **UCR4** — Date handling: always check `instanceof Date`; never `new Date(timeString)` or `` new Date(`${d}T${t}`) `` alone. (Directly traceable to Rider OS's BUG-2 — the "1899 date" / `NaN:31 AM` failure mode.)
- **UCR5** — Single calculation source (EP4 at implementation level)
- **UCR6** — Sheet appends go through the project's TruthEngine-equivalent, never a raw `sheet.appendRow()`

**File & Test Organization**
- Numbered-prefix-by-layer file naming, all `.txt`
- Test files: `NN_Tests_<FeatureId>.txt`, exposing `runAllXTests()`, each carrying its own **MANUAL VERIFICATION CHECKLIST** for the I/O-dependent half.

**AI Coding Rules**
- One feature fully closes (Implementation → Regression Test → Governance Update → Documentation Update) before the next starts — no parallel development of multiple large features.
- Every edit gets grep/read-verified against its own diff before being called done.

---

## 5. Validation

| Generic tier | Status | Where it lives |
|---|---|---|
| Unit Tests | ✅ Active | Pure-logic functions, `NN_Tests_<FeatureId>.txt` |
| Regression Tests | ✅ Active | Explicit re-verification after every fix |
| Contract Tests | ✅ Active | Backward Compatibility Review (§3) |
| Migration Tests | ✅ Active | Migration Plan step 4 (§3, §6) |
| Scenario / Boundary / Acceptance | ~ Folded in | Manual verification checklist |
| Integration Tests | ❌ Deferred | Sheet/Telegram I/O verified manually — no heavy-mocking suite chasing 100% automated coverage |
| Performance / Security / Recovery Tests | ❌ Deferred | No current trigger on a single-user personal system — revisit if a real need appears (EP3) |

---

## 6. Release

**Versioning:**
- **MAJOR** — Constitution-level architecture change
- **MINOR** — a new Business_Rules principle/feature ships
- **PATCH** — same-day bug fix or sub-feature within a MINOR batch

**Migration/Rollback/Compatibility** = the P17 four-part template, required whenever §0.3's trigger fires.
**Release Checklist** = Definition of Production-Ready (§0.5).
**Change Log** = `00_Project_State.txt`'s versioned `CHANGES` sections — required, not just habit.

---

## 7. Evolution

- **Deprecation, not deletion** — a deprecated function stays as a thin, tagged alias until an ecosystem-wide audit confirms zero remaining callers.
- **Tech debt register** — folds into `00_Project_State.txt`'s `KNOWN BUGS` section, renamed `KNOWN BUGS + TECH DEBT`.
- **Module split/merge** — already happened once (Personal AI Core → Rider OS / Reminder OS / Productivity OS). `92_ReminderEngine.txt`, flagged as possibly stale post-split, is a currently-open Evolution item worth closing under this framework.
- **Long-term roadmap** — per-project roadmap in each State file's `NEXT STEPS`; ecosystem-level roadmap in the Knowledge Base.

---

## 8. Review Framework

**Review Profiles (Tier 1 — defined now, real precedent):**

| Profile | Real precedent |
|---|---|
| Feature Review | Each P-numbered feature (P16, P20, P15, P17, P23-25) |
| Schema/Migration Review | P17 — the only sheet-schema-touching change so far |
| Full Audit Review | Personal AI Core's multi-round security/quality audit |
| Cross-AI Proposal Review | Recurring pattern of evaluating another AI's suggestions against actual code — including this document's own D1-D6 review |

Platform/Connector/Infrastructure/Storage/API/Security-as-its-own-profile etc.: **Tier 3, named, deferred** — zero precedent yet in a 3-project ecosystem. Defined for real the first time a review actually needs one.

**Checklist Library:** Separation of Concerns · Dependencies/Ownership · Contracts · Single-Source-of-Truth (DRY) · Documentation Drift · Architecture Drift · KISS/YAGNI · *(added)* "does this Constitution now contain business rules that should move to `00_Business_Rules.txt`?"

**Review Workflow:**
```
Review Request → Profile Selection → Evidence Collection (empirical
code check, EP5) → Findings → Risk Assessment → Decision Matrix →
Disposition → Gate Decision → ADR (when disposition changes
architecture/contracts/dependencies) → Review History entry
```

**Risk Matrix v1** *(finalized in v1.0 — Decision D2)*

| Tier | Meaning | Criteria | Examples |
|---|---|---|---|
| **CRITICAL** | Stop immediately | Data loss · security breach · irreversible corruption · production failure · an incorrect financial/consequential transaction | A permanent core-data delete · a wrong trade in a future Investment OS · a wrong payment in a future Finance OS |
| **HIGH** | Must fix before continuing | Broken contract · wrong architecture dependency · backward-compat failure · migration with no rollback | P17-style schema change shipped without a rollback plan |
| **MEDIUM** | Can continue, must be logged | Technical debt · missing tests · documentation drift · performance concern | The `92_ReminderEngine.txt` stale-file question |
| **LOW** | Observe | Style issue · minor optimization | — |

CRITICAL is defined narrowly on purpose — if it's used for "big problems" generally, HIGH stops meaning anything.

**Decision Matrix** — Question → Options → Evaluation Criteria → Advantages → Disadvantages → Trade-offs → Recommendation → Decision → ADR. Required shape for any non-trivial call. D3 in the Decision Log below is the worked example.

---

## 9. Audit Framework

**Fields per finding**: Evidence · Risk Level (§8) · Recommendation · Priority · Owner (always Steven, solo dev — trivial today, kept for template consistency) · Verification (re-check after fix).

**Audit types actually used**: Security + Code Quality (Personal AI Core). Production Readiness (Rider OS, new — see below). Architecture / Governance / Data / AI / Compliance / Performance audits are named, available — not yet triggered.

**Production Readiness Audit Workflow** *(new in v1.2 — validated through Rider OS's first full run, not designed ahead of the evidence)*:
```
Per pending item: Automated tier check (already passing, confirmed
  once at audit start) → Manual checklist run → Bug found?
    → yes: ADR (Evidence/Impact per §0.7) + fix → re-verify the
      SAME item against the fix → clean? → item closes
    → no: item closes
  → next item → all items closed → Audit complete, Review History
    entry records the whole run
```
Ran for real across five items (Rider OS, 2026-07-20): 53 automated + 30 manual checks, three real bugs found this way and fixed (ADR-002, ADR-003, ADR-004) — each one in how a function was *called* against live state, not in logic the automated tier already covered. That gap is exactly why both tiers stay mandatory (§5) rather than treating passing unit tests as sufficient.

---

## Engineering Knowledge Base

**Failure Catalog**

| Lesson | Source |
|---|---|
| Never `new Date(timeString)` or `` new Date(`${d}T${t}`) `` alone in GAS | BUG-2, Rider OS — "1899 date" / `NaN:31 AM` |
| A module-local "last known X" lookup that only checks its own sheet will eventually be wrong | BUG-11, Rider OS — 622km false gap |
| Read the override source (e.g. a per-entity profile sheet) before the global config fallback | BUG-4, Rider OS |
| An append-key operation needs duplicate-key protection at the batch level, not just per-row | `batchUpsertRowsByKey_`, Personal AI Core |
| A checkpoint bypass on one status branch can silently skip validation other branches get | `_syncDailyShifts_`, Personal AI Core |
| A private helper used only internally still needs to be in the module's export list, or external callers (incl. test files) can't see it | `_getSessionRemainingMins`, Rider OS |
| A function meant to fire on one specific day via an automatic trigger, but also manually testable on any day, needs date math computed relative to whatever day it actually runs on — not a fixed offset that assumes the trigger day | ADR-003, Rider OS — P20's WeekStart showed the wrong date when manually tested on a non-trigger day |
| A "current record" selector that filters by a status flag needs that flag actively closed out on the superseded record when a new one is written, or more than one record can satisfy "current" at once | ADR-003, Rider OS — old weekly reward configs stayed marked active indefinitely |
| A rolling aggregate used to derive a smaller-scope target must exclude that smaller scope's own still-accumulating data, or progress-in-the-small silently shrinks the target-in-the-small | ADR-004, Rider OS — logging today's own orders was shrinking today's own daily target |

**Reference Architecture** — the proven layer stack (Truth → State → Event → Service → Observability → Operational Intelligence → Reminder → Intelligence-stub) is the default starting template for a new Domain OS project, unless Initiation documents a reason to deviate.

**Glossary** — *Truth Layer*, *Single Calculation Source*, *Domain Ownership*, *Documentation Drift*, *Append-only Contract* — defined once at ecosystem level, referenced from each project.

---

## Decision Log — D1-D6 (ratified 2026-07-18)

### D1 — Where does UEF physically live?
**Decision:** Independent docs-only directory, no GAS deployment, no project "imports" it — UEF governs, projects don't depend on it as code. Each project's Constitution gets a one-line pointer to it.

### D2 — Risk tiers
**Decision:** 4 tiers (CRITICAL/HIGH/MEDIUM/LOW) — full definition in §8. CRITICAL kept deliberately narrow so it doesn't dilute HIGH.

### D3 — Is `00_Business_Rules` mandatory or conditional? *(worked example, per §8's Decision Matrix)*
**Question:** Should every project be required to have `00_Business_Rules`, even nearly empty?
**Options:** (a) Mandatory everywhere · (b) Conditional — created once a project accumulates non-trivial domain/business logic distinct from architecture
**Advantages of (a):** every project looks identical on day one
**Disadvantages of (a):** an empty mandatory file on an infrastructure-shaped project is the exact speculative-design smell EP3 exists to prevent
**Advantages of (b):** matches what actually happened — Rider OS split it out organically, the moment it needed it
**Disadvantages of (b):** a project could accumulate business-shaped logic inside its Constitution for a while before anyone notices — mitigated by the new Checklist Library item (§8)
**Decision:** (b), conditional. Which project needs it, right now, isn't argued in the abstract — it's checked against actual code at that project's review.

### D4 — Retrofit history, or only enforce going forward?
**Decision:** Forward only. Rider OS's `00_ADR_Log.txt` starts at ADR-001 ("Adopt UEF"), not a reconstruction of past decisions. Backfilling a fake decision trail (e.g. writing "ADR-001: we decided to use EventBus" today, when no such formal process existed at the time) would pollute the governance record rather than document it. The 2026-07-14 governance review is well-documented enough in State's changelog to be worth copying into Review History as a first retroactive entry once that file is created; older history generally isn't.

### D5 — Rollout order
**Decision:** Pilot on Rider OS. Richest, most recent governance activity to validate against — event architecture, schema migration, business rules, testing, a live command interface, maintenance history. Personal AI Core and Reminder OS retrofit once it proves out.

### D6 — UEF Version Governance *(added this round)*
**Decision:** PATCH/MINOR/MAJOR scheme per §0.8. MAJOR changes require an ecosystem-level ADR; MINOR/PATCH don't.

---

## Format note

This document is Markdown, not the `.txt`-comment-block style project-level meta docs use. Deliberate, not drift: `00_Project_Constitution.txt` etc. live inside each GAS project, wrapped in `/* */` so they sit alongside real code inside the Script Editor. UEF doesn't need to — governance docs only, no runtime code — so it doesn't inherit that constraint. All project-level governance files, and any other file this framework produces for a specific project, use `.txt`, never `.gs`.

## What this rollout did and did not do

- Did not touch the Universal Domain OS Blueprint's content — §2 is process-only.
- Did not write any implementation code, migration script, or module file.
- Did not retroactively rewrite any existing governance file.
- Did not fully specify the deferred Review Profiles or Validation tiers — zero real precedent yet, so they stay named and available (EP3).
- Did produce two adoption ADRs (`UEF_ADR_Log.md`, `00_ADR_Log.txt`) as the concrete first application of §0.7, then refined the template itself (Impact/Next Steps split, Review Date → Review Trigger, Related ADRs added) after review, before treating either as locked — a same-day v1.0 → v1.1 MINOR bump.
- Did not yet run Rider OS's first UEF Review, backfill its Review History, or start its Production Readiness Audit — proposed as the next step, not started here.

---

Next: Rider OS's first full UEF Review (Review Framework, §8) — evidence collection against the live project, findings, risk assessment, disposition, then backfilling Review History and starting the Production Readiness Audit that closes out P16/P20/P15/P17's `pending` status. That's a substantive pass on its own — say when to start it.
