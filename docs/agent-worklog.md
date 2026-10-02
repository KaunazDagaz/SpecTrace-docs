# Agent worklog

How AI was used to build SpecTrace: what the coding agent did, what the student decided, where the agent was wrong,
and how each mistake was caught. The student is the author of every decision, and this page cites the record that
shows each one. The agent drafted this page from that record.

## How the work was split

- **The agent** was Claude Code. Its commits carry a co-author trailer naming Claude Opus 5.5; a few early ones in
  `spectrace-dev` name Claude Opus 5 or Claude Sonnet 5.
- **The commits without an agent trailer:**
  - in `spectrace-dev`: the initial commit, the SPEC-5 project setup, and the gold standard (`3657ef3`);
  - in SpecTrace-docs: the initial commit, which holds the first Blueprint, TOR and plan.
- **Authorship:** every commit is under the student's name, and the student merged every pull request.
- **The agent's rules** are in [`CLAUDE.md`][claude] in `spectrace-dev`, and `AGENTS.md` is byte-identical to it.
  The student approved their latest revision on 1 October 2026 ([plan][plan] §12).

## What the agent did, card by card

| Milestone | Work | Pull requests |
|---|---|---|
| M1 | SPEC-5: project setup (its setup commit carries no agent trailer) | dev [#1][d1] |
| M1 | SPEC-6: the offset map and section index; the plan's section regex checked against the real corpus and corrected | dev [#2][d2] |
| M1 | SPEC-7: extraction through a cached, swappable client, and fail-closed verification | dev [#3][d3], [#4][d4], [#5][d5], [#6][d6] |
| M1 | SPEC-8: test cases, the derived matrix and the gap rule (TOR 1.1) | dev [#7][d7]; docs [#1][s1] |
| M1 | SPEC-9: offline reproduction in CI; the M1 acceptance note | dev [#8][d8]; docs [#2][s2] |
| M2 | The M2 plan and cards | docs [#3][s3] |
| M2 | SPEC-10: the baseline arm, the claim parser, the headline on two documents | dev [#9][d9]; docs [#4][s4] |
| M2 | SPEC-11: the annotation rules, the worksheet and the gold-file check (the annotation itself is the student's) | dev [#10][d10]; docs [#5][s5] |
| M2 | SPEC-12: quality metrics, the chunking decision, the error analysis | dev [#11][d11]; docs [#6][s6] |
| M2 | SPEC-13: the review UI and its log; the student's review committed | dev [#12][d12], [#14][d14]; docs [#7][s7] |
| M2 | SPEC-14 and after: the container, the deploy scripts, the live mode and its fixes; the M2 acceptance note | dev [#13][d13], [#15][d15], [#16][d16], [#17][d17], [#18][d18]; docs [#8][s8], [#9][s9], [#10][s10] |
| M3 | The M3 plan; the agent rules applied | docs [#11][s11], [#12][s12]; dev [#19][d19] |
| M3 | SPEC-15: the demonstration script and the slides, drafted for the student | docs [#13][s13] |
| M3 | SPEC-16: the decision on the live service, the privacy page, the corrected banner | dev [#20][d20]; docs [#14][s14] |
| M3 | SPEC-17: this page, [`limitations.md`][limitations], the README and the report it is translated into | this pull request |

## What the student decided

Each item links the record that shows it.

- **The concept, the user and the scope:** [`BLUEPRINT.md`][blueprint] 1.0, approved by the supervisor on 16
  September 2026, including RFC 6902 as the primary document (§4) and .NET (§8).
- **Every plan and card,** by merging it, and every other merge. The acceptance of M1 and M2, by merging their
  acceptance notes ([M1][m1], [M2][m2]).
- **The second document,** RFC 10050, chosen from three candidates the agent proposed, as the SPEC-10 card required
  ([plan][plan] §12; [`spec/TOR.md`][tor] 1.3).
- **The decisions recorded as the student's** in [`research/decisions.md`][decisions]:
  - the review UI starts runs (27 September);
  - the public demo, offline (28 September);
  - then live (29 September);
  - kept live against the provider's terms (1 October).
- **The gold standard:** annotated by the student under rules frozen beforehand (`corpus/gold/rfc6902.gold.yaml`,
  annotator field).
- **The chat transcripts:** captured by the student from the chat interface (`experiments/a0/`).
- **The review decisions on the reference run:** in its log, under the student's self-declared name
  (`experiments/review/`).
- **The course format, the live document for the demonstration, and the approval of the nine agent-rules changes:**
  answers of 1 October 2026 ([plan][plan] §12).

## Where the agent was wrong, and how it was caught

| Episode | What went wrong | How it was caught | What changed |
|---|---|---|---|
| Comments in code | The agent wrote comments the project does not want | Found after SPEC-3 merged, and stripped twice ([M1 note][m1] §8.4) | A rule against agent comments, dev [#4][d4] |
| Line endings | Files written with Windows line endings broke byte comparisons | Four separate fixes, in `476c705`, `417fa38`, `da0936e` and `9e37019` ([M1 note][m1] §8.4) | A `.gitattributes` rule per data folder, now one rule in `CLAUDE.md` |
| The verifier | When the model gave one quote two readings, it kept the first and dropped the other without a record: on the recorded run, the MUST NOT reading of §5's sentence. It also computed the verification rate over distinct outcomes (86.7%) instead of quotes returned (77.8%). | On the recorded run, the day SPEC-7 merged (dev [#6][d6]) | Conflicting readings now go to the decision queue, and a check fails if any reading is not accounted for |
| The quota | The daily limit was not checked before the first model was used; Flash's free tier allowed 20 requests a day, less than one run | The quota stayed exhausted for days | The model switched to Flash-Lite, dev [#5][d5]; recorded in [`decisions.md`][decisions] |
| Judgments meant for a person | The agent wrote the expected sections of the sampled lookups (10 in M1, 13 for RFC 10050), and entered the error-analysis verdicts at the student's request | The acceptance notes ([M1][m1] row 2.5, [M2][m2] §8.4) | An agent rule: such judgments are entered by the person |
| The parser | It counted its own failure to read the RFC 10050 baseline answer against the model | Before the figure reached the headline ([M2 note][m2] §8.1) | Fixed in `f91de60` |
| A live request in a test run | `dotnet test` ran with the key set and spent one request (SPEC-11) | The agent's own record | An agent rule: tests run without the key |
| An untrue sentence on the run list | It said cached documents still run live with no key; they do not | While building the live mode (`ba50791`) | The sentence corrected and pinned by a test |
| The provider's terms | The terms were not read before the public service went live; they restrict unpaid use for users in the EEA | Raised in the brief for the M2 acceptance note, and verified in the note, on 1 October | The decision recorded with its risk (SPEC-16); provider terms joined the agent's VERIFY list |
| The stored key | The agent handed over an untested command, and Cloud Shell's paste wrapped the key in escape codes | `smoke-test.sh --live` on the first live deploy | Fixed in dev [#17][d17]; an agent rule: untested commands are labelled |
| Pull request descriptions | Six M2 PRs went in empty. In M3, the descriptions the agent wrote to a file did not reach PRs #13, #14, #19 and #20, which carry their commit messages instead. | The M2 acceptance note; on 2 October, while writing this page | An agent rule: a PR without its criteria is not ready to merge; the commit messages now carry the criteria |

## What the agent never did

- **Annotation and capture:** it did not annotate the gold standard, capture a chat transcript, or make a review
  decision on the reference run.
- **Deployment:** it did not deploy, run `gcloud` against the student's projects, or receive or store the Gemini key.
- **Judgments and approvals:** it did not approve its own work or decide a criterion a person must judge; where the
  student asked it to enter such a judgment, the record says so.
- **The student's own note:** it did not draft the one-page note the student writes for the supervisor.

[claude]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/main/CLAUDE.md
[blueprint]: ../BLUEPRINT.md
[tor]: ../spec/TOR.md
[plan]: ../research/IMPLEMENTATION_PLAN.md
[decisions]: ../research/decisions.md
[m1]: ../acceptance/m1-acceptance.md
[m2]: ../acceptance/m2-acceptance.md
[limitations]: limitations.md
[d1]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/1
[d2]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/2
[d3]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/3
[d4]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/4
[d5]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/5
[d6]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/6
[d7]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/7
[d8]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/8
[d9]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/9
[d10]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/10
[d11]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/11
[d12]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/12
[d13]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/13
[d14]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/14
[d15]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/15
[d16]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/16
[d17]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/17
[d18]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/18
[d19]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/19
[d20]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/20
[s1]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/1
[s2]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/2
[s3]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/3
[s4]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/4
[s5]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/5
[s6]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/6
[s7]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/7
[s8]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/8
[s9]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/9
[s10]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/10
[s11]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/11
[s12]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/12
[s13]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/13
[s14]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/14
