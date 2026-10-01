# M2 acceptance note

[Русская версия](m2-acceptance.ru.md)

| | |
|---|---|
| Milestone | M2 (≈ G3), due 30 September 2026: [implementation plan §12][plan] |
| Version under acceptance | `spectrace-dev` `main` at [`8c9bfcd9848a1416617dc51b32179491128983a2`][c-8c9bfcd], the merge of PR [#17][d17] on 29 September 2026; `SpecTrace-docs` `main` at [`4e63b1d`][c-4e63b1d], the merge of PR [#9][s9] on 29 September 2026 |
| Checks re-run | 1 October 2026, 06:47–07:25 UTC, on that exact commit; the live URL at 06:49–06:51 UTC |
| Prepared by | the coding agent (Claude Opus 5.5), for the student's review |
| Acceptance decision | the student merging this PR, after the two decisions under Status. No tag exists yet. |

Fields left for the student:
- the two decisions under Status, and the budget alert, which only the student can confirm;
- the "Supervisor told" column of §6;
- both answers in §7;
- the hours in §8.3;
- a review of every conclusion in §8 and every proposal in §9.

---

## Status

**Two decisions are needed before merge, and one fact must be confirmed. No check failed.**

| Check | Command | Result on 1 October 2026 |
|---|---|---|
| Build | `dotnet build --warnaserror` | Passed: 0 warnings, 0 errors |
| Tests | `dotnet test` | Passed: 573 passed, 0 failed, 1 skipped. The skipped test is `LiveGeminiTests`, which skips itself when no key is set. |
| Offline reproduction | `dotnet run --project src/SpecTrace.Cli -- run --document corpus/rfc6902.txt --offline --out runs/reference` | Passed: `git status` shows only `runs/reference/manifest.json` changed, and `git diff` only its `started_at` and `git_sha` lines |
| Headline recomputation | `dotnet run --project src/SpecTrace.Cli -- score --headline --documents corpus/rfc6902.txt,corpus/rfc10050.txt --gold corpus/gold/rfc6902.gold.yaml` | Passed: exit 0, and `git status -- experiments` is empty, so every regenerated file is byte-identical to the committed one |
| Gold standard check | `dotnet run --project src/SpecTrace.Cli -- score --gold corpus/gold/rfc6902.gold.yaml --document corpus/rfc6902.txt` | Passed: 20 candidates, 18 kept and 2 dropped, none undecided; 19 requirements, each quote found exactly once |
| Review log summary | `dotnet run --project src/SpecTrace.Cli -- score --reviews experiments/review/rfc6902-3ff2234db6aa.reviews.jsonl` | Ran: 5 lines from one author; 2 test cases decided (1 accepted, 0 edited, 1 rejected); 1 queue item decided (testable) |
| Smoke test, live URL | `bash deploy/smoke-test.sh https://spectrace-5zrm6uxcja-lz.a.run.app --live` | Passed: 17 of 17 checks. The service still runs live, so `--live` was used, as the task asked. Its live check made 2 model calls, 0 from the cache, 690 tokens in and 230 out, at 06:51:23 UTC. |

How the checks were run:
- **Where:** in a fresh clone from GitHub at `8c9bfcd`, on Windows 11 (10.0.26300) with .NET SDK 10.0.301 and Git `core.autocrlf=true`. `GEMINI_API_KEY` and `SPECTRACE_OFFLINE` were removed from the environment.
- **Why not the working copy:** the student's working copy holds untracked corpus documents and cache entries: 48 cache files against 30 committed.
- **What changed in the clone:** the reproduction rewrote `runs/reference/manifest.json` in that clone only, and it was restored before the next check.

Tests per project: `SpecTrace.Core.Tests` 210 passed; `SpecTrace.Llm.Tests` 59 passed, 1 skipped; `SpecTrace.Pipeline.Tests` 304 passed. At M1 the total was 251.

### Needs a decision before merge (not a check failure)

**1. The live public service and the Gemini API Additional Terms.** The provider's terms were read on 1 October 2026 ([Gemini API Additional Terms of Service][terms], "Last updated 2026-04-28 UTC"). Under "Use Restrictions" they say:

> "You may use only Paid Services when making API Clients available to users in the European Economic Area, Switzerland, or the United Kingdom."

The same section opens with:

> "Use of Google AI Studio and Gemini API is for developers building with Google AI models for professional or business purposes, not for consumer use."

The service at the live URL fits the first sentence's case:
- It is open to anyone, users in the EEA included; Lithuania is in the EEA.
- Since 29 September it calls the Gemini API with the AI Studio key of `gen-lang-client-0785466808`, as the README records. The agent cannot see which key the secret holds.
- That project has no billing by design (REQ-DEP-02, plan §10.2), so the calls run on unpaid quota.

`research/decisions.md` (entry of 29 September) and TOR 1.6 record the quota, data-use, reproducibility and cost risks of the live service, but not this restriction. This note reports the text and gives no legal reading. Options, for the student:

- **(a) Switch the service back to offline now.** The README kill switch does it without a rebuild. The offline configuration passed 18 of 18 smoke checks on 29 September (README, Deployment). TOR §10 then returns to its 1.5 wording; §6 row D4 proposes it.
  - `deploy/deploy.sh` must not be re-run afterwards as it stands, because it always deploys live (see §5).
  - **This is the agent's recommendation.**
- **(b) Keep the service live on Paid Services**, by enabling billing on the key project.
  - That ends the free tier the project is built around (Blueprint §8, plan §10.2), and bills every call.
  - It needs a decision entry, a TOR revision, and for Blueprint §8 the supervisor's re-approval.
- **(c) Keep it as it is.** On the quoted text, that means operating against the provider's terms. Not recommended.

**2. The SPEC-13 walkthrough is partial.**
- The committed review decides 2 of 23 test cases and 1 of 4 queue items.
- SPEC-13 accepts on "a manual walkthrough of the full RFC 6902 review"; REQ-REV-01 accepts on "Manual walkthrough of one full document's review."

Options:
- **(a) Finish the walkthrough before merging this note.** Then commit the new log and re-run `score --reviews`; §2 row 13.1 and §4 change with it.
- **(b) Accept M2 with REQ-REV-01 partly met**, and carry the walkthrough to M3, recorded as §6 row D1 proposes.

**To confirm: the budget alert** on `spectrace-deploy` (REQ-DEP-02).
- The README still marks it pending, and no PR carries the screenshot the SPEC-14 card asks for as evidence.
- The agent has no access to the Google Cloud console.

### Sources and their status

| Short name | What | Where it is |
|---|---|---|
| TOR, plan, blueprint, decisions | [`spec/TOR.md`][tor] 1.6; [`research/IMPLEMENTATION_PLAN.md`][plan]; [`BLUEPRINT.md`][blueprint] 1.1, awaiting the supervisor's re-approval; [`research/decisions.md`][decisions] | `SpecTrace-docs` `main` at `4e63b1d` |
| Annotation rules | [`spec/annotation-rules.md`][rules], frozen at [`69ed50e`][c-69ed50e] | `SpecTrace-docs` |
| PRs, commits, CI | `spectrace-dev` PRs [#9][d9] to [#17][d17]; `SpecTrace-docs` PRs [#3][s3] to [#9][s9]; their commits; GitHub Actions runs 17 to 34 | public on GitHub; read through the GitHub API on 1 October 2026 |
| Experiments | [`headline.md`][headline], [`error-analysis.md`][ea], [`chunking-decision.md`][chunk], [`rfc6902.quality.md`][quality], the six metrics files, [`experiments/review/`][outcomes] | `spectrace-dev` `main` at `8c9bfcd` |
| Provider terms | [Gemini API Additional Terms of Service][terms], last updated 2026-04-28 UTC | read on 1 October 2026: page text extracted from the raw HTML, and each quoted sentence matched in it |
| Live service | [https://spectrace-5zrm6uxcja-lz.a.run.app][live], also served at https://spectrace-38594812553.europe-north1.run.app | checked over HTTP on 1 October 2026. The Cloud Run configuration itself is not readable to the agent. |
| Linear | the M2 cards | **not read**: there was no Linear access. Plan §12 says its card IDs are the Linear issue IDs, and the criteria in §2 come from it. |
| Student's logs | the `deploy.sh` and `smoke-test.sh` output of the first deploy, pasted in the agent session on 29 September | not in either repository; cited only where the README records them |

| Card | Branch | `spectrace-dev` PR | `SpecTrace-docs` PR | Commits |
|---|---|---|---|---|
| SPEC-10 Every arm has a measured share of claims that cannot be found | `SPEC-10-unlocatable-claims` | [#9][d9], no description | [#4][s4] | 11, from [`975525b`][c-975525b] to [`28a7979`][c-28a7979] |
| SPEC-11 A hand-made ground truth the tooling trusts | `SPEC-11-gold-standard` | [#10][d10], no description | [#5][s5] | [`d2c5e60`][c-d2c5e60], [`40a4b4d`][c-40a4b4d], [`3657ef3`][c-3657ef3] |
| SPEC-12 Extraction quality against the ground truth | `SPEC-12-quality-metrics` | [#11][d11], no description | [#6][s6] | 6, from [`3c0fc04`][c-3c0fc04] to [`613cb35`][c-613cb35] |
| SPEC-13 Accept, edit or reject each case, every decision kept | `SPEC-13-review-ui` | [#12][d12], no description | [#7][s7] | [`d248ee0`][c-d248ee0], [`4381b69`][c-4381b69], [`9cbd91d`][c-9cbd91d] |
| SPEC-14 The review UI at a public URL | `SPEC-14-deployment` | [#13][d13], no description | [#8][s8] | [`4a51587`][c-4a51587], [`afa89c4`][c-afa89c4], [`4f39751`][c-4f39751] |
| No card: the review log committed | `reference-review-walkthrough` | [#14][d14] | — | [`c6560cc`][c-c6560cc] |
| No card: the deployment recorded | `record-live-deployment` | [#15][d15] | — | [`9c63393`][c-9c63393] |
| No card: the public demo runs new documents live | `public-live-runs` | [#16][d16], no description | [#9][s9] | [`ba50791`][c-ba50791], [`4afb266`][c-4afb266], [`54ba698`][c-54ba698] |
| No card: the key stored without paste escape codes | `clean-secret-input` | [#17][d17] | — | [`ac9f09f`][c-ac9f09f] |
| M2 plan: cards SPEC-10 to SPEC-14, TOR 1.2, Blueprint 1.1 | `m2-plan` | — | [#3][s3] | merged as [`6540f95`][c-6540f95] |

PRs #14, #15 and #17 have a description only because GitHub copies the message of a single-commit PR.

---

## 1. Working scenario

### At the live URL

What a person outside the project does, and what they see. Each step was observed today, through `smoke-test.sh` or a direct request.

1. **Open the run list** at [https://spectrace-5zrm6uxcja-lz.a.run.app][live]. After a quiet spell the first response takes a few seconds while an instance starts; `/health` took 2.3 seconds today. A yellow banner on every page says:
   - this is a public demo that calls a model;
   - a document in the committed cache replays without a call, and any other upload goes to Gemini's free tier through the author's key;
   - upload public specifications only, and every visitor shares one daily quota;
   - the reference run is read-only, reviewer names are self-declared, and runs and decisions made here disappear when the instance restarts.
2. **Open the reference run**, `rfc6902-3ff2234db6aa`:
   - the manifest;
   - a quote verification rate of 77.8%;
   - 12 requirements, 23 proposed test cases and 4 decision-queue items.
3. **Open its review page.** It shows:
   - every requirement in its source context, with its test cases;
   - the author's 5 logged decisions, read-only.

   A decision sent anyway, from a form or by hand, is refused with 403, and the log keeps its 5 lines.
4. **Open the reviewed matrix**: 11 requirements covered, 1 gap, with Markdown and CSV export.
5. **Replay a corpus document**: choose `rfc6902.txt` in the new-run form.
   - It replays from the cache in seconds, to run `rfc6902-3ff2234db6aa`, the reference run's own ID.
   - That run is the visitor's to review: enter a name, then accept, edit or reject; the reviewed matrix follows.
   - `rfc10050.txt` replays the same way.
6. **Upload a document of your own**: one UTF-8 plain-text file of at most 64 KiB.
   - It runs live, at no more than ten model requests a minute, and spends the key's daily quota.
   - Today's smoke-test document, one sentence, took 2 calls and about 8 seconds.
   - **This step is the one decision 1 under Status is about.**
7. **Come back later**: runs and decisions made at the URL live only in that instance, and a restart, a scale to zero or a redeploy drops them.

### Locally

A reader with Git and the .NET SDK 10.0.x follows §3. They see:
- **The reproduction:** the CLI summary shown in §3. `git status` lists only `runs/reference/manifest.json`, and the headline recomputes with no change.
- **The review UI, offline:** `dotnet run --project src/SpecTrace.Web -- --offline`, at http://localhost:5000.
  - The run list says "This server runs offline." and offers `rfc10050.txt` and `rfc6902.txt`.
  - The reference run's review page shows **0 of 23 cases and 0 of 4 items decided**, because a fresh clone has no `runs/web/reviews/reference.jsonl`.
  - After the two copy commands now in the README (§3), it shows the committed review: 2 of 23 cases, 1 of 4 items, 5 log lines.
- **The demo as the image runs it:** `dotnet run --project src/SpecTrace.Web -- --offline --public-demo`.
  - The offline banner, "This is an offline demo.", is on every page.
  - The reference run is read-only: "This run is read-only on this server."

---

## 2. Evidence, per card

Where the criteria come from:
- the M2 description in plan §12, plus SPEC-14's result, split into its three parts;
- the "Acceptance" line of each M2 card, split into single checkable statements;
- the acceptance criterion of every TOR requirement each card names;
- each card's "Evidence" line.

The Linear cards were not available. If a card states a criterion that is missing here, add a row.

The column **Re-verified 1 Oct** says what was actually run or read today. "Tests pass" means the named tests were among the 573 that passed today. "The live smoke test" means today's run in Status.

### M2 as a whole

| # | Part of the result | Evidence | Re-verified 1 Oct | Result |
|---|---|---|---|---|
| R.1 | A reviewer can accept, edit or reject every proposed test case, with each decision permanently recorded | The review UI (SPEC-13); the append-only log; [`WebReviewTests`][t-webreview]`.EachReviewActionAppendsExactlyOneLineAndLeavesEveryEarlierLineByteIdentical`; [`ReviewLogTests`][t-reviewlog]`.ConcurrentAppendsNeverInterleaveOrTruncateALine`; on the public demo, [`WebDemoTests`][t-webdemo]`.OnThePublicDemoAVisitorsOwnRunFromTheCorpusChoiceAcceptsDecisionsInItsOwnLog` | Yes: tests pass, the live smoke test | Met as a capability. The human walkthrough that shows it on the reference run is partial: row 13.1. |
| R.2 | A measured number, not an impression, for how the verified pipeline compares with an unverified one, on the primary document and on a second, unfamiliar one | [`headline.md`][headline]: the share of claims not located, per arm, on RFC 6902 and RFC 10050; precision, recall and F1 on RFC 6902 | Yes: recomputed today, byte-identical | Met. Sample sizes are in §4. |
| R.3 | The review UI runs at a public URL | [The live URL][live]; the README, Deployment | Yes: the live smoke test, 17 of 17 | Met. On the quoted terms, its live mode needs a decision: Status, decision 1. |

### SPEC-10: every arm has a measured share of claims that cannot be found

Requirements: REQ-EXP-01, REQ-EXP-04, REQ-EXP-05, and the verification-rate and cost fields of REQ-EXP-03. TOR §12's second document, closed in TOR 1.3.

| # | Acceptance criterion | Evidence | Re-verified 1 Oct | Result |
|---|---|---|---|---|
| 10.1 | `experiments/` holds a metrics file per arm per document, with the three numbers | Six files: `rfc6902-chat-2026-09-26`, `rfc6902-baseline-4227a0d51f3d`, `rfc6902-3ff2234db6aa`, `rfc10050-chat-2026-09-27`, `rfc10050-baseline-61a4ac164379`, `rfc10050-9759f1bdfc79` (`.metrics.json`). Each holds the shares not located, found once and found more than once. | Yes: files read; regenerated byte-identical | Met |
| 10.2 | For the pipeline, both the raw and the delivered share are reported, the delivered one presented as zero by design | Rows "B pipeline, model's raw claims" and "B pipeline, delivered register — zero by design" in [`headline.md`][headline] | Yes | Met |
| 10.3 | The chat arm is labelled as not reproducible, with the reason | [`headline.md`][headline], "How to read the table", the A0 entry | Yes | Met |
| 10.4 | Both chat results are reported whichever way they come out | The A0 rows on both documents; [`error-analysis.md`][ea] §8 reports RFC 6902's 0.0% and RFC 10050's 9.5% | Yes | Met |
| 10.5 | The headline regenerates offline in CI, and CI fails if it differs | [`ci.yml`][ci] steps "Recompute the headline…" and "Fail if the recomputed experiments differ…"; [`HeadlineReproductionTests`][t-headline]`.TheCommittedHeadlineAndEveryFileItGeneratesRegenerateOfflineByteForByte`, `.CiRunsExactlyTheCommandTheHeadlineSaysRebuildsIt`; both steps green in run [34][run34] | Yes: tests pass; recomputed today; run 34 read | Met |
| 10.6 | The RFC 6902 row is complete by 27 September, the second-document row by 29 September | RFC 6902 row: [`ee4e4c7`][c-ee4e4c7], 26 September. Both documents: [`01219d2`][c-01219d2], 27 September. Merged with PR [#9][d9] on 27 September, 11:17 UTC. | Yes: `git log` | Met |
| 10.7 | REQ-EXP-01: the baseline run produces a verification-rate figure | A baseline: found once 6.3% on RFC 6902, 81.8% on RFC 10050; [`BaselineArmTests`][t-baseline]`.TheBaselineRunReplaysOfflineWithoutASingleNetworkRequestAndKeepsTheAnswerVerbatim` | Yes | Met |
| 10.8 | REQ-EXP-04: a transcript archived verbatim, with timestamp and model identifier, yields a quote-verification rate; the results say the arm is not reproducible, and why | [`a0/rfc6902.md`][a0-6902] (captured 2026-09-26 13:05 +03:00, "Gemini 3.5 Flash-Lite", share link) and [`a0/rfc10050.md`][a0-10050] (2026-09-27 11:55 +03:00); [`ScoreClaimsCommandTests`][t-claims]`.AnExternallyProducedAnswerIsScoredByTheSameParserAndVerifierAndItsMetricsAreWritten` | Yes | Met |
| 10.9 | One deterministic parser for the baseline and the chat transcripts | `ClaimParser`; [`ClaimParserTests`][t-parser], including `TheRealChatAnswerOnRfc6902SplitsIntoOneClaimPerNumberedHeading` and `TheRealBaselineAnswerOnRfc10050LabelsItsQuotesExactSentenceAndEachIsRead` | Yes: tests pass | Met |
| 10.10 | REQ-EXP-05 and the second document: chosen, licence-checked, section index checked | [`corpus/SOURCES.md`][sources]: RFC 10050, 31,890 bytes, IETF Trust Legal Provisions 3.c.i. [`Rfc10050SectionIndexTests`][t-10050]`.TheIndexHoldsExactlyTheSectionsOfTheTableOfContentsInOrder`, `.ASampledSpanReportsTheSectionThatContainsIt`, 13 spans | Yes: tests pass | Met. The 13 sampled spans were written by the agent ([`13797a7`][c-13797a7]), so they are not a person's inspection. |
| 10.11 | The expected number of live calls is stated before the first live run | No written record in any commit or PR | No | Not verified |
| 10.12 | `dotnet test` is run without `GEMINI_API_KEY`, so the live check spends nothing | Cannot be checked after the fact. The agent's own memory note, outside both repositories, records one live request spent by accident during SPEC-11 on 27 September. | No | Not verified; one known lapse |
| 10.13 | Evidence: the PR link | PR [#9][d9] has no description | Yes | Partly met |

### SPEC-11: a hand-made ground truth the tooling trusts

Requirements: TOR §9 (the gold standard), Blueprint §4 and §6.

| # | Acceptance criterion | Evidence | Re-verified 1 Oct | Result |
|---|---|---|---|---|
| 11.1 | The rules file is committed before the first annotation commit | The rules: [`7a64311`][c-7a64311], merged as [`69ed50e`][c-69ed50e] on 27 September at 12:05 UTC. The gold file: [`3657ef3`][c-3657ef3], 12:40 UTC the same day. | Yes: `git log` in both repositories | Met |
| 11.2 | The gold file names that commit | `annotation_rules_commit: 69ed50ea13564b3da3b17705b3bd39336a082349` in [`rfc6902.gold.yaml`][gold]; `AnnotationRules.FrozenCommit` ([`40a4b4d`][c-40a4b4d]) | Yes | Met |
| 11.3 | The loader rejects a gold file that does not | [`GoldCheckTests`][t-goldcheck]`.AFileNamingNoRulesCommitOrAnotherOneIsRejected`, `.NoFileLoadsWhileNoFrozenRulesCommitIsRecorded` | Yes: tests pass | Met |
| 11.4 | The gold file loads with zero unresolved quotes | Today's `score --gold` output in Status; the CI step "Load the committed gold standard", green in run [34][run34] | Yes | Met |
| 11.5 | It holds every requirement the rules define, however many; the count is reported and never padded | The keyword scan's 20 candidates are all decided, 18 kept and 2 dropped, giving 19 requirements. [`AnnotationWorksheetTests`][t-worksheet]`.TheCommittedWorksheetIsExactlyWhatTheKeywordScanWrites` | Yes | Met for what code can check. Whether each decision is right is the one annotator's judgment (TOR §11). |
| 11.6 | Evidence: the rules commit, the gold file commit, the loader test output | The rows above, and today's output in Status | Yes | Met |

### SPEC-12: extraction quality is measured against the ground truth

Requirements: REQ-EXP-02, REQ-EXP-03, REQ-EXP-05, REQ-EXT-03.

| # | Acceptance criterion | Evidence | Re-verified 1 Oct | Result |
|---|---|---|---|---|
| 12.1 | REQ-EXP-02: matching and metrics are unit-tested on hand-built fixtures with known expected values | [`SpanMatchingTests`][t-span]`.FiftyOnePercentOverlapOfTheShorterSpanMatchesAndFortyNinePercentDoesNot`, `.GreedyMatchingAtThirtyFiftyAndSeventyPercentGivesTheHandCalculatedPairs`; [`QualityScoreTests`][t-quality]`.PrecisionRecallF1AndModalityAccuracyEqualTheHandCalculatedValues` | Yes: tests pass | Met |
| 12.2 | REQ-EXP-03: `experiments/{runId}.metrics.json` holds every field REQ-EXP-03 lists | The RFC 6902 pipeline and baseline files hold precision, recall, F1, the verification rate, modality accuracy, tokens and the cache hit rate. The chat file has no tokens and no cache rate, because the chat interface reports none, as [`headline.md`][headline] states. The RFC 10050 files have no quality fields, because RFC 10050 has no gold standard, which is out of SPEC-11's scope. | Yes: every file's fields read | Met for the two controlled arms; n/a where stated |
| 12.3 | The chunking decision and the error analysis are written in `experiments/` | [`chunking-decision.md`][chunk]; [`error-analysis.md`][ea]; TOR 1.4 | Yes | Met |
| 12.4 | The error analysis reports both chat figures, including the case where the chat arm does well on RFC 6902 | [`error-analysis.md`][ea] §8 | Yes | Met |
| 12.5 | REQ-EXT-03: recall by position, and the decision recorded either way | [`chunking-decision.md`][chunk]: no gold requirement lies in the last third, so the rule cannot fire, and chunking is not added | Yes | Met. The check cannot see degradation late in a long document. |
| 12.6 | Evidence: the PR link, test output, metrics files, the error analysis | PR [#11][d11] has no description. In the error analysis, the verdict column was entered by the agent on the student's instruction ([`cfa85f8`][c-cfa85f8]), and §9 still says no review has been recorded. | Yes | Partly met |

### SPEC-13: a reviewer can accept, edit or reject each test case, and every decision is kept

Requirements: REQ-REV-01, REQ-REV-02, REQ-MTX-01 as amended in TOR 1.2, and the export command in TOR §8.

| # | Acceptance criterion | Evidence | Re-verified 1 Oct | Result |
|---|---|---|---|---|
| 13.1 | REQ-REV-01: a manual walkthrough of the full RFC 6902 review | The committed [log][log]: 2 of 23 cases decided (1 accepted, 1 rejected, 0 edited) and 1 of 4 queue items (testable). That item's quote occurs twice, so the decision changes no row. Neither repository holds walkthrough notes or screenshots. | Yes: `score --reviews`; the reviewed matrix exported | **Partly met**: Status, decision 2 |
| 13.2 | REQ-REV-02: after several decisions, the log shows one line per decision, and no line has changed | 5 lines for 5 decisions. [`ReviewLogTests`][t-reviewlog]`.EveryKindOfDecisionIsWrittenAsOneLineAndReadBackUnchanged`; [`WebReviewTests`][t-webreview]`.EachReviewActionAppendsExactlyOneLineAndLeavesEveryEarlierLineByteIdentical`. At the live URL, still 5 lines after the refused decisions. | Yes | Met |
| 13.3 | A rejected case and a human "not testable" decision change the matrix exactly as I4 says | [`WebReviewTests`][t-webreview]`.ARejectedCaseAndAHumanNotTestableDecisionChangeTheReviewedMatrixExactlyAsI4Says`; [`ReviewedRunTests`][t-reviewedrun]`.StatusFollowsI4ThroughTheLogOverRandomRunsAndDecisions` | Yes: tests pass | Met by tests. Not shown in the human review: its one rejection left the requirement covered by its other case, and it holds no "not testable" decision. |
| 13.4 | `runs/reference/` is byte-identical before and after the walkthrough | No commit touches `runs/reference/` after [`f0556b7`][c-f0556b7] of 24 September. Today's reproduction matches it. The live service refuses decisions on it. | Yes | Met |
| 13.5 | The walkthrough's log is committed under `experiments/` | [`experiments/review/rfc6902-3ff2234db6aa.reviews.jsonl`][log], PR [#14][d14], SHA-256 `b03e811e…bc802` | Yes | Met |
| 13.6 | The accepted, edited and rejected counts are computed from the log and written to `experiments/` | [`rfc6902-3ff2234db6aa.review-outcomes.md`][outcomes]; today's `score --reviews` gives the same counts | Yes | Met |
| 13.7 | The Markdown and CSV exports hold the same rows and statuses as the matrix page | [`WebReviewTests`][t-webreview]`.TheMarkdownAndCsvExportsHoldTheSameRowsAndStatusesAsTheMatrixPage`; [`ReviewCommandsTests`][t-reviewcmd]`.ExportWritesTheReviewedMatrixAsMarkdownAndCsvWithTheSameRowsAsTheReviewedView`. Today's export of the committed review: 11 covered, 1 gap. | Yes | Met |
| 13.8 | REQ-MTX-01: the matrix derives from the register, the cases and the log; regenerating it gives an identical result | [`RunReplayTests`][t-replay]`.DeletingAndRegeneratingTheMatrixFromTheSameInputsGivesIdenticalBytes`; [`ReviewedRunTests`][t-reviewedrun]`.WithAnEmptyLogEveryCaseStaysProposedAndTheMatrixEqualsThePipelinesMatrix` | Yes: tests pass | Met |
| 13.9 | Evidence: the PR link, walkthrough notes or screenshots, a sample of the log, the counts, a sample of each export | PR [#12][d12] has no description; no notes or screenshots exist. The log and the counts are committed. Exports were generated today and not committed. | Yes | Partly met |

### SPEC-14: the review UI runs at a public URL with no key anywhere

Requirements: REQ-DEP-02, NFR-06, REQ-DEP-01, and the TOR §10 deliverable as amended in TOR 1.6.

| # | Acceptance criterion | Evidence | Re-verified 1 Oct | Result |
|---|---|---|---|---|
| 14.1 | `/health` answers | The live smoke test; the CI `container` job in run [34][run34-container] | Yes | Met |
| 14.2 | The reference matrix and the review pages render at the public URL | The live smoke test | Yes | Met |
| 14.3 | No key is present in the image | CI step "Fail if the image holds an .env file or a model key variable", green in run [34][run34-container]; the [`Dockerfile`][dockerfile] | Yes: step result read | Met |
| 14.4 | No key is present in the service configuration | Changed by the decision of 29 September (TOR 1.6): the configuration now holds a reference to a Secret Manager version, not the key. [`deploy.sh`][deploysh] checks exactly that shape before it reports success. The agent has not seen that check's output for the live deploy, and cannot read the configuration. | Partly: the live banner reports a key, and a live upload completed today | Changed by decision; the amended form is not verified |
| 14.5 | The README records both project IDs | [README][readme], Deployment: `spectrace-deploy` and `gen-lang-client-0785466808` | Yes | Met |
| 14.6 | The README records the service URL | [README][readme], Deployment | Yes | Met |
| 14.7 | The README records the teardown steps | [README][readme], "Teardown after the defense" | Yes: read; not executed | Met |
| 14.8 | The README says that decisions on the public demo are anonymous and do not survive the instance stopping | [README][readme], "What the public service does, and what it does not"; the banner; [`WebDemoTests`][t-webdemo]`.TheBannerSaysWhatThePublicDemoIsOnEveryPageAndIsAbsentWithoutIt` | Yes | Met |
| 14.9 | REQ-DEP-02: two distinct project IDs in the README, and a budget alert on the billing-enabled one | The IDs are recorded. The budget alert is "*pending*" in the README, and there is no screenshot. | IDs: yes. Alert: no. | **Partly met**: to be confirmed by the student |
| 14.10 | REQ-DEP-01, NFR-06: CI succeeds offline, with no secret | [`ReproductionDocsTests`][t-docs]`.TheWorkflowReferencesNoSecret`; runs 17 to 34, all green, with `SPECTRACE_OFFLINE: 1` and no secret | Yes | Met |
| 14.11 | Evidence: the service URL, the deploy log, a screenshot of the budget alert | The URL: yes. The first, offline deploy's log was pasted in the session on 29 September and is summarised in the README. The live deploy's log was not received. No budget screenshot. | Partly | Partly met |

---

## 3. Launch

### Reproduce offline

What a reader needs: Git, and the .NET SDK 10.0.x. No API key, no container runtime and no access to the model provider; the network only for `git clone` and the NuGet restore.

```
git clone https://github.com/KaunazDagaz/SpecTrace-dev.git
cd SpecTrace-dev
git checkout 8c9bfcd9848a1416617dc51b32179491128983a2
dotnet run --project src/SpecTrace.Cli -- run --document corpus/rfc6902.txt --offline --out runs/reference
git status
git diff
dotnet run --project src/SpecTrace.Cli -- score --headline --documents corpus/rfc6902.txt,corpus/rfc10050.txt --gold corpus/gold/rfc6902.gold.yaml
git status -- experiments
```

What it printed today, trimmed:

```
quotes         18 returned by the model, 14 located exactly once, 4 ambiguous, 0 not found
register       12 requirements
covered        11 by at least one proposed case
gaps           1
test cases     23, of which 23 not yet reviewed by a person
decision queue 4 items for a person

-  "started_at": "2026-09-24T13:08:30.3300697+00:00",
+  "started_at": "2026-10-01T06:48:56.699598+00:00",
-  "git_sha": "9e37019ee5403e9a85b82fd3a8bcc8e7efc4251e"
+  "git_sha": "8c9bfcd9848a1416617dc51b32179491128983a2"

rfc6902      A   baseline  13 of 16 claims not located
rfc10050     B   pipeline  1 of 30 claims not located
rfc6902      B delivered             precision 100.0% (12/12)   recall 63.2% (12/19)    F1 77.4%   modality 83.3% (10/12)
mode           offline, replayed from the cache; no key is read and no request is sent
```

The last `git status` prints nothing.

The review UI, with the committed review shown, as the image runs it:

```
mkdir -p runs/web/reviews
cp experiments/review/rfc6902-3ff2234db6aa.reviews.jsonl runs/web/reviews/reference.jsonl
dotnet run --project src/SpecTrace.Web -- --offline --public-demo
```

### Redeploy

The student runs this from Cloud Shell; the agent never runs `gcloud` against the student's projects:

```
cd ~/SpecTrace-dev && git pull
bash deploy/deploy.sh spectrace-deploy gen-lang-client-0785466808 europe-north1
bash deploy/smoke-test.sh https://spectrace-5zrm6uxcja-lz.a.run.app --live
```

To switch the running service back to offline without a rebuild, which is option (a) of decision 1, run the kill switch, then the smoke test without `--live`:

```
gcloud run services update spectrace --project spectrace-deploy --region europe-north1 --remove-env-vars SPECTRACE_OFFLINE --remove-secrets GEMINI_API_KEY --cpu-throttling
bash deploy/smoke-test.sh https://spectrace-5zrm6uxcja-lz.a.run.app
```

### Checked against the README

| Command | In the README | How it was checked today |
|---|---|---|
| Reproduce | identical, under "Reproduce" | Run. [`ReproductionDocsTests`][t-docs]`.TheReadmeReproduceCommandIsTheExactCommandCiRuns` passes. |
| Headline | identical, under "Commands", and the command `headline.md` prints | Run. `HeadlineReproductionTests.CiRunsExactlyTheCommandTheHeadlineSaysRebuildsIt` passes. |
| Gold check, review summary, export | identical | Run |
| Local web UI, offline | identical | Run; pages read over HTTP |
| Local demo, `--offline --public-demo` | **missing**: the README named the flag only in prose | Run; fixed on branch `readme-launch-commands` |
| Copying the committed review into place | **missing**: a fresh clone showed the reference run unreviewed | Run in bash and in PowerShell; fixed on the same branch |
| `deploy.sh`, `smoke-test.sh`, the kill switch | identical, under "Deploy, redeploy and check" | `smoke-test.sh --live` run today. `deploy.sh` was not run by the agent: it was exercised against a stand-in for `gcloud` in PRs [#16][d16] and [#17][d17], and its flags were checked against the `gcloud` reference on 29 September. The kill switch was not run. |

The README fix is the one commit on `spectrace-dev` branch `readme-launch-commands`. It touches only the README.

---

## 4. Results

### Conditions

| | RFC 6902 | RFC 10050 |
|---|---|---|
| Document | RFC Editor plain text, April 2013; 26,405 bytes, 1,011 lines | RFC Editor plain text, September 2026, after the model's March 2026 knowledge cutoff, though its drafts were public from February 2025; 31,890 bytes, 876 lines |
| Gold standard | 19 requirements, one annotator, annotated on 27 September 2026, under the rules frozen at [`69ed50e`][c-69ed50e] | none |
| Arms A and B | `gemini-3.5-flash-lite` through the Gemini API, temperature 0. B: 13 calls on 23 September. A: 1 call on 26 September. | Same model and settings. B: 14 calls on 27 September. A: 1 call on 27 September. |
| Arm A0 | "Gemini 3.5 Flash-Lite" as gemini.google.com shows it, captured 26 September 13:05 +03:00. Its version, system prompt and sampling are not disclosed. | The same interface, captured 27 September 11:55 +03:00 |
| Repetitions | One run per arm and document. No arm was sampled twice. | One run per arm |
| Matching | An exact substring match after whitespace is collapsed, with no fuzzy matching; a claim counts only through a quote that locates. A match needs at least 50% overlap of the shorter span, one to one, greedy by overlap size; ties go to the earlier gold requirement, then to the earlier claim. | Only the headline metric: there is no gold standard |

The call dates are each cache entry's `createdAt` in the committed `cache/`.

### Claims whose quote cannot be located (the headline)

| Document | Arm | Claims | Not located | Found once | Found more than once |
|---|---|---:|---:|---:|---:|
| RFC 6902 | A0 chat, illustrative, not reproducible | 18 | 0 (0.0%) | 14 (77.8%) | 4 (22.2%) |
| RFC 6902 | A baseline | 16 | 13 (81.3%) | 1 (6.3%) | 2 (12.5%) |
| RFC 6902 | B pipeline, raw claims | 18 | 0 (0.0%) | 14 (77.8%) | 4 (22.2%) |
| RFC 6902 | B pipeline, delivered register | 12 | 0 (0.0%), by design | 12 (100.0%) | 0 |
| RFC 10050 | A0 chat, illustrative, not reproducible | 21 | 2 (9.5%) | 18 (85.7%) | 1 (4.8%) |
| RFC 10050 | A baseline | 22 | 2 (9.1%) | 18 (81.8%) | 2 (9.1%) |
| RFC 10050 | B pipeline, raw claims | 30 | 1 (3.3%) | 27 (90.0%) | 2 (6.7%) |
| RFC 10050 | B pipeline, delivered register | 17 | 0 (0.0%), by design | 17 (100.0%) | 0 |

### Quality on RFC 6902, against 19 gold requirements

| Arm | Claims | Matched | Precision | Recall | F1 | Modality accuracy |
|---|---:|---:|---:|---:|---:|---|
| A0 chat, illustrative | 18 | 17 | 94.4% | 89.5% | 91.9% | n/a: the arm states no modality |
| A baseline | 16 | 3 | 18.8% | 15.8% | 17.1% | n/a |
| B raw | 18 | 18 | 100.0% | 94.7% | 97.3% | 77.8% (14/18) |
| B delivered | 12 | 12 | 100.0% | 63.2% | 77.4% | 83.3% (10/12) |

**Sensitivity of the matching rule.** At 30% and at 70% overlap, every figure for every arm is the same as at 50%. The smallest overlap of any matched pair is 100%: each located claim quotes the whole sentence or clause around its gold requirement. On RFC 6902 the threshold does not matter. That would change on a document where models quote fragments that straddle sentences.

**The cost of verification.**
- **Calls and tokens.**
  - A: 1 call, 7,647 tokens in and 1,788 out on RFC 6902; 7,930 and 2,220 on RFC 10050.
  - B: 13 calls, 11,708 and 3,374 on RFC 6902; 14 calls, 12,344 and 4,285 on RFC 10050.
  - Every replay is served 100% from the cache.
- **What verification held back from the pipeline's register.** 6 of the 19 gold requirements, 31.6 points of recall: 94.7% raw against 63.2% delivered.
  - Four are quotes that occur twice in the document.
  - Two are one sentence claimed with two readings.
  - All six sit in the human decision queue; none is lost.
  - No pipeline quote failed verification on RFC 6902.
- **What verification denied the baseline.** Of its 13 unlocated quotes, 12 are at or above 0.90 similarity (the measure in [`headline.md`][headline]). They land on 12 gold requirements no located baseline claim matched. The baseline reached 15 of 19 and is credited with 3.
  - The cause, per [`error-analysis.md`][ea] §2: one consistent formatting change, double quotation marks rewritten as single.
  - No unlocated quote in the whole experiment is invented or reworded.

### Review outcomes

From `score --reviews` on the committed log, today:

| Measure | Value |
|---|---|
| Log lines | 5, one author, self-declared (Mikita Besau), 28 September 2026 |
| Test cases decided | 2 of 23: 1 accepted as proposed, 0 edited, 1 rejected |
| Decision-queue items decided | 1 of 4: testable |
| Superseded decisions | 2: a later line on the same case |
| Reviewed matrix | 11 covered, 1 gap, as before review: the rejected case's requirement keeps its other, accepted case |

**Sample sizes, and what is not claimed.**
- Claims per arm number 16 to 30, and there is one run per arm, with no repeated sampling.
- On RFC 10050 the arms differ by one or two claims, which is within what a second run could change.
- Quality rests on one document and one annotator, and no confidence interval was computed.
- Two decided cases cannot estimate an acceptance rate, so Blueprint §6's third measure of usefulness has no figure yet.
- No single example in this note is offered as proof of quality.

What the numbers do say, read by the agent for the student's review:
- On RFC 6902, the controlled comparison, A against B with the same model and verifier, is 81.3% against 0.0% of claims not located. Almost all of A's share is one formatting habit, not invention.
- The uncontrolled chat arm located every quote on RFC 6902, and came within one F1 band of the pipeline's raw claims: 91.9% against 97.3%.
- On the less familiar RFC 10050, the three arms lie close together: 9.5%, 9.1% and 3.3%.

---

## 5. Honest limitations

### Deferred by plan §12, to M3

| What | Where it is required |
|---|---|
| `docs/limitations.md`, `docs/privacy-safety.md`, `docs/agent-worklog.md` | TOR §10 |
| The 5–7 minute demo ending on a gap the system found | TOR §10, Blueprint §6 |

### What does not work, or works only in part

- **The walkthrough is partial**, at 2 of 23 cases and 1 of 4 items. No person has used the edit action on the reference run: 0 edited.
- **A human decision on a quote found more than once changes no register in M2.** The 6 gold requirements held back stay held back, and the one queue decision in the log changed nothing. Error-analysis §10 lists anchoring them as a finding for later.
- **"Testable" is logged but generates no case**, so such a requirement stays a gap.
- **The public service**:
  - has no accounts;
  - shares one daily quota among all visitors;
  - accepts any upload, which only the page's wording keeps to public specifications;
  - keeps nothing past a restart;
  - is not reproducible from the repository for its live runs;
  - can briefly exceed its one-instance cap;
  - is billed by instance since 29 September.
- **`deploy/deploy.sh` always deploys live.** The kill switch turns the running service offline, but the next run of `deploy.sh` turns it live again. An offline redeploy needs a change to the script, which is outside this task's boundaries.
- **The banner, the README and `decisions.md` may overstate the provider's data use.** They say inputs "may be used to improve Google's models". For a developer in the EEA, the provider's terms apply the paid data terms to unpaid quota too: §6, row D5.
- **NFR-04 is still unmet**: the provider and the model are constants in [`LlmClientFactory`][factory], as at M1.
- **Several documents lag the system**: §6, rows D3, D7, D8 and D12.

### Not verified, or verified only in part

| What | Why not |
|---|---|
| A legal reading of the Gemini API terms | Not the agent's to give. The text is quoted, and the decision is the student's. |
| The budget alert on `spectrace-deploy` | The agent has no console access, and the README marks it pending |
| The service configuration: the key only as a secret reference, no plain variable, CPU allocated between requests, 0 to 1 instances | The agent cannot read it, and the live deploy's `deploy.sh` output was not received. Indirect evidence only: the live banner and a completed live upload. |
| Whether secret version 1, the copy with the escape byte, was destroyed | No access |
| Which key the secret holds, and that it was made in `gen-lang-client-0785466808` | The agent never sees the key, by rule; the student created the secret |
| The CI job logs, for example the container's smoke-test output | The logs API answers 403 without a token. Only the step results were read. |
| SPEC-10's expected live-call count, stated before the first live run | No written record |
| Every `dotnet test` in M2 run without the key | Not checkable afterwards; one known lapse during SPEC-11 |
| The Linear cards | No Linear access |
| Whether the supervisor has been told of each divergence | Only the student knows: §6 |
| The sampled section checks as a person's inspection: 10 spans in M1, 13 for RFC 10050 | Written by the agent |
| NFR-07, a full live run within free-tier limits | Not re-run, because it spends quota |
| The cost of the live service so far | No billing access |

---

## 6. Divergences

Every place where the shipped system, or a document, differs from the TOR or the plan.
- **Recorded** says where the divergence is written down, if anywhere.
- **Supervisor told** is for the student to fill in.
- Under TOR §13, an unrecorded divergence is a defect in the documents. The wording proposed below would fix each one. **The agent has not edited the TOR, the Blueprint or the plan.** The student decides.

| # | Divergence | Original | What shipped | Recorded | Supervisor told |
|---|---|---|---|---|---|
| D1 | The reference review | SPEC-13 acceptance and REQ-REV-01: a walkthrough of the full review | 2 of 23 cases and 1 of 4 queue items decided | No. PR [#14][d14] states the counts, but not the gap. | |
| D2 | The size of the gold standard | Blueprint 1.0 §4, approved: 30–50 requirements. TOR 1.0 and 1.1 §9, the same. | 19 requirements under the frozen rules, one annotator | Yes: TOR 1.2 §9, with the §13(b) note in docs PR [#3][s3]. Blueprint 1.1 §4 changes with it but **awaits the supervisor's re-approval**. TOR §9 has not said 30–50 since version 1.2. | |
| D3 | The model | Blueprint §8 and plan §4.5: Gemini Flash | `gemini-3.5-flash-lite` since 23 September (PR [#5][d5]) | Partly: PR #5 and the M1 note §6, row 1. Not in the Blueprint, the plan or `decisions.md`. | |
| D4 | The live public service against the provider's terms | TOR 1.6 §10 and `decisions.md`, 29 September | Live calls on unpaid quota for users in the EEA, which the quoted Use Restriction allows only on Paid Services | No | |
| D5 | Data use on the free tier | Blueprint §4: "on a free usage tier, submitted text may be used by the provider to improve its models". Blueprint §7 and TOR §12 list the policy as open. The same claim is on the banner, in the README and in `decisions.md`. | The provider's terms: for a developer in the EEA, the paid data terms apply to unpaid quota | No | |
| D6 | The budget alert | REQ-DEP-02. The SPEC-14 card: created by 28 September, a screenshot as evidence. | "*pending*" in the README | Partly: the README says it is pending | |
| D7 | The data contracts | Plan §6: `ReviewRecord(string Decision, string? Comment, DateTimeOffset At)`. TOR §7 calls the plan's contracts frozen. | `ReviewRecord(RunId, TestCaseId, Decision, Edit, Author, At)`, plus `QueueResolution`, `CaseEdit` and `LoggedDecision` | No; plan §10.1 describes the log's fields in prose | |
| D8 | CI | Plan §10.3: restore, build, test, end-to-end run, invariants, upload | Adds a Windows job, the headline recomputation and its drift check, the gold check, and the `container` job | Partly: in the README and `CLAUDE.md`; not in the plan | |
| D9 | The gold file name | The SPEC-11 card: `corpus/gold/rfc6902.gold.json` | `rfc6902.gold.yaml` | Yes, in plan §5.1, §8.3 and the rules; only the card line still says `.json` | |
| D10 | NFR-04 | "swappable via configuration" | Constants in code, one provider | Only in the M1 note, §5 and §6 row 10 | |
| D11 | The redistribution terms of RFC 6902 | TOR §12 lists them as open | Recorded in [`corpus/SOURCES.md`][sources]: TLP 5.0, 3.c.i | Recorded in `spectrace-dev`; TOR §12 still open | |
| D12 | Documents that lag the system | — | The README still says the switch to live runs is "*pending its deploy*". `error-analysis.md` §9 says "No review has been recorded yet". Blueprint §8 says Cloud Run is "Free at this traffic level", against instance-based billing since 29 September. | No | |
| D13 | Paths in `CLAUDE.md` and `AGENTS.md` | — | Both still cite `spectrace-docs/spec/tor.md`, `blueprint.md` and `research/implementation-plan.md`, which do not exist | In the M1 note, §8.7 item 1; not fixed | |

The SPEC-14 card's "no model calls" and "no key in the service configuration" were changed on purpose, by TOR 1.6 and `decisions.md` of 29 September. They are not a separate divergence. What remains open there is D4 and D5.

### Proposed wording

**D1.** Either finish the walkthrough, or add to `research/decisions.md`:

> **1 October 2026 — The reference walkthrough is partial.** The committed review of the reference run decides 2 of 23 test cases and 1 of 4 decision-queue items. REQ-REV-01 ("Manual walkthrough of one full document's review") and the SPEC-13 acceptance are partly met in M2. The walkthrough is carried to M3; the review UI, the log and the counts are unchanged.

**D2.** No new wording. The supervisor's re-approval of Blueprint 1.1 closes it.

**D3.** Plan §4.5, the sentence on the default implementation:

> Default implementation targets `gemini-3.5-flash-lite` through the AI Studio key's free tier. It replaced Gemini 3.5 Flash on 23 September 2026, when Flash's free tier allowed 20 requests a day, less than one full run (NFR-07); Flash-Lite allowed 500 (`spectrace-dev` PR #5).

Blueprint §8, the provider row, at its next re-approval: "Gemini Flash-Lite (`gemini-3.5-flash-lite`), behind a swappable interface". With the reason above, and a `decisions.md` entry carrying the same text.

**D4.** If decision 1 is (a): TOR §10 returns to "Deployed review UI (offline mode, no key required)". TOR 1.7's revision row:

> §10 returns to the offline deployment of version 1.5, and §11's live-service risk is removed. The Gemini API Additional Terms (Use Restrictions, last updated 2026-04-28) allow only Paid Services when an API client is available to users in the European Economic Area, Switzerland or the United Kingdom, and the key project has no billing by design (REQ-DEP-02). A same-principle refinement under §13(b).

Plus a `decisions.md` entry quoting the restriction.

If decision 1 is (b), the revision row says instead that the key project moves to Paid Services and that plan §10.2's "Project 2 — no billing" changes. Blueprint §8's remaining-risk row changes with it, which needs the supervisor's re-approval.

**D5.** TOR §12, closing the data-use half of the open item:

> The provider's data-use terms are confirmed (Gemini API Additional Terms, last updated 2026-04-28): content submitted on unpaid quota is used to improve Google's products, except that for developers in the European Economic Area, Switzerland or the United Kingdom the Paid Services data terms apply to all Services, including unpaid quota. The project's developer is in the EEA.

The student confirms the last sentence. Blueprint §4's caveat and §7's open question change to match, at the next re-approval. The banner text is a code change for M3.

**D6.** In the README table, once confirmed: "Budget alert: €N on `spectrace-deploy`, emails at 50%, 90% and 100% of actual spend, created on DATE". Attach the screenshot to the PR that adds it.

**D7.** Plan §6: replace the `ReviewRecord` line with the shipped records, `LoggedDecision`, `ReviewRecord`, `CaseEdit` and `QueueResolution`, with the fields above. Note under TOR §7 that §6 changed in M2.

**D8.** Plan §10.3:

> Two jobs, `ubuntu-latest` and `windows-latest`: restore, build, test, the README Reproduce command, the headline recomputation with a drift check on `experiments/`, the gold check. A `container` job on `ubuntu-latest` builds the image, starts it with no key, runs `deploy/smoke-test.sh` and compares the RFC 6902 run started through the form with `runs/reference/`.

**D9.** In the SPEC-11 card, `rfc6902.gold.json` becomes `rfc6902.gold.yaml`.

**D10.** Either amend NFR-04 to "The LLM provider SHALL be replaceable through a single interface; the model is a named constant" with a §13(b) note, or make it configuration. That is a product decision.

**D11.** TOR §12, closing the other half: "The redistribution terms of both corpus documents are recorded in `corpus/SOURCES.md` in `spectrace-dev` (IETF Trust Legal Provisions 5.0, 3.c.i)."

**D12.**
- The README paragraph: "Switched to live runs on 29 September 2026 (revision …)". If decision 1 is (a), it records the switch back as well.
- `error-analysis.md` §9: "One review is recorded, of 2 of 23 cases (1 accepted, 1 rejected): too few to analyse which cases reviewers reject."
- Blueprint §8: "Cloud Run, scale to zero; instance-based billing since 29 September 2026", at re-approval.

**D13.** §9, change 1.

---

## 7. Control questions

The student answers these personally. Both answers are left empty on purpose. When the student gives them, the agent inserts them verbatim and translates them into the other version.

1. Can I explain M2 and its cards in plain words to someone outside the project?

   Answer:

2. If the agent became unavailable tomorrow, would I understand what was done and why?

   Answer:

---

## 8. Retrospective

Each question gives the facts first, with links to the evidence, and then conclusions. **The conclusions were drafted by the agent for the student's review.** Every conclusion rests on the facts listed above it. Nothing has been changed on the strength of them.

### 8.1 What worked

Facts:

- All 18 CI runs on `spectrace-dev` since 26 September, runs [17][run17] to [34][run34], finished green, on every PR and every merge. The checks re-run today pass, and the headline regenerates byte-identical.
- Measurement defects were caught before they reached the headline:
  - The parser had reported all 20 items of the RFC 10050 baseline answer as claims without a quote. Its own miss was being "counted against the model" ([`f91de60`][c-f91de60]).
  - A free-text answer cut short is now recorded, not refused ([`8d33d6f`][c-8d33d6f]).
- Rules and thresholds were fixed before the figures:
  - The annotation rules merged 35 minutes before the first gold commit (row 11.1).
  - The 0.90 similarity threshold and the modality-blind matching rule were kept even where changing them would have flattered a figure ([`error-analysis.md`][ea], §3 and §6).
- The fail-closed layers caught deployment faults before any visitor saw them:
  - The first deploy's build stopped on an IAM grant that had not yet applied (README).
  - The `--live` smoke check caught a key stored with an escape byte on the first live deploy.
  - [`deploy.sh`][deploysh] now refuses such a secret (PR [#17][d17]).
- `SpecTrace-docs` kept pace with the code. Every card had a docs PR ([#4][s4] to [#8][s8]), and `decisions.md`, cited by the TOR since 1.0, was finally created ([#7][s7]). M1 §8.4 had found these missing.
- The test suite grew from 251 at M1 to 573.

Conclusions (drafted by the agent, for the student's review):

- The headline is a reproducible artifact, not a claim. It holds because every arm goes through one parser and one verifier, and the rules were fixed before any figure existed.
- Each fault that reached a live system in M2 was stopped by a check written for it before it happened. This is the same pattern M1 credited to "verify, don't assume".

### 8.2 Which assumption turned out wrong

Facts:

- **The gold standard.** Blueprint 1.0 expected 30–50 gold requirements. Under rules frozen in advance, RFC 6902 has 19 (TOR 1.2).
- **How an unverified approach fails.** On RFC 6902:
  - The chat arm located all 18 of its quotes.
  - The baseline's 13 unlocated quotes were one formatting habit, and 12 of them reached a gold requirement.
  - No unlocated quote in the experiment is invented or reworded ([`error-analysis.md`][ea], §2 and §3).
- **The chunking check.** It cannot fire on RFC 6902, because no gold requirement lies in the last third (TOR 1.4).
- **The free tier.** On 29 September the agent presented the option to make the public service live, with the quota, data-use, reproducibility and cost risks listed, and the student chose it. The agent did not check the provider's terms, so the EEA restriction was missed (Status, decision 1).
- **The data-use caveat.** Blueprint §4 and the banner assume the free tier may train on inputs. For a developer in the EEA, the provider applies its paid data terms instead (§6, row D5).
- **A sentence on the run list.** Since SPEC-13 it had said that, live with no key, cached documents still run. They do not. This was found only when the live mode was built ([`ba50791`][c-ba50791]).
- **The paste into `read -rs`.** It was assumed to store pasted text as it is. Cloud Shell's bracketed paste wrapped the key in escape codes (PR [#17][d17]).

Conclusions (drafted by the agent, for the student's review):

- External rules failed again, as the quota did in M1: provider terms and terminal behaviour this time. The VERIFY list in `CLAUDE.md` covers endpoints, limits and the regex, not terms of use.
- The experiment supports a narrower statement than the Blueprint's problem description. On RFC 6902, verification's measured value is two things:
  - catching quotes that are not verbatim;
  - routing duplicated and double-read sentences to a person.

  It did not catch invented requirements, because none were invented. RFC 10050's differences of one or two claims are too few to say more.

### 8.3 Where time was lost

Facts:

- **29 September, after SPEC-14.** SPEC-14 merged at 08:16 UTC. By 14:03 UTC four more `spectrace-dev` PRs had followed, with one docs PR: the review log (#14), the deploy record (#15), the scope change to live runs (#16, docs #9), and its fix (#17).
  - The live change was decided, built, deployed, broken by the escaped key, and fixed on one day.
  - The provider-terms conflict was raised two days later, in the brief for this note, and verified while writing it.
- **The wrong merge.** Docs PR [#9][s9], branch `public-live-runs`, merged at 09:24 UTC. The `spectrace-dev` PR [#16][d16], from a branch of the same name, was opened only at 10:56 UTC. In between, Cloud Shell ran the old scripts, and `smoke-test.sh --live` printed its usage, as the student's terminal output, pasted in the agent session, showed.
- **The first deploy** failed with `PERMISSION_DENIED`, because the build's role grant had not yet applied, and was run again (README).
- **SPEC-10.** The RFC 10050 pipeline run stopped on our own validation of an empty boundary input ([`d989828`][c-d989828]). It was fixed in [`8cbc341`][c-8cbc341] and completed in [`ba64d41`][c-ba64d41].
- **SPEC-12.** Two review rounds ([`ed8bbdd`][c-ed8bbdd], [`cfa85f8`][c-cfa85f8]) and a reverted link to an unpublished findings file ([`613cb35`][c-613cb35]), within 30 minutes.
- Hours per card are not recorded anywhere the agent can read.

Conclusions (drafted by the agent, for the student's review):

- The largest loss was the unplanned live change on the day before the freeze. It took most of a day and two failed attempts, the escaped key and the wrong merge, and it led to a terms conflict.
- A scope change late in a milestone skipped the one check that would have stopped it. Under change control (TOR §13), checking the provider's terms belongs before the decision, not after the deploy.

To be filled in by the student, hours per card if tracked:

### 8.4 Which defects repeated

Facts, against M1 §8.4:

- **The PR record.**
  - Every `spectrace-dev` PR in M2 with more than one commit has an empty description: [#9][d9], [#10][d10], [#11][d11], [#12][d12], [#13][d13], [#16][d16].
  - Every PR was merged 6 to 74 seconds after its CI finished. None of PRs #9 to #17 has a review or a comment, as the GitHub API showed today.
  - The descriptions the agent wrote for #13 and #16 were not used.
  - M1 §8.7 proposed items 3 and 9 for exactly this. Neither was applied.
- **Documents behind the system.**
  - The model in the Blueprint and the plan: M1 §6 row 1, still open.
  - The plan's §6 contracts: M1 rows 7 to 9, now again with the review records.
  - Plan §10.3, `error-analysis.md` §9, the README's live paragraph: §6, rows D3, D7, D8 and D12.
- **The paths in `CLAUDE.md`**: M1 §8.7 item 1, still not fixed.
- **Human judgments made by the agent.**
  - M1 row 2.5: the ten sampled spans were written by the agent.
  - In M2: the 13 RFC 10050 sampled spans ([`13797a7`][c-13797a7]), and the error-analysis verdicts, entered on the student's instruction ([`cfa85f8`][c-cfa85f8]).
- **The live test spending quota**: one request during SPEC-11, recorded only in the agent's own memory note. M1 §8.7 item 7 was not applied.
- **Two descriptions of one command set drifting apart.** `CLAUDE.md` listed the `Demo` command and the README did not, until today's fix. `CLAUDE.md` gained 69 lines in M2 and lost 3, mostly descriptions that the README also holds.
- **Line endings** did not recur as a defect in M2. Each new folder still got its own `.gitattributes` rule (+9 lines); M1 §8.7 item 4 was not applied.

Conclusions (drafted by the agent, for the student's review):

- What repeated are defects at handover points that no CI check sees: the PR, the documents, the rules file.
- The M1 note proposed nine rule changes, and none was applied. Four of the repeats above are the ones those changes were meant to stop: the PR record, the paths, the live test's quota and the per-folder line-ending rules. A retrospective that changes nothing repeats.

### 8.5 What should change in the product

Candidates for M3 task selection, not decisions (drafted by the agent, for the student's review):

- Settle the live service (decision 1). If it goes offline, give `deploy.sh` an offline mode, so that a redeploy cannot switch live calls back on.
- Correct the banner's data-use sentence once the student confirms where the developer is (D5).
- Let a logged human decision anchor a quote found more than once, which recovers delivered recall through a human action (P6): error-analysis §10, finding 1.
- The extraction reads `MUST NOT` as `MUST` in both of RFC 6902's `MUST NOT` sentences: error-analysis §10, finding 2. It is a prompt change, so it means new live calls and a new reference run.
- Make NFR-04 true, or amend it (D10). Make `LiveGeminiTests` opt-in, as M1 §8.5 proposed.

### 8.6 What should change in the documents

Facts: §6, rows D1 to D13.

Conclusions (drafted by the agent, for the student's review):

- Group the Blueprint changes into one version for one re-approval: the model (D3), the data-use caveat and open question (D5), and the hosting row (D12).
- Apply D7, D8 and D9 to the plan, and D4 (as decided), D5 and D11 to the TOR, with §13(b) notes.
- Record D1 and the terms decision in `decisions.md`.
- Before the final report, `docs/privacy-safety.md` (M3) should quote the provider's terms as the Status section of this note has them.

### 8.7 What should change in the agent rules

Facts: §8.4. The proposals are in §9.

---

## 9. Agent rules revision

Proposed changes to `CLAUDE.md`, and identically to `AGENTS.md`, which are byte-identical on `main`. Each comes from an M2 episode. Most sharpen an existing rule; the largest removes text. **None is applied. They wait for the student's approval.**

| # | Change | Kind | Reason, and the episode behind it |
|---|---|---|---|
| 1 | The real document paths: `spec/TOR.md`, `BLUEPRINT.md`, `research/IMPLEMENTATION_PLAN.md` | Sharpen | The rules cite files that do not exist; M1 §8.7 item 1, still open (D13) |
| 2 | Add provider terms to the VERIFY list | Sharpen an existing list | The live public service was decided without the provider's EEA restriction (Status, decision 1) |
| 3 | "Never report a check as done…" covers commands handed to the student | Sharpen | The untested `read -rs` command stored the key with an escape byte (PR [#17][d17]) |
| 4 | Every sentence a page prints about the system is pinned by a test | Sharpen the wording rule | The run list's untrue sentence on live mode with no key, from SPEC-13 to [`ba50791`][c-ba50791] |
| 5 | A PR description holds the criteria; an empty one is not ready to merge | Sharpen | Six M2 PRs with empty descriptions; M1 item 3 not applied |
| 6 | One change across both repositories: different branch names, each PR linking the other | Sharpen the branch rule | Docs PR #9 was merged in place of `spectrace-dev` PR #16, both from `public-live-runs` (§8.3) |
| 7 | Run `dotnet test` without the key unless the live path is the task | Sharpen a description into a rule | One request spent during SPEC-11; M1 item 7 not applied |
| 8 | Judgments a criterion gives to a person are entered by the person | Sharpen the human-authored rule | The agent-written sampled spans ([`13797a7`][c-13797a7]) and the agent-entered verdicts ([`cfa85f8`][c-cfa85f8]) |
| 9 | Remove the per-card descriptions that the README already holds; keep only their rules, and add the README to the reading list | Remove | The README and `CLAUDE.md` drifted (the `Demo` command). The SPEC-14 paragraph was written on 28 September and rewritten 17 hours later ([`4f39751`][c-4f39751], [`54ba698`][c-54ba698]). `CLAUDE.md` gained 69 lines in M2. |

The diff below was produced by applying the nine changes to `CLAUDE.md` at `8c9bfcd`. Every target line was checked to occur exactly once. The result shrinks the file from 269 to 216 lines. Hunk 1 holds changes 1 and 9, hunk 3 holds changes 7 and 9, and every other hunk holds one change.

````diff
--- a/CLAUDE.md
+++ b/CLAUDE.md
@@ -13,12 +13,13 @@ This file governs work in `spectrace-dev`. The authoritative documents live in t
 Read, in this order:
 
-1. `spectrace-docs/spec/tor.md` — the frozen, numbered requirements. This is *what* to build.
-2. `spectrace-docs/blueprint.md` §9 — the seven non-negotiable principles. Repeated below, but read them at the source too.
-3. `spectrace-docs/research/implementation-plan.md` §12 — the current milestone and its tasks.
+1. `SpecTrace-docs/spec/TOR.md` — the frozen, numbered requirements. This is *what* to build.
+2. `SpecTrace-docs/BLUEPRINT.md` §9 — the seven non-negotiable principles. Repeated below, but read them at the source too.
+3. `SpecTrace-docs/research/IMPLEMENTATION_PLAN.md` §12 — the current milestone and its tasks.
 4. The Linear card for the specific task you were given.
+5. The `README.md` of this repository — every command, page and folder, which this file does not repeat.
 
 Then, before writing any code, state: which files you read, your plan, and how you will check the result. If required context or access is missing, say so and stop — do not start implementing around a gap.
 
-**Precedence when documents disagree:** `blueprint.md` → `tor.md` → `implementation-plan.md`. The implementation plan is engineering reference, not authority; it may be refined as work reveals better approaches, provided every TOR requirement still holds.
+**Precedence when documents disagree:** `BLUEPRINT.md` → `TOR.md` → `IMPLEMENTATION_PLAN.md`. The implementation plan is engineering reference, not authority; it may be refined as work reveals better approaches, provided every TOR requirement still holds.
 
 ---
@@ -46,5 +47,6 @@ These hold regardless of model, document, or developer. They are the project's r
 - No fuzzy quote matching in the default path. A `--fuzzy` flag may exist for error analysis; it defaults to off and its results never count as verified.
 - A test case cannot exist without at least one requirement ID. Enforce by type, not by convention.
-- Nothing in the output claims completeness. Wording matters here as much as logic.
+- Nothing in the output claims completeness, and every sentence a page or a command prints about what the
+  system does is pinned by a test. Wording matters here as much as logic.
 
 ---
@@ -74,89 +76,26 @@ Deploy:   bash deploy/deploy.sh <deployment project ID> <key project ID> europe-
 ```
 
-Everything above works today; `Deploy` is the student's alone, as described below. `Score` accepts the run ID of any headline row over the document the
-gold file annotates, replays it offline and adds the quality fields to its metrics file.
-`run --arm baseline` sends the one naive prompt in
-`Prompts/baseline.user.md` and scores the answer with the same verifier. `score --claims` scores
-an externally produced answer, such as a chat transcript, through the same parser and verifier.
-`score --headline` always replays from `cache/`, never calls the provider, and rewrites
-`experiments/headline.md`, every metrics file it names and, with `--gold`, the quality reports
-`experiments/{documentId}.quality.md` and `experiments/chunking-decision.md`; CI runs the command written in that
-file and fails if anything under `experiments/` changes. Run it after anything that changes a
-figure, and commit the result.
-
-`score --worksheet` writes the annotation worksheet: every sentence of the de-paginated text that
-carries an uppercase BCP 14 keyword, found by a keyword scan and never by a model, with every
-annotator field empty. It never writes over an existing file. `score --gold` without `--run`
-checks a gold file and loads it only if it names the frozen rules commit recorded in
-`AnnotationRules.FrozenCommit`, every scanned sentence is decided, every value is allowed, and
-every quote is found exactly once through the existing resolver, within its section when the
-entry names one; it lists every problem with its line. CI runs the `Gold` command above.
-
-`run` extracts, verifies, generates test cases for the requirements the model classified
-testable, and writes `manifest.json`, `requirements.json`, `rejected-quotes.json`,
-`decisions.json`, `test-cases.json`, `matrix.json` and `matrix.html` under `runs/{runId}/`, or
-under `--out`. `--offline` and `SPECTRACE_OFFLINE=1` are equivalent: the run replays from
-`cache/`, needs no key, and a cache miss is an error that names the missing request, never a
-network call. Use the flag in anything a reader types, because it works in every shell.
-`extract --document <path>` still does extraction and verification only.
-
-As of SPEC-5, `runs/reference/` holds the committed reference run.
-`tests/SpecTrace.Pipeline.Tests/OfflineEndToEndTests.cs` regenerates it offline, with an HTTP
-handler that refuses every request, and compares every file byte for byte. The only exclusions
-are `started_at` and `git_sha` in `manifest.json`. Do not add to that list to get a test
-passing. CI runs the README `Reproduce` command verbatim on ubuntu-latest and windows-latest;
-`ReproductionDocsTests` fails if the README and `ci.yml` disagree. When the output is meant to
-change, follow README "Changing the reference run on purpose": record any new cache entries
-with one online run to `runs/{runId}/`, then rewrite `runs/reference/` with the `Reproduce`
-command and commit that diff, with the reason, in the PR. Never edit `runs/reference/` by hand.
-
-A live call needs `GEMINI_API_KEY` in the environment — the Windows user environment or the
-shell, never a file in this repository, `.env` included. `tests/SpecTrace.Llm.Tests` holds one
-live check that skips itself when the key is absent, so CI stays keyless. When the key is
-present, every `dotnet test` spends one real request from the daily quota on that check.
-
-As of SPEC-13, `src/SpecTrace.Web` serves the review UI at http://localhost:5000: a run list with
-a new-run form, a run overview that also shows a run in progress, review, and the reviewed
-matrix. Start it from the repository root. A review decision is one JSON line appended to
-`runs/web/reviews/{runKey}.jsonl`, where the run key is `reference` or a run ID; earlier lines
-are never changed, and the latest decision per case or item wins. The reviewed matrix is computed
-on each request from the run's files plus that log, so a decision never rewrites any file the
-pipeline produced. The log is not kept in `runs/{runId}/` because the offline end-to-end test
-compares the file list of `runs/reference/`. An upload identical to a `corpus/*.txt` file runs as
-that corpus document; any other upload is copied to `runs/web/uploads/`, and its cache entries go
-to `runs/web/cache/`, both ignored by git. `export --run <key>` writes the reviewed matrix as
-Markdown and CSV through the renderer the matrix page uses; `score --reviews <log>` counts cases
-accepted, edited and rejected by their latest decision.
-
-The review log is the evidence that a person made each decision, and it is append-only, so a
-stray decision can never be removed. An agent never makes review decisions on the reference run;
-the student does. No test writes under `runs/reference/`: review tests work on a temporary copy
-of a run, and run tests use a temporary runs directory.
-
-As of SPEC-14, the `Dockerfile` builds the review UI as the public demo Cloud Run serves. The image
-sets `SPECTRACE_PUBLIC_DEMO=1` and, by default, `SPECTRACE_OFFLINE=1`. The demo makes the reference
-run read-only on the server, puts a banner on every page saying whether the server runs offline or
-live, and refuses to start live without a key. The image carries the committed reference review log,
-`experiments/review/{runId}.reviews.jsonl`, at `runs/web/reviews/reference.jsonl`. The `container`
-CI job builds the image, starts it offline with no key, runs `deploy/smoke-test.sh` against it, and
-compares the RFC 6902 run started through the form with `runs/reference/`. Since the scope change of
-29 September 2026, the deployed service runs new documents live: `deploy/deploy.sh` sets
-`SPECTRACE_OFFLINE=0` and `GEMINI_API_KEY` as a reference to a pinned version of the Secret Manager
-secret `gemini-api-key` in the deployment project, and the student runs
-`deploy/smoke-test.sh <url> --live` against the live URL. The student deploys from Cloud Shell and
-creates the secret. An agent never deploys, never runs `gcloud` against the student's projects, and
-never asks for, receives or stores a Google credential, a service-account key or the Gemini key. The
-key reaches the service only through that secret reference: never put a key, a build argument for
-one, or a plain variable holding one in the image, the service configuration, this repository or
-CI. `.gitattributes` keeps `*.sh`, `Dockerfile`, `.dockerignore` and `.gcloudignore` LF, because bash
-fails on CRLF.
-
-`corpus/rfc6902.txt` is in the repository as of SPEC-2. `.gitattributes` marks `corpus/**` as
-`-text` so git performs no end-of-line conversion on it: the file is LF on every platform, and
-the character offsets the offset map and section index are tested against are identical on
-Windows and on CI. Do not remove that rule, and do not re-save the corpus with CRLF. As of
-SPEC-5 the same rule covers `cache/**` and `runs/reference/**`, because Git for Windows would
-otherwise convert them to CRLF on checkout and break the byte comparison. The pipeline writes
-LF on every platform.
+Everything above works today; `Deploy` is the student's alone. The README describes what each command
+writes and every page and folder; this file keeps only the rules:
+
+- Run `score --headline` after anything that changes a figure, and commit the result. It always replays
+  from `cache/`, and CI fails if anything under `experiments/` changes.
+- In anything a reader types, use `--offline` rather than `SPECTRACE_OFFLINE=1`: the flag works in every
+  shell.
+- `runs/reference/` changes only through README "Changing the reference run on purpose", never by hand.
+  The byte comparison excludes only `started_at` and `git_sha` in `manifest.json`; do not add to that list
+  to get a test passing.
+- A live call needs `GEMINI_API_KEY` in the environment, never in a file in this repository, `.env`
+  included. Run `dotnet test` with `GEMINI_API_KEY` removed from the environment unless the task is to
+  exercise the live path: when the key is present, the live check spends one real request.
+- The review log is append-only evidence that a person decided. An agent never makes review decisions on
+  the reference run; the student does. No test writes under `runs/reference/`.
+- An agent never deploys, never runs `gcloud` against the student's projects, and never asks for,
+  receives or stores a Google credential, a service-account key or the Gemini key. The key reaches the
+  deployed service only as the Secret Manager reference that `deploy/deploy.sh` sets: never in the image,
+  a build argument, a plain variable, this repository or CI.
+- `.gitattributes` keeps `corpus/**`, `cache/**`, `runs/reference/**` and the committed experiment files
+  `-text`, and `*.sh`, `Dockerfile`, `.dockerignore` and `.gcloudignore` LF. Do not remove those rules,
+  and do not re-save those files with CRLF. The pipeline writes LF on every platform.
 
 ---
@@ -212,8 +151,11 @@ These run in CI against the committed cache. Do not merge with any of them red,
 
 - One Linear issue → one branch → one PR. Branch names carry the issue ID (`spec-2-offset-map`) so Linear links them automatically.
+  When one change needs a PR in each repository, the two branches get different names, and each PR links the other.
 - Work only within the task's stated boundaries. Do not expand the product, and never weaken an acceptance criterion to get a test passing.
 - Hit a blocker: stop and state the fact, the cause, and the options. Do not guess past it or silently pick a direction.
-- Every PR states which acceptance criteria are met and how each one was checked.
-- Never report a check as done that you did not actually run.
+- Every PR description states which acceptance criteria are met and how each one was checked. A PR with an
+  empty description is not ready to merge.
+- Never report a check as done that you did not actually run. This includes a command handed to the student:
+  one the agent has not run in the same kind of environment is labelled untested when it is handed over.
 - Replacing a real integration with a stub is acceptable only as an explicitly agreed interim step, stated in the PR.
 - A new idea does not interrupt the current task — write it to `spectrace-docs` and let it be decided later. A blocking defect is the opposite: fix it or renegotiate the boundary, never hide it.
@@ -248,4 +190,7 @@ These run in CI against the committed cache. Do not merge with any of them red,
 - `runs/` is gitignored except the single reference run.
 - The gold standard in corpus/gold/ and the chat transcripts in experiments/a0/ are human-authored. The agent may load and validate them, but never creates, completes or edits them.
+  The same holds for every judgment an acceptance criterion gives to a person: a verdict in the error analysis,
+  a sampled inspection, a review decision. The agent prepares the material, and the person enters the judgment;
+  if the person asks the agent to enter it, the file says so.
 
 ---
@@ -258,4 +203,6 @@ A claim about an API's behaviour or a library's capability is not verification u
 - Gemini endpoint shape, request format, JSON-mode field names, and current rate limits — these have changed recently and are no longer published as one fixed number
 - the section-header regex, against the real corpus file — RFC formatting varies, and a wrong regex silently mislabels every section
+- the provider's terms of use, before anything lets someone other than the student reach the model — the Gemini API
+  Additional Terms restrict unpaid use by region and set what happens to submitted data
 
 If something in this file or the implementation plan contradicts what you observe when you actually run it, the observation wins. Say so rather than coding around it.
@@ -265,5 +212,5 @@ If something in this file or the implementation plan contradicts what you observ
 ## Out of scope
 
-See `spectrace-docs/spec/tor.md` §2.2–2.3. Do not build anything listed there, however small it looks — scope discipline is graded on this project.
+See `SpecTrace-docs/spec/TOR.md` §2.2–2.3. Do not build anything listed there, however small it looks — scope discipline is graded on this project.
 
 The tempting ones, repeated: no database, no authentication, no SPA or client-side build step, no PDF or OCR input, no executing the generated test cases, no retrieval or embeddings, no abstraction with a single implementation, no requirement diffing across versions, no GitHub or Linear access from the system itself.
````

[blueprint]: ../BLUEPRINT.md
[tor]: ../spec/TOR.md
[plan]: ../research/IMPLEMENTATION_PLAN.md
[decisions]: ../research/decisions.md
[rules]: ../spec/annotation-rules.md
[terms]: https://ai.google.dev/gemini-api/terms
[live]: https://spectrace-5zrm6uxcja-lz.a.run.app
[readme]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/README.md
[ci]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/.github/workflows/ci.yml
[dockerfile]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/Dockerfile
[deploysh]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/deploy/deploy.sh
[factory]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/src/SpecTrace.Pipeline/LlmClientFactory.cs
[headline]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/experiments/headline.md
[ea]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/experiments/error-analysis.md
[chunk]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/experiments/chunking-decision.md
[quality]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/experiments/rfc6902.quality.md
[gold]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/corpus/gold/rfc6902.gold.yaml
[sources]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/corpus/SOURCES.md
[log]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/experiments/review/rfc6902-3ff2234db6aa.reviews.jsonl
[outcomes]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/experiments/review/rfc6902-3ff2234db6aa.review-outcomes.md
[a0-6902]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/experiments/a0/rfc6902.md
[a0-10050]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/experiments/a0/rfc10050.md
[t-span]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Core.Tests/SpanMatchingTests.cs
[t-quality]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Core.Tests/QualityScoreTests.cs
[t-goldcheck]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Core.Tests/GoldCheckTests.cs
[t-reviewedrun]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Core.Tests/ReviewedRunTests.cs
[t-10050]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Core.Tests/Rfc10050SectionIndexTests.cs
[t-headline]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Pipeline.Tests/HeadlineReproductionTests.cs
[t-baseline]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Pipeline.Tests/BaselineArmTests.cs
[t-parser]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Pipeline.Tests/ClaimParserTests.cs
[t-claims]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Pipeline.Tests/ScoreClaimsCommandTests.cs
[t-worksheet]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Pipeline.Tests/AnnotationWorksheetTests.cs
[t-webreview]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Pipeline.Tests/WebReviewTests.cs
[t-reviewlog]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Pipeline.Tests/ReviewLogTests.cs
[t-reviewcmd]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Pipeline.Tests/ReviewCommandsTests.cs
[t-replay]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Pipeline.Tests/RunReplayTests.cs
[t-webdemo]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Pipeline.Tests/WebDemoTests.cs
[t-docs]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/8c9bfcd9848a1416617dc51b32179491128983a2/tests/SpecTrace.Pipeline.Tests/ReproductionDocsTests.cs
[d5]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/5
[d9]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/9
[d10]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/10
[d11]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/11
[d12]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/12
[d13]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/13
[d14]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/14
[d15]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/15
[d16]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/16
[d17]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/17
[s3]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/3
[s4]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/4
[s5]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/5
[s6]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/6
[s7]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/7
[s8]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/8
[s9]: https://github.com/KaunazDagaz/SpecTrace-docs/pull/9
[run17]: https://github.com/KaunazDagaz/SpecTrace-dev/actions/runs/36315111508
[run34]: https://github.com/KaunazDagaz/SpecTrace-dev/actions/runs/36579714655
[run34-container]: https://github.com/KaunazDagaz/SpecTrace-dev/actions/runs/36579714655/job/109444355333
[c-8c9bfcd]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/8c9bfcd9848a1416617dc51b32179491128983a2
[c-975525b]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/975525b60addd60e60a6ffab66648c7dfd455596
[c-ee4e4c7]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/ee4e4c7c2a7cfa35f4d140c2ebaab09f1ae5bb86
[c-01219d2]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/01219d22919492b1b34c4354d14bf47ac7ecf686
[c-28a7979]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/28a797946ce3fc3fdd0449b6def250f6e27883fc
[c-13797a7]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/13797a7165909c1b70bc40626c272c4d27bd3142
[c-f91de60]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/f91de6015adb3c6285470a696f662a93723fc21b
[c-d989828]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/d989828ec845643165117776e5f2ab359a02abf3
[c-8cbc341]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/8cbc3418bff4a8babbf002242710e520d9a8f97c
[c-ba64d41]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/ba64d410d4d5a33a669145bac925f8316270ba22
[c-8d33d6f]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/8d33d6f7052296bc3bf75e22774e362086dfe272
[c-d2c5e60]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/d2c5e60c3de3d0cb8d1b41681e441dfb59fd7ae7
[c-40a4b4d]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/40a4b4d769df4dcaecb1e3a1c4f6e205f4d6f2b3
[c-3657ef3]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/3657ef3473580a4b53b3aa475205ca8e15279902
[c-3c0fc04]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/3c0fc049927a122d392617a1c3f5d634f5e13e2d
[c-ed8bbdd]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/ed8bbddfea455200283be6c6f3d64d9bb51da4d1
[c-cfa85f8]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/cfa85f8128068be11e773f89f0533aed1963e283
[c-613cb35]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/613cb35868cc218665eb5d952cc8e5a83aa1ed4e
[c-d248ee0]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/d248ee04be21ffb7047c7955dee3b7987eb5ce39
[c-4381b69]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/4381b693e77827d95b350b72264382d4523be6b2
[c-9cbd91d]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/9cbd91d486be31e7b7de123ea681517e499c8637
[c-4a51587]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/4a51587aaaad542e4ac46d6cccd6b541a5f2890a
[c-afa89c4]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/afa89c4b5e0cb7df99447039dae16461753688aa
[c-4f39751]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/4f397514bf482470e932f6514ec39d5fa74acb27
[c-c6560cc]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/c6560ccd16a504bee7d6fc125492bc9a54786cde
[c-9c63393]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/9c63393cfee027c580fffa63dc95aacd99e224e3
[c-ba50791]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/ba507917409d3272ec2264e8dde7a0e3dd824ffc
[c-4afb266]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/4afb26655c868bfb0e2684ec2a5a76a15b45e6f5
[c-54ba698]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/54ba6989c7e1ab2fa5c56b85d1ccf67e00bf853b
[c-ac9f09f]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/ac9f09f93287520a7e0c02579112101ddfef6c86
[c-f0556b7]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/f0556b7247374523b95ecb3835891125c6fae425
[c-4e63b1d]: https://github.com/KaunazDagaz/SpecTrace-docs/commit/4e63b1dba3c1a0ec34614d4688225106762ac113
[c-7a64311]: https://github.com/KaunazDagaz/SpecTrace-docs/commit/7a643114980dfede8d7e06b6573d7d671fedfd1c
[c-69ed50e]: https://github.com/KaunazDagaz/SpecTrace-docs/commit/69ed50ea13564b3da3b17705b3bd39336a082349
[c-6540f95]: https://github.com/KaunazDagaz/SpecTrace-docs/commit/6540f950ad6f3b109e024a014fcb76670dfb6f7f
