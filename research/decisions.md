# SpecTrace — Decisions

Engineering choices, each with the reasoning behind it and what it changes. `spec/TOR.md` names this file as
"the *why* behind the requirements".

This file was created on 27 September 2026. The TOR cites a version 1.0 of it, with sections §1–5, that was never
written: the M1 acceptance note, item 17, records that gap. Entries below are identified by their date and title,
not by section numbers, so that no entry is mistaken for one of those cited sections.

---

## 27 September 2026 — The review UI can also start a run (SPEC-13)

**Decided by:** the student, on 27 September 2026, as a scope change to SPEC-13.

**Decision.** Besides reviewing runs, the web UI can start one. A reviewer uploads one plain-text specification;
the web layer runs it through the same pipeline the CLI runs, wired the same way; the run then appears in the
run list for review. Review is built first and the run form second, so that if the SPEC-13 timebox runs out,
the missing piece is the run form, not review.

**Why.** The demo and the usefulness measures in `BLUEPRINT.md` §6 depend on a person reviewing a real run.
Until now, a run could be produced only from the command line, which puts a step outside the browser between
the reviewer and the thing being reviewed. Starting a run from the page closes that loop without changing what
a run is.

**Boundaries.** These hold for the run form; each one keeps a principle in `BLUEPRINT.md` §9 or a TOR
requirement intact.

- **Same pipeline, same wiring.** The web layer holds no pipeline logic. The CLI's wiring — the orchestrator, the
  caching decorator, the rate limiter, and the API key read from the environment — moves into
  `SpecTrace.Pipeline` so that both entry points share it. Starting a run is the only path from the UI to the
  model; review actions never call it, not even through the cache (P5, P6).
- **One document, plain text only.** An upload is one UTF-8 text file with a size limit that keeps one live run
  well inside the daily request quota. No PDF, Word, HTML or scanned input (TOR §2.3). The form says the pipeline
  is built for IETF RFC plain text and public specifications only; other formats may lose section labels or
  produce quotes that fail verification, which are dropped and logged (P3).
- **The bytes are the document.** The uploaded bytes are kept exactly as received, because spans are resolved
  against them (P2, REQ-ING-01). No path is built from the uploaded file name; a document name is derived from
  it only after reducing it to a safe slug.
- **Corpus documents stay corpus documents.** An upload byte-identical to a file in `corpus/` is that corpus
  document, whatever the upload was called: same document name, same requirement IDs, same committed cache.
- **Nothing uploaded reaches a commit.** Any other document — its copy, its run and its cache entries — goes only
  to locations git ignores, so nothing uploaded through the UI can reach `cache/`, `corpus/` or `runs/reference/`
  in a commit (NFR-05).
- **Offline stays offline.** With `SPECTRACE_OFFLINE=1`, only documents in the committed cache can run, and the
  form says so up front. A cache miss ends the run with a clear message. The deployed UI runs offline, so it never
  calls the model and holds no key (REQ-DEP-02, NFR-06).
- **One run at a time.** A second submission while a run is in progress gets a clear message, not a queue.
- **A short wait, then a status page.** The upload request waits for the run for a short time, then redirects to
  a page that refreshes itself. On Cloud Run's request-based billing, CPU is allocated only while a request is
  being processed, so a cache replay finishes inside the request. Only a long live run, which happens only
  locally, continues in the background.
- **A run that did not finish is never offered for review.** A failed, cancelled or interrupted run shows its
  reason instead. An exhausted daily quota is reported as such.

**Choices made while building it (SPEC-13, 28 September 2026).**

- **The size limit is 64 KiB.** A run makes one model call for extraction and one per testable requirement: 13 calls
  for RFC 6902 (26 KB), 14 for RFC 10050 (32 KB), and 1.6 calls per KB for the densest document tried. At a
  pessimistic 2 calls per KB, a 64 KiB document needs at most about 131 calls. That is about a quarter of the 500
  requests a day the project's key had on 27 September 2026; AI Studio is the source of truth for the current
  figure. Extraction's 16,384 output tokens also cap how many requirements one document can yield, whatever its
  size.
- **A corpus file is any `corpus/*.txt` file**, as it is for the CLI. On a machine with local, uncommitted corpus
  files, those count too, and a live run of one records missing cache entries into `cache/` exactly as
  `spectrace run` does.
- **Everything the UI writes goes under `runs/`**, which `.gitignore` already ignores except `runs/reference/`:
  runs in `runs/{runId}/`, and the review logs, run states, uploaded copies and their cache entries under
  `runs/web/`. No `.gitignore` change was needed.

**What this changes in other documents.**

- `spec/TOR.md` 1.5: the §8 Web UI row names the run list with its new-run form, and the run overview's
  in-progress state, besides the run overview, review and matrix pages. This is a same-principle refinement under
  TOR §13(b). No requirement is added or removed, and no principle in `BLUEPRINT.md` §9 changes.
- `research/IMPLEMENTATION_PLAN.md` §10.1 describes the pages, the review log and the run form as built. The review
  log moves from `runs/{runId}/reviews.jsonl` to `runs/web/reviews/{runKey}.jsonl`. The offline end-to-end test
  compares the list of files in `runs/reference/`, so a log kept there would break it. The log would also sit in
  a committed folder, and REQ-REV-02's append-only rule means a stray decision could never be removed.
- `research/IMPLEMENTATION_PLAN.md` §12 notes the scope change under the SPEC-13 card.

**Risk recorded, not resolved.** In live mode an uploaded document is sent to Gemini's free tier, where inputs
may be used to improve the provider's models (`BLUEPRINT.md` §4). The CLI's `--document` option already allowed
this. The form states it and asks for public specifications only; nothing else enforces it.

**Out of scope, unchanged.** Authentication, a database, any JavaScript framework, deployment (SPEC-14), live
model calls from the deployed app, PDF, Word or HTML input, generating cases for a single requirement from the
review page, and several runs at once.

---

## 28 September 2026 — The review UI as a public, offline demo (SPEC-14)

**Decided by:** the student, in the SPEC-14 brief of 28 September 2026, which refines the SPEC-14 card. The
agent made the choices the brief left open, and they are listed as such below.

**Decision.** The review UI runs on Cloud Run as a public demo, in offline mode, from a billing-enabled
deployment project kept apart from the AI Studio key project. The image switches two settings on itself:
`SPECTRACE_OFFLINE=1` and `SPECTRACE_PUBLIC_DEMO=1`. On the demo, the reference run is read-only, and a visitor
who wants to try reviewing starts their own run of a corpus document.

**Why.** TOR REQ-DEP-02 and §10 ask for a deployed review UI with no key. A public link can be handed to anyone.
The reference run's review log is the evidence that a person made each decision (P6), and it is append-only, so
a decision a visitor made there could never be removed. A visitor's own run gives the same review experience
without touching that evidence.

**Boundaries.**

- **No key anywhere.** There is no key in the image, the service configuration or Secret Manager. Offline mode
  and the demo are set in the image, so the image CI tests is configured like the deployed one.
  `deploy/deploy.sh` reads the service configuration back and fails if it holds any variable or secret (NFR-06,
  P5).
- **The demo is offline or it does not start.** Its banner says this is an offline demo, so a server with the demo
  on and offline off refuses to start rather than show an untrue banner.
- **Read-only is enforced on the server.** `RunCatalog` refuses a decision on the reference run before it reads
  or appends anything, and the page answers 403. Hiding the decision forms on that page is presentation only.
- **Two projects.** The key project has no billing. The deployment project has billing and a budget alert.
  `deploy/deploy.sh` stops before deploying anything if billing on the key project is enabled or cannot be read
  (Implementation Plan §10.2).
- **One instance, scale to zero, request-based billing,** so that every visitor sees the same runs and the
  one-run-at-a-time rule holds.
- **Nothing a visitor does persists.** Runs and decisions made on the service live only in the instance and are
  lost when it stops. The banner on every page and the README say so, and say that reviewer names are
  self-declared.
- **The student deploys, by hand, from Cloud Shell.** CI builds and smoke-tests the image, but never deploys: that
  would need Google credentials in CI (NFR-06). The agent never handles a Google credential or a key.

**Choices made while building it (the agent, 28 September 2026).**

- **Region `europe-north1`** (Hamina, Finland). It is the Tier 1 Cloud Run region nearest to Lithuania, is
  marked low CO2, and has Cloud Build and Artifact Registry. `europe-central2` (Warsaw) is closer but Tier 2, and
  latency does not matter for this demo.
- **Base images `mcr.microsoft.com/dotnet/sdk:10.0-noble` and `aspnet:10.0-noble`.** They match `net10.0` and
  the installed SDK, 10.0.301; .NET 10 ships no Debian images. The tags float, so CI and Cloud Build may pull
  different patch builds. The smoke test run against the live URL closes that gap.
- **One setting, `SPECTRACE_PUBLIC_DEMO` or `--public-demo`,** covers the read-only reference and the banner,
  since both say the same thing: this server is the public demo. Outside the image it is off.
- **The run form offers every `corpus/*.txt` document by name,** in every mode. CI replays each offered document
  to completion in the container, and checks that RFC 6902 reproduces the reference run's artifacts.
- **The image also carries the reference run's metrics file,** which the run overview reads for the quote
  verification rate. It also carries the committed review log, `experiments/review/{runId}.reviews.jsonl`,
  placed where the UI reads it. The brief listed the review log but not the metrics file; without that file the
  public overview would say the rate is not recorded.
- **`/health`, not `/healthz`,** because Cloud Run reserves paths ending in `z`.
- **The service-level `--max 1`** rather than the revision-level `--max-instances`, as Cloud Run's documentation
  recommends. Cloud Run may exceed it briefly, for example during a traffic spike.
- **The billing check is `gcloud billing projects describe <key project> --format='value(billingEnabled)'`.** It
  is documented to need only `resourcemanager.projects.get`. The agent could not run it, having no `gcloud` and no
  access, so the script fails closed on an answer it cannot read, and a console check is the student's first
  manual step whatever the script says.

**Deviations from `IMPLEMENTATION_PLAN.md` §10.2 and the SPEC-14 card.**

1. The card has the service deployed from the student's machine; it is deployed from Cloud Shell.
2. The card has decisions on the public demo accepted into the instance's temporary storage; they are accepted
   only on visitors' own runs, and the reference run is read-only.
3. §10.2 has the deployment project used only for Cloud Run and Artifact Registry. A deploy from source also uses
   Cloud Build and a Cloud Storage bucket for the uploaded source, in the same project.
4. The run form gains the choice of a corpus document, which §10.1 and §10.2 do not describe.
5. The image carries the reference run's metrics file and its committed review log.
6. "Maximum instances capped at one" is Cloud Run's service-level cap, which it may briefly exceed.
7. The base-image tags float, as described above.

**What this changes in other documents.**

- `research/IMPLEMENTATION_PLAN.md` §10.2 and the SPEC-14 card gain a note pointing here.
- `spec/TOR.md`: nothing. §8 already names the run list with its new-run form, and §10 already asks for the
  deployed review UI in offline mode with no key. No requirement is added or removed, and no principle in
  `BLUEPRINT.md` §9 changes.

**Risk recorded, not resolved.** The service is public and has no accounts: anyone can replay corpus documents
and review their own runs under any name. Cost is bounded by one instance, request-based billing and the budget
alert, and a budget alert only notifies; it does not stop spending.

**Out of scope, unchanged.** Live model calls from the deployed app; a key in the image, the service
configuration or Secret Manager; authentication; a custom domain; deploying from GitHub Actions; infrastructure
as code; persistent storage for visitors' runs or decisions.
