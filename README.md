# SpecTrace

[Русская версия](README.ru.md)

SpecTrace extracts the normative requirements of a technical specification and anchors each one to an exact span of
the source. It then proposes test cases traceable to those requirements, and shows which requirements have none.
Every claim in its output is checkable by code against the document.

This repository holds the concept, the agreed requirements, the decisions and the acceptance record. The code, the
data and every result are in [spectrace-dev][dev].

## The problem, and for whom

- **The user.** A tester or developer has been given a specification. They must produce test cases from it, and be
  able to show that nothing in it was left unchecked.
- **By hand,** the work is accurate but slow.
- **With a chat assistant,** it is fast, but a requirement the assistant states can be checked only by re-reading
  the whole document — the very work it was meant to save.
- **What SpecTrace changes:** every requirement is anchored to an exact place in the source. Anything a model claims
  that cannot be found there is set aside and shown. Every requirement without a test case appears as a named gap.

## Why this approach

The model proposes and code verifies. Seven rules hold whatever the model is:

- The model returns quote text only. Code finds where that text is, by exact match after whitespace is collapsed,
  and never accepts a position from the model (P1, P2).
- A quote that cannot be found is dropped and listed separately. A quote found more than once goes to a person
  (P3, P6).
- Every model call is cached under its full request, so the whole run replays offline, with no key (P5).
- A test case cannot exist without a requirement. The output never says that the specification is fully covered:
  only that nothing in the register is uncovered (P7).

A different model changes the numbers, not these guarantees. The main choices behind them, with the alternatives and
the evidence, are under [Key decisions](#key-decisions).

## What has been done

| Milestone | Result | Status |
|---|---|---|
| M1 | One document in, and out come a verified register, traceable test cases and a matrix with visible gaps. The run replays offline in CI with no key. | Accepted on 26 September 2026, tagged `m1-accepted` in [`spectrace-dev`][t-m1-dev] and [SpecTrace-docs][t-m1-docs] |
| M2 | A reviewer accepts, edits or rejects each case, and every decision is kept. Measured comparisons with unverified approaches on two documents. The review UI at a public URL. | Accepted on 1 October 2026, tagged `m2-accepted` in [`spectrace-dev`][t-m2-dev] and [SpecTrace-docs][t-m2-docs] |
| M3 | These documents, the decision on the live service, and the 5–7 minute demonstration | Done on 3 October 2026, tagged `defense` in [`spectrace-dev`][t-def-dev] and [SpecTrace-docs][t-def-docs] |

- **The system:** a .NET pipeline and command line, and a server-rendered review UI. It runs publicly at
  [spectrace-5zrm6uxcja-lz.a.run.app][live], where corpus documents replay from the committed cache and a new
  document runs live.
- **Who did what:** a coding agent wrote most of the code and documents. The student set the scope, took the
  decisions recorded as theirs, annotated the gold standard, captured the chat transcripts and reviews the reference
  run. The record is in [`docs/agent-worklog.md`][worklog].

## What confirms the result

Each figure is in the linked file, which anyone can regenerate with one command. The sample size is beside each one.

- **What verification holds back.** On RFC 6902, with the same model, temperature 0 and the same verifier:
  - 81.3% of a naive prompt's quotes are not in the document verbatim (13 of 16). Almost all of them were changed in
    one way: double quotation marks rewritten as single ones.
  - The pipeline's quotes: 0.0% (0 of 18).
  - A chat assistant's answer, captured by hand: every one of its 18 quotes was found.
  - On RFC 10050, published after the model's knowledge cutoff, the three are close: chat 9.5%, naive prompt 9.1%,
    pipeline 3.3%.

  One run per arm, 16 to 30 claims each: [`experiments/headline.md`][headline],
  [`experiments/error-analysis.md`][ea] §2.
- **Quality against a hand-made ground truth.** The pipeline's register has precision 100% (12 of 12), recall 63.2%
  (12 of 19) and F1 77.4%. Before verification held quotes back, recall was 94.7%. The 6 requirements held back are
  in the human decision queue: 4 are quotes that occur twice in the document, and 2 come from one sentence the model
  read two ways. 19 gold
  requirements, one annotator: [`experiments/headline.md`][headline], [`experiments/error-analysis.md`][ea] §3.
- **How often a reviewer kept the proposals.** This is how the project measures whether its test cases help: how many
  a reviewer accepts as proposed, edits or rejects. On the reference run, 11 of 23
  cases were accepted as proposed, 9 edited and 3 rejected. There was one reviewer, who is also the author, on one
  document. 21 of the 23 decisions were made with the coding agent's opinion on each case, at the reviewer's request:
  [`experiments/review/`][outcomes], [`experiments/error-analysis.md`][ea] §9.
- **Anyone can repeat it.** One command replays the reference run offline, with no key.
  - CI runs that command on Linux and Windows on every push, and compares every file byte for byte except the run's
    start time and commit hash.
  - CI also regenerates the headline and the quality figures, and fails if either differs from the committed files.
  - All 40 CI runs between 20 September and 1 October 2026 passed: [CI][actions].

## Why it is considered right and ready

M1 and M2 were accepted after a criterion-by-criterion check against evidence re-run on the accepted commit.

**Right, in the sense the project sets itself.**
- Every requirement in the register is a verbatim quote that code located in the source.
- Every test case names a requirement.
- Every requirement without a test case stays a visible gap until a person decides otherwise.

These properties hold by construction and are checked by tests and the invariants I1–I8 on every push. All 44 CI runs
from 20 September to 2 October 2026 passed. The headline and quality figures are not asserted: one command
regenerates them from the committed cache, and CI fails if one of them changes. Where the system could not decide —
a quote found twice, a sentence read two ways, a requirement no case could be written for — it handed the question
to a person instead of guessing.

**Ready for what the course asks, and not more.** Someone outside the project can rerun it, read why each decision
was made, and see it handle a document nobody recorded. But:
- its quality is measured on one document, with one model and one annotator;
- its review was decided by one person, who is also its author.

Those limits are stated in [`docs/limitations.md`][limitations], not hidden.

## Where it stops

- [`docs/limitations.md`][limitations] says what these results allow the project to claim and what they do not.
- [`docs/privacy-safety.md`][privacy] says what a run sends to the model and where. It also covers the use
  restriction the public service runs against.
- NFR-04 is not met: the provider and the model are constants in [`LlmClientFactory.cs`][factory], so another
  provider or model is a code change, not a configuration change.

## How to rerun it

With Git and the .NET SDK 10.0, no key, and network only for the clone and the package restore:

```
git clone --branch defense https://github.com/KaunazDagaz/SpecTrace-dev.git
cd SpecTrace-dev
dotnet run --project src/SpecTrace.Cli -- run --document corpus/rfc6902.txt --offline --out runs/reference
git status
```

`git status` lists only `runs/reference/manifest.json`, whose start time and commit hash change. The other commands,
the review UI and the experiment are in [spectrace-dev's README][dev-readme].

## Key decisions

| Decision | Alternatives | Evidence | Status |
|---|---|---|---|
| .NET and C# | Python, Node and TypeScript | The student's existing skill, and a codebase a diploma can reuse | Kept |
| The model returns quote text; code finds its position | Positions reported by the model | Code can check a quote's position exactly; a model's offsets cannot be checked. By exact match, the reference run located 14 of 18 quotes once and sent the 4 found twice to a person. | Kept |
| Every model call cached under its full request, temperature 0 | No cache; a live call on every run | CI replays the reference run offline on Linux and Windows and compares it byte for byte | Kept |
| Gemini 3.5 Flash-Lite as the model, from 23 September | Gemini 3.5 Flash, as planned | Flash's free tier allowed 20 requests a day, less than one run; Flash-Lite allowed 500 | The planned model replaced |
| No chunking: the whole document in one request | Chunked extraction | The rule fixed in advance cannot fire on RFC 6902: no gold requirement lies in its last third. Absence of evidence, not evidence ([`chunking-decision.md`][chunk]). | Kept |
| A gold standard sized by rules frozen in advance: 19 requirements | 30–50 requirements, as first planned | A keyword scan finds 20 candidate sentences; under the frozen rules 18 are kept, giving 19 requirements. 30 would mean counting what the extraction excludes, or padding. | Changed; awaits the supervisor's approval |
| A baseline prompt that asks for quotes | The first baseline prompt, which asked for test cases only | Nothing in an answer of test cases alone can be checked against the source | The first prompt replaced |
| The review log outside the run folder | A log inside `runs/{runId}/` | A log inside the reference run would break its byte-for-byte test, and a stray decision in a committed folder could never be removed | Changed during SPEC-13 |
| The review UI can start a run | Runs from the command line only | Review and the demonstration need a person reviewing a real run in the browser | Kept |

[dev]: https://github.com/KaunazDagaz/SpecTrace-dev
[dev-readme]: https://github.com/KaunazDagaz/SpecTrace-dev/tree/defense#reproduce
[live]: https://spectrace-5zrm6uxcja-lz.a.run.app
[actions]: https://github.com/KaunazDagaz/SpecTrace-dev/actions
[headline]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/experiments/headline.md
[ea]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/experiments/error-analysis.md
[chunk]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/experiments/chunking-decision.md
[outcomes]: https://github.com/KaunazDagaz/SpecTrace-dev/tree/defense/experiments/review
[t-m1-dev]: https://github.com/KaunazDagaz/SpecTrace-dev/tree/m1-accepted
[t-m1-docs]: https://github.com/KaunazDagaz/SpecTrace-docs/tree/m1-accepted
[t-m2-dev]: https://github.com/KaunazDagaz/SpecTrace-dev/tree/m2-accepted
[t-m2-docs]: https://github.com/KaunazDagaz/SpecTrace-docs/tree/m2-accepted
[t-def-dev]: https://github.com/KaunazDagaz/SpecTrace-dev/tree/defense
[t-def-docs]: https://github.com/KaunazDagaz/SpecTrace-docs/tree/defense
[factory]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/src/SpecTrace.Pipeline/LlmClientFactory.cs
[limitations]: docs/limitations.md
[privacy]: docs/privacy-safety.md
[worklog]: docs/agent-worklog.md
