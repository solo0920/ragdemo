<!--
Sync Impact Report
- Version change: 2.0.0-DRAFT.3 → 2.0.0-DRAFT.4
- Modified principles: G2 (Complexity Tracking / Non-Negotiable violation handling)
- Modified sections: G2, Migration Notes
- Added sections: None
- Removed sections: None
- Follow-up TODOs: Ratification date is "(pending)" (TODO(RATIFICATION_DATE)).
-->

# ragdemo.win Constitution v2.0.0-DRAFT.4

Purpose: a stable core, legal correctness, and evidence-first operations,
delivered as small, end-to-end, verifiable slices.

## How to Read This Document

- Principle IDs are stable. Use them verbatim in plans, reviews, and commits:
  `CM1-CM4`, `A1-A3`, `D1-D3`, `E1-E5`, `G1-G5`.
- A heading marked **(Non-Negotiable)** may not be waived by a Complexity Tracking entry.
  It can only change through the amendment process (G3).
  Heading marks are the single source of truth; the Governance section only summarizes them.
- Keywords: MUST / MUST NOT = mandatory; SHOULD = default, deviations need a recorded reason.
- Evidence labels apply to factual claims, measurements, and observations:
  - `[verified]`: supported by the applicable evidence requirements (E1).
  - `[inferred]`: an inference, or a factual claim that has not been verified.

  Normative requirements, design decisions, and explicitly stated proposals do not need these labels
  unless they also assert factual results. No other labels are used.

---

## Core Mission

### CM1. Task outcome over surface size (Non-Negotiable)
Every feature MUST produce a small end-to-end working slice with observable behavior.
No speculative infrastructure without a concrete, current use case.

### CM2. Provenance of legal answers (Non-Negotiable)
Every user-facing answer based on regulation or legal text MUST be traceable at two levels.
The response contract MUST distinguish them.

**User-visible citation.** Sufficient for a reader to identify and inspect the cited authoritative source and passage:
the document's human identifiers (e.g. title, court, case number, article number as applicable),
the cited passage quoted verbatim, and a link or locator to the source.

**Audit provenance.** Retained in an audit record linked to the response by a stable, queryable `response_id`
(one response may link to multiple sources and chunks), retrievable for verification, and not required to be displayed to the user:

| Field | Meaning |
|---|---|
| `source_id` | Identifier of the authoritative document |
| `raw_sha256` | Hash of the original ingested file bytes (see CM3) |
| `canonical_sha256` | Hash of the canonical source bytes (see CM3) |
| `canonical_version` | Version of the rule that derived canonical bytes from the raw file |
| `chunk_id` | Identifier of the retrieved chunk |
| `start_byte`, `end_byte` | Byte offsets of the chunk within the canonical source bytes |
| `retrieval_method` | e.g. dense / sparse / hybrid, plus reranker if used |
| `model` | Generation model identifier, when an LLM produced any part of the answer |

The audit record holds provenance only. Query, prompt, and response content are governed by CM4.

CM2 defines WHAT provenance must exist. CM3 defines how source text is PROTECTED.

### CM3. Source immutability (Non-Negotiable)
1. **Raw preservation.** The original ingested file (e.g. archive, JSON) MUST be preserved byte-for-byte with its `raw_sha256`.
2. **Canonical source bytes.** Each citable authoritative document MUST have defined canonical source bytes:
   the UTF-8 encoding of its authoritative text, derived from the raw file by a documented, deterministic, lossless rule
   (e.g. extracting and JSON-decoding a named text field). The rule is versioned as `canonical_version`.
   Canonical bytes are the single basis for offsets, chunking, hashing, and reslice.
3. **Distinct hashes.** `raw_sha256` and `canonical_sha256` are different artifacts and MUST NOT be substituted for each other.
4. **Offsets.** Offsets MUST be zero-based byte offsets into the canonical source bytes.
   Ranges are half-open intervals `[start_byte, end_byte)`: `start_byte` inclusive, `end_byte` exclusive.
   Offsets MUST NOT be interpreted as Unicode character positions (a Chinese character typically occupies 3 UTF-8 bytes).
5. **Rule changes.** Changing a derivation rule requires a new `canonical_version` and re-ingest.
   Offsets are valid only against the canonical version they were created with.
6. **No normalization.** Derivation MUST NOT apply Unicode normalization (NFC/NFD/NFKC), CRLF/LF conversion,
   full-width/half-width conversion, or whitespace trimming/collapsing. Decoding escapes (e.g. JSON `\uXXXX`) is decoding, not normalization.
   Normalized forms MAY exist only as separate derived search fields and MUST NOT be shown as source text.
7. **Verbatim chunks.** Every retrieved chunk MUST be a verbatim substring of its canonical source bytes.
8. **Contract tests** (outputs archived per E3). Tests MUST operate on `bytes` objects, never on decoded strings:
   - Reslice: `canonical_bytes[start_byte:end_byte]` equals the chunk text bytes exactly.
   - Hash: `sha256(canonical_bytes)` equals `canonical_sha256`.
   - Re-derivation: applying rule `canonical_version` to the raw file reproduces `canonical_sha256`.
   - Cross-environment determinism: re-derivation yields the same `canonical_sha256` in every environment listed in `docs/ENVIRONMENTS.md`.
9. **LLM separation.** LLM output MUST be visibly separated from source text in every response and MUST NOT be presented as authority.

### CM4. Security and privacy boundary (Non-Negotiable)
1. Secrets and sensitive configuration MUST NOT enter source control or uncontrolled logs.
2. AuthN/AuthZ boundaries and least privilege MUST NOT be bypassed.
3. User queries, prompts, and model responses are treated as potentially sensitive.
   If they are logged or stored, the plan MUST state purpose, retention period, and access scope.
   Default: do not log full query or prompt content outside controlled, access-restricted storage.

---

## Architecture

### A1. Stable core vs replaceable component
Core capabilities (ingest, retrieval, query, eval, models, rule management) stay stable.
Providers, vector DBs, storage, and UI components MUST be replaceable through explicit adapters.

### A2. Minimal complexity
Use the simplest architecture that satisfies the current need.
Each added dependency, service, or abstraction requires a current concrete requirement and a verification plan.

### A3. Explicit contracts
Public API, schema, database, retrieval interface, adapter interface, and frontend/backend contracts MUST be explicit and version-aware.
Implicit contract changes are prohibited.

---

## Delivery and Verification

### D1. Behavior tests scaled to risk
Tests validate behavior and contracts, not implementation artifacts.
Choose unit / integration / E2E by risk, not by habit.

### D2. Operable pipeline
The key pipeline exposes explicit stages:
`ingest -> parse -> chunk -> embed -> index -> retrieve -> rerank -> generate -> evaluate`.
- Failures MUST identify the stage.
- Each stage MUST have at least one independently runnable verification step (contract test or check command).

### D3. Reproducible environment
Supported environments MUST remain reproducible using `uv`, version-controlled config, and documented compose commands.
The list of supported environments and their specifics lives in `docs/ENVIRONMENTS.md`, not in this document.
Changing that list does not require an amendment; changing the reproducibility requirement does.

---

## Evidence and Measurement

### E1. Evidence-backed quantitative claims (Non-Negotiable)
Scope: quantitative claims in reports, PRs, commit messages, specs, and documentation.

**Measured claims** (pass rates, hit rates, latency, throughput, quality or reliability results) MUST state:
1. the exact command or reproducible measurement procedure,
2. the archived raw output location,
3. the denominator / population / sample size.

A denominator is always required for any ratio, rate, or percentage.

**Specification claims** (e.g. a model's vector dimension, a configured limit) MUST state their authoritative source
(spec, config file path, or vendor documentation); no separate measurement run is required.

A claim lacking its required evidence is labeled `[inferred]` and MUST NOT be presented or recorded as a verified result.
Evidence MUST NOT be fabricated or manufactured only to satisfy this principle.
In exploratory conversation, unevidenced numbers are `[inferred]` and MUST NOT be copied into the artifacts above until evidenced.

### E2. Baseline required (Non-Negotiable)
Changes to behavior, performance, retrieval quality, classification, or reliability MUST establish a reproducible, archived
pre-change baseline when a meaningful baseline exists. Before and after MUST be shown together,
use comparable methods, and disclose material differences in datasets, configurations, or environments.

New capabilities with no valid baseline MUST, before implementation, define acceptance criteria and a reproducible initial measurement.

Any baseline exception MUST record its rationale and the alternative verification method.

### E3. Completion requires proof (Non-Negotiable)
No task is complete without a verification step and its actual output.
On failure, retry or report; never mark complete.

### E4. Verified vs inferred labeling
Factual claims, measurements, and observations MUST carry the `[verified]` / `[inferred]` distinction defined in "How to Read This Document".

### E5. No invented references
File, function, and field names MUST be confirmed to exist before being cited.
If confirmation fails, report the failure instead of citing.

---

## Governance and Agent Discipline

### G1. Agent scope (Non-Negotiable)
Agents stay within the requested scope.
For out-of-scope concerns: stop -> explain -> propose the smallest change -> obtain approval.

### G2. Constitution Check gate (Non-Negotiable)
Checks are defined by workflow events, not by template phase names.

1. **Before plan research and design:** check the proposed plan's intent against CM1-CM4, A1-A3, and D1-D3
   before beginning plan-level research or design.
   Preliminary reading needed to understand the request (reading the spec and existing code) may precede this check,
   but MUST NOT produce design decisions.
2. **After design artifacts exist (data model, contracts, verification plan) and before tasks are generated:**
   re-check against the same set, and confirm the plan names
   - the applicable evidence requirements (E1),
   - the baseline or the justified baseline exception (E2),
   - the completion verification steps and expected evidence (E3).
3. **After implementation:** review the actual diff and verification evidence against the same requirements,
   and check E4, E5, G1, G4, G5.

**Complexity Tracking** records, with rationale and the rejected simpler alternative:
- deviations from SHOULD requirements,
- unresolved conflicts between principles,
- exceptions this constitution explicitly permits (e.g. a baseline exception under E2).

Complexity Tracking is a record, not an approval channel. A violation of a Non-Negotiable principle MUST NOT be treated as an approved exception.
On detecting one, the agent MUST stop and report it.
Work may resume only after the violation is resolved or the constitution is formally amended under G3.
A recorded decision alone MUST NOT authorize a violation.

**Complexity Tracking template**

| Principle | Deviation / conflict / permitted exception | Justification | Rejected simpler alternative | Approver | Date |
|---|---|---|---|---|---|

### G3. Amendments (Non-Negotiable)
An amendment MUST document: changed principle, rationale, effects, affected plans, new version number, and a migration note.
Version rules: MAJOR = principle removed or meaning reversed; MINOR = principle added or materially extended; PATCH = wording or clarification only.

### G4. Evidence-driven policy evolution
If a feature repeatedly violates a principle, the constitution SHOULD be reconsidered through G3.
Implementation MUST NOT bypass the principle in the meantime.

### G5. Definition of Done and incremental additivity
A feature is done when behavior, tests, docs/contracts, scope, secrets handling, and constitution alignment all pass.

Changes MUST stay within the approved specification and task scope. Additive changes are preferred when they satisfy the requirement.
A change that exceeds approved scope, breaks an existing contract, or alters existing behavior without authorization in the approved specification
MUST stop for explanation and approval before implementation.
Behavior changes explicitly authorized by the approved specification do not need separate approval.

---

## Technology Constraints

- Backend: Python + uv + FastAPI.
- Data: PostgreSQL + Qdrant.
- Infra: Docker Compose; Cloudflare as needed.

Changes to this stack are architectural and require constitutional justification (A1, A2).
Environment details: see `docs/ENVIRONMENTS.md` (D3).

---

## Development Workflow

Required for non-trivial work:
1. specify
2. clarify (if needed)
3. plan (Constitution Check, G2 steps 1-2)
4. tasks
5. analyze (as useful)
6. implement
7. tests
8. review against the constitution (G2 step 3)

Spec defines what/why, plan defines how, tasks define actions.
Implementation MUST NOT silently redefine requirements.

---

## Governance

- This constitution supersedes other project practices.
- Conflict resolution order:
  1. Non-Negotiable principles take precedence over all others.
  2. Within the same status, tier order applies: Core Mission > Architecture > Delivery > Evidence > Governance.
  3. If two Non-Negotiable principles conflict, stop and request a decision; record the outcome as an amendment or clarification (G3).
- Non-Negotiable principles (summary; headings are authoritative): CM1, CM2, CM3, CM4, E1, E2, E3, G1, G2, G3.
- Principle IDs remain stable. A deprecated principle requires a versioned migration note and its ID MUST NOT be reused.

## Release Gate

Each item MUST link to evidence (command output path or artifact link), per E1 and E3.

- [ ] Behavior and scope verified
- [ ] Tests pass; no unrelated regressions
- [ ] No source-text mutation (CM3 reslice, hash, re-derivation, and cross-environment test outputs attached)
- [ ] No secret exposure (CM4)
- [ ] Feature metrics evidenced (E1, E2)
- [ ] Constitution Check passed at all G2 checkpoints
- [ ] Affected paths documented

---

## Version

- Version: 2.0.0-DRAFT.4
- Ratified: (pending)
- Last amended: 2026-10-14

## Migration Notes

### DRAFT.3 -> DRAFT.4

| Change | Reason |
|---|---|
| G2: a recorded decision alone cannot authorize a Non-Negotiable violation; work resumes only after resolution or formal amendment under G3 | Close the gap between "recording a decision" and formally amending the constitution (PATCH-level wording change) |

### DRAFT.2 -> DRAFT.3

| Change | Reason |
|---|---|
| CM3 item 4: zero-based, half-open byte offsets; not character positions | Prevent agents from using different offset conventions |
| CM3 item 8: tests operate on `bytes`; added cross-environment determinism test | Avoid string-slice tests passing by accident; catch environment-dependent canonical hashes |
| Evidence labels limited to factual claims, measurements, observations (How to Read, E4) | "Everything else is inferred" wrongly covered normative text |
| G2 Complexity Tracking redefined as a record, not an approval channel; Non-Negotiable violation means stop and report | "Justified exceptions" contradicted the Non-Negotiable rule |
| G2 checkpoint 1 reworded; preliminary reading allowed but may not produce design decisions | Remove ambiguity without opening a bypass |
| CM2: `response_id` must be stable, queryable, and support multiple sources | Make the audit link an explicit contract |

### DRAFT.1 -> DRAFT.2

| Change | Reason |
|---|---|
| CM3 defines raw bytes vs canonical source bytes, `canonical_version`, distinct hashes, re-derivation test | Offsets need one unambiguous basis when sources are layered |
| CM2 splits user-visible citation from audit provenance | Keep UI clean without losing reproducibility |
| E1 split into measured vs specification claims; denominator rule objective; "false assertion" replaced by `[inferred]` | Missing evidence means unverified, not false |
| E2 allows new capabilities with no baseline via acceptance criteria + initial measurement | Avoid deadlock on first-time features |
| G2 rewritten as three workflow-event checkpoints | "Phase 0 / Phase 1" were undefined |
| G5 permits behavior changes authorized by the approved spec | Avoid redundant approval stops |

### DRAFT -> DRAFT.1

| Change | Reason |
|---|---|
| Unified `C1-C4` references to `CM1-CM4` | ID mismatch made G2 un-checkable mechanically |
| Non-Negotiable marked in every heading; Governance list is a summary | Single source of truth |
| Added precedence rule: Non-Negotiable first, then tier order | Tier order conflicted with Non-Negotiable marks |
| G2 covers E1-E3 as planning items; added Complexity Tracking template | Baseline and verification must be planned |
| CM2/CM3 overlap removed; reslice test added | Make CM3 verifiable |
| CM4 adds query/prompt logging rule | RAG-specific privacy gap |
| D2 requires per-stage verification step | Link D1 and D2 |
| Environment list moved to `docs/ENVIRONMENTS.md` | Hardware changes should not need an amendment |
| Release Gate requires evidence links | Align with E3 |
| `Ratified` set to pending | A draft cannot be ratified |
