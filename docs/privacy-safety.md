# Privacy and safety

This page covers what SpecTrace sends to a language model, where it goes, and what the provider's terms say happens
to it. It also covers what the system keeps, and what nothing in it enforces. It is for anyone deciding what they may
put into SpecTrace. The terms are quoted as read on 1 October 2026, and the provider can change them.

## What a run sends to the model

| Call | What is sent | Code |
|---|---|---|
| Extraction, one per run | The whole document, after the line `DOCUMENT ID: <name>` | [`RequirementExtractor.cs`][extractor] |
| Test case generation, one per testable requirement | That requirement's section number and its quote, nothing else | [`TestCaseGenerator.cs`][generator] |
| The naive baseline, from the command line only | The baseline prompt and the whole document | [`BaselineRun.cs`][baseline] |

Each call also carries its system prompt, one of the files in [`src/SpecTrace.Pipeline/Prompts/`][prompts]. Nothing
else is sent. No reviewer's name or review decision ever reaches the model, because review actions never call it
([README, Review UI][readme-review]).

## Where it goes

| How SpecTrace runs | Where the calls go |
|---|---|
| Offline: `--offline` or `SPECTRACE_OFFLINE=1`, in CI, and in the container image by default | Nowhere. Every answer is replayed from the committed cache, no key is read, and a call that is not in the cache ends the run. |
| Live on your own machine: the command line or the review UI without `--offline` | The Gemini API, through the key in your environment, on your Google Cloud project. The terms that apply are the ones for you as that key's developer. |
| [The public service][readme-deployment] | The Gemini API, through the author's key, read from Secret Manager in the deployment project. Every visitor's upload goes through it. |

## What the provider's terms say happens to it

From the [Gemini API Additional Terms][terms], last updated 2026-04-28. Under "Unpaid Services":

> "When you use Unpaid Services, including, for example, Google AI Studio and the unpaid quota on Gemini API, Google
> uses the content you submit to the Services and any generated responses to provide, improve, and develop Google
> products and services and machine learning technologies, including Google's enterprise features, products, and
> services, consistent with our Privacy Policy."

> "To help with quality and improve our products, human reviewers may read, annotate, and process your API input and
> output."

> "If you're in the European Economic Area, Switzerland, or the United Kingdom, the terms under "How Google uses Your
> Data" in "Paid Services" apply to all Services, including Google AI Studio and unpaid quota in the Gemini API, even
> though they are offered free of charge."

Under "Paid Services", Google "doesn't use your prompts (…) or responses to improve our products", and "logs prompts
and responses for a limited period of time, solely for detecting and preventing violations of the Prohibited Use
Policy to maintain the safety and security of the Services, and any required legal or regulatory disclosures."

What that means here:

- **On the public service**, the key's developer is the author, who is in the European Economic Area. The Paid
  Services terms apply: uploads are not used to improve Google's products, and are logged for a limited period.
- **Live on your own machine**, it depends on where you are. Outside the EEA, Switzerland and the United Kingdom, the
  unpaid terms apply to what you send: Google may use it to improve its products, and people may read it.
- **Either way, upload public specifications only.** Nothing personal and nothing belonging to an employer enters
  the system ([`BLUEPRINT.md`][blueprint] §4–§5), whatever the provider does with it.

## The public service runs against a use restriction

The same terms say, under "Use Restrictions":

> "You may use only Paid Services when making API Clients available to users in the European Economic Area,
> Switzerland, or the United Kingdom."

The public service runs on the key project's unpaid quota. The student decided on 1 October 2026 to keep it live, to
show the system working on new documents, and accepts that risk. The decision, the options it was chosen from and
the way back are in [`research/decisions.md`][decision]. The README's [kill switch][readme-deploy] turns the service
offline without a rebuild.

## What is kept, where, and for how long

- **`cache/` in `spectrace-dev`:** every request and response of the committed runs, document text included, in a
  public repository, indefinitely. It holds only the corpus documents, which are public specifications whose
  redistribution terms are recorded in [`corpus/SOURCES.md`][sources].
- **`runs/` on a local machine:** run outputs, review logs, and the copies and cache entries of uploaded documents.
  Git ignores them, and they stay until deleted.
- **On the public service:** uploads, their cache entries, runs and review decisions live only in the instance. They
  are lost when it stops: on a restart, a scale to zero after a quiet spell, or a redeploy.
- **Names:** a reviewer's name is self-declared and is stored in the review log. The committed review log and the
  gold standard carry the author's name.

## What nothing enforces

- That an upload is a public specification. Only the banner and the form ask for it.
- Who reviews. There are no accounts, and anyone can review under any name.
- How the daily quota is shared. Every visitor to the public service draws on the same one.

## The key

- **Locally:** `GEMINI_API_KEY` in the environment, never in a file of the repository. `.env.example` holds names, not
  values.
- **On the public service:** a reference to a pinned version of the Secret Manager secret `gemini-api-key`, which
  `deploy/deploy.sh` sets and then checks. The key is never in the image, a build argument, a plain variable, the
  repository or CI ([README, Reproducibility and secrets][readme-secrets]).
- **In CI:** no key, offline (REQ-DEP-01, NFR-06).
- **Billing:** the key's project has no billing, so a call on it cannot be billed (REQ-DEP-02).

[terms]: https://ai.google.dev/gemini-api/terms
[blueprint]: ../BLUEPRINT.md
[decision]: ../research/decisions.md
[extractor]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/src/SpecTrace.Pipeline/RequirementExtractor.cs
[generator]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/src/SpecTrace.Pipeline/TestCaseGenerator.cs
[baseline]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/src/SpecTrace.Pipeline/BaselineRun.cs
[prompts]: https://github.com/KaunazDagaz/SpecTrace-dev/tree/defense/src/SpecTrace.Pipeline/Prompts
[sources]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/corpus/SOURCES.md
[readme-review]: https://github.com/KaunazDagaz/SpecTrace-dev#review-ui
[readme-deployment]: https://github.com/KaunazDagaz/SpecTrace-dev#deployment
[readme-deploy]: https://github.com/KaunazDagaz/SpecTrace-dev#deploy-redeploy-and-check
[readme-secrets]: https://github.com/KaunazDagaz/SpecTrace-dev#reproducibility-and-secrets
