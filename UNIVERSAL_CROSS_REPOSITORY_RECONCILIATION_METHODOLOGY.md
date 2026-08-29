# Universal Cross-Repository Reconciliation Methodology

**Status:** DRAFT — EVIDENCE-DERIVED METHODOLOGY
**Source case:** Personal Life OS ↔ Master Cross-Repository Reconciliation,
August 2026
**Companion document:** `UNIVERSAL_OS_GOVERNANCE_AUDIT_METHODOLOGY.md`

---

## 0. Purpose and Scope

This methodology defines a reusable, phase-gated procedure for reconciling a
Domain OS repository with a Master/Universal governance repository **after
the two have evolved independently** — most commonly because governance work
was performed against a snapshot of the OS repository while real,
unrelated domain development continued on the OS side in parallel.

It exists because that exact situation occurred: a Universal-governance
audit and adoption were completed against an uploaded snapshot of Personal
Life OS, after which a fresh upload of the real, live repository revealed
that real domain development had continued in the meantime — including a
genuine identifier collision between a real domain decision and the
governance decision the earlier audit had produced. This methodology is the
generalized form of how that was actually resolved.

This methodology assumes the companion Governance Audit methodology has
already been run at least once against some snapshot of the OS. It governs
what to do when a **later** snapshot no longer agrees with what the
Universal layer currently believes.

---

## 1. Shared Core Invariants

*(Identical to the companion document; repeated here for completeness.)*

1. Read-only before mutation.
2. Source-of-truth before synchronization.
3. Never overwrite authoritative local history merely to resolve a Master
   discrepancy.
4. Never present inference as fact.
5. No scope expansion without explicit authorization.
6. One file at a time for controlled governance mutations.
7. Persist before proceeding to the next mutation.
8. Independent verification is mandatory.
9. Meaningful mutations require checkpoint evidence.
10. Governance adoption does not equal implementation compliance.
11. Reconciliation completion does not equal final ecosystem acceptance.
12. Universal propagation must be selective, not wholesale repository
    copying.
13. Identifier collisions must be resolved semantically, not textually.
14. Historical records must remain historically truthful.
15. Stale references must be corrected without rewriting unrelated valid
    records.
16. A successful audit must not automatically authorize implementation
    changes.
17. STOP means STOP — do not infer authorization for the next phase.

Two invariants are specific to reconciliation and are added here:

18. **Neither repository is presumed current merely by role.** "Master" is
    not automatically newer than a Domain OS, and a Domain OS is not
    automatically newer than Master. Currency is established by reading
    both, not by title.
19. **Collision Resolution Heuristic (bounded — not a universal
    invariant).** When there is strong evidence that one candidate for a
    contested identifier is already materially integrated/depended upon,
    and the competing candidate is demonstrably less integrated or not yet
    integrated anywhere, preserving the materially-integrated identity may
    be treated as a safer working hypothesis, and the less-integrated
    candidate renumbered instead. This heuristic does **not** decide, and
    must not be applied to automatically decide, any case where: both sides
    are materially integrated; both sides have meaningful external
    dependencies; both repositories are independently authoritative over
    the decision in question; the decision dates are equal or ambiguous;
    the evidence conflicts; integration depth and chronology point in
    different directions; or the situation otherwise falls outside this
    methodology's defined OS-owns-domain / Master-owns-governance authority
    split. All such cases **must** escalate to an explicit owner/human
    governance decision — this methodology provides no automatic answer for
    them, and applying the heuristic outside its bounded scope is itself a
    methodology failure (see Phase 3's Failure condition).
20. **Reconciliation does not mean making the repositories identical.** The
    objective is to restore semantic consistency while preserving
    legitimate repository-specific history, authority boundaries, and
    domain ownership. A record or file existing only on one side is not
    automatically a defect, and a difference is not automatically something
    to "fix." Every difference must be resolved into one of: legitimate /
    stale / contradictory / missing / out of scope — "merely different" is
    not itself one of those categories and is not, by itself, a reason to
    change anything.

---

## 2. Phase Definitions

### PHASE 0 — Independent State Reconstruction

- **Entry condition:** A newly-provided or newly-relevant snapshot of the OS
  repository exists, and there is reason to believe it may differ from what
  the Universal layer currently assumes about that OS (e.g., time has
  passed, development was known to be ongoing, or a fresh upload arrives
  after governance work already completed against an older snapshot).
- **Actions:** Treat the OS repository and the Master repository as two
  independent states. Do not diff the new OS snapshot only against the
  Master's records of it — also diff it against whatever *prior* OS
  snapshot the Master-side work was actually performed against, so that
  "what changed, and on which side" can be established precisely rather
  than assumed. Recover, on the OS side: Constitution, decision log, state
  tracker, file map, and any incident/history documents. Recover, on the
  Master side: the manifest and registry entries concerning this OS.
- **Evidence required:** A full-repository, file-by-file diff of the new OS
  snapshot against (a) the prior snapshot the Master-side work assumed, and
  (b) the current OS working copy if one exists from in-progress governance
  work.
- **Exit condition:** A precise inventory: which files are identical across
  all three states, which differ, and — for each that differs — whether the
  difference is explained by governance work already done, by independent
  domain development, or by something not yet accounted for.
- **Failure condition:** Assuming a fresh upload is "just a duplicate" or
  "just newer" without actually diffing it.
- **STOP condition:** After the inventory is reported.
- **Mutation allowed:** No.
- **Evidenced by:** on receiving a fresh Personal Life OS upload, diffing it
  both against the original pre-governance-work snapshot *and* against the
  working copy that had already received governance edits — which revealed
  the fresh upload contained the earlier governance-reconciliation output
  (confirming it had been round-tripped through the real project) plus
  substantial independent domain development the working copy had never
  seen, including four entirely new files.

### PHASE 1 — Bidirectional Comparison

- **Entry condition:** Phase 0 inventory complete.
- **Actions:** For every governance-relevant artifact (Constitution,
  decision log, state tracker, file map, Universal manifest, Universal
  registry), build an explicit comparison. Do not limit this to files the
  Universal-side work touched — check the OS's own file map or equivalent
  for files it registers that the Master side has never seen.
- **Evidence required:** A table: artifact, OS-side content/state,
  Master-side content/state, classification.
- **Exit condition:** Every artifact classified as one of: identical, OS
  newer, Master newer, OS-only, Master-only, stale reference,
  contradiction, missing, or derived summary.
- **Failure condition:** A classification asserted without having actually
  opened both sides' content for that artifact.
- **STOP condition:** After the comparison table is reported.
- **Mutation allowed:** No.

### PHASE 2 — Collision Detection *(mandatory)*

- **Entry condition:** Phase 1 comparison exists.
- **Actions:** For every identifier the Universal-side work assigned or
  relied on (an ADR number, a section number, a file name, a cross-
  reference), check not only whether the *same string* exists on the OS
  side, but what it *means* there. Search for the identifier in every
  plausible form: full canonical format, abbreviated/shorthand format, and
  any format a human author might use in prose. A search for one exact
  string is not sufficient — a collision can be missed entirely if only the
  canonical long form is checked while short-form references exist
  elsewhere. Where a string matches, read the surrounding content and
  determine independently what topic, decision, or event it actually
  refers to on each side before concluding either "same thing" or "true
  collision."
- **Evidence required:** For every match found, the exact location, the
  surrounding context, and an explicit semantic judgment (same referent /
  different referent) with reasoning.
- **Exit condition:** A complete list of every identifier examined, each
  marked either "no collision" or "TRUE COLLISION," with the reasoning
  shown for each.
- **Failure condition:** Concluding "no collision" based on a single
  exact-string search that did not also check abbreviated forms.
- **STOP condition:** After the collision list is reported. Do not resolve
  a collision in the same pass that discovers it.
- **Mutation allowed:** No.

**Canonical real-world case, generalized below:**

A Universal-governance-adoption ADR had been written against an older
snapshot of an OS's decision log, using the next available number at that
time. By the time a fresh snapshot of the same OS arrived, the OS's real,
independent decision log had already used that same number for an
unrelated domain decision — with real code depending on it, decided two
days earlier in wall-clock terms than the governance decision. A first pass
correcting Universal-layer references searched only for the canonical
full-format identifier and appeared to succeed; a subsequent, explicitly
re-run audit that also searched for the shorthand form of the identifier
found two additional occurrences the first pass had missed, in
free-text prose rather than a structured field. Both were confirmed, by
reading their surrounding sentences, to refer to the governance decision,
not the domain one, and were corrected identically.

The general rule this establishes: **collision detection must be repeated
with every plausible textual form of an identifier, and a first
correction pass should not be assumed complete until a second, differently-
targeted search comes back empty.**

### PHASE 3 — Source-of-Truth Arbitration

- **Entry condition:** One or more TRUE COLLISIONs confirmed.
- **Actions:** First check whether the collision falls within the bounded
  scope of the Collision Resolution Heuristic (Invariant #19) — that is,
  whether the evidence clearly shows one candidate materially integrated
  and depended upon, and the other clearly not yet integrated anywhere.
  Relevant evidence includes: which decision has an earlier real-world
  decision date; which decision already has real, integrated dependencies
  (application code, other documents that reference it); which decision has
  already been merged back into the live/authoritative environment versus
  which exists only as output not yet integrated anywhere. Never resolve by
  a positional rule ("Master always wins" or "whichever is newer wins").
  If the evidence clearly fits the heuristic's narrow scope, the
  less-integrated side yields the identifier. If the evidence does **not**
  clearly fit — both sides integrated, both authoritative, dates equal or
  ambiguous, evidence conflicting, or chronology and integration depth
  pointing opposite ways — do not attempt to resolve it here. Present the
  collision, the evidence for each side, and the fact that it falls outside
  the heuristic's bounded scope, and escalate it for an explicit
  owner/human governance decision.
- **Evidence required:** An explicit statement of the deciding factors for
  each collision, citing the actual dates and dependencies found, and an
  explicit statement of whether the collision fits the heuristic's bounded
  scope or must be escalated.
- **Exit condition:** For every TRUE COLLISION, either (a) a stated
  resolution reached via the bounded heuristic, with the supporting
  evidence shown, or (b) an explicit escalation to the owner, with the
  reason the heuristic does not apply.
- **Failure condition:** Resolving a collision by preferring a side because
  of its role (Master vs. OS) or because one file "looked newer," without
  citing the actual dependency/chronology evidence; or applying the
  heuristic to a collision that does not clearly fit its bounded scope
  instead of escalating it.
- **STOP condition:** After the resolution is proposed, before any file is
  touched.
- **Mutation allowed:** No.
- **Evidenced by:** the real due-date decision keeping its number because it
  had a two-days-earlier real decision date *and* real application-code
  dependencies already referencing it, while the not-yet-integrated
  governance decision was renumbered instead — explicitly not because
  Master or the audit process "lost," but because renumbering the
  not-yet-integrated side was strictly cheaper and safer.

### PHASE 4 — Reconciliation Plan

- **Entry condition:** All collisions resolved in principle (Phase 3); all
  other Phase 1 differences classified.
- **Actions:** Produce an explicit KEEP / MODIFY / ADD / EXCLUDE table
  covering every file that could plausibly be touched by the reconciliation,
  including files that will explicitly **not** be touched (state this
  positively, not by omission). For each MODIFY or ADD entry, specify the
  exact scope, the exact reason, the source of truth being followed, and
  anything the change is explicitly prohibited from altering.
- **Evidence required:** The full plan, in enough detail that the owner
  could execute it themselves from the document alone.
- **Exit condition:** Explicit authorization, ideally granted (and to be
  exercised) one file at a time rather than as a single blanket approval.
- **Failure condition:** A plan that would touch domain-implementation files
  merely because they were part of the same upload, when the actual gap is
  purely about governance identifiers.
- **STOP condition:** Immediately after the plan is presented.
- **Mutation allowed:** No.

### PHASE 5 — OS-Side Reconciliation

- **Entry condition:** Explicit, file-scoped authorization for one specific
  OS-side file.
- **Actions:** Reconcile using the **current, real** OS snapshot as the base
  — not an earlier working copy the Universal-side audit had produced,
  even if that working copy is more "familiar." Add the renumbered/
  corrected governance content without touching the real, authoritative
  content that already occupies the contested identifier. Follow the
  companion methodology's `READ → MODIFY → VALIDATE → PERSIST → VERIFY →
  CHECKPOINT → STOP` cycle for each file.
- **Evidence required:** Confirmation that the file used as the base
  matches the real current snapshot (by hash, if the same file was
  previously seen elsewhere), plus the standard diff/hash/scope evidence
  for the change itself.
- **Exit condition:** One file checkpoint per authorized file, explicitly
  confirming the real, pre-existing identifier's content is byte-identical
  before and after.
- **Failure condition:** Reconciling against a stale working copy instead of
  the real current snapshot; or altering, even slightly, the content
  already occupying the contested identifier.
- **STOP condition:** After each file.
- **Mutation allowed:** Yes — exactly one OS-side file per cycle, and never
  a domain-implementation file merely because it was part of the same
  upload.

### PHASE 6 — Master-Side Reconciliation

- **Entry condition:** All required OS-side files (Phase 5) complete and
  independently verified.
- **Actions:** Correct only the specific Universal-layer references
  identified in Phase 2 as stale. Before executing a bulk textual
  replacement, verify — by reading every matching line's context — that
  every occurrence being replaced genuinely belongs to the governance
  decision and none belongs to the domain decision or to an unrelated OS.
  Never perform a replacement across a whole Universal document without
  first confirming, line by line, that no other OS's record and no
  unrelated content contains the same string.
- **Evidence required:** A pre-replacement audit listing every matching
  line, its context, and a KEEP/REPLACE judgment per occurrence; a
  post-replacement count confirming the number of replacements equals the
  number judged REPLACE; and a diff confirming zero other lines changed.
- **Exit condition:** One file checkpoint per Universal file, explicitly
  confirming other OS's records and the canonical Universal rule document
  are untouched.
- **Failure condition:** A replacement count that does not match the
  pre-audited REPLACE count; any change outside the identified lines.
- **STOP condition:** After each Universal file.
- **Mutation allowed:** Yes — exactly one Universal file per cycle, limited
  to the confirmed-stale references.
- **Evidenced by:** the manifest correction, performed only after
  confirming that every one of four matching lines belonged to the
  governance decision and that forty-six unrelated historical lines
  contained no trace of the collided identifier; and the registry
  correction, which required a second, explicitly re-run audit (per Phase
  2's general rule) after an initial exact-string-only pass missed two
  shorthand occurrences sitting in free-text prose.

### PHASE 7 — Bidirectional Verification

- **Entry condition:** All Phase 5 and Phase 6 files complete.
- **Actions:** Independently re-verify, from the actual current files on
  both sides, that: the real, authoritative identifier's content is
  unchanged; the renumbered/corrected identifier is uniquely and correctly
  used everywhere it needs to appear; no reference in either direction
  still points to the pre-collision identifier; the Universal layer's
  summary of the OS's state matches what the OS repository actually
  contains; no domain implementation detail has leaked into the Universal
  layer; no Universal rule text has been duplicated into the OS repository;
  and no status field has been overstated as a result of the
  reconciliation itself.
- **Evidence required:** Fresh hash and diff computations for every changed
  file on both sides; explicit occurrence counts for both the old and new
  identifier, on both sides, in every plausible textual form.
- **Exit condition:** A single PASS / FAIL determination for the whole
  reconciliation, with the supporting evidence table shown.
- **Failure condition:** Any stale reference, in any form, discovered on
  either side.
- **STOP condition:** After the determination.
- **Mutation allowed:** No.

### PHASE 8 — Reconciliation Checkpoint

- **Entry condition:** Phase 7 = PASS.
- **Actions:** Record that cross-repository reconciliation is complete.
  Explicitly state, as three separate facts, that this does **not** mean
  the adopted governance rule is now operationally active, and does **not**
  mean the ecosystem as a whole has received final acceptance.
- **Evidence required:** None beyond what Phase 7 already produced — this
  phase is a statement, not a new audit.
- **Exit condition:** A short, explicit handoff statement naming all three
  states independently.
- **Failure condition:** Collapsing "reconciliation complete" with either of
  the other two states in the same sentence without qualification.
- **STOP condition:** Immediately. Do not begin final ecosystem acceptance,
  another OS's audit, or any further mutation without a separate, explicit
  authorization for each.
- **Mutation allowed:** No.

---

## 3. Evidence Requirements

Same categories as the companion methodology (primary / secondary / derived
/ unverified), with one reconciliation-specific addition:

- **Cross-repository primary evidence** requires reading *both* sides
  directly in the same session — a claim about "what Master currently says"
  based on an earlier turn's summary, without re-reading Master's actual
  current file, is not primary evidence once independent OS-side
  development is known or suspected to have occurred in the interim.

---

## 3.1 Evidence Maturity / Generalization Status

Three levels, identical in meaning to the companion document's Section 3.1:

- **Proven by Current Real-World Case** — actually exercised in this session
  and directly supported by concrete artifacts (diffs, hashes, quoted
  context) from that exercise.
- **Reasonable Abstraction / Provisional** — a reasonable generalization
  from the current case, not yet independently exercised and confirmed
  across multiple OS-repository pairs.
- **Case-Specific** — a concrete filename, identifier, decision date, or
  incident detail belonging to the one real case; not intended for reuse
  as-is.

| Item | Status |
|---|---|
| Invariants 1–18, 20 | Proven by Current Real-World Case |
| Invariant 19 (Collision Resolution Heuristic) | **Reasonable Abstraction / Provisional, and explicitly bounded** — validated only for the one collision pattern actually encountered (one side integrated, one side not); untested for every other case named in its own text |
| Phases 0–2, 4–8 (structure and gating) | Proven by Current Real-World Case |
| Phase 3's bounded-heuristic branch | Proven by Current Real-World Case (the one pattern it covers) |
| Phase 3's escalation branch | Reasonable Abstraction / Provisional — the escalation *path* is a sound default, but has not itself been exercised, since the one real collision fit the heuristic's scope and never needed to escalate |
| Phase 2's requirement to search abbreviated/shorthand forms | Proven by Current Real-World Case — a first pass that searched only the canonical form genuinely missed real occurrences, caught only by a second, differently-targeted search |
| Worked Example's specific identifiers, dates, and file names | **Case-Specific** |

Do not claim cross-OS-pair validation for anything marked provisional above
until it has actually been exercised on further, independent
reconciliations.

---

## 4. Failure / Edge Case Handling

- **OS evolved after the Master-side snapshot:** the default and expected
  case this methodology exists for. Diff against the actual prior snapshot,
  not against assumption.
- **Master evolved independently of the OS** (e.g., another OS's governance
  work, or a Universal rule update, happened in the interim): apply the
  same Phase 0 discipline symmetrically — do not assume the OS side is the
  only one that could have moved.
- **Duplicate identifier (ADR number, section number, or equivalent):**
  Phase 2/3 as above. Resolve by dependency cost and chronology, not by
  repository role.
- **Abbreviated/shorthand references missed by an exact-string search:**
  always re-run collision detection with a second, broader search pattern
  after any first-pass correction, before declaring it complete.
- **Conflicting dates** between an identifier's embedded prefix and its
  actual decision-date field: trust the explicit decision-date field, and
  confirm the prefix's actual meaning (fixed epoch vs. literal date) using
  multiple real entries rather than assuming.
- **Local-only domain decisions:** must never be represented in the
  Universal layer, even summarized, unless a separate task explicitly
  authorizes ecosystem-wide propagation of implementation state.
  Universal-only governance decisions must never be copied verbatim into a
  Domain OS's local documents; reference them instead.
  Reference: see Section 2 of the companion **Universal OS Governance
  Audit Methodology** for the counterpart discipline (only the
  governance-relevant fact — never domain implementation detail —
  travels upward into the Universal layer).
- **Partially persisted changes from an earlier, possibly-interrupted
  session:** re-verify from actual files before trusting a prior
  checkpoint's narrative; a genuinely interrupted session's last claimed
  checkpoint may not have actually persisted.
- **AI-generated proposed state conflicting with real repository content:**
  the real, persisted repository always wins over anything an AI session
  proposed or drafted but that was never actually integrated back into that
  repository.

---

## 5. Anti-Patterns

- "Master wins automatically."
- "Whichever file looks newest wins automatically."
- Renumbering an authoritative, already-integrated historical decision
  merely to make it match a not-yet-integrated one.
- Copying an entire OS repository's implementation state into Master to
  "keep them in sync."
- Copying the Universal rule's full canonical text into an OS's local
  governance document.
- Modifying both an OS-side file and a Master-side file before verifying
  either.
- Treating "reconciliation complete" as equivalent to "final ecosystem
  acceptance."
- Silently fixing an unrelated discovered issue (a stale header, an
  unrelated broken cross-reference) while reconciling something else.
- Continuing to the next phase, or the next file, without an explicit stop.
- Relying on a single exact-string search to declare a collision search
  complete.
- Recording a reconciliation checkpoint for a correction that has not
  actually been persisted and independently re-verified yet.
- Treating this methodology, or Invariant #19's heuristic in particular, as
  an autonomous algorithm capable of silently resolving every genuine
  identifier or governance collision. The methodology guides investigation,
  classifies differences, and narrows the decision space; it does not
  replace owner authority. Final authority over a true governance or
  identifier collision remains a human/owner decision — never
  auto-renumber an authoritative governance record without that explicit
  authorization, and never apply the bounded heuristic to a case outside
  its stated scope instead of escalating it.

---

## 6. Gate Summary

| Gate | Meaning | Mutation allowed after this gate alone? |
|---|---|---|
| `INDEPENDENT STATE PASS` | Both sides' actual current content established | No |
| `COMPARISON PASS` | Full artifact-by-artifact classification complete | No |
| `COLLISION AUDIT PASS` | Every identifier checked in every plausible form | No |
| `ARBITRATION PASS` | Every true collision has an evidenced resolution | No |
| `RECONCILIATION PLAN PASS` | Explicit KEEP/MODIFY/ADD/EXCLUDE table shown | No — requires explicit, file-scoped authorization |
| `OS FILE CHECKPOINT PASS` | One OS-side file reconciled and verified | Only the next authorized file |
| `MASTER FILE CHECKPOINT PASS` | One Master-side file reconciled and verified | Only the next authorized file |
| `BIDIRECTIONAL VERIFICATION PASS` | Both sides independently re-confirmed consistent | No |
| `RECONCILIATION COMPLETE` | Handoff recorded, three states kept separate | No — does not authorize final ecosystem acceptance or another OS |

---

## 7. Worked Example (illustrative, not a template of numbers)

A governance-adoption decision was written into an OS's decision log
against an older snapshot, using the next number available at the time. A
later, fresh snapshot of the same OS's real repository showed that number
had independently been used, in the meantime, for a real domain decision
with genuine code dependencies, decided two days earlier. Comparing the
fresh snapshot against both the original pre-governance snapshot and the
governance-adoption working copy established exactly what had happened on
each side. The collision was confirmed by reading, not assuming: the
"same" identifier meant two unrelated things. Because the domain decision
was already integrated and earlier, and the governance decision was not
yet integrated anywhere, the governance decision was renumbered — its
content otherwise unchanged, with a short note added explaining the
renumbering. The same collision recurred at a second level (a section
number in the OS's status tracker) and was resolved the same way. Master's
two governance-registry files were corrected to reference the new number —
but only after an initial correction pass, which searched for the
canonical long-form identifier only, was re-run with a broader search and
found two additional shorthand occurrences it had missed. A full
bidirectional verification then confirmed: the real domain decision
untouched, the governance decision uniquely and correctly identified
everywhere, no stale reference in any form remaining on either side, and no
domain implementation detail present in the Universal layer. The
reconciliation was recorded as complete — explicitly distinct from a claim
that the governance rule was now in daily operational use, and explicitly
distinct from any ecosystem-wide final acceptance.

---

## 8. Methodology Status

This document is a methodology draft. It does not modify any Universal
Engineering Framework, any OS's governance files, or any Universal
registry. It creates no ADR and records no checkpoint. It has not itself
been audited, adopted, or propagated anywhere.

This methodology is **not** Universal-adopted policy, and has **not** yet
satisfied an evidentiary threshold for ecosystem-wide adoption. Its evidence
base is currently one real reconciliation case (see Section 3.1 for which
specific parts that limits — in particular Invariant #19's heuristic, which
is validated for only one collision pattern out of the several its own text
names). It must not be represented as having been independently validated
across multiple OS-repository pairs.

UEF §0.9 sets a precedent, for Blueprint-level ecosystem changes, of
requiring evidence from at least two independent projects rather than a
single project's design preference. That §0.9 wording governs Blueprint
changes specifically, not methodology documents — it is cited here as an
evidence-threshold precedent and a reason for caution, not as a UEF rule
that already formally governs this methodology's adoption. Nothing in UEF
has been modified or reinterpreted to say otherwise.

**Status:** DRAFT — PROVISIONAL — VALIDATED AGAINST ONE REAL CASE ONLY
**Source case:** Personal Life OS Governance Audit + Personal Life OS ↔
Master Cross-Repository Reconciliation, August 2026
