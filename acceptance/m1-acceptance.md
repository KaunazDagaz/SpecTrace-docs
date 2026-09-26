# M1 acceptance note

[Русская версия](m1-acceptance.ru.md)

| | |
|---|---|
| Milestone | M1 (≈ G2), vertical slice: [implementation plan §12][plan] |
| Version under acceptance | `spectrace-dev` `main` at [`a976acee87b3cd1ecf02bb8788e8d1179ceef85c`][c-a976ace], the merge of PR [#8][pr8] on 25 September 2026 |
| Checks re-run | 26 September 2026, 09:33 UTC, on that exact commit |
| Prepared by | the coding agent (Claude Opus 5.5), for the student's review |
| Acceptance decision | the student merging this PR. No tag exists yet. The student tags the version after sign-off. |

Fields left for the student: the results of criteria 2.5 and M.2 in §2, row 18 in §6, both answers in §7, the hours field in §8.3, and a review of every conclusion in §8.

---

## Status

**No blocker.** Build, tests and the README reproduction command all pass on `a976ace` today.

| Check | Command | Result on 26 September 2026 |
|---|---|---|
| Build | `dotnet build --warnaserror` | Passed: 0 warnings, 0 errors |
| Tests | `dotnet test` | Passed: 251 passed, 0 failed, 1 skipped. The skipped test is `LiveGeminiTests`, which skips itself when no key is set. |
| Offline reproduction | `dotnet run --project src/SpecTrace.Cli -- run --document corpus/rfc6902.txt --offline --out runs/reference` | Passed: `git status` shows only `runs/reference/manifest.json` changed, and `git diff` shows only its `started_at` and `git_sha` lines |

How the checks were run: in a fresh clone from GitHub, checked out at `a976ace`, on Windows 11 (10.0.26200) with .NET SDK 10.0.301 and Git `core.autocrlf=true`. `GEMINI_API_KEY` and `SPECTRACE_OFFLINE` were removed from the environment. The checks were not run in the student's working copy, because it holds extra untracked cache entries that could hide a cache miss.

Tests per project: `SpecTrace.Core.Tests` 76 passed; `SpecTrace.Llm.Tests` 54 passed, 1 skipped; `SpecTrace.Pipeline.Tests` 121 passed.

### Needs a decision before merge (not a check failure)

The accepted code implements **TOR 1.1**, but `SpecTrace-docs` `main` still holds **TOR 1.0**.

- TOR 1.1 changes REQ-MTX-02 and the §3 definition of *Gap*. The same commit changes plan §2.3, §2.4, §7.2 and invariant I4. That commit, `a772f38`, exists only on the local branch `SPEC-8-gap-definition` in the student's clone. It was never pushed, and `SpecTrace-docs` has had no PR before this one.
- `spectrace-dev` PR [#7][pr7] says "TOR 1.1 changes REQ-MTX-02 in SpecTrace-docs", and `CLAUDE.md` and `AGENTS.md` already carry the amended I4. On the published record, the code and the TOR therefore disagree. TOR §13 treats such divergence as "a defect in the document".
- Options: push `SPEC-8-gap-definition` and merge it before this PR; or merge this PR and carry the amendment as an open M2 item. This note treats TOR 1.1 as the version under acceptance, because the code, the tests and `CLAUDE.md` implement it.

### Sources and their status

| Short name | What | Where it is |
|---|---|---|
| TOR, plan, blueprint | [`spec/TOR.md`][tor], [`research/IMPLEMENTATION_PLAN.md`][plan], [`BLUEPRINT.md`][blueprint] | `SpecTrace-docs` `main`; the TOR there is version 1.0 |
| Amendment | TOR 1.1; plan §2.3, §2.4, §7.2, §9 I4 | commit `a772f38`, local branch `SPEC-8-gap-definition`, not pushed |
| N3 | `research/spec-3-notes.md` | student's local clone of `SpecTrace-docs`, untracked |
| N4 | `research/spec-4-notes.md` | student's local clone of `SpecTrace-docs`, untracked |
| N5 | `research/spec-5-notes.md` | commit `6a11b4d`, local branch `SPEC-9-offline-reproduction`, not pushed |
| PRs, commits, CI | `spectrace-dev` PRs #1 to #8, their commits, GitHub Actions runs 1 to 16 | public on GitHub; read through the GitHub API on 26 September 2026 |
| Linear | the M1 cards | **not read**: there was no Linear access in this session. The acceptance criteria in §2 come from plan §12 and the TOR instead. |

Task numbers and Linear IDs name different things (N5 §2). This note uses the task numbers SPEC-1 to SPEC-5 from the commit titles, with the Linear ID beside each.

| Task | Linear issue and branch | PR | Commits |
|---|---|---|---|
| SPEC-1 Repositories and environment | SPEC-5, `SPEC-5-project-setup` | [#1][pr1] | [`7a6b750`][c-7a6b750] |
| SPEC-2 Offset map and section index | SPEC-6, `SPEC-6-offset-map` | [#2][pr2] | [`476c705`][c-476c705] |
| SPEC-3 Extraction and verification | SPEC-7, `SPEC-7-extraction-and-verification` | [#3][pr3] | [`417fa38`][c-417fa38] |
| SPEC-4 Generation, matrix, decision queue | SPEC-8, `SPEC-8-generation-and-matrix` | [#7][pr7] | [`da0936e`][c-da0936e] |
| SPEC-5 Offline reproduction | SPEC-9, `SPEC-9-offline-reproduction` | [#8][pr8] | [`9e37019`][c-9e37019], [`f0556b7`][c-f0556b7] |
| No task: agent comments removed | none, `remove-agent-comments` | [#4][pr4] | [`76b51c4`][c-76b51c4] |
| No task: model switched to Flash-Lite | none, `flash-lite-model` | [#5][pr5] | [`496f4a0`][c-496f4a0] |
| No task: verification fixes | none, `verification-fixes` | [#6][pr6] | [`50c5a73`][c-50c5a73] |

---

## 1. Working scenario

On the accepted version, one command takes RFC 6902 and produces a register of verified requirements, proposed test cases for them, and a traceability matrix with visible gaps.

The pipeline collapses the document's whitespace while keeping a map back to the raw file, and indexes its sections. It sends the whole document in one request to `gemini-3.5-flash-lite` and asks for candidate requirements as quote text, modality and testability, never positions. Code then locates every quote by exact match. A quote that is not found goes to `rejected-quotes.json`. A quote found more than once, or claimed with conflicting readings, goes to the human decision queue. Every other quote becomes a requirement with a content-hash ID, a character span and a section number.

For each requirement the model classified testable, a second call sends only the section number and the quote, and gets back up to three test cases. Code, not the model, attaches the requirement ID. A matrix is derived from the register and the cases alone: each requirement is covered or a gap, and cases naming a missing requirement are listed as orphans. `matrix.html` says at the top that coverage is by proposed, unreviewed cases, and that it does not claim the specification is fully covered.

Every model call passes through a disk cache keyed by a hash of the full request. With `--offline`, the whole run replays from `cache/` with no key and no network, and a cache miss is an error.

The command that demonstrates it, from the repository root. It was run today on `a976ace` and exited with code 0:

```
dotnet run --project src/SpecTrace.Cli -- run --document corpus/rfc6902.txt --offline
```

It writes seven files to `runs/rfc6902-3ff2234db6aa/`; the matrix is `matrix.html` there. Today, the six files other than `manifest.json` were byte-identical to `runs/reference/`.

---

## 2. Evidence, per task

Where the criteria come from: the "Verifiable result" column of plan §12 for each task, split into single checkable statements, plus the acceptance criterion of every TOR requirement with gate G2, placed under the task that delivered it. The Linear cards were not available. If a card states a criterion that is missing here, add a row.

The column **Re-verified 26 Sep** says what was actually run or read today. **Result** is the verdict on the accepted version. It is left empty where only the student can check. "Tests pass" means the named tests were among the 251 that passed in today's `dotnet test` run.

### SPEC-1: repositories and environment are ready

| # | Acceptance criterion | Evidence | Re-verified 26 Sep | Result |
|---|---|---|---|---|
| 1.1 | Both repositories exist | [SpecTrace-dev][dev], [SpecTrace-docs][docs] | Yes: `git ls-remote` on both | Met |
| 1.2 | The repositories are linked (plan §5.1 adds the Linear project) | `spectrace-dev` [`README.md`][readme] links `SpecTrace-docs` and its three documents. `SpecTrace-docs` `main` has no README and no link to `spectrace-dev`. | Yes, both repositories read. Linear links not checked. | Partly met: no link from docs to dev |
| 1.3 | `AGENTS.md` and `CLAUDE.md` are present | Both at the repository root, byte-identical since `7a6b750` | Yes: `cmp AGENTS.md CLAUDE.md` | Met |
| 1.4 | The solution builds | The build check in Status | Yes: `dotnet build --warnaserror` | Met |
| 1.5 | CI runs green with no secrets configured | [`ci.yml`][ci] references no secret; `ReproductionDocsTests.TheWorkflowReferencesNoSecret`; CI run [1][run1] on PR #1 green | Yes: tests pass, `ci.yml` read. The repository's secret settings were not checked; that needs admin access. | Met |
| 1.6 | `.gitignore` excludes `runs/` except the reference run | `.gitignore` lines 485–486: `/runs/*` and `!/runs/reference/` | Yes: after today's scenario run in the fresh clone, `git check-ignore` matched `runs/rfc6902-3ff2234db6aa/` to `.gitignore:485`, and `git ls-files` lists the 7 files of `runs/reference/` | Met |
| 1.7 | The target framework is chosen from the installed SDK and written down (plan §5.2) | `CLAUDE.md` "Environment"; `Directory.Build.props` sets `net10.0` | Yes: `dotnet --list-sdks` reports `10.0.301` only | Met |

### SPEC-2: any quote can be located exactly, with its section

| # | Acceptance criterion | Evidence | Re-verified 26 Sep | Result |
|---|---|---|---|---|
| 2.1 | REQ-ING-01: any substring of the normalised text maps back to its exact place in the raw file, line wraps included | [`OffsetMapCorpusTests`][t-offset]: `EveryNormalisedSubstringOfTheCorpusMapsBackToRawTextThatNormalisesToIt`, `AQuoteWrappedAcrossFourLinesIsLocatedInTheRawFile` | Yes: tests pass | Met |
| 2.2 | The system names the section a span falls in | [`SectionIndexTests`][t-section]; `QuoteVerifierTests.AVerifiedRequirementNamesTheSectionItsSpanFallsIn` | Yes: tests pass | Met |
| 2.3 | The plan §9 normalisation cases pass: tabs, CRLF against LF, indentation, a quote across a line break, quotes at the start and end of the document, empty and whitespace-only quotes | `OffsetMapCorpusTests`: `TabsCollapseLikeAnyOtherWhitespaceRunAndLeaveTheNormalFormUnchanged`, `TheSameQuoteResolvesWhetherTheDocumentUsesLfOrCrlf`, `AQuoteInsideAnIndentedBlockIsLocatedWithoutItsIndentation`, `AQuoteWrappedAcrossOneLineIsLocatedInTheRawFile`, `AQuoteAtTheStartOfTheDocumentSkipsTheSixLeadingBlankLines`, `AQuoteAtTheEndOfTheDocumentStopsBeforeTheTrailingPageBreak`, `AnEmptyQuoteIsRejectedRatherThanMatchedAtTheStartOfTheDocument`, `AWhitespaceOnlyQuoteIsRejectedRatherThanMatchedAtTheStartOfTheDocument` | Yes: tests pass | Met |
| 2.4 | The section-header regex is verified against the corpus | PR [#2][pr2]: the regex suggested in plan §5.5 missed the appendix headers and would have labelled 389 of 1011 lines "9.2". `SectionIndexTests`: `TheIndexFindsEveryHeaderInTheDocumentAndNothingElse`, `TheTableOfContentsDoesNotRegisterAsThirtyFourExtraSections`, `RunningPageHeadersAndFootersDoNotRegisterAsSections`, `AHeaderShapedLineThatIsIndentedIsNotAHeader` | Yes: tests pass | Met |
| 2.5 | REQ-ING-02: section lookups for ten sampled spans match manual inspection | `SectionIndexTests.ASampledSpanReportsTheSectionThatContainsIt` holds ten spans. The agent wrote the expected sections in it, so the test cannot stand in for a person's inspection. | To be confirmed by the student | |
| 2.6 | REQ-VER-01: exact substring match only, no fuzzy matching in the default path | No `--fuzzy` flag exists: "fuzzy" does not occur in `src/` or `tests/`. `OffsetMapCorpusTests.AQuoteThatIsNotInTheDocumentIsRejectedRatherThanApproximated` | Yes: search and tests | Met |

### SPEC-3: the model is asked, and nothing it says is trusted unverified

| # | Acceptance criterion | Evidence | Re-verified 26 Sep | Result |
|---|---|---|---|---|
| 3.1 | Candidate requirements come through a cached, swappable LLM client | `ILlmClient`, composed as `CachingLlmClient(RateLimitedLlmClient(GeminiLlmClient))` in [`LlmClientFactory`][factory]; [`CachingLlmClientTests`][t-cache] | Yes: tests pass, code read | Met, with the limit on "swappable" stated in §5 |
| 3.2 | REQ-VER-02: a claim that is not in the text is dropped and shown separately, and a corrupted quote is rejected, not repaired | `QuoteVerifierTests.AFabricatedQuoteIsDroppedFromTheRegisterAndRecordedVerbatimAsRejected`; `rejected-quotes.json` | Yes: tests pass | Met |
| 3.3 | At least five requirements are verified end to end | `OfflineReplayTests.AtLeastFiveRequirementsAreVerifiedEndToEnd`; the reference run has 12 | Yes: tests and reproduction | Met |
| 3.4 | REQ-EXT-01, NFR-02: the prompt schema contains no offset or line-number field | `ExtractionPromptTests.TheResponseSchemaDeclaresNoFieldThatCouldCarryAPosition`, `ExtractionPromptTests.TheOutputSchemaWrittenInThePromptDeclaresTheSameFieldsAndNoPosition`, and the same two tests in `GenerationPromptTests` | Yes: tests pass | Met |
| 3.5 | REQ-EXT-02: re-running over an unchanged document and cache yields identical IDs | `QuoteVerifierTests.TheSameQuotesGetTheSameIdsOnEveryRunWhateverOrderTheyArriveIn`; `OfflineEndToEndTests.I6RequirementIdsAreIdenticalAcrossTwoRunsAndUniqueWithinEach` | Yes: tests pass | Met |
| 3.6 | REQ-VER-03: a duplicated sentence is marked `ambiguous` and goes to the decision queue | `QuoteVerifierTests.AQuoteFoundMoreThanOnceBecomesAQuestionForAPersonAndNotARequirement`; two such items in the reference run | Yes: tests and reproduction | Met |
| 3.7 | NFR-01: temperature 0, and a cache keyed by the full request | `CachingLlmClientTests.ChangingAnyFieldOfTheRequestChangesTheCacheKey`; all 14 committed cache entries record `temperature` 0 | Yes: tests pass, the 14 entries read | Met |
| 3.8 | NFR-03, I8: `SpecTrace.Core` has no dependency on `SpecTrace.Llm` | [`CoreDoesNotDependOnLlmTests`][t-core], against both the project file and the compiled assembly | Yes: tests pass | Met |

### SPEC-4: test cases, a matrix, visible gaps

| # | Acceptance criterion | Evidence | Re-verified 26 Sep | Result |
|---|---|---|---|---|
| 4.1 | REQ-GEN-01: every requirement classified testable gets proposed cases, grounded only in its own quote | `PipelineRunTests.OnlyTestableRequirementsAreSentForGenerationEachWithItsSectionAndQuoteAlone`, `PipelineRunTests.NoCaseNamesARequirementTheModelFlagged`, `GenerationPromptTests.TheUserPromptIsTheSectionAndTheQuoteAndNothingElse` | Yes: tests pass | Met |
| 4.2 | REQ-GEN-02, I5: a test case cannot be constructed without a requirement ID | `TestCaseTests.ATestCaseThatNamesNoRequirementCannotBeConstructed` | Yes: tests pass | Met |
| 4.3 | I2: every case names a requirement in the register. REQ-MTX-03: a case whose requirement is gone is listed as an orphan, not dropped | `OfflineEndToEndTests.InvariantsI1ToI8HoldOnTheRegeneratedRun`; `RunReplayTests.ARemovedRequirementsCasesLandInTheOrphansSectionRatherThanBeingDropped` | Yes: tests pass | Met |
| 4.4 | The matrix shows covered against gap, and I3, I4 and I7 hold | `TraceabilityMatrixTests`; `OfflineEndToEndTests.InvariantsI1ToI8HoldOnTheRegeneratedRun`; the reference run has 11 covered and 1 gap | Yes: tests and reproduction | Met |
| 4.5 | REQ-MTX-01: deleting and regenerating the matrix from the same inputs gives an identical result | `RunReplayTests.DeletingAndRegeneratingTheMatrixFromTheSameInputsGivesIdenticalBytes` | Yes: tests pass | Met |
| 4.6 | REQ-MTX-02 as in TOR 1.1: a property test over random combinations of requirements, cases and human decisions, and a requirement the model flagged, with no human decision, is a gap | `TraceabilityMatrixTests.StatusIsGapIfAndOnlyIfThereIsNoNonRejectedCaseAndNoHumanDecisionOverRandomCombinations`, `TraceabilityMatrixTests.ARequirementTheModelFlaggedStaysAGapUntilAHumanDecides` | Yes: tests pass | Met against TOR 1.1, which is not on docs `main` (see Status) |
| 4.7 | P7: no output claims completeness | `MatrixHtmlTests.TheTopOfThePageSaysCoverageIsByProposedUnreviewedCasesAndClaimsNoCompleteness`; the text at the top of `runs/reference/matrix.html` read today | Yes: tests pass, page read | Met |

### SPEC-5: anyone can rerun it and get the same answer, with no key

| # | Acceptance criterion | Evidence | Re-verified 26 Sep | Result |
|---|---|---|---|---|
| 5.1 | The full document-to-matrix run replays from the committed cache, offline, with no key, and a test asserts it | [`OfflineEndToEndTests`][t-e2e] runs the CLI once with `SPECTRACE_OFFLINE=1` and once with `--offline`, each with a handler that refuses every request and no key, and compares every artifact with `runs/reference/` byte for byte | Yes: tests pass | Met |
| 5.2 | The README command reproduces the committed reference run | The reproduction check in Status | Yes: command run | Met |
| 5.3 | REQ-DEP-01: the CI job succeeds with `SPECTRACE_OFFLINE=1` and no secrets | `ci.yml` sets `SPECTRACE_OFFLINE: 1` and runs the README command. CI run [16][run16] on `a976ace`: both jobs green, on `ubuntu-latest` and `windows-latest` | Result read today through the GitHub API; CI was not re-triggered | Met |
| 5.4 | The README and CI run the same command | `ReproductionDocsTests.TheReadmeReproduceCommandIsTheExactCommandCiRuns` | Yes: tests pass | Met |
| 5.5 | TOR §9: I1 to I8 run in CI against the committed cache | `OfflineEndToEndTests.InvariantsI1ToI8HoldOnTheRegeneratedRun`, in the CI Test step | Yes locally; in CI through run 16 | Met |

### M1 as a whole

Plan §12: "M1 is accepted when task 5 is green and a human has watched a real matrix come out of a real run."

| # | Acceptance criterion | Evidence | Re-verified 26 Sep | Result |
|---|---|---|---|---|
| M.1 | Task 5 is green | The SPEC-5 rows above | Yes | Met |
| M.2 | A human has watched a real matrix come out of a real run | No record of it in the PRs or commits | To be confirmed by the student | |

---

## 3. Reference run in numbers

Source: `runs/reference/` at `a976ace`, counted from the files today and matched against the CLI summary of today's reproduction. Run ID `rfc6902-3ff2234db6aa`, model `gemini-3.5-flash-lite`, temperature 0.

**These numbers describe one run over one document. Quality against a ground truth is measured in M2, and no number here is a claim about quality.** In particular, 12 is how many requirements the model returned and the code verified. How many requirements RFC 6902 actually contains stays unknown until the M2 gold standard exists.

| Measure | Value | From |
|---|---|---|
| Quotes returned by the model | 18 claims, covering 15 distinct sentences | cache entry `bde351f7…`; CLI summary |
| Located exactly once in the source | 14 claims | CLI summary |
| … of which became requirements in the register | 12 requirements, all `MUST`, all classified `testable` by the model | `requirements.json` |
| … of which went to the decision queue with conflicting readings | 2 claims on 1 sentence, in §5: `SHOULD` and `MUST_NOT` | `decisions.json` |
| Found more than once (`ambiguous`) | 4 claims on 2 sentences, each of which occurs twice in RFC 6902 | `decisions.json` |
| Rejected: not found in the source | 0 | `rejected-quotes.json` |
| Verification rate, as plan §8.5 defines it: verified quotes ÷ quotes returned | 14 / 18 = 77.8% | CLI summary |
| Test cases | 23: 11 positive, 12 negative, 0 boundary. All 23 `proposed`; none reviewed. | `test-cases.json` |
| Covered by at least one proposed case | 11 | `matrix.json` |
| Gaps | 1: `REQ-rfc6902-5a829e`, §4.1 | `matrix.json` |
| Marked not testable or deferred by a person | 0 | `matrix.json` |
| Orphan cases | 0 | `matrix.json` |
| Decision-queue items | 4: 2 found more than once, 1 with conflicting readings, 1 generation blocked | `decisions.json` |
| Model calls | 13: 1 extraction and 12 generation. All 13 served from the cache. | `manifest.json` |
| Tokens | 11,708 input and 3,374 output. Extraction 7,980 and 1,153; generation 3,728 and 2,221. | `manifest.json`; the 13 cache entries |

The token totals in `manifest.json` equal the sum over the 13 cache entries.

Two facts about this run, neither of them about quality. First, all 4 claims not counted as verified are sentences that genuinely occur twice in the document, and no claim failed to be found. Second, the one gap is the quote "When the operation is applied, the target location MUST reference one of:", which stops before the list it introduces. The model blocked generation on it and gave its reason, which `decisions.json` records.

---

## 4. How to reproduce

What a reader needs installed:

- Git.
- .NET SDK 10.0.x. `global.json` asks for `10.0.100` or a later feature band.

Network access is needed only for `git clone` and the NuGet package restore. No API key, no access to the model provider, no database, no container runtime.

The commands:

```
git clone https://github.com/KaunazDagaz/SpecTrace-dev.git
cd SpecTrace-dev
git checkout a976acee87b3cd1ecf02bb8788e8d1179ceef85c
dotnet run --project src/SpecTrace.Cli -- run --document corpus/rfc6902.txt --offline --out runs/reference
git status
git diff
```

The fourth line is the README "Reproduce" command, verbatim. What the reader should see is what was observed today. The command prints:

```
spectrace 1.0.0

document       rfc6902
model          gemini-3.5-flash-lite
mode           offline
extraction     from cache, 7980 tokens in, 1153 out
generation     12 calls, 12 from cache, 3728 tokens in, 2221 out

quotes         18 returned by the model, 14 located exactly once, 4 ambiguous, 0 not found
register       12 requirements
covered        11 by at least one proposed case
gaps           1
blocked        1 generations returned no case
orphans        0

test cases     23, of which 23 not yet reviewed by a person
  positive     11
  negative     12
  boundary     0

decision queue 4 items for a person
  generation_blocked                           1
  quote_found_more_than_once                   2
  same_quote_claimed_with_different_readings   1

written to     runs/reference

Coverage is by proposed, unreviewed test cases. The matrix does not claim the specification is fully covered: it shows only whether each requirement in the register has a proposed case.
```

Then `git status` lists only `runs/reference/manifest.json` as modified, and `git diff` changes two lines. `started_at` becomes the time of the reader's run. `git_sha` becomes the commit they built: the committed value is `9e37019ee5403e9a85b82fd3a8bcc8e7efc4251e`, and a build of `a976ace` records `a976acee87b3cd1ecf02bb8788e8d1179ceef85c`. The two commits hold identical code: `git diff 9e37019 a976ace` touches only `runs/reference/manifest.json`. Every other byte is what was committed.

Optionally, `dotnet build --warnaserror` and `dotnet test` should report 0 warnings, and 251 passed and 1 skipped, when `GEMINI_API_KEY` is not set. With the key set, the live test spends one real request from the daily quota.

---

## 5. Honest limitations

### Not built in M1, deferred by plan §12

| What | Requirement | Milestone |
|---|---|---|
| Review UI: accept, edit or reject a case, with an append-only decision log | REQ-REV-01, REQ-REV-02 | M2 |
| Baseline arm, gold standard, scoring, metrics, error analysis. `score` prints usage and exits non-zero. | REQ-EXP-01 to REQ-EXP-05 | M2 |
| The chunking decision, from recall by position | REQ-EXT-03 | M2 |
| A second, less-known document for the contamination check | TOR §12, REQ-EXP-05 | M2 |
| Deployment to Cloud Run, with two separate cloud projects | REQ-DEP-02 | M2 |
| `docs/limitations.md`, `docs/privacy-safety.md`, `docs/agent-worklog.md`, the demo | TOR §10 | M3 |

`export` also appears in the CLI usage text. It is not implemented, and no task lists it.

### What this means for the M1 output

- **No human decision exists anywhere.** All 23 cases are `proposed`. "Covered" means that a model proposed a case no person has read yet. The not-testable and deferred statuses have never appeared in a real run; they exist only in unit tests.
- **Four real requirements are missing from the register.** "The target location MUST exist for the operation to be successful." occurs in §4.2 and §4.3, and "The "from" location MUST exist for the operation to be successful." in §4.4 and §4.5. Without context, neither can be resolved to one span, so both wait in the decision queue (N3 §4).
- **One sentence carries two obligations.** The §5 sentence has a `SHOULD` and a `SHALL NOT`. The model returned it twice with different modalities, so it went to the queue with both readings, and neither reading is in the register (PR [#6][pr6]).
- **The only gap comes from extraction.** It is a list lead-in quoted without its list (N4 §2).
- **The model classified all 18 claims `testable`.** The `needs_human_decision` and `not_testable` paths are exercised only by test fixtures.
- **Contamination.** RFC 6902 is well known and very likely in the model's training data (plan §14).

### Not verified, or verified only in part

- The ten section lookups (2.5) and a human watching a real run (M.2) are the student's to confirm.
- NFR-07, a full live run within free-tier limits without manual intervention, was not re-run, because it spends daily quota. The recorded live generation took 6.5 minutes for 12 calls, against about 1.5 minutes of pure rate limiting. Retries are not logged, so the cause is a guess (N4 §5). On `gemini-3.5-flash`, the free tier gave 20 requests a day, "less than one full run" (PR [#5][pr5]).
- The live Gemini path was not exercised today: `LiveGeminiTests` skipped because no key was set. The newest recorded live response is from 23 September 2026.
- NFR-04, "swappable via configuration": the provider `gemini` and the model `gemini-3.5-flash-lite` are constants in [`LlmClientFactory`][factory]. Switching is a code change, not a configuration change, and only one provider implementation exists. P4's promise, that a swap changes quality but not correctness, has not been tested with a second provider.
- Daily-quota detection treats a `quotaId` containing "PerDay" as exhausted. The values of that field are not documented (PR [#5][pr5]).
- The generation schema sends no `maxItems`. The parser enforces "at most 3 cases" and fails the run above that (N4 §3).
- CI's "Reproduce" step proves that the README command runs on both platforms. The byte-for-byte comparison is done earlier, by `OfflineEndToEndTests` in the "Test" step, not by the "Reproduce" step.
- `git_sha` in `manifest.json` comes from the build. A build without `.git` records `unknown`, and a dirty working tree is not flagged (N5 §3).
- `extract` writes no manifest (N5 §4).
- One of the 14 committed cache entries, the `gemini-3.5-flash` extraction `246ce833…`, is not used by the reference run (N5 §5).
- `LiveGeminiTests` spends a real request whenever `GEMINI_API_KEY` is set (N4 §4, N5 §6).
- Two TOR §12 items are still open: the provider's free-tier data-use policy, and the redistribution terms of RFC 6902.
- Only Windows and Linux are tested.

---

## 6. Changes against the original plan

"Updated" says whether the document that held the original has been changed to match.

| # | Change | Original | What M1 did | Decided in | Updated |
|---|---|---|---|---|---|
| 1 | Model | Blueprint §8, plan §4.5: Gemini Flash on the AI Studio free tier | First live extraction on `gemini-3.5-flash` (cache entry `246ce833…`, 23 September). Then switched to `gemini-3.5-flash-lite`: Flash's free tier gave 20 requests a day, less than one full run, so NFR-07 did not hold. Flash-Lite gives 500. | PR [#5][pr5] | No: blueprint §8 and plan §4.5 still say Flash |
| 2 | Provider behaviour | Blueprint §8: Gemini. Plan §6: "exponential backoff on HTTP 429" | Provider unchanged: Gemini `generateContent`, key in the `x-goog-api-key` header. Added: the retry hint is read from the error body, because Gemini sends no `Retry-After`; 503 is treated as transient; an exhausted daily quota fails fast with `LlmQuotaExhaustedException` | PR [#3][pr3], PR [#5][pr5], N3 §3 | No |
| 3 | REQ-MTX-02, the definition of *Gap*, I4 | TOR 1.0: a gap "if and only if" there are zero non-rejected cases; plan I4 the same | TOR 1.1: a gap unless there is a non-rejected case or a logged human decision; the model's classification never moves a row. Plan §2.3, §2.4 and I4 amended; I4 in `CLAUDE.md` and `AGENTS.md` amended. | Amendment `a772f38`, local and unpushed; dev PR [#7][pr7] | Dev side yes. Docs side **not on `main`**. The amended TOR §3 table also runs the *Gap* and *Arm* rows together on one line. |
| 4 | Generation user prompt | Plan §7.2 gave the system prompt only | The user prompt is the section number and the whitespace-normalised quote, nothing else; the document is never sent to a generation call | PR [#7][pr7] | Plan §7.2 on the amendment branch only |
| 5 | Section-header regex | Plan §5.5: `^(\d+(?:\.\d+)*)\.?\s+(\S.*)$` | Headers at column 0 only, plus appendix headers (`Appendix A.`, `A.1.` to `A.16.`) with a required trailing dot | PR [#2][pr2] | No |
| 6 | Verification rate and conflicting readings | Plan §8.5: verified quotes ÷ quotes returned. Plan §2.4 lists "a sentence carrying more than one obligation" for the queue. | SPEC-3 first computed the rate over distinct outcomes (86.7%) and kept only the first reading of a quote claimed twice. Corrected to plan §8.5 (77.8%), with conflicting readings sent to the queue. | PR [#6][pr6] | Plan §2.4 on the amendment branch only |
| 7 | `LlmRequest` contract | Plan §6, which TOR §7 calls frozen | Gained `PromptSha256`, because the cache key must include the prompt file hash | PR [#3][pr3], N3 §2 | No |
| 8 | Decision queue as written to disk | Plan §6: `DecisionQueueItem(Id, Quote, Section, Question, Resolution)` | The Core record is unchanged. `decisions.json` is written from `QueuedDecision` in `SpecTrace.Pipeline`, which adds `reason`, `verification`, `requirement_id`, `blocked_reason` and `claims` | PR [#6][pr6], PR [#7][pr7] | No |
| 9 | Human decisions in Core | Plan §6 has no type for them | `HumanCoverageDecision` and `HumanCoverageVerdict` added to `SpecTrace.Core` as an input to the matrix. M1 passes none. | PR [#7][pr7] | No |
| 10 | NFR-04, "no more than two implementations" | Plan §6 lists a provider and two decorators | Read as counting providers | N3 §2 | Open question |
| 11 | Windows CI job | Plan §10.3 names no platform; SPEC-1's CI ran on `ubuntu-latest` only | `ubuntu-latest` and `windows-latest`, each running the README command verbatim | [`9e37019`][c-9e37019] | README and `CLAUDE.md` describe it; the plan does not |
| 12 | `.gitattributes` | Not in the plan | `-text` on `corpus/**` (SPEC-2), `src/SpecTrace.Pipeline/Prompts/**` (SPEC-3), `cache/**` and `runs/reference/**` (SPEC-5). The pipeline writes LF on every platform. | [`476c705`][c-476c705], [`417fa38`][c-417fa38], [`9e37019`][c-9e37019] | `CLAUDE.md` yes; the plan no |
| 13 | CLI surface | TOR §8: `run`, `score`, `export`. Plan §9: offline through `SPECTRACE_OFFLINE=1` | Added `extract`, `--offline` (equivalent to the variable, and it works in any shell) and `--out`. `score` and `export` are not implemented. | PR [#3][pr3], [`9e37019`][c-9e37019] | README and `CLAUDE.md` yes; the plan no |
| 14 | No-comments rule | Not in plan §13's `AGENTS.md` | Added to `CLAUDE.md` and `AGENTS.md` after agent comments were stripped twice | PR [#4][pr4] | `CLAUDE.md` yes |
| 15 | Work outside the five tasks | Plan §0 and `CLAUDE.md`: one Linear issue, one branch, one PR | PRs #4, #5 and #6 came from branches with no issue ID | PRs [#4][pr4], [#5][pr5], [#6][pr6] | Whether they had cards: the student to confirm |
| 16 | Schedule | Plan §12: M1 due 20 September | The last M1 PR merged on 25 September | PR [#8][pr8] | No |
| 17 | Documentation layout | Plan §5.1: `blueprint.md`, `spec/tor.md`, `research/implementation-plan.md`, a `README.md`. The TOR cites `research/decisions.md`. | Upper-case file names, no README. `research/decisions.md` has never existed. `CLAUDE.md` cites the lower-case names. | N3 §1, N5 §1 | No |
| 18 | Scope changes on the Linear cards | — | Not visible to the agent | — | To be filled in by the student: |

---

## 7. Control questions

The student answers these personally. Both answers are left empty on purpose.

1. Can I explain M1 and its tasks in plain words to someone outside the project?

   Answer:

2. If the agent became unavailable tomorrow, would I understand what was done and why?

   Answer:

---

## 8. Retrospective

Each question gives the facts first, with links to the evidence, and then conclusions. **The conclusions were drafted by the agent for the student's review.** Every conclusion rests on the facts listed above it. Nothing has been changed on the strength of them.

### 8.1 What worked

Facts:

- All 16 CI runs on `spectrace-dev`, runs [1][run1] to [16][run16], finished green, on every PR and on every merge commit. The three checks re-run today pass.
- The reference run reproduces byte for byte on Linux and Windows. Today it replayed 13 of 13 model calls from the cache, with no key.
- No fabricated quote reached the register: 0 of 18 claims failed verification in the reference run, and the fabricated-quote fixture is rejected (`AFabricatedQuoteIsDroppedFromTheRegisterAndRecordedVerbatimAsRejected`).
- Checking against the real corpus and the real API found three errors in the plan before they shipped: the section regex (PR [#2][pr2]), the missing `Retry-After` header (N3 §3), and the unconfirmed `maxItems` keyword (N4 §3).
- The contradiction between TOR 1.0 REQ-MTX-02 and the plan's statuses surfaced during SPEC-4 and was resolved by amending the TOR, not in code alone (`a772f38`, PR [#7][pr7]).
- 16 ideas and observations went into notes instead of interrupting a task: 5 in N3, 5 in N4, 6 in N5.
- The test suite grew from 138 passing at SPEC-3 (PR [#4][pr4]) to 251 today.

Conclusions (drafted by the agent, for the student's review):

- For M1, the project's central bet held: verification and the cache make the output checkable and replayable without trusting either the model or the key.
- "Verify, don't assume" paid for itself. Each of the three plan errors would have been silent: wrong section labels, retries that ignore the provider's hint, a schema keyword that may be ignored.

### 8.2 Which assumption turned out wrong

Facts:

- Blueprint §8 chose Gemini Flash for "Zero marginal cost at this scale". The 3.5 Flash free tier gave 20 requests a day, less than one full run (PR [#5][pr5]). Across 20–23 September the quota stayed exhausted for longer than any retry hint said (N3 §3).
- The plan §5.5 regex would have labelled 389 of 1011 lines "9.2" (PR [#2][pr2]).
- TOR 1.0 REQ-MTX-02 could not hold together with the plan's `NotTestable` and `DeferredByHuman` statuses (`a772f38`).
- The plan §6 contracts, called frozen, needed a new field, `PromptSha256`, and a richer queue record (N3 §2; PR [#6][pr6]).
- The plan treated a quote found twice as an edge case, to "count how often it happens" (§14). In the real run it affects 2 of 15 distinct sentences, and 4 real requirements (N3 §4).
- The plan expected M1 by 20 September. It finished on 25 September.

Conclusions (drafted by the agent, for the student's review):

- The assumptions that failed were about outside facts: vendor quotas, the RFC's layout, how often the text repeats itself. The architecture held. Quota needs its own budget in the M2 plan, because the baseline arm and any second model will each spend it (N3 §3).
- "Frozen" in TOR §7 promises more than the plan could keep. Either the contracts get a version and a revision history like the TOR, or the TOR stops calling them frozen.

### 8.3 Where time was lost

Facts:

- SPEC-2 merged on 20 September at 17:40 UTC. The SPEC-3 PR opened on 23 September at 07:07 UTC. N3 §3 records the Flash quota as exhausted across 20–23 September.
- Work done twice:
  - agent comments stripped after SPEC-3 (the second part of [`417fa38`][c-417fa38]), then again from the SPEC-1 and SPEC-2 files (PR [#4][pr4]);
  - the extraction re-recorded on a new model (PR [#5][pr5]);
  - the verifier and the verification rate corrected after merge (PR [#6][pr6]);
  - the reference run regenerated because its manifest named an earlier commit ([`f0556b7`][c-f0556b7]).
- The 12 live generation calls took 6.5 minutes, against about 1.5 minutes of rate limiting (N4 §5).
- Hours per task are not recorded anywhere the agent can read.

Conclusions (drafted by the agent, for the student's review):

- The largest single loss was the wait on the free-tier quota between SPEC-2 and SPEC-3, and the quota was spent on a model that was then replaced. Checking the per-day limit before choosing the model, a VERIFY item in TOR §12, would have saved it.
- Most of the rework was agent output missing either a rule that already existed (no comments) or a definition already in the plan (§8.5). Both are cheaper to catch before the PR than after the merge.

To be filled in by the student, hours per task if tracked:

### 8.4 Which defects repeated

Facts:

- **Line endings on Windows**, in four tasks: the corpus (SPEC-2, [`476c705`][c-476c705]), the prompt files (SPEC-3, [`417fa38`][c-417fa38]), the JSON artifacts (SPEC-4, [`da0936e`][c-da0936e]), and the cache writer, which wrote CRLF (SPEC-5, [`9e37019`][c-9e37019]).
- **Agent-written comments**, removed twice ([`417fa38`][c-417fa38], PR [#4][pr4]) before a rule forbade them.
- **PR records**:
  - PRs [#1][pr1] and [#8][pr8] have no description;
  - none of the eight PRs lists its acceptance criteria one by one with how each was checked, which `CLAUDE.md` requires;
  - no PR has a human review or comment; the only reviews are two Copilot comments saying it could not review because of a quota limit;
  - six of the eight PRs were merged 61 to 126 seconds after they were opened; PR #5 after 1 hour 40 minutes, PR #6 after 5 minutes 25 seconds.
- **Numbering**: PR [#1][pr1] is titled "SPEC-5-projec setup", PR [#3][pr3] "SPEC-7: …" over a commit titled "SPEC-3: …", and PR [#8][pr8] "Spec 9 offline reproduction" (N5 §2).
- **Documentation written but not published**: N3 and N4 are untracked; N5 and the TOR 1.1 amendment are on unpushed branches; `SpecTrace-docs` has had no PR before this one.
- **Wrong paths in documents**: the TOR cites `research/decisions.md`, which does not exist (N3 §1), and `CLAUDE.md` cites lower-case file names (N5 §1).
- **The live test spending quota** (N4 §4, N5 §6).

Conclusions (drafted by the agent, for the student's review):

- The line-ending defect came back because each fix covered only the file at hand. One general rule would have prevented the next occurrence: everything the pipeline writes or hashes is LF, and every committed data folder gets `-text`.
- The PR record is the weakest evidence in M1. CI passed every time, but the record shows no review, and no PR maps criteria to checks; this note had to rebuild that mapping. If review happened outside GitHub, it left no trace a supervisor can see.
- `SpecTrace-docs` is behind the code. The rule "write it to `spectrace-docs`" was followed, but a note that is written and never pushed is invisible to everyone else.

### 8.5 What should change in the product

Facts: N3 §4 (repeated sentences), N4 §1 (decisions naming a requirement no longer in the register), N4 §2 (list lead-ins), N4 §5 (retries not logged), N5 §3 (where `git_sha` comes from), N5 §4 (`extract` writes no manifest), and NFR-04 as stated in §5.

Conclusions (drafted by the agent, for the student's review). These are candidates for M2 task selection, not decisions:

- Measure before changing extraction. Decide on section-scoped disambiguation of repeated sentences, and on list lead-ins, only after the M2 gold standard shows what each costs in recall.
- Log retry counts and waits in the manifest, so that NFR-07 becomes a measurement instead of a guess.
- Make the provider and model configuration rather than constants, or amend NFR-04 to say what M1 actually does.
- Make `LiveGeminiTests` opt-in through its own variable.

### 8.6 What should change in the documents

Facts: rows 1, 3 to 9, 16 and 17 of §6, and "Needs a decision before merge" in Status.

Conclusions (drafted by the agent, for the student's review):

- Push the TOR 1.1 amendment and merge it, and fix the *Gap* and *Arm* rows in TOR §3.
- Update blueprint §8 and plan §4.5 to name `gemini-3.5-flash-lite`, and say why.
- Update plan §5.5 (the regex) and plan §6 (`PromptSha256`, `HumanCoverageDecision`, the queue record written to disk).
- Correct the M1 dates in plan §12, and re-plan M2, which is due 27 September.
- Add a `README.md` to `SpecTrace-docs` that links `spectrace-dev`, which closes criterion 1.2.
- Settle the file names, by renaming or by fixing the references, and restore or remove `research/decisions.md`.
- After acceptance, update the "Status" section of the `spectrace-dev` README, which still says "Milestone M1 (vertical slice) is in progress."

### 8.7 What should change in the agent rules

Facts: §8.4.

Conclusions (drafted by the agent, for the student's review). Proposed changes to `CLAUDE.md`, and identically to `AGENTS.md`. **None of them is applied in this task.**

1. **Paths.** Replace `spectrace-docs/spec/tor.md`, `spectrace-docs/blueprint.md` and `spectrace-docs/research/implementation-plan.md` with the real names `spec/TOR.md`, `BLUEPRINT.md` and `research/IMPLEMENTATION_PLAN.md`, or rename the files in `SpecTrace-docs`.
2. **A docs change is published.** Under "Workflow", add: "A change to `spectrace-docs` is done only when it is pushed and has its own PR. The `spectrace-dev` PR that depends on it links that PR. A dev PR that implements an amended requirement is not merged before the amendment is."
3. **PR description.** Under "Workflow", add: "Every PR description has an acceptance-criteria table: the criterion, the evidence (test name, file or command), and how it was checked. A PR with no description is not ready for review."
4. **Line endings.** Under "Code conventions", add: "Every file the pipeline writes, and every file whose hash goes into a cache key, uses LF on every platform. A new committed data folder gets a `-text` rule in `.gitattributes` in the same PR."
5. **One naming scheme.** Replace the branch example `spec-2-offset-map` with the scheme the student chooses: either commit titles, PR titles and branch names all use the Linear issue ID, or they all use the task number and the PR body gives the Linear ID.
6. **Work without a card.** Under "Workflow", add: "Work outside the current task, even a fix, gets its own Linear issue before it gets a branch."
7. **Quota.** Under "Commands", add: "Run `dotnet test` with `GEMINI_API_KEY` removed from the environment, unless the task is to exercise the live path."
8. **Order of the reference run.** In the reference-run paragraph, add: "Commit the code first. Then regenerate `runs/reference/` with the Reproduce command, in a commit of its own, so that `git_sha` names the code that produced it."
9. **Review.** Plan §13's line "Do not merge on a green CI alone; the diff must be read." did not carry over into `CLAUDE.md`. Restore it under "Workflow".

[blueprint]: ../BLUEPRINT.md
[tor]: ../spec/TOR.md
[plan]: ../research/IMPLEMENTATION_PLAN.md
[dev]: https://github.com/KaunazDagaz/SpecTrace-dev
[docs]: https://github.com/KaunazDagaz/SpecTrace-docs
[readme]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/a976acee87b3cd1ecf02bb8788e8d1179ceef85c/README.md
[ci]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/a976acee87b3cd1ecf02bb8788e8d1179ceef85c/.github/workflows/ci.yml
[factory]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/a976acee87b3cd1ecf02bb8788e8d1179ceef85c/src/SpecTrace.Pipeline/LlmClientFactory.cs
[t-offset]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/a976acee87b3cd1ecf02bb8788e8d1179ceef85c/tests/SpecTrace.Core.Tests/OffsetMapCorpusTests.cs
[t-section]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/a976acee87b3cd1ecf02bb8788e8d1179ceef85c/tests/SpecTrace.Core.Tests/SectionIndexTests.cs
[t-cache]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/a976acee87b3cd1ecf02bb8788e8d1179ceef85c/tests/SpecTrace.Llm.Tests/CachingLlmClientTests.cs
[t-core]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/a976acee87b3cd1ecf02bb8788e8d1179ceef85c/tests/SpecTrace.Pipeline.Tests/CoreDoesNotDependOnLlmTests.cs
[t-e2e]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/a976acee87b3cd1ecf02bb8788e8d1179ceef85c/tests/SpecTrace.Pipeline.Tests/OfflineEndToEndTests.cs
[pr1]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/1
[pr2]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/2
[pr3]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/3
[pr4]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/4
[pr5]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/5
[pr6]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/6
[pr7]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/7
[pr8]: https://github.com/KaunazDagaz/SpecTrace-dev/pull/8
[c-7a6b750]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/7a6b7507d28ba824f01789b6e4449361bd4d7268
[c-476c705]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/476c705726fd0374cc1282874cb8eb9979778895
[c-417fa38]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/417fa386f5269bf106fd40faefe96f3da8c7d724
[c-76b51c4]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/76b51c4bf74b5daf06cd5c5038ee17be40b693fe
[c-496f4a0]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/496f4a0ac2dbc6ded3f37aa9067b21011f516d59
[c-50c5a73]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/50c5a73ddaa9a216c2b017dae53bd316667fce79
[c-da0936e]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/da0936e01fe963abfbc04b076aabf6d4306e984e
[c-9e37019]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/9e37019ee5403e9a85b82fd3a8bcc8e7efc4251e
[c-f0556b7]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/f0556b7247374523b95ecb3835891125c6fae425
[c-a976ace]: https://github.com/KaunazDagaz/SpecTrace-dev/commit/a976acee87b3cd1ecf02bb8788e8d1179ceef85c
[run1]: https://github.com/KaunazDagaz/SpecTrace-dev/actions/runs/35503394365
[run16]: https://github.com/KaunazDagaz/SpecTrace-dev/actions/runs/36109843572
