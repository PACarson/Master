# Universal OS Governance Audit Methodology

**Status:** DRAFT — EVIDENCE-DERIVED METHODOLOGY
**Source case:** Personal Life OS Governance Audit, August 2026
**Companion document:** `UNIVERSAL_CROSS_REPOSITORY_RECONCILIATION_METHODOLOGY.md`

---

## 0. Purpose and Scope

This methodology defines a reusable, phase-gated procedure for auditing a single
Domain OS against a Universal Governance framework (a UEF-style document plus a
cross-project registry/manifest), and — where authorized — for bringing that
Domain OS into local compliance with a specific Universal rule.

It was not designed in the abstract. It is a direct generalization of the
Personal Life OS audit actually performed in this session, including the
mistakes discovered and corrected along the way. Where a phase exists only
because a real problem was found, that provenance is noted.

This methodology governs **auditing and adopting inside one repository**. It
does **not** govern reconciling two independently-evolved repositories against
each other — that is the companion methodology's job. The two are meant to be
used together: this one first (per OS), the other whenever a repository is
found to have diverged from what a Universal audit assumed about it.

This methodology is distinct from UEF §9's existing Audit Framework. UEF §9
concerns a different class of audit — security, code quality, and production
readiness of a codebase. This methodology concerns a different subject
entirely: whether a Domain OS's *governance* (its Constitution, decision log,
and adoption of Universal rules) is current, consistent, and correctly
represented — governance-rule compliance and governance-state consistency,
not code quality.

---

## 1. Shared Core Invariants

*(This section is intentionally identical in both methodology documents.)*

1. **Read-only before mutation.** Every audit begins with a phase in which no
   file may be modified. Findings must be reported before any authorization
   to change anything is assumed.
2. **Source-of-truth before synchronization.** Establish what is actually true
   in the actual persisted files before comparing, propagating, or announcing
   any status.
3. **Never overwrite authoritative local history merely to resolve a
   discrepancy.** If a Universal-layer record and a local record disagree,
   the default assumption is that the Universal-layer record is the one that
   went stale, not the local one — because local history is usually the
   product of real, already-executed decisions.
4. **Never present inference as fact.** A conclusion drawn from pattern-matching
   or absence of evidence is not the same as a conclusion drawn from reading
   the actual current file. Label the difference.
5. **No scope expansion without explicit authorization.** Discovering an
   unrelated defect (a stale file, a broken cross-reference, an inconsistent
   header) is a *finding to report*, not an invitation to fix it in the same
   pass.
6. **One file at a time for controlled governance mutations.** Never modify
   two governance files and then persist both together. Each file gets its
   own complete cycle.
7. **Persist before proceeding to the next mutation.** A change that exists
   only in a working copy or container is not yet a change.
8. **Independent verification is mandatory.** A tool reporting "write
   succeeded" is not verification. Reading the persisted artifact back and
   comparing it — by diff and by hash — is verification.
9. **Meaningful mutations require checkpoint evidence.** A checkpoint is a
   claim backed by a specific diff, a specific hash, and a specific persisted
   path — not a narrative summary.
10. **Governance adoption does not equal implementation compliance.** A rule
    being formally adopted (an ADR accepted, a Constitution reference added,
    a status record written) is a governance fact. Whether people or AI
    sessions actually follow that rule day to day is a separate, later,
    empirically-observed fact. The two must never share one status field.
11. **Reconciliation completion does not equal final ecosystem acceptance.**
    Closing out a specific piece of work is not the same as declaring the
    whole ecosystem consistent. Only the latter authorizes moving on
    unconditionally.
12. **Universal propagation must be selective, not wholesale.** Only the
    governance-relevant facts about an OS belong in the Universal registry.
    Domain implementation detail does not travel upward merely because it
    exists.
13. **Identifier collisions must be resolved semantically, not textually.**
    Two things can share a spelling and mean different things. Read what an
    identifier actually refers to before treating a string match as a
    content match.
14. **Historical records must remain historically truthful.** A checkpoint
    that was correct when written, against the information available at the
    time, should not be silently rewritten as though the earlier state never
    existed. If a later correction is needed, add a correction, don't erase
    the record of what happened.
15. **Stale references must be corrected without rewriting unrelated valid
    records.** A narrow, surgical correction of exactly the wrong reference
    is the goal — not a broad rewrite of the file containing it.
16. **A successful audit does not automatically authorize implementation
    changes.** "The audit passed" and "you may now modify the files" are two
    separate authorizations, even when the same person will grant both.
17. **STOP means STOP.** A phase boundary is not a suggestion. The absence of
    an explicit instruction to continue is not authorization to continue.

---

## 2. Phase Definitions

Each phase below specifies: entry condition, actions, evidence required, exit
condition, failure condition, STOP condition, and whether mutation is allowed.

### PHASE 0 — Recovery / Scope Preflight

- **Entry condition:** A request to audit or continue auditing a Domain OS
  against Universal Governance.
- **Actions:** Identify exactly which OS, which repository snapshot, and
  which version is under discussion. Read the Universal cross-project
  registry/manifest to recover what has *already* been verified for this and
  other OS's. Do not treat conversational summary or memory of prior turns as
  sufficient — re-read the actual persistent registry.
- **Evidence required:** The actual current content of the Universal registry
  and manifest, read directly, not recalled.
- **Exit condition:** A clear statement of what is already-verified
  ("ALREADY VERIFIED — NO ACTION REQUIRED") versus what remains open, for
  every OS mentioned, based on the actual files rather than an assumed
  narrative.
- **Evidence-tier caution:** Cross-checking against the Universal registry or
  manifest is *Secondary evidence* for whatever OS that record describes — it
  confirms what the registry/manifest currently claims, not what that OS's
  own actual files currently say. Reopening that OS's own authoritative files
  directly is *Primary evidence*. Do not report or imply that an OS was
  freshly audited in this phase when only its registry/manifest
  representation was inspected; state plainly which of the two occurred.
- **Failure condition:** The registry and the narrative disagree in a way
  that cannot be resolved by re-reading — report the conflict rather than
  picking a side.
- **STOP condition:** After reporting current state, before any file is
  touched.
- **Mutation allowed:** No.
- **Evidenced by:** the session's own opening recovery audit, which
  cross-checked five previously-processed OS's *registry and manifest
  entries* (Secondary evidence for those five OS's — their own files were not
  reopened in that pass) against each other for internal consistency before
  touching anything, and explicitly flagged that a registry's own explanatory
  note can itself go stale relative to the table it explains (a real,
  low-priority finding surfaced and correctly *not* acted on outside its
  authorized scope).

### PHASE 1 — Authority / Source-of-Truth Discovery

- **Entry condition:** Phase 0 complete; a specific OS is selected for audit,
  and its repository is actually available (not merely referenced).
- **Actions:** Identify the OS's own governance mechanism — which document is
  its Constitution, which is its decision log (ADR or equivalent), which is
  its implementation-state tracker, which is its file/module map. Do not
  assume this OS's mechanism mirrors another OS's. Determine local
  identifier/numbering conventions by reading actual recent entries, not by
  assuming a pattern — for example, confirm whether an ID's embedded date is
  a literal decision date or a fixed epoch prefix by comparing several real
  entries' *separate* decision-date fields.
- **Evidence required:** Direct reads of the governance files' own headers
  and recent entries.
- **Exit condition:** A stated authority model for this specific OS: which
  file is the governance root, which files carry decision-level authority,
  which carry detailed/specialized authority, which carry implementation
  status.
- **Failure condition:** The repository does not actually exist yet for this
  OS — in that case classify as *PENDING — REPOSITORY NOT AVAILABLE* and do
  not invent its governance structure from the registry entry alone.
- **STOP condition:** After the authority model is stated.
- **Mutation allowed:** No.
- **Evidenced by:** the numbering-convention discovery for Personal Life
  OS's ADR log (confirming a fixed ID-prefix versus a真实 per-entry decision
  date by checking three consecutive recent entries rather than assuming).

### PHASE 2 — Constitution Preflight

This phase is split into two layers with different obligations. A future
executor must not read this split as permission to skip the whole phase —
Layer A is mandatory for every OS, unconditionally.

#### Layer A — Mandatory Lightweight Constitution Preflight

- **Entry condition:** Phase 1 complete. A candidate governance-root document
  (Constitution or equivalent) has been identified.
- **Actions:** Perform a lightweight check, for every OS, regardless of
  whether any problem is suspected. At minimum verify: (1) the document's own
  self-identification — does it actually name this OS, not another one; (2)
  its declared version/status fields; (3) whether it appears to belong to the
  correct OS at all, checked by content, not filename or path; (4) whether
  the governance baseline it declares is internally plausible against a
  quick cross-check of one or two sibling governance files' own version
  markers. This is a preflight check, not the full reconciliation — it should
  be cheap.
- **Evidence required:** Direct reads of the candidate document's own
  self-identification and version fields, plus at least one sibling file's
  equivalent fields for comparison.
- **Exit condition:** Either "no anomaly found, proceed to Phase 3," or an
  anomaly is identified and Layer B is triggered.
- **Failure condition:** Skipping this layer because no problem is
  suspected, or relying on filename/path instead of the document's own
  content to confirm OS ownership.
- **STOP condition:** After Layer A's finding is reported — either the clean
  result or the anomaly.
- **Mutation allowed:** No.
- **Evidence-maturity caution:** This lightweight preflight is currently
  *provisional* — see Section 3.1. It is a reasonable abstraction from one
  case (see Layer B's "Evidenced by," below) in which skipping an equivalent
  check would have caused a wrong-file error and a stale-version error to go
  undetected. It has not yet been exercised, and found to be worth its cost,
  across multiple independent OS audits. Future audits should continue
  applying it and note whether it earns its keep on OS's where nothing turns
  out to be wrong.

#### Layer B — Conditional Deep Constitution Reconciliation

*This layer is genuinely conditional: it activates only when Layer A finds an
anomaly — drift, mismatch, a stale version, a wrong-content file, unclear
authority, or other evidence requiring deeper investigation. On an OS where
Layer A finds nothing, Layer B does not run, and that is the expected,
correct outcome — not a shortcut.*

- **Entry condition:** Layer A found an anomaly.
- **Actions:** Cross-check the governance root's own self-declared version,
  name, and date against every other file's self-declared version. Do not
  trust a single file's header in isolation — check whether *sibling*
  governance/architecture files that should track the same evolution
  actually do, and whether any of them explicitly disclaim redefining what
  the root document already defines. If a genuine version gap is found,
  determine precisely which provisions of the stale root are still valid
  (unchanged principles), which have been extended, which have had their
  detail moved to a more specific document, and which are simply outdated —
  using direct textual evidence for each classification, not assumption.
- **Evidence required:** Full reads of the governance root and of every file
  claimed to be "current," with specific citations (quoted phrases, explicit
  cross-references) supporting each classification.
- **Exit condition:** A scoped, minimal, additive reconciliation proposal
  (append a bridge, do not rewrite history) is presented and explicitly
  authorized before Phase 3 begins.
- **Failure condition:** The uploaded/available governance-root file does not
  even belong to this OS (wrong file entirely) — this must be caught by
  checking for the OS's own name/identity inside the file, not merely by
  filename or file path. Treat a mismatch here as blocking, not cosmetic.
- **STOP condition:** After presenting the reconciliation proposal, before
  writing it; and again after each individual file-level persistence step if
  the reconciliation itself requires multiple files.
- **Mutation allowed:** Only after explicit authorization, and using the
  Phase 6 file-discipline pattern below.
- **Evidenced by:** the Personal Life OS Constitution being found, on first
  upload, to contain a different OS's completed governance content entirely
  (a wrong-file error, caught by checking self-identification inside the
  file rather than trusting the filename); and, after correction, being
  found to be a genuinely older architectural snapshot (v4.7) than the rest
  of the repository (v5.1/v5.2) — resolved by an additive "bridge" section
  that preserved the entire historical text and pointed forward to the
  documents that now carry current detail, rather than by rewriting the
  Constitution.



### PHASE 3 — OS-N Read-Only Governance Audit

- **Entry condition:** The governance root is confirmed current (directly, or
  via a completed and accepted Phase 2 reconciliation).
- **Actions:** Read the OS's actual governance and decision-log files in
  full. Search — comprehensively, using multiple related terms, not one
  exact phrase — for existing local mechanisms that might already achieve
  what the Universal rule requires, under different terminology. Distinguish
  a superficially similar term (a quality/acceptance gate) from the actual
  concept being audited (a persistence/recovery discipline) by reading what
  each term is actually used to mean in context, not by string similarity.
  Look for direct local evidence — a real past incident, a real gap, a real
  cost already paid — rather than treating the audit as a purely abstract
  compliance check.
- **Evidence required:** Grep-level terminology sweeps across the whole
  repository (not just the "expected" files), plus close reading of any
  incident/history documents that surface.
- **Exit condition:** A concrete classification of each relevant Universal
  requirement against this OS's actual state: not adopted, adopted under
  different terms, partially adopted, or already fully covered.
- **Failure condition:** A terminology hit that turns out, on inspection, to
  be an unrelated coincidence (same word, different meaning) — report this
  explicitly as a non-collision rather than silently ignoring it.
- **STOP condition:** After the audit findings are reported.
- **Mutation allowed:** No.
- **Evidenced by:** the full-repository terminology sweep for
  "checkpoint / persist / export / recovery / verified / container reset"
  across Personal Life OS, which found zero hits for the exact concept but
  did find a real, previously undocumented incident (a genuine container
  reset that had already cost real work) sitting in an incident-report file
  that was not part of the "expected" governance file list; and the correct
  determination that the OS's own quality-gate mechanism was a different
  axis entirely from the persistence discipline being audited, despite both
  using overlapping vocabulary.

### PHASE 4 — Evidence / Collision Analysis

- **Entry condition:** Phase 3 findings exist.
- **Actions:** For every provision or mechanism examined, assign exactly one
  status from a fixed vocabulary (e.g. `STILL CURRENT`, `CURRENT + EXTENDED`,
  `MODIFIED`, `SUPERSEDED`, `AUTHORITY TRANSFERRED`, `UNCLEAR`). These six
  labels are an example of a sufficiently precise, non-overlapping
  vocabulary — they are not a mandatory Universal lexicon. A different OS's
  governance culture may use its own equivalent controlled vocabulary; the
  actual requirement is precision and non-overlap (every label means one
  thing, and no two labels can both apply), not these exact six words. Do not
  collapse distinct meanings into one label merely for convenience — a
  provision whose detail moved to a more specific document is not the same
  as a provision that was proven wrong. Resolve `UNCLEAR` items using
  repository evidence before asking the owner to resolve them; only escalate
  what evidence genuinely cannot settle.
- **Evidence required:** A direct quotation or citation supporting every
  status assigned.
- **Exit condition:** Zero remaining `UNCLEAR` items without either a
  resolution or an explicit, justified escalation.
- **Failure condition:** A status assigned without a specific supporting
  citation — treat this as insufficiently evidenced and re-derive it.
- **STOP condition:** After the classification table is complete.
- **Mutation allowed:** No.
- **Evidenced by:** the resolution of three initially-`UNCLEAR` items (an
  engineering-standard document, a structural template, and a
  process-rhythm decision) purely from direct quotations already present in
  sibling governance files, without needing to ask the owner.

### PHASE 5 — Governance Adoption Proposal

- **Entry condition:** Phase 4 complete; a genuine gap has been identified
  (not merely a terminology difference).
- **Actions:** Draft the exact proposed change: which file(s), which exact
  section, current text, proposed new text, the reasoning, which existing
  decisions it must reference (and how, without duplicating them), and
  which existing decisions it must explicitly *not* be confused with.
  Explicitly list which files this proposal would touch and, just as
  importantly, which files it would deliberately not touch.
- **Evidence required:** The full draft text of the proposed change, ready
  for review — not merely a description of what it will contain.
- **Exit condition:** Explicit authorization to proceed, file by file.
- **Failure condition:** A proposal that touches more files than the gap
  requires, or that would duplicate content already authoritative elsewhere.
- **STOP condition:** Immediately after presenting the draft. No file may be
  touched in the same turn the draft is presented.
- **Mutation allowed:** No.

### PHASE 6 — Controlled File-Level Implementation

- **Entry condition:** Explicit, file-scoped authorization received.
- **Actions:** For the one authorized file:
  `READ → MODIFY → VALIDATE → PERSIST → INDEPENDENTLY VERIFY → CHECKPOINT →
  STOP`. Validation includes: structural integrity (e.g. balanced
  delimiters), that the change is a pure, minimal addition unless a
  correction was explicitly authorized, that nothing outside the intended
  scope changed, and that any identifier or numbering used is confirmed
  correct against the actual current repository state (not assumed from an
  earlier snapshot). Persistence means writing the result to a location the
  owner can actually retrieve, not merely leaving it in a working
  container. Independent verification means re-reading the persisted copy
  from that location and comparing its hash/diff against the working
  version, freshly computed, not reused from an earlier step.
- **Evidence required:** A pristine-vs-current diff showing zero unintended
  deletions or edits, a hash computed at least twice independently
  (working copy and persisted copy), and a full-repository scope check
  confirming no other file changed.
- **Exit condition:** The checkpoint report, covering the exact diff, the
  hash, the persisted location, and the scope-check result.
- **Failure condition:** Any unexpected diff outside the intended scope, or
  any hash mismatch between working and persisted copies.
- **STOP condition:** After every single file, unconditionally, regardless
  of how many files the overall proposal contains.
- **Mutation allowed:** Yes — exactly one file per cycle.

### PHASE 7 — Governance Verification

- **Entry condition:** All files from Phase 5's proposal have completed
  Phase 6.
- **Actions:** Perform a fresh, read-only, cross-file consistency check —
  re-derive each claim from the actual current file content rather than
  from memory of having written it. Explicitly verify each of: Rule Defined,
  Rule Inherited, Rule Locally Adopted, File-level Verified, Overall
  Governance Verified, Implementation Checkpoint (or equivalent operational)
  System Active. The last of these must be verified as still `Pending`
  unless independent operational evidence — actual observed day-to-day
  compliance — exists. Do not infer it from the mere existence of the
  adoption.
- **Evidence required:** Re-computed hashes, re-read cross-references,
  explicit confirmation that no forbidden implication is present in the new
  text (e.g. a document claiming a specialized file "replaces" the
  governance root globally, when it should only carry authority for the
  specific detail that moved).
- **Exit condition:** A single PASS / FAIL / PASS-WITH-NON-BLOCKING-
  OBSERVATIONS determination, stated explicitly.
- **Failure condition:** Any of the six states above cannot be independently
  re-derived from the current files.
- **STOP condition:** Immediately after the determination.
- **Mutation allowed:** No.

### PHASE 8 — Universal Propagation

- **Entry condition:** Phase 7 = PASS, and explicit authorization to
  propagate.
- **Actions:** Update only the Universal-layer artifacts that actually
  represent cross-project status (a recovery/checkpoint manifest, an OS
  registry) — and only the specific fields/rows concerning this OS. Reuse
  the exact terminology and status vocabulary already established for other
  OS's; do not invent new labels. Never copy domain-specific implementation
  detail into the Universal layer merely because it exists in the OS
  repository.
- **Evidence required:** A before/after diff of each Universal file showing
  only the intended row/section changed, plus a full-repository scope check
  of the Universal layer confirming no other OS's record and no canonical
  Universal rule document changed.
- **Exit condition:** Checkpoint report per Universal file, one file at a
  time, matching the Phase 6 discipline.
- **Failure condition:** Any change to another OS's row, or to the canonical
  Universal rule document itself, merely because this OS adopted it.
- **STOP condition:** After each Universal file, and again after the last
  one, before treating anything as ecosystem-final.
- **Mutation allowed:** Yes — exactly one Universal file per cycle, and only
  the rows/sections concerning the OS under audit.

### PHASE 9 — Final Read-Only Verification

- **Entry condition:** Phase 8 complete for all authorized Universal files.
- **Actions:** Independently re-verify, from the actual current files, every
  claim made across the whole audit: that only the expected files changed in
  both the OS repository and the Universal layer, that no stale reference
  remains anywhere reachable, that the governance state is internally
  consistent across all touched documents, and that the implementation
  status has not been overstated anywhere.
- **Evidence required:** Fresh, independently recomputed hashes and diffs
  for every changed file; a fresh full-repository and full-Universal-layer
  scope check against the true original state (not against the
  immediately-preceding step).
- **Exit condition:** An explicit table distinguishing **Governance Audit
  PASS** from **Implementation Active** from **Final Ecosystem Acceptance**
  — the three must never be reported as a single combined status.
- **Failure condition:** Any of the three states above cannot be
  independently supported by the current files.
- **STOP condition:** After the final table. Do not begin another OS, and do
  not treat this as ecosystem-wide acceptance, without a separate, explicit
  authorization for each.
- **Mutation allowed:** No.

---

## 3. Evidence Requirements

- **Primary evidence:** the actual current content of a persisted file, read
  directly in the same turn the claim is made; a freshly computed hash; a
  diff against a freshly re-derived pristine baseline.
- **Secondary evidence:** an earlier turn's verified checkpoint report,
  *when* the file in question has not been touched since — confirmed by
  re-reading, not assumed.
- **Derived evidence:** a conclusion reached by cross-referencing two or more
  primary sources (e.g. inferring an ID-numbering convention from several
  entries' dates). Always label this as derived, and show the sources.
- **Unverified claim:** anything asserted by a prior conversational summary,
  a memory note, or a registry entry that has not been checked against the
  actual current file in this session. Treat as a hypothesis to verify, not
  as a fact.

Do not require a hash for every conceptual judgment (e.g. "is this
provision still philosophically valid") — use hashes and diffs where they
are the actual mechanism of proof (file identity, persistence, scope), and
textual citation where the question is one of meaning.

---

## 3.1 Evidence Maturity / Generalization Status

This section classifies this methodology's own phases and rules by how
generalized they actually are, so a future reader does not mistake a
single-case abstraction for an established universal. Three levels:

- **Proven by Current Real-World Case** — the procedure was actually
  exercised, in this session, and is directly supported by concrete
  artifacts (diffs, hashes, quoted text) from that exercise.
- **Reasonable Abstraction / Provisional** — a reasonable generalization
  from the current case, not yet independently exercised and confirmed
  across multiple OS's.
- **Case-Specific** — a concrete filename, identifier, incident, number, or
  date belonging to the one real case; not intended for reuse as-is.

| Item | Status |
|---|---|
| Invariants 1–9, 14–17 | Proven by Current Real-World Case |
| Invariant 10 (governance ≠ implementation) | Proven by Current Real-World Case |
| Invariant 11 (reconciliation ≠ final acceptance) | Proven by Current Real-World Case |
| Invariant 12 (selective propagation) | Proven by Current Real-World Case |
| Invariant 13 (semantic collision resolution) | Proven by Current Real-World Case |
| Phases 0, 1, 3, 5–9 (structure and gating) | Proven by Current Real-World Case |
| Phase 2, Layer A (mandatory lightweight preflight) | **Reasonable Abstraction / Provisional** — see Layer A's own note above |
| Phase 2, Layer B (deep reconciliation procedure) | Proven by Current Real-World Case |
| Phase 4's controlled-vocabulary *principle* | Proven by Current Real-World Case |
| Phase 4's specific six labels | **Case-Specific** — an example, not a required lexicon |
| Worked Example's specific file names, dates, numbers | **Case-Specific** |

Do not claim cross-OS validation for anything marked provisional above until
it has actually been exercised on further, independent OS audits.

---

## 4. Anti-Patterns

- Treating "the newest-looking file" as automatically authoritative.
- Assuming a governance-root document's filename or path guarantees its
  content — always verify self-identification inside the file.
- Searching for a collision or a gap using only one exact string when
  abbreviated, shorthand, or reworded forms of the same reference are
  plausible.
- Recording a governance adoption and an operational-compliance claim in the
  same status field.
- Treating a passed audit as authorization to implement changes.
- Batching multiple file modifications before any persistence or
  verification.
- Fixing an unrelated discovered defect silently, inside an unrelated
  authorized change.
- Copying an entire Universal rule document's text into a local adoption
  record instead of referencing it.
- Continuing to the next phase without an explicit stop-and-report boundary.
- Declaring "Overall Governance Verified" or "Implementation Active" without
  a specific, independently-reproducible reason for each.
- Treating this methodology as an autonomous algorithm capable of silently
  resolving a genuine governance collision on its own. The methodology
  guides investigation and narrows the decision space; it does not replace
  owner authority. Final authority over a true identifier or governance
  collision remains a human/owner decision — never auto-resolve one without
  that explicit authorization.

---

## 5. Gate Summary

| Gate | Meaning | Mutation allowed after this gate alone? |
|---|---|---|
| `READ-ONLY PREFLIGHT PASS` | Current registry/manifest state confirmed | No |
| `SOURCE-OF-TRUTH PASS` | Local authority model established | No |
| `CONSTITUTION PREFLIGHT PASS` (Layer A) | Lightweight identity/version check completed for this OS, clean or anomaly-flagged | No |
| `CONSTITUTION RECONCILED` (Layer B, only if triggered) | Governance root confirmed current via an authorized bridge | No (the bridge itself required its own file-level authorization) |
| `AUDIT COMPLETE` | Phase 3–4 findings finalized, zero unresolved `UNCLEAR` | No |
| `PROPOSAL PRESENTED` | Exact draft change shown | No — requires explicit, file-scoped authorization |
| `FILE CHECKPOINT PASS` | One file modified, persisted, independently verified | Only the *next* authorized file, not automatically |
| `GOVERNANCE CHECKPOINT PASS` | Cross-file consistency independently re-verified | No |
| `UNIVERSAL PROPAGATION CHECKPOINT PASS` | One Universal file updated and verified | Only the *next* authorized Universal file |
| `FINAL VERIFICATION PASS` | Whole audit independently re-confirmed | No — does not imply Final Ecosystem Acceptance |

No gate authorizes the next phase automatically. Every mutation-bearing phase
requires its own explicit authorization, even immediately following a PASS.

---

## 6. Failure / Edge Case Handling

- **Wrong file uploaded entirely:** detected by checking self-identification
  text inside the file, not the filename. Treat as blocking; do not guess
  which content is "probably right."
- **Stale governance root:** handled by Phase 2; resolved by an additive
  bridge, never a rewrite.
- **Terminology that looks like a match but isn't:** verify by reading how
  the term is actually used in context (what question does it answer?)
  before concluding a mechanism already exists.
- **Real incident found mid-audit:** use it as genuine local evidence in the
  proposal, but do not overstate what the incident's own historical response
  already established — distinguish "a recovery happened" from "a
  prevention rule already existed."
- **Missing files for an otherwise-registered OS:** classify explicitly as
  *PENDING — REPOSITORY NOT AVAILABLE*; never fabricate the missing
  structure from the registry entry alone.
- **A `Pending` status the owner did not expect:** state plainly why it
  remains pending, and what specific future observation would change it.

---

## 7. Worked Example (illustrative, not a template of numbers)

Personal Life OS's Constitution was found, on first inspection, to be a
different OS's completed governance content entirely; corrected to the
right file, it was then found to be an older architectural snapshot than
the rest of the repository. An additive bridge section reconciled this
without rewriting the 874 lines of original historical text. Only then did
the OS-N audit proceed, finding zero existing local terminology for the
Universal persistence rule but a real, previously-undocumented incident
that made the case concrete. A proposal covering exactly three files was
drafted, shown in full, and implemented one file per turn, each
independently persisted and verified before the next began. A 13-point
cross-file governance check and an 8-section status synthesis both passed
before two Universal-layer files were updated — narrowly, only the rows
concerning this one OS — and independently re-verified. Throughout,
"Implementation Checkpoint System Active" remained `Pending`, and was
never reported as anything else, because no operational evidence for it
existed yet.

The specific file names, ADR numbers, and section numbers above belong to
one real case. Future applications of this methodology will have different
numbers; the phase structure and gating discipline are what generalizes.

---

## 8. Methodology Status

This document is a methodology draft. It does not modify any Universal
Engineering Framework, any OS's governance files, or any Universal registry.
It creates no ADR and records no checkpoint. It has not itself been audited,
adopted, or propagated anywhere.

This methodology is **not** Universal-adopted policy, and has **not** yet
satisfied an evidentiary threshold for ecosystem-wide adoption. Its evidence
base is currently one real OS case (see Section 3.1 for which specific parts
that limits). It must not be represented as having been independently
validated across multiple OS's.

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
