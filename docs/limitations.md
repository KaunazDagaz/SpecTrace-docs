# Limitations

What SpecTrace's results allow the project to claim, and what they do not. Each statement links to the file that
holds its figure.

## The samples are small

- **Claims:** 16 to 30 claims per arm and document, one run per arm, with no repeated sampling
  ([`headline.md`][headline]). On RFC 10050 the arms differ by one or two claims, which a second run could change.
- **The gold standard:** 19 requirements in one document, annotated by one person, so there is no inter-annotator
  agreement figure ([`rfc6902.quality.md`][quality]; [`spec/TOR.md`][tor] §11).
- **The review:** one reviewer, who is also the author, on one document. 21 of its 23 case decisions and 3 of its 4
  queue decisions were made with the coding agent's opinion on each, at the reviewer's request, and each equals that
  opinion. So the counts are not an independent human judgment ([`experiments/review/`][review];
  [`error-analysis.md`][ea] §9).
- **Confidence intervals:** none were computed.

## One model, one provider

- **The controlled arms:** every controlled arm used `gemini-3.5-flash-lite` at temperature 0 through the Gemini API.
- **The chat arm:** it used the Gemini app, whose model version, system prompt and sampling are not disclosed. It was
  captured by hand and cannot be replayed ([`headline.md`][headline]).
- **NFR-04 is unmet:** the provider and the model are constants in code ([`LlmClientFactory.cs`][factory]), so a
  different provider is a code change, not a configuration change.
- **An open question:** whether a cheap model with strict verification beats an expensive one trusted without it
  ([plan][plan] §8.1, §15) is not answered here.

## What the model may already know

- **RFC 6902 (2013) is well known.** The model has most likely seen it and its test suites, so the extraction quality
  measured on it is an upper bound ([plan][plan] §14).
- **RFC 10050 is the check on that.** It was published in September 2026, after the model's March 2026 knowledge
  cutoff, but its drafts were public from February 2025 ([`corpus/SOURCES.md`][sources]). On it the three arms lie
  close together: chat 9.5%, naive prompt 9.1%, pipeline 3.3% of claims not located ([`headline.md`][headline]).

## What verification was not shown to do

- **No invented requirements appeared.** No quote that could not be located, in any arm or document, was invented or
  reworded. Each was the source tidied: its quotation marks, its capitalisation, or a mark where the model cut it
  ([`error-analysis.md`][ea] §2). So the experiment does not show verification catching an invented requirement, the
  failure [`BLUEPRINT.md`][blueprint] §1 describes.
- **What it did show,** on RFC 6902: quotes that are not verbatim are refused, and sentences found twice or read two
  ways go to a person.
- **The cost of that strictness.** It refuses a quote whose only fault is cosmetic. 12 of the naive prompt's 13
  unlocated quotes were within 0.90 similarity of the source, and they were its only way to 12 gold requirements
  ([`headline.md`][headline], the cost of verification).
- **The pipeline's own register** holds back 6 of the 19 gold requirements for a person: recall 63.2% delivered,
  against 94.7% before verification ([`error-analysis.md`][ea] §3).

## The chunking check could not fire

The rule fixed in advance compares recall in the first and the last third of the document. No gold requirement lies
in RFC 6902's last third, so the rule cannot fire. The whole document is 7,980 input tokens in the extraction call,
so a model losing requirements late in a long document could not be detected ([`chunking-decision.md`][chunk]; [`spec/TOR.md`][tor] 1.4).

## Completeness is not proven

The system claims only that no requirement in its register lacks a test case that has not been rejected. It never
claims that the specification is covered (P7). Against the gold standard, the delivered register's recall is 63.2%
([`headline.md`][headline]).

## The results may not transfer

RFC normative language, with MUST, SHOULD and MAY in capitals, is far stricter than most requirements, and the
keyword makes extraction unusually easy. Nothing here shows that the results hold for requirements written as
ordinary prose ([plan][plan] §14).

## What the system does not do

- **A duplicated quote stays unresolved.** A person's answer on a quote found more than once is logged, but changes
  no register. The 6 held-back gold requirements stay held back ([README, Review UI][readme-review];
  [`error-analysis.md`][ea] §10).
- **"Testable" generates nothing.** It is logged, but no test case is generated, so the requirement stays a gap.
- **The public service** has no accounts and shares one daily quota among all visitors. Only the page's wording
  keeps uploads to public specifications, its live runs cannot be replayed from the repository, and it runs against
  a use restriction in the provider's terms ([`docs/privacy-safety.md`][privacy]).

[headline]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/experiments/headline.md
[quality]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/experiments/rfc6902.quality.md
[ea]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/experiments/error-analysis.md
[chunk]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/experiments/chunking-decision.md
[review]: https://github.com/KaunazDagaz/SpecTrace-dev/tree/defense/experiments/review
[factory]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/src/SpecTrace.Pipeline/LlmClientFactory.cs
[sources]: https://github.com/KaunazDagaz/SpecTrace-dev/blob/defense/corpus/SOURCES.md
[readme-review]: https://github.com/KaunazDagaz/SpecTrace-dev#review-ui
[blueprint]: ../BLUEPRINT.md
[tor]: ../spec/TOR.md
[plan]: ../research/IMPLEMENTATION_PLAN.md
[privacy]: privacy-safety.md
