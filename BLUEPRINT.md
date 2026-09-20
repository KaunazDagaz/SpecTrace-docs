# SpecTrace — Project Blueprint

**Status:** Approved
**Version:** 1.0
**Course:** EHU AI-Native Engineering Practice 2026 — LAB 04 Applied Intelligent Systems
**Author:** Mikita
**Approved:** 16 September 2026, by the practice supervisor *(date assumed as today — correct if the actual date differs)*

---

## 0. Document control

This blueprint follows the seven-section template from the practice's own methodology guide (Avdeichik, *Agentic Product Development Student Guide*, v1.0, 12 Sep 2026 — Этап 02, "Зафиксировать смысл"). Worth naming plainly: the guide identifies its author as the supervisor of this very practice, so matching its structure is not a stylistic choice — it is very likely the actual expectation at sign-off.

Sections 1–7 below answer the seven questions the template requires, in the template's own order and naming (Russian label kept alongside the English one). Two further sections go beyond the template: the engineering choices already explored (§8), which the guide's own methodology places in a *later* research stage rather than in the blueprint itself, and the non-negotiable rules those choices must obey (§9).

Two documents already exist downstream of this one — `TOR.md` (frozen, numbered requirements) and `IMPLEMENTATION_PLAN.md` (engineering detail) — written before this restructuring. `TOR.md`'s pointers into this document have been updated to match the new section numbers below; `IMPLEMENTATION_PLAN.md` does not cross-reference Blueprint sections and needs no change on that account.

---

## 1. User and problem (Пользователь и проблема)

*Кто сталкивается с проблемой и в какой ситуации?*

A tester or developer who has been handed a specification and must produce test cases from it, while being able to prove that nothing in the document was left unchecked.

Today: manual traceability — reading the document, pulling out checkable statements by hand, building a requirement-to-test table by hand — is accurate but slow. Handing the same document to a general chat assistant is fast, but untrustworthy: it returns plausible test cases for requirements that are not actually in the document, and the only way to catch this is to re-read the whole specification — exactly the work the assistant was supposed to remove.

---

## 2. Useful outcome (Полезный результат)

*Что человек сможет сделать лучше, чем сейчас?*

Today this person either spends hours building traceability by hand, or trusts a model's output and has no way to catch an invented requirement short of redoing the reading themselves.

With this system: every requirement is anchored to an exact, checkable location in the source; anything a model claims that cannot be located in the source is dropped automatically and shown separately, never mixed into the trusted result; every requirement without a test case shows up as a named gap instead of a silent omission. The tester's real cost shifts from "read the whole document to check the machine's work" to "review a short, specific list of flagged decisions."

---

## 3. Scenarios and functions (Сценарии и функции)

*Как проходит основной путь от входа до результата?*

1. The tester provides one specification document.
2. The system proposes candidate requirements, each as an exact quote from the document.
3. Every quote is checked against the source. One that cannot be found is set aside and shown separately — it never enters the trusted result.
4. For each verified requirement, the system proposes one or more test cases.
5. The tester reviews each proposal — accept, edit, or reject — and every decision is kept in a record that is never silently changed afterward.
6. The tester sees one table: which requirements are covered, which are gaps — and can export it.

---

## 4. Data (Данные)

*Какие сведения нужны, откуда они берутся, кому доступны?*

- **Input:** one public specification document. The working candidate is IETF RFC 6902 (JSON Patch) — precise, numbered, normatively dense, and small enough to fit whole in a single request. Exact redistribution terms are assumed permissive but not yet formally confirmed — see §7.
- **Ground truth:** a small hand-annotated set (30–50 requirements), built by the student directly from the same document, kept in the project's own documentation for scoring only.
- **Generated data:** candidate requirements and test cases produced by the language model, cached to disk so nothing is requested twice.
- **What never enters the system:** anything personal, and anything belonging to an employer — not as a default, as a hard boundary (§5).
- **Access:** the whole project — code, cache, corpus, gold standard, run outputs — lives in a public repository. Nothing here is access-restricted, because nothing sensitive is ever admitted in the first place.
- **One honest caveat:** on a free usage tier, submitted text may be used by the provider to improve its models. Acceptable only because the input is a public specification; would not be acceptable for a private document. Exact terms to confirm — see §7.

---

## 5. First version and boundaries (Первая версия и границы)

*Что входит в практику, а что сознательно откладывается?*

**In this version:** one document per run; requirement extraction with mandatory source verification; test case generation limited to verified requirements; a traceability matrix with visible gaps; a human review step with a permanent decision record; comparison against a naive single-prompt baseline and, for demonstration only, against an untooled public chat run.

**Deliberately set aside for later (the diploma horizon):** following a specification as it changes across versions; the system itself reading from or writing to GitHub or Linear; connecting requirements to real automated tests in a codebase; a controlled study measuring time actually saved by QA staff; agent-authored pull requests.

**Never in scope for this system, at any horizon:** any employer's code or specifications; running or compiling the generated test cases; PDF or scanned input; more than one document in a single run.

---

## 6. Success verification (Проверка успеха)

*Как мы покажем работоспособность и оценим полезность?*

**Demonstration:** a live run over the chosen document, ending on a real, visible coverage gap the system found — reviewed live, not staged.

**Usefulness, measured rather than asserted:**
- how often a claim cannot be verified against the source, compared across three conditions: an untooled public chat (for scale, not as a fair comparison), a controlled single-prompt baseline, and the full pipeline;
- extraction precision, recall, and F1 against the hand-built ground truth;
- how many generated test cases a reviewer accepts as-is, edits, or rejects outright.

**Reproducibility as its own proof:** the entire run replays from a committed cache, offline, with no API key required — so "it works" does not rest on the demo going well on the day; anyone reading the repository can rerun it afterward and get the same numbers.

---

## 7. Open questions (Неопределённость)

*Какие предположения и вопросы пока не проверены?*

- We assume a tester actually wants this exact shape of output — a register, a matrix, a decision queue — rather than a direct export into whatever test-management tool they already use. Not checked against anyone but the student so far.
- We assume normative language (MUST / SHOULD / MAY) reads closely enough to ordinary specifications that the result says something beyond this one document's style. Untested.
- We do not yet know how much of the model's apparent success on the primary document is genuine reading versus memorized familiarity with a well-known public text. A second, less-known document is planned specifically to probe this; the result is not in yet.
- Redistribution terms for the chosen document, and the provider's exact data-use policy on its free tier, are assumed acceptable but not yet formally confirmed.
- Whether the document needs to be split into chunks for reliable extraction is genuinely unknown until the first real run.
- Which second document will be used for the memorization check is not yet decided.

---

## 8. Decision map — engineering choices already explored (beyond the template)

The template's own later stage (Этап 03, "накопить облако знаний") asks for exactly this shape of record before it feeds into the ТЗ: question → options → choice → reason → remaining risk. This project did that exploration conversationally, in text, ahead of writing this blueprint rather than strictly after it — the guide explicitly allows the same open dialogue in text as in voice, so the sequence differs but the discipline doesn't. Recorded here rather than left implicit:

| Question | Options considered | Chosen | Reason | Remaining risk |
|---|---|---|---|---|
| Runtime for the pipeline | Python (richer LLM tooling), .NET/C#, Node/TypeScript | .NET/C# | Matches the student's existing skill; produces a codebase directly reusable for a same-domain diploma; no new-tool overhead inside a short window | .NET's LLM-specific tooling is less mature than Python's — more of the provider client may need to be hand-written |
| LLM provider for the pipeline | Gemini (free tier), OpenAI, Anthropic | Gemini Flash, behind a swappable interface | Zero marginal cost at this scale; context window comfortably fits the whole document in one request | Free-tier quotas and model availability can change without notice — mitigated by treating the provider as a parameter and caching every response, not by trusting the quota |
| Where the review UI runs | Local only, Cloud Run, a third-party PaaS | Cloud Run, scale-to-zero | Free at this traffic level; the course toolchain already assumes Google Cloud | Enabling billing on the same project removes the Gemini free tier — the deployment project and the API-key project must stay separate, as a rule, not a reminder |
| How to stay reproducible without repeated cost | No caching, or content-addressed caching | Caching every LLM call by a hash of the full request, temperature 0 | Turns a cost-saving trick into the project's strongest reproducibility argument: the whole pipeline replays offline, with no key, indefinitely | Ordinary cache-invalidation care needed if a prompt changes; nothing else identified yet |

---

## 9. Non-negotiable rules (beyond the template)

These hold regardless of which specific technology from §8 ends up in place, and are the direct, mechanical answer to the most likely objection this project will face — that it is a thin wrapper around a language model.

| # | Principle | What it rules out |
|---|---|---|
| P1 | **Verification over trust.** Every claim in the output is checkable by code against the source document. | A model's assertion being the final word on anything. |
| P2 | **The model never reports position.** It returns quote text only; the system resolves that text to a character span. | Model-reported offsets — confident, plausible, and wrong. |
| P3 | **Fail closed.** A requirement whose quote cannot be located in the source is dropped and logged, never guessed into place. | Silent hallucination reaching the output. |
| P4 | **Provider is a parameter.** Swapping the LLM changes output quality; it must never change correctness guarantees or require a redesign. | Betting the project's validity on one vendor's model. |
| P5 | **Reproducibility by construction.** Every model call is cached deterministically; the whole pipeline replays offline, with no key, indefinitely. | An experiment only its author can rerun. |
| P6 | **Human decision is visible and separate.** The system proposes; a logged human action (accept / edit / reject / defer) is what turns a proposal into a decision. | Silent auto-acceptance of generated content. |
| P7 | **No verdict of completeness.** The system claims only "nothing in the register is uncovered," never "the specification is fully covered." | Overclaiming what verification can actually prove. |

Substitute a stronger or weaker model into this pipeline, and P1, P2, P3, P5, P6, P7 do not move — only the numbers on the metrics report do. That is the difference between value sitting in the scaffolding and value sitting entirely in the model.

---

## 10. What comes after this document

`TOR.md` and `IMPLEMENTATION_PLAN.md` already exist and do not need to be rewritten on account of this restructuring — only `TOR.md`'s pointers into this document changed. The template's own next stages apply to the *other* documents, not this one: Этап 06 (two-repository structure, `AGENTS.md`/`CLAUDE.md`) and Этап 07 (the first veha broken into no more than five outcome-phrased tasks) are now reflected in `IMPLEMENTATION_PLAN.md` §5.1 and §12. What remains is Этап 02's own checkpoint — supervisor sign-off, recorded above — and Этап 06's physical half: actually creating the two repositories.
