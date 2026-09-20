# SpecTrace — Implementation Plan

**Working title (RU):** Проверяемый ассистент тест-дизайна: от нормативных требований спецификации к матрице трассировки с видимыми пробелами покрытия
**Working title (EN):** Verifiable test-design assistant: from normative requirements to a traceability matrix with visible coverage gaps
**Repository name:** `spec-trace`
**Course:** EHU AI-Native Engineering Practice 2026 — LAB 04 Applied Intelligent Systems (related: LAB 02)
**Project type:** Hybrid (minimal product + separate experiment)

---

## 0. How to use this document

This is an implementation brief for a coding agent. Read the whole thing before writing code.

Rules of engagement:

1. Implement in the order given in §12. Each task is one Linear issue, one branch, one PR.
2. Never skip the invariant tests in §9. They are the core claim of the project, not decoration.
3. If a rule in §4 conflicts with something that seems more convenient, follow §4 and raise the conflict in the PR description.
4. When a task is done, the PR must state which acceptance criteria are met and how they were checked.
5. Anything in §11 (Out of scope) must not be built, even if trivial. Scope discipline is graded.
6. Where this document says **VERIFY**, do not trust the detail written here — check current official documentation first, because this plan was written ahead of implementation and vendor APIs change.

---

## 1. Problem and user

**User.** A software tester or developer who receives a specification and must turn it into test cases, and must be able to prove that no requirement was left unchecked.

**Current situation.** The work is manual: read the document, extract the checkable statements, write cases, then hand-assemble a "requirement → test" table. Slow, and nobody can demonstrate completeness.

**What goes wrong when this is handed to a chatbot.** It returns plausible test cases for requirements that are not in the document. The tester cannot tell an invented requirement from a real one without re-reading the whole specification — which is exactly the work they wanted to save. The output looks like evidence of coverage while covering nothing.

**Job to be done.** "Turn this specification into a set of test cases I can defend line by line, and show me what is still uncovered."

**The design consequence.** The system's output is not prose. It is structured data in which every claim is anchored to a byte range in the source document, verified by code rather than asserted by a model.

---

## 2. What the system produces

One run over one document produces four artifacts.

### 2.1 Requirements register

Every atomic normative statement found in the document, each anchored to an exact character span.

```json
{
  "id": "REQ-rfc6902-a3f1c2",
  "document_id": "rfc6902",
  "section": "4.1",
  "modality": "MUST",
  "text": "The \"path\" member of the object designates the location within the target document where the operation is performed.",
  "span": { "start": 8421, "end": 8535 },
  "testability": "testable",
  "testability_note": null,
  "verification": "exact"
}
```

`span` is resolved by our code, never by the model. See §4.1.

### 2.2 Test cases

```json
{
  "id": "TC-a3f1c2-01",
  "requirement_ids": ["REQ-rfc6902-a3f1c2"],
  "title": "Operation with an unknown op value is rejected",
  "type": "negative",
  "precondition": "A valid target JSON document is loaded.",
  "input": "A patch object with op set to \"frobnicate\".",
  "expected_result": "The implementation rejects the patch and does not modify the target document.",
  "status": "proposed",
  "review": null
}
```

A test case with an empty `requirement_ids` is never created. This is enforced by the type system and by an invariant test, not by prompt wording.

### 2.3 Traceability matrix

A derived view. Pure projection over the two collections above — it holds no data of its own.

| Requirement | Modality | Section | Cases | Status |
|---|---|---|---|---|
| REQ-…-a3f1c2 | MUST | 4.1 | TC-a3f1c2-01, TC-a3f1c2-02 | covered |
| REQ-…-77b0e9 | MUST | 4.3 | — | **gap** |
| REQ-…-1d4e08 | SHOULD | 5 | — | deferred by human |
| REQ-…-90cc31 | MUST NOT | 4.4 | — | not testable in isolation |

Plus an **orphan cases** section: test cases whose requirement no longer exists after a re-run.

### 2.4 Human decision queue

Everything the system is not allowed to decide alone: ambiguous wording, a sentence carrying more than one obligation, a `SHOULD` with no context, a statement that cannot be checked by a single case. Each entry carries the quote and a question, never a guess.

---

## 3. What the system must not produce

- Executable test code. Different project.
- A verdict that coverage is complete. The system may only claim "no uncovered requirement in the register". Register completeness stays a human question and must be stated as a limitation.
- Free-form prose anywhere in the output. The moment a paragraph appears, nothing can be checked automatically.
- A test case for a requirement whose quote failed verification. Such requirements are dropped from the register and counted separately.

---

## 4. Non-negotiable design rules

### 4.1 The model never reports positions

The model returns quote **text** only. Our code finds where that text lives in the document. If asked for offsets, a model will produce confident, plausible, wrong integers. There is no prompt that fixes this. It is an architectural rule.

### 4.2 Whitespace normalisation with an offset map

RFC `.txt` files are hard-wrapped at ~72 columns, so a quoted sentence in the raw file contains newlines mid-phrase. A model returns the same sentence with normal spacing. Naive `IndexOf` on the raw text will therefore fail on almost every correct quote, and it will look like the model is hallucinating when it is not.

Implement this **first**, before any LLM code:

```
NormalizedDocument
  ├─ Raw      : string      // original file content, untouched
  ├─ Normal   : string      // every run of whitespace collapsed to a single space, trimmed
  └─ Map      : int[]       // Map[i] = index in Raw of the character at Normal[i]
```

Resolution algorithm for a quote `q`:

1. Normalise `q` the same way.
2. `idx = Normal.IndexOf(qNormal, StringComparison.Ordinal)`.
3. If `idx >= 0`: `span = (Map[idx], Map[idx + qNormal.Length - 1] + 1)`, `verification = "exact"`.
4. If not found: the requirement is **rejected**, recorded in a `rejected_quotes.json` report with the model's returned text, and excluded from the register. `verification = "failed"`.
5. If found more than once: prefer the occurrence inside the chunk the quote came from. If there is no chunk context, mark `verification = "ambiguous"` and route to the human decision queue.

No fuzzy matching in the default path. A `--fuzzy` flag may exist for error analysis only, must be off by default, and its results must never be counted as verified.

### 4.3 Deterministic, stable IDs

`REQ-{documentId}-{sha1(qNormal)[0..6]}` — order-independent, stable across runs, so two runs can be diffed. Test case IDs derive from the requirement ID plus an ordinal.

### 4.4 Everything the model says is cached

Every LLM call is cached to disk, keyed by `SHA256` of the canonical JSON of the request (model + system prompt + user prompt + temperature + max tokens + schema). Temperature is `0`.

This is the highest-leverage 30 lines in the project. It gives three things at once:

- development re-runs cost nothing;
- `SPECTRACE_OFFLINE=1` makes a cache miss throw, so the entire experiment replays with **zero API calls and no API key**;
- therefore **CI can run the full end-to-end pipeline deterministically with no secrets**, which is unusually strong reproducibility evidence for G3.

The `cache/` directory is committed to the repository.

### 4.5 The LLM provider is a parameter, not a decision

One interface, ~15 lines. Default implementation targets Gemini Flash via the AI Studio free tier. Swapping providers must be a config change, not a refactor.

---

## 5. Architecture

### 5.1 Repository layout

Two repositories, per the practice's own methodology — stated as a rule, not a stylistic option: `spectrace-docs` holds the concept and its evolution, `spectrace-dev` holds the code. Folder names below are illustrative; the split between the two repos is not.

**`spectrace-docs/`** — concept, decisions, history: what and why

```
spectrace-docs/
├── README.md                 # map of documents: what's agreed, what's draft
├── blueprint.md               # BLUEPRINT.md — current concept, versioned
├── spec/
│   └── tor.md                  # TOR.md — agreed requirements, versioned
├── research/
│   ├── implementation-plan.md  # this document — engineering detail, not authoritative
│   ├── architecture.md
│   ├── limitations.md
│   └── privacy-safety.md
├── decisions/
│   └── agent-worklog.md        # what the coding agent did, what was rejected and why
├── conversations/               # cleaned transcripts of blueprint/TOR discussions
└── acceptance/                   # per-milestone acceptance reports
```

**`spectrace-dev/`** — implementation: how it works

```
spectrace-dev/
├── AGENTS.md                     # Codex instructions (§13)
├── CLAUDE.md                     # Claude Code instructions — own file, same rules (§13)
├── README.md                     # run + reproduce instructions, links to spectrace-docs
├── SpecTrace.sln
├── src/
│   ├── SpecTrace.Core/           # domain types, normalisation, matrix. NO I/O, NO HTTP.
│   ├── SpecTrace.Llm/            # ILlmClient, Gemini client, caching decorator, rate limiter
│   ├── SpecTrace.Pipeline/       # extraction + generation stages, prompts, orchestration
│   ├── SpecTrace.Cli/            # entry point: run, score, export
│   └── SpecTrace.Web/            # minimal API + server-rendered review UI
├── tests/
│   ├── SpecTrace.Core.Tests/     # normalisation, offset map, ID stability, matrix
│   └── SpecTrace.Pipeline.Tests/ # invariants, offline end-to-end
├── corpus/
│   ├── rfc6902.txt
│   └── gold/rfc6902.gold.json
├── cache/                        # committed LLM response cache
├── runs/                         # run outputs (gitignored except the reference run)
├── experiments/                  # metric outputs, error analysis
├── .env.example                  # no real secrets, ever
└── .github/workflows/ci.yml
```

Both repositories, and the Linear project, link to each other. They must never hold three different versions of the same requirement — `spectrace-dev` reads `tor.md` from `spectrace-docs`; it never keeps its own copy.

`SpecTrace.Core` must have no dependency on `SpecTrace.Llm`. Enforce with a test that inspects assembly references — this is the architectural boundary that keeps the verifiable part independent of the probabilistic part.

### 5.2 Target framework

Use the latest LTS SDK installed. **VERIFY** with `dotnet --list-sdks` before choosing the TFM and write the choice into `AGENTS.md`. Do not pin to a version you have not confirmed is present.

### 5.3 Pipeline stages

```
load document (.txt)
  → NormalizedDocument (Raw, Normal, Map)
  → section index (regex over Raw)
  → [optional] chunking          ← start WITHOUT this, see §5.4
  → LLM: extract normative statements
  → resolve + verify every quote → span        ← rejects here, not later
  → assign stable IDs, attach section by binary search over the index
  → LLM: generate test cases per testable requirement
  → validate requirement_ids against the register
  → assemble matrix + decision queue
  → write run artifacts to runs/{runId}/
```

### 5.4 Start without chunking

Flash-class models have a very large context window and the target document is roughly 15k tokens, so it fits whole in one call. Do not build chunking for G2 — one request, all requirements.

Add chunking later **only if** measurement shows the model drops requirements from the middle of a long document. If that happens, "does chunking improve extraction recall?" becomes a second measured result rather than time spent on speculative engineering.

### 5.5 Section index

Parse headers from `Raw` with a regex over line starts, roughly `^(\d+(?:\.\d+)*)\.?\s+(\S.*)$`. Build a sorted `(rawOffset, number, title)` list. Resolve a span's section with a binary search for the last header at or before `span.start`. **VERIFY** the regex against the actual file — RFC formatting varies, and a wrong regex silently mislabels every section.

---

## 6. Contracts

```csharp
// SpecTrace.Core
public enum Modality { Must, MustNot, Should, ShouldNot, May }
public enum Testability { Testable, NeedsHumanDecision, NotTestable }
public enum Verification { Exact, Ambiguous, Failed }
public enum CaseType { Positive, Negative, Boundary }
public enum ReviewStatus { Proposed, Accepted, Edited, Rejected }
public enum CoverageStatus { Covered, Gap, DeferredByHuman, NotTestable }

public readonly record struct TextSpan(int Start, int End)
{
    public int Length => End - Start;
    public bool Overlaps(TextSpan other) => Start < other.End && other.Start < End;
}

public sealed record Requirement(
    string Id,
    string DocumentId,
    string Section,
    Modality Modality,
    string Text,
    TextSpan Span,
    Testability Testability,
    string? TestabilityNote,
    Verification Verification);

public sealed record TestCase(
    string Id,
    IReadOnlyList<string> RequirementIds,   // never empty
    string Title,
    CaseType Type,
    string Precondition,
    string Input,
    string ExpectedResult,
    ReviewStatus Status,
    ReviewRecord? Review);

public sealed record ReviewRecord(string Decision, string? Comment, DateTimeOffset At);

public sealed record MatrixRow(
    string RequirementId,
    Modality Modality,
    string Section,
    IReadOnlyList<string> TestCaseIds,
    CoverageStatus Status);

public sealed record DecisionQueueItem(
    string Id, string Quote, string Section, string Question, string? Resolution);

public sealed record RunManifest(
    string RunId, string DocumentId, string Provider, string Model,
    double Temperature, DateTimeOffset StartedAt,
    int PromptCount, int CacheHits, int InputTokens, int OutputTokens,
    string PipelineVersion, string GitSha);
```

```csharp
// SpecTrace.Llm
public interface ILlmClient
{
    Task<LlmResponse> CompleteAsync(LlmRequest request, CancellationToken ct);
}

public sealed record LlmRequest(
    string SystemPrompt, string UserPrompt, string Model,
    double Temperature, int MaxOutputTokens, string? JsonSchema);

public sealed record LlmResponse(
    string Text, int InputTokens, int OutputTokens, bool FromCache);
```

Implementations: `GeminiLlmClient`, `CachingLlmClient` (decorator), `RateLimitedLlmClient` (decorator, token bucket ~10 requests/minute default, exponential backoff on HTTP 429).

Run artifacts written to `runs/{runId}/`: `manifest.json`, `requirements.json`, `test-cases.json`, `matrix.json`, `matrix.html`, `decisions.json`, `rejected-quotes.json`.

---

## 7. Prompts

Store as files under `src/SpecTrace.Pipeline/Prompts/`, load at runtime, and include the file hash in the cache key. Prompts are versioned artifacts, not string literals scattered through the code.

### 7.1 P1 — requirement extraction (`extract.system.md`)

```
You extract normative requirements from a technical specification.

Rules:
1. Quote verbatim from the document. Never paraphrase, correct, complete or
   normalise the wording. The quote must be copyable from the source.
2. Extract only normative statements — those using MUST, MUST NOT, SHALL,
   SHALL NOT, SHOULD, SHOULD NOT, MAY, REQUIRED, RECOMMENDED, OPTIONAL.
   Skip descriptive, historical and explanatory prose.
3. One atomic obligation per requirement. If a sentence carries two obligations
   joined by "and" or "or", emit two entries, each quoting only its own clause.
4. Never report character positions, line numbers or offsets. You do not know
   them. Return the quote text only.
5. If a statement is normative but cannot be checked from its own text alone
   (it depends on another document, on runtime environment, or on a second
   implementation), set testability to "needs_human_decision" and explain why
   in one sentence.
6. If you are unsure whether something is normative, include it with
   testability "needs_human_decision" rather than omitting it.
7. Return valid JSON only. No markdown fences, no commentary.

Output schema:
[
  {
    "modality": "MUST" | "MUST_NOT" | "SHOULD" | "SHOULD_NOT" | "MAY",
    "quote": "<verbatim text from the document>",
    "testability": "testable" | "needs_human_decision" | "not_testable",
    "testability_note": "<one sentence, or null>"
  }
]
```

User prompt: the document text, preceded by `DOCUMENT ID: {id}`.

### 7.2 P2 — test case generation (`generate.system.md`)

```
You write black-box test cases for a single requirement.

You are given one requirement quote and its section number. Nothing else about
the specification is available to you, and you must not assume anything that is
not stated in the quote.

Rules:
1. Produce between 1 and 3 cases. Fewer good cases beat more weak ones.
2. Every case must be derivable from the quote alone. Do not invent error codes,
   field names, limits, formats or behaviours that the quote does not state.
3. Prefer one positive and one negative case where the quote supports both.
4. Expected results describe observable behaviour, not implementation detail.
5. If the quote does not contain enough information to write a single meaningful
   case, return an empty array and give the reason in "blocked_reason".
6. Return valid JSON only. No markdown fences, no commentary.

Output schema:
{
  "cases": [
    {
      "title": "<short imperative sentence>",
      "type": "positive" | "negative" | "boundary",
      "precondition": "<string>",
      "input": "<string>",
      "expected_result": "<string>"
    }
  ],
  "blocked_reason": "<string or null>"
}
```

### 7.3 P0 — naive baseline (`baseline.system.md`)

Deliberately weak. This is the control arm, not a strawman to be improved.

```
Generate test cases for the following specification. Return them as a numbered
list with a title, input and expected result for each.
```

Baseline output is parsed loosely and then run through the same quote-verification
machinery to measure how many of its claims can be located in the source at all.

---

## 8. The experiment

### 8.1 Framing

The honest framing matters more than the result. Arm B's unverifiable-claim rate is zero **by construction** — that is not a finding, it is the design. So the measured question is the price of that guarantee:

> Strict quote verification drives unverifiable claims to zero. What does it cost in recall, and what is the exchange rate?

Secondary question, once the provider is a parameter: **is a cheap model with strict validation better than an expensive model without it?** This is the "honest next question" the course asks for, and it is a defensible diploma continuation.

### 8.2 Arms

| Arm | Description |
|---|---|
| A — baseline | P0 single naive prompt, no traceability, no verification. Same model, temperature 0. |
| B — treatment | Full pipeline with quote verification and traceability. |
| C — optional | Arm B pipeline on a second model. Only if time permits. |

### 8.3 Gold standard

Manually annotate one document. Budget 5–7 hours; schedule it, it is the single largest non-code cost in the project.

```json
{
  "document_id": "rfc6902",
  "annotator": "<name>",
  "annotated_at": "2026-09-…",
  "annotation_rules": "docs/annotation-rules.md",
  "requirements": [
    { "gold_id": "G-001", "modality": "MUST", "section": "4.1",
      "quote": "<verbatim>", "span": {"start": 8421, "end": 8535},
      "testability": "testable" }
  ]
}
```

Write `docs/annotation-rules.md` **before** annotating and do not change it during. Decide up front: does a sentence with two `MUST` clauses count as one requirement or two? Are `RECOMMENDED` and `SHOULD` the same class? Rules written afterwards to fit the results are not a gold standard.

### 8.4 Matching predicate

A predicted requirement matches a gold requirement when their spans overlap and the overlap is at least 50% of the shorter span. Span overlap is far more robust than string equality — it survives a predicted quote that is one clause longer or shorter than the gold one. Each gold requirement matches at most one prediction (greedy by overlap size, highest first).

### 8.5 Metrics

| Metric | Definition |
|---|---|
| Extraction precision / recall / F1 | against gold, span-overlap matching per §8.4 |
| Quote verification rate | verified quotes ÷ quotes returned by the model |
| Unverifiable claim rate | 1 − verification rate. The headline safety metric. |
| Modality accuracy | correct modality among matched pairs |
| Gap detection accuracy | system's gap set vs. gap set after human review |
| Review outcome distribution | accepted / edited / rejected, per arm |
| Cost | input + output tokens, wall-clock, cache hit rate |

`spectrace score --run <id> --gold corpus/gold/rfc6902.gold.json` writes `experiments/{runId}.metrics.json`. Metric computation lives in `SpecTrace.Core` and is unit-tested against hand-built fixtures — a metric you cannot test is a metric you cannot defend.

### 8.6 Error analysis

Required for G3, not optional. Sample and categorise:

- quotes that failed verification — was the model paraphrasing, merging two sentences, or correcting the source?
- gold requirements the system missed — where in the document did they sit?
- predicted requirements with no gold match — genuine misses in the annotation, or genuine false positives?
- test cases rejected on review — what pattern do they share?

---

## 9. Invariants and test plan

These run in CI in offline mode against the committed cache. They are the reason the project can claim anything at all.

| # | Invariant |
|---|---|
| I1 | For every requirement, `Raw[span.start..span.end]` normalises to exactly `requirement.text` normalised. |
| I2 | Every `TestCase.RequirementIds` entry references a requirement present in the register. |
| I3 | Every requirement in the register appears in the matrix exactly once. |
| I4 | `Status == Gap` if and only if the requirement has zero non-rejected test cases. |
| I5 | No test case has an empty `RequirementIds`. |
| I6 | Requirement IDs are unique within a run and identical across two runs over the same document with the same prompts. |
| I7 | No requirement with `Verification == Failed` appears in the register or the matrix. |
| I8 | `SpecTrace.Core` has no assembly reference to `SpecTrace.Llm`. |

Unit tests beyond the invariants:

- normalisation and offset map: hard-wrapped input, tabs, CRLF vs LF, leading indentation, quote spanning a line break, quote at document start and end, empty and whitespace-only quote;
- section index: nested numbering, a header-shaped line inside a code block (this will happen in an RFC — decide and test the behaviour);
- matching predicate: exact overlap, 49% and 51% overlap, one gold matching two predictions, nested spans;
- metric computation against fixtures with a hand-calculated expected F1;
- matrix assembly: orphan cases, requirement with mixed accepted and rejected cases.

**Offline end-to-end test.** `SPECTRACE_OFFLINE=1 spectrace run --document corpus/rfc6902.txt` completes using only the committed cache, produces the reference run, and all invariants hold. This single test is the G2 deliverable and the G3 reproducibility evidence.

---

## 10. Interface and deployment

### 10.1 Review UI

Server-rendered HTML from `SpecTrace.Web`. Razor Pages or minimal API with a templating helper — no SPA framework, no client build step, no component library.

Three pages:

1. **Run overview** — manifest, counts, quote verification rate, link to matrix.
2. **Review** — requirement quote with its surrounding source context, its test cases, and three buttons per case: accept / edit / reject. Editing a case sets status `Edited` and stores the original alongside.
3. **Matrix** — the table, gaps highlighted, orphan section, export to Markdown and CSV.

Persistence is JSON files on disk. No database, no migrations, no ORM. Review decisions append to `runs/{runId}/reviews.jsonl` with author and timestamp — that append-only log **is** the visible human-decision boundary the lab requires, so do not overwrite entries.

Hard ceiling: two evenings. If a third is needed, drop the UI and ship `matrix.html` generated by the CLI instead.

### 10.2 Cloud Run

Multi-stage `Dockerfile`, non-root user, `PORT` from environment, `/health` endpoint, scale to zero, minimum instances 0.

**The billing trap — this one will cost you the free tier if ignored.** Enabling billing on a Google Cloud project removes the Gemini free tier *on that project*, and every call bills from the first token. The practice requires a billing-enabled project for Cloud Run. Therefore:

- **Project 1** — billing enabled, used only for Cloud Run and Artifact Registry. Budget alert configured.
- **Project 2** — no billing, used only for the Gemini API key, obtained through AI Studio.

Never generate the API key inside the deployment project. **VERIFY** current free-tier model availability and rate limits in AI Studio before relying on them; Google revises these without notice, and free-trial cloud credits granted after March 2026 do not pay for Gemini API usage.

Deployed service runs in offline mode against the committed cache, so **no API key is deployed**. The demo is the review UI over the reference run. Record the teardown plan in `README.md` (G4 requires it).

### 10.3 CI

`.github/workflows/ci.yml`, triggered on PR and push to main:

```
dotnet restore → dotnet build --warnaserror → dotnet test
→ offline end-to-end run → invariant checks → upload run artifacts
```

No secrets. Public repository, so Actions minutes are free. If CI ever needs an API key, the caching design has been broken — fix the design, not the workflow.

---

## 11. Out of scope

Do not build any of these, however small they look:

- database of any kind, ORM, migrations
- authentication, user accounts, multi-tenancy
- React, Vue, or any client-side build step
- PDF or scanned input, OCR
- more than one document per run
- executing or compiling the generated test cases
- Jira, TestRail, Azure DevOps integration
- streaming responses
- a plugin system, more than two `ILlmClient` implementations, or any abstraction with a single implementation
- fuzzy quote matching in the default path
- model-reported offsets, under any circumstances
- retrieval or embeddings — the document fits in context, RAG here is complexity with no question attached

Also out of scope as a matter of law and course rules: any specification, code or data belonging to your employer. The corpus is public RFCs only.

---

## 12. Milestones and tasks

The practice's own methodology calls a demonstrable, verifiable step a *veha* (milestone) and asks for no more than five tasks on the current one, each phrased as a result rather than a technical action — and asks that only the current milestone be broken into tasks; later ones stay at the outcome level until the current one is accepted. G-numbers below are the course's calendar gates; M1/M2/M3 are how the work is actually organised in Linear underneath them.

### M1 (≈ G2), due 20 September — vertical slice

Result: given one specification document, the pipeline produces a verified requirements register, traceable test cases, and a matrix with visible gaps — and the whole run replays offline, from a committed cache, with no API key, in CI.

| # | Task | Verifiable result |
|---|---|---|
| 1 | Repositories and environment are ready to run something, even trivially | Both repos exist and are linked; `AGENTS.md`/`CLAUDE.md` present; solution builds; CI runs green on an empty test with no secrets configured; `.gitignore` excludes `runs/` except the reference run |
| 2 | Any quote from the document can be located back in the source, exactly | Given any substring of the normalized text, the system points to precisely where it sits in the raw file, line-wraps included, and names its section — §9 normalisation tests pass, section-index regex verified against the corpus |
| 3 | The system asks a model for candidate requirements and never trusts an answer it can't verify | Given the document, it returns verified requirement candidates via a cached, swappable LLM client; anything claimed that isn't actually in the text is dropped and shown separately, never mixed into the trusted list; ≥5 requirements verified end to end |
| 4 | Every verified requirement gets test cases you can check, and gaps are visible | For each verified requirement the system proposes traceable cases (I2, I5 hold); a matrix shows covered vs. gap (I3, I4, I7 hold); no case can exist without pointing to a real requirement |
| 5 | Anyone can rerun the whole thing and get the same answer, with no key | The full document-to-matrix run replays from the committed cache in CI, offline, with `SPECTRACE_OFFLINE=1` and no secrets — asserted by a test, not just claimed |

**M1 is accepted when task 5 is green and a human has watched a real matrix come out of a real run.**

### M2 (≈ G3), due 27 September — outcome level only

Result: a reviewer can accept, edit, or reject every proposed test case with the decision permanently recorded; there is a measured number — not an impression — for how much better the verified pipeline does than an unverified one, on both the primary document and a second, unfamiliar one.

Depends on: M1 accepted, and its retrospective.

Not broken into tasks yet, per the methodology — only the current milestone gets that detail. When M1 lands, M2's tasks are drawn from the REQ-REV, REQ-EXP, and REQ-DEP-02 items already specified in `TOR.md` §4.6–4.8, checked against what M1 actually produced — not rewritten from this plan in advance.

### M3 (≈ G4), due 4 October — outcome level only

Result: someone outside the project can read `spectrace-docs`, understand what was built and why, rerun it themselves from `spectrace-dev`'s README, and watch a 5–7 minute demonstration that ends on a real gap the system found.

Depends on: M2 accepted. Deliverables list already in `TOR.md` §10; tasks drawn from there once M2 lands.

Feature freeze 30 September. Rehearsal 2 October. Final upload to the EHU Teams contour 4 October.

---

## 13. `AGENTS.md` and `CLAUDE.md` (drop into `spectrace-dev`)

Two files, one per tool, per the methodology's own distinction — Codex reads `AGENTS.md`, Claude Code reads `CLAUDE.md`. Content is intentionally identical for now; genuine tool-specific divergence is its own exercise later, not something to invent ahead of a real need.

**`AGENTS.md`:**

```markdown
# SpecTrace — agent instructions

## Purpose
Extract normative requirements from a technical specification, generate test
cases traceable to exact source spans, and expose coverage gaps. Every claim in
the output must be verifiable against the source document by code.

## Context
Blueprint and TOR live in the sibling `spectrace-docs` repository —
`blueprint.md` and `spec/tor.md`. Read the current TOR before starting any
task; it is the frozen source of what to build. This plan's engineering detail
(reference, not authoritative) lives at `spectrace-docs/research/implementation-plan.md`.
Current milestone: this document §12 (M1/M2/M3). Do not start a future
milestone's tasks before the current one is accepted.

## Commands
- Build:      dotnet build --warnaserror
- Test:       dotnet test
- Run:        dotnet run --project src/SpecTrace.Cli -- run --document corpus/rfc6902.txt
- Offline:    SPECTRACE_OFFLINE=1 dotnet run --project src/SpecTrace.Cli -- run --document corpus/rfc6902.txt
- Score:      dotnet run --project src/SpecTrace.Cli -- score --run <id> --gold corpus/gold/rfc6902.gold.json

## Architectural boundaries
- SpecTrace.Core contains domain logic only. No HTTP, no file I/O, no LLM
  dependency. A test enforces this.
- The LLM never reports character offsets. It returns quote text; our code
  resolves the position. Do not add offsets to any prompt schema.
- No fuzzy quote matching in the default path.
- Every LLM call goes through the caching decorator. Never call a provider
  client directly.
- Temperature is 0 everywhere.

## Data rules
- Corpus is public RFCs only. Never add employer or third-party material.
- No secrets in the repository. API keys come from environment variables only.
- cache/ is committed on purpose — it is reproducibility evidence, not clutter.

## Workflow
- One Linear issue → one branch → one PR.
- Every PR states which acceptance criteria are met and how they were checked.
- Do not merge on a green CI alone; the diff must be read.
- If a task cannot be completed within its stated scope, stop and say so rather
  than expanding the scope.

## Out of scope
See `spec/tor.md` §2.2–2.3 in `spectrace-docs`. Do not build anything listed there.
```

**`CLAUDE.md`:** the same content as `AGENTS.md` above, saved as its own file.

---

## 14. Risks and limitations

State these in the report before the committee asks.

**Training data contamination.** The model has almost certainly seen RFC 6902 and its known test suites. Extraction quality here is an upper bound and will not transfer directly to a private specification. Mitigation for the report: run the pipeline on one obscure or recent document and report the gap between the two.

**Domain narrowness.** RFC normative language is far stricter than ordinary business requirements. `MUST`/`SHOULD` keywords make extraction unusually easy. Do not claim the result generalises to prose requirements without evidence.

**Register completeness is unproven.** The system can show that no requirement *in the register* is uncovered. Whether the register captured every requirement in the document is exactly what the gold standard measures, and recall will not be 100%.

**Single annotator.** One person annotated the gold standard, so there is no inter-annotator agreement figure. Name this rather than hiding it; frozen annotation rules are a partial mitigation.

**Equivalent-quote ambiguity.** A quote appearing twice in the document is genuinely ambiguous without chunk context. Currently routed to the human queue; count how often it happens.

**Free-tier data handling.** On the Gemini free tier, inputs and outputs may be used to improve the provider's models. Acceptable here because the corpus is a public RFC, and stated explicitly in `docs/privacy-safety.md`. It would not be acceptable for a private specification, which is itself a finding worth reporting.

**Vendor quota volatility.** Free-tier limits change without notice. The cache makes the project immune to this after the first run — a design property worth stating as a result rather than a workaround.

---

## 15. Diploma continuation

The practice ends with a working, reproducible slice and one honest unanswered question. The strongest candidate:

> Does strict source verification let a cheap, fast model match or beat a frontier model that is trusted without verification — and at what point does the recall penalty outweigh the safety gain?

That question needs the provider abstraction (built), the cache (built), the metrics (built) and the gold standard (built). It needs a second and third document, a second annotator, and a cost model. That is a diploma, and it starts from a green CI rather than from zero.
