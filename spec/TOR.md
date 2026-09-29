# SpecTrace — Terms of Reference (ТЗ)

**Document status:** Draft, derived from `BLUEPRINT.md` — ready for technical review
**Version:** 1.6
**Course:** EHU AI-Native Engineering Practice 2026 — LAB 04 Applied Intelligent Systems
**Related documents:** `BLUEPRINT.md` (principles and scope, authoritative), `research/decisions.md` (engineering choices and their reasoning — the *why* behind the requirements below), `IMPLEMENTATION_PLAN.md` (detailed engineering design, prompts, and task sequence)
**Approved by / date / version:** *(fill in once the supervisor signs off — the methodology requires the TOR agreed and saved with a version, same as the blueprint)*

---

## 0. Document control

This TOR translates the principles in `BLUEPRINT.md` into numbered, verifiable requirements. It is the contract between "what was approved" and "what the coding agent builds."

Precedence, if the three documents ever disagree:

1. `BLUEPRINT.md` — if a requirement here contradicts a principle there, the principle wins and this document is amended.
2. **This TOR** — the frozen set of what the system must do, and by when.
3. `IMPLEMENTATION_PLAN.md` — how it gets built. Implementation details may be refined as engineering work reveals better approaches, provided every requirement below still holds.

Requirements are numbered so each one traces to the milestone that demonstrates it and to the test that verifies it — the same discipline this project asks of the specifications it processes.

### Revision history

| Version | Date | Change | Reason |
|---|---|---|---|
| 1.0 | 16 September 2026 | First frozen version. | — |
| 1.1 | 23 September 2026 | REQ-MTX-02 and the §3 definition of *Gap* now exclude a requirement only when a logged human decision marks it not testable or deferred. A model's testability classification alone never does. Requirements the model flags go to the human decision queue and stay gaps until a human decides. REQ-MTX-02's acceptance criterion now covers human decisions as well as requirements and cases. | Version 1.0 said a requirement is a gap *if and only if* it has zero non-rejected test cases. The implementation plan also has `NotTestable` and `DeferredByHuman` statuses, and by REQ-GEN-01 those requirements also have zero cases. The two could not both hold. Resolved under P6 (a logged human action turns a proposal into a decision) and P7 (no verdict of completeness): otherwise the model could make coverage look better without a human deciding anything. This is a same-principle refinement under §13(b), so `BLUEPRINT.md` does not change. Invariant I4 in `IMPLEMENTATION_PLAN.md` §9, `AGENTS.md` and `CLAUDE.md` changes with it. |
| 1.2 | 26 September 2026 | Four changes. (1) REQ-EXT-03, REQ-REV-01, REQ-REV-02, REQ-EXP-02, REQ-EXP-03, REQ-EXP-05 and REQ-DEP-02 move from gate G3 to *G3 / freeze*, which §4 defines. REQ-EXP-01 and REQ-EXP-04 stay at G3. (2) REQ-MTX-01 derives the matrix from the requirements register, the test case set and the review log. (3) §9: the gold standard holds every requirement the frozen annotation rules define, instead of 30–50 requirements. (4) §3: the *Gap* and *Arm* definitions, which ran together on one line in 1.1, are two rows again. | (1) The M2 plan in `IMPLEMENTATION_PLAN.md` §12 delivers these requirements on 29–30 September, after G3 on 27 September. The gate column would otherwise claim what the schedule does not deliver. (2) Version 1.1 made logged human decisions an input of REQ-MTX-02 but left REQ-MTX-01 naming only the register and the test cases. With the review UI (REQ-REV-02), the review log is where rejections and human decisions come from. (3) RFC 6902 carries a BCP 14 keyword in 18 sentences. Reaching 30 would mean counting statements the extraction prompt excludes, or padding the set to meet a number. The frozen rules fix the count, not a target. `BLUEPRINT.md` §4 changes with it, in version 1.1. (4) A formatting defect. All four are same-principle refinements under §13(b). No principle in `BLUEPRINT.md` §9 changes. |
| 1.3 | 27 September 2026 | §12: the open item on the second document is closed. The second, less-known document is RFC 10050, Protocol-Specific Profiles for JSContact (September 2026), and §11 names it. | SPEC-10 needed the document to run the pipeline, the baseline and the chat arm on it. RFC 10050 was chosen from three candidates (RFC 10050, RFC 10031, RFC 10022): recent Standards Track RFCs of a size close to RFC 6902 that revise or update no other RFC. It was published after the knowledge cutoff of the pipeline's model (March 2026), although its Internet-Drafts were public from February 2025. Like RFC 6902, it is subject to BCP 78 and the IETF Trust Legal Provisions, whose section 3.c.i allows copying it in full and without modification; the source, size, SHA-256 and licence are recorded in `corpus/SOURCES.md` in `spectrace-dev`. A same-principle refinement under §13(b). No principle in `BLUEPRINT.md` §9 changes. |
| 1.4 | 27 September 2026 | §12: the open item on REQ-EXT-03's chunking trigger is closed. Measured under SPEC-12 on RFC 6902, the trigger does not fire: chunked extraction is not added, and single-request extraction stays, as REQ-EXT-03 says. | The rule, fixed before the measurement, was that chunking is needed if the pipeline's recall in the last third of the document is at least 20 percentage points below its recall in the first third. All 19 gold requirements sit in lines 184–422 of 1011, so the last third holds none and the rule cannot fire. That is an absence of evidence, not evidence that recall holds up late in a document: RFC 6902 is about 8k tokens, and the check cannot detect degradation late in a long document. The decision and its caveats are recorded in `experiments/chunking-decision.md` in `spectrace-dev`. REQ-EXT-03 itself does not change. `BLUEPRINT.md` §7 still lists chunking as an open question; it changes only with the supervisor's re-approval. A same-principle refinement under §13(b). No principle in `BLUEPRINT.md` §9 changes. |
| 1.5 | 28 September 2026 | §8: the Web UI row names a run list with a new-run form, and a run overview that also shows a run in progress, besides the review and matrix pages. Starting a run from the UI runs the same pipeline as the CLI and is the UI's only path to the model. | On 27 September 2026 the student decided that the review UI can also start a run (SPEC-13). The decision, its boundaries and the risk it carries are recorded in `research/decisions.md`. Version 1.4 described three pages, and §13 treats silent divergence between this document and the shipped system as a defect. No requirement is added or removed. One `.txt` document per run (§2.1), no PDF or OCR input (§2.3), offline replay with no key (REQ-DEP-01) and the deployed UI in offline mode (§10) all still hold. A same-principle refinement under §13(b). No principle in `BLUEPRINT.md` §9 changes. |
| 1.6 | 29 September 2026 | §10: the deployed review UI also runs a new document live, through the Gemini key made in the key project, which has no billing. The service reads the key from Secret Manager in the deployment project; it is never in the repository, the image or CI. Cached documents still replay with no call. §11 adds the risk this carries. | On 29 September 2026, after SPEC-14 deployed the UI offline, the student decided that the public service should run new documents too. The decision, its boundaries and the risks it accepts are recorded in `research/decisions.md`. REQ-DEP-01 (CI offline, with no key), REQ-DEP-02 (two projects, the key project without billing), NFR-05 and NFR-06 (no secret committed or required by CI) all still hold. Treated as a same-principle refinement under §13(b): no principle in `BLUEPRINT.md` §9 changes. `BLUEPRINT.md` §4 accepts the free tier's use of inputs "only because the input is a public specification", and on the public service nothing but the page's wording keeps uploads to public specifications. Whether §4 needs amending, with supervisor re-approval, is the student's call, and `research/decisions.md` records it as open. |

---

## 1. Purpose

Build SpecTrace: a pipeline that extracts normative requirements from a specification document, anchors each one to an exact span of the source text, generates test cases traceable to those requirements, and exposes a traceability matrix with visible coverage gaps — with every claim checkable by code, never asserted by a model alone.

Primary corpus for this phase: IETF RFC 6902 (JSON Patch) — chosen per `research/decisions.md` §3.

Full problem statement, user, and design rationale: `BLUEPRINT.md` §1–3, §9.

---

## 2. Scope

### 2.1 In scope (this document governs)

- single-document pipeline: ingest one `.txt` specification per run
- LLM-based extraction of candidate requirements, each as verbatim quote text only
- deterministic verification of every quote against the source, with automatic rejection on failure
- LLM-based test case generation, scoped to verified requirements only
- a derived traceability matrix with gap detection
- a human review workflow with a logged, append-only decision trail
- a naive single-prompt baseline arm for comparison
- scoring against a hand-annotated gold standard
- an offline-replayable, cached pipeline requiring no API key to reproduce
- deployment of the review UI to a free-tier cloud target

### 2.2 Out of scope for this document (diploma continuation)

- tracking a specification across multiple versions; requirement diffing
- the system reading from or writing to GitHub or Linear as part of its own behavior
- linking requirements to real automated tests in a codebase
- a controlled, participant-based time-saved study
- agent-authored pull requests

### 2.3 Never in scope for this system

- employer code or specifications of any kind
- executing or compiling the generated test cases
- PDF, scanned, or OCR input
- multi-document runs in a single execution

---

## 3. Definitions

| Term | Meaning |
|---|---|
| Requirement | An atomic normative statement extracted from the source document |
| Quote | The verbatim text of a requirement, as returned by the model |
| Span | The `[start, end)` character range in the raw source document where a quote is located |
| Verification | The deterministic act of locating a quote's span in the source: `exact`, `ambiguous`, or `failed` |
| Gap | A requirement with zero non-rejected test cases and no logged human decision marking it not testable or deferred |
| Arm | One configuration of the pipeline run for comparison (baseline vs. treatment) |
| Gold standard | The hand-annotated reference set of requirements for one document |

---

## 4. Functional requirements

Each item states what the system SHALL do, how it is accepted, and the gate at which it must be demonstrated. Gate *G3 / freeze* means in progress at G3 on 27 September and complete by the feature freeze on 30 September. Full technical detail behind each cross-reference lives in `IMPLEMENTATION_PLAN.md`.

### 4.1 Ingestion and anchoring

| ID | Requirement | Acceptance criterion | Gate |
|---|---|---|---|
| REQ-ING-01 | The system SHALL load a `.txt` document and produce a normalized form with all whitespace runs collapsed, alongside a map from normalized-text position back to raw-text offset. | Round-trip test: for any substring of the normalized text, the mapped raw-text span, once normalized, equals that substring. *(algorithm: Implementation Plan §4.2)* | G2 |
| REQ-ING-02 | The system SHALL build a section index from the document's header structure and resolve any span to the nearest preceding section number. | Section lookups for ten sampled spans on a known document match manual inspection. | G2 |

### 4.2 Requirement extraction

| ID | Requirement | Acceptance criterion | Gate |
|---|---|---|---|
| REQ-EXT-01 | The system SHALL send the full document text to an LLM and receive candidate requirements as quote text, modality, and testability — never as positions or offsets. | Prompt schema contains no offset or line-number field. *(prompt: Implementation Plan §7.1)* | G2 |
| REQ-EXT-02 | The system SHALL assign each requirement a stable ID derived from the document ID and a hash of its normalized quote. | Re-running extraction over an unchanged document and cache yields identical IDs. | G2 |
| REQ-EXT-03 | IF measurement on the primary document shows extraction recall degrading for requirements located later in the document, THEN chunked extraction SHALL be added as a follow-up task; otherwise single-request extraction is final. | Recall-by-position analysis recorded under `experiments/`; the decision is documented either way. | G3 / freeze |

### 4.3 Verification

| ID | Requirement | Acceptance criterion | Gate |
|---|---|---|---|
| REQ-VER-01 | The system SHALL resolve every candidate quote to a span by exact substring match on normalized text, and SHALL NOT use fuzzy matching in the default path. | A `--fuzzy` flag, if present, defaults to off; default-path output contains zero fuzzy-resolved spans. | G2 |
| REQ-VER-02 | A quote that cannot be located SHALL be excluded from the requirements register and recorded in a separate rejected-quotes report together with the model's original text. | A deliberately corrupted quote is rejected, not silently repaired, in a fixture test. | G2 |
| REQ-VER-03 | A quote located more than once without disambiguating context SHALL be marked `ambiguous` and routed to the human decision queue, never silently resolved to the first match. | A test fixture with a duplicated sentence produces an `ambiguous` entry. | G2 |

### 4.4 Test case generation

| ID | Requirement | Acceptance criterion | Gate |
|---|---|---|---|
| REQ-GEN-01 | The system SHALL generate test cases only for requirements with `testability = testable`, using only the requirement's own quote as grounding. | No test case references a `needs_human_decision` or `not_testable` requirement. | G2 |
| REQ-GEN-02 | Every test case SHALL name at least one requirement ID; a test case naming none SHALL NOT be constructible. | Enforced by type (non-empty list) and covered by a unit test. | G2 |

### 4.5 Matrix and gaps

| ID | Requirement | Acceptance criterion | Gate |
|---|---|---|---|
| REQ-MTX-01 | The system SHALL derive the traceability matrix purely from the requirements register, the test case set and the review log, holding no independent state. | Deleting and regenerating the matrix from the same inputs yields an identical result. | G2 |
| REQ-MTX-02 | A requirement's status SHALL be `gap` if and only if it has zero non-rejected test cases and no logged human decision marks it not testable or deferred. A model's classification alone SHALL NOT set a requirement to not testable or deferred. Requirements the model flags SHALL go to the human decision queue and remain gaps until a human decides. | Property test over random requirement, test case and human-decision combinations. A requirement the model flagged, with no human decision, is a gap. | G2 |
| REQ-MTX-03 | A test case whose requirement no longer exists SHALL appear in a separate orphans section, never silently dropped. | Fixture: remove a requirement, confirm its case appears as orphaned. | G2 |

### 4.6 Human review

| ID | Requirement | Acceptance criterion | Gate |
|---|---|---|---|
| REQ-REV-01 | The system SHALL provide a review surface listing each requirement's quote, source context, and its test cases, with accept / edit / reject actions. | Manual walkthrough of one full document's review. *(page structure: Implementation Plan §10.1)* | G3 / freeze |
| REQ-REV-02 | Every review decision SHALL be appended to a per-run, append-only log with author and timestamp; prior entries SHALL NOT be overwritten. | Inspecting the log after several decisions shows one line per decision, none mutated. | G3 / freeze |

### 4.7 Experiment and scoring

| ID | Requirement | Acceptance criterion | Gate |
|---|---|---|---|
| REQ-EXP-01 | The system SHALL provide a baseline arm using a single naive prompt with no verification or traceability, scored through the same quote-verification machinery as the treatment arm. *(prompt: Implementation Plan §7.3)* | Baseline run produces a verification-rate figure, not merely raw text. | G3 |
| REQ-EXP-02 | The system SHALL score a run against a gold-annotated document using span-overlap matching: a match requires ≥50% overlap of the shorter span, each gold requirement matched at most once, greedy by overlap size. | Metric computation unit-tested against hand-built fixtures with a known expected result. | G3 / freeze |
| REQ-EXP-03 | The system SHALL report, at minimum: extraction precision/recall/F1, quote-verification rate, modality accuracy, and cost (tokens, cache hit rate) per arm. | `experiments/{runId}.metrics.json` contains every listed field after a scored run. | G3 / freeze |
| REQ-EXP-04 | The system SHALL accept an externally produced list of claimed requirements — including output captured by hand from a public chat interface — and score it through the same verification machinery, without that output having been generated by the pipeline. | A captured transcript, archived verbatim with timestamp and model identifier, yields a quote-verification rate. The results document states that this arm is not reproducible and explains why. | G3 |
| REQ-EXP-05 | The illustrative arm SHALL be run against both the primary document and the less-known second document, and both results reported regardless of outcome. | Both figures appear in the error analysis, including the case where the untooled workflow performs well on the primary document. | G3 / freeze |

### 4.8 Deployment and CI

| ID | Requirement | Acceptance criterion | Gate |
|---|---|---|---|
| REQ-DEP-01 | The full pipeline SHALL run end-to-end in CI using only the committed cache, with no API key and no network call to the LLM provider. | CI job succeeds with `SPECTRACE_OFFLINE=1` and no secrets configured. | G2 |
| REQ-DEP-02 | The review UI SHALL be deployed to a billing-enabled cloud project kept separate from the project holding the LLM API key. | Two distinct project IDs recorded in `README.md`, with a budget alert on the billing-enabled one. | G3 / freeze |

---

## 5. Non-functional requirements

| ID | Requirement | Rationale |
|---|---|---|
| NFR-01 | All LLM calls SHALL run at temperature 0 and SHALL be cached by a content hash of the full request (model, prompts, parameters, schema). | Determinism and reproducibility — Blueprint P5. |
| NFR-02 | No prompt, contract, or code path SHALL request or accept a model-reported character offset or line number. | Blueprint P2 — architectural, not a style preference. |
| NFR-03 | `SpecTrace.Core` SHALL have no compile-time or runtime dependency on any LLM client. | Enforced by an assembly-reference test; keeps the verifiable core independent of the probabilistic edge. |
| NFR-04 | The LLM provider SHALL be swappable via configuration, through a single interface with no more than two implementations at this stage. | Blueprint P4 — the direct answer to "this is just a wrapper." |
| NFR-05 | No employer code, specification, or data SHALL appear anywhere in the repository, cache, or corpus. | Legal and course-rules constraint; non-negotiable. |
| NFR-06 | No secret (API key or otherwise) SHALL be committed to the repository or required by CI. | Follows directly from NFR-01's caching requirement. |
| NFR-07 | A full run over the primary document SHALL complete within free-tier rate limits without manual intervention (backoff on HTTP 429). | Keeps the project's operating cost at zero. *(verify current limits at build time — Implementation Plan §10.2)* |

---

## 6. Architecture (condensed)

```
Specification (.txt)
   → Anchoring layer (deterministic)
   → Extraction (LLM)
   → Verification (deterministic — accept, reject, or flag ambiguous)
   → Case generation (LLM, verified requirements only)
   → Matrix & gap detection (deterministic)
   → Human review (logged)
   → Export
```

Two independent halves: a **verifiable core** (no network, no LLM, fully unit-tested) and a **probabilistic edge** (the two LLM calls, isolated behind one interface, cached, swappable). NFR-03 and NFR-04 exist specifically to keep this boundary real rather than aspirational.

Module layout, exact types, and prompt text: `IMPLEMENTATION_PLAN.md` §5–7.

---

## 7. Data contracts (index)

Full field-level definitions are frozen in `IMPLEMENTATION_PLAN.md` §6 and SHALL NOT be restated or allowed to diverge here. This TOR fixes only which entities must exist and what each is for:

| Entity | Purpose |
|---|---|
| `Requirement` | One verified, anchored normative statement |
| `TestCase` | One case, traceable to ≥1 requirement |
| `MatrixRow` | Derived coverage status for one requirement |
| `DecisionQueueItem` | One item requiring a human decision |
| `ReviewRecord` | One logged human decision on a test case |
| `RunManifest` | Metadata for one pipeline run (provider, cost, versions) |

---

## 8. External interfaces

| Interface | Summary |
|---|---|
| `ILlmClient` | Single method, request in, response out. Decorated by caching and rate-limiting. No provider-specific code outside its implementation. |
| CLI | `run`, `score`, `export` commands, per Implementation Plan §5. |
| Web UI | Server-rendered pages — a run list with a new-run form, a run overview that also shows a run in progress, review, and matrix — per Implementation Plan §10.1. No SPA framework. Starting a run is the UI's only path to the model, through the same pipeline as the CLI. |

---

## 9. Validation plan

- **Invariants I1–I8** (quote-span consistency, referential integrity, matrix correctness, Core/Llm isolation) run in CI against the committed cache on every push. Full list: `IMPLEMENTATION_PLAN.md` §9.
- **Gold standard**: every requirement the annotation rules define in the primary document, hand-annotated, with the rules frozen before annotation begins. The count follows from the rules and is never padded towards a target.
- **Experiment**: baseline vs. treatment, scored per REQ-EXP-02/03, with error analysis categorizing rejected quotes, missed requirements, and false positives.

---

## 10. Deliverables

- Public repository, green CI, `AGENTS.md` present
- `README.md` with one-command offline reproduction
- Deployed review UI, public: it replays the committed cache with no key, and runs a new document live through the key project's Gemini key, read from Secret Manager in the deployment project, never from the repository, the image or CI
- Gold standard, metrics output, error analysis document
- `docs/limitations.md`, `docs/privacy-safety.md`, `docs/agent-worklog.md`
- 5–7 minute demo ending on a gap the system found

---

## 11. Risks and assumptions

Condensed; full treatment in `BLUEPRINT.md` §7 and `IMPLEMENTATION_PLAN.md` §14.

- Training-data contamination on the primary public document is assumed; a second, less-known document measures the gap: RFC 10050, Protocol-Specific Profiles for JSContact, published in September 2026.
- Free-tier vendor quotas may change without notice; NFR-01's cache is the mitigation, not a workaround to remove later.
- The deployed review UI runs new documents live for anyone with the link. Every visitor shares the key's daily request quota, and an upload is sent to the provider's free tier whatever it contains; only the page's wording asks for public specifications. A live run made there cannot be replayed from the repository, so it is a demonstration, never evidence. Recorded in `research/decisions.md`, 29 September 2026.
- Single-annotator gold standard; no inter-annotator agreement figure will exist.
- The illustrative arm (REQ-EXP-04) is not reproducible by construction and is reported as a demonstration with a verification figure attached, never as a controlled result. Extracting its claimed requirements from unstructured prose is manual work; budget for it.

---

## 12. Open items pending technical breakdown

Everything else in this TOR is frozen. These remain conditional or unresolved on purpose, and should be closed during the component-by-component technical breakdown rather than guessed here:

- exact provider SDK call shape and current free-tier rate limits — **VERIFY** at implementation time
- confirmation of the primary document's redistribution terms and the provider's free-tier data-use policy, ahead of the final report (Blueprint §7)

---

## 13. Change control

Any change to a functional or non-functional requirement above requires either: (a) a matching update to `BLUEPRINT.md` if a principle is affected, or (b) a note in the PR description if it is a same-principle refinement. Silent divergence between this document and the shipped system is treated as a defect in the document, not tolerated as drift.

---

## Appendix: the prompt this TOR was assembled from

Per the practice's own methodology (Этап 05: form the prompt first, then use it to build the ТЗ). Kept here so a future revision — or a future milestone's TOR extension — can be regenerated the same way rather than freehand.

```
You are assembling the Terms of Reference (ТЗ) for SpecTrace, from the
following inputs. Do not invent functionality beyond what they state.

Input documents and versions: BLUEPRINT.md v1.0 (approved 16 September 2026);
research/decisions.md v1.0; the project's working conversation, cleaned to
decisions and open questions only.

Agreed technical decisions to build from, not re-litigate: as recorded in
research/decisions.md §1-5.

First-version boundaries not to exceed: as recorded in BLUEPRINT.md §5.

What the result must contain: functions, module boundaries, data, external
interfaces, error handling, constraints, how to run it, and how to check it —
each requirement numbered so tasks can reference it later.

Hard rules:
1. Every requirement must trace to a scenario or data point in BLUEPRINT.md
   §3-4; one with no such anchor does not belong here.
2. Every requirement needs a verifiable acceptance criterion, not a
   description of intent.
3. Never present an assumption, a convenience, or your own suggestion as an
   agreed decision — if it isn't in the blueprint or the decision map, it
   goes in §12, explicitly marked as not yet decided.
4. If a genuine contradiction exists between the blueprint and the decision
   map, or between two decisions, name it as a question. Do not silently
   pick a side.
5. Number every requirement (REQ-xxx, NFR-xxx).
```
