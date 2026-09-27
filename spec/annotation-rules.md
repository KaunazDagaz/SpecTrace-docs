# SpecTrace — Annotation rules for the gold standard

**Status:** Frozen when the pull request that adds this file merges into `main`. The gold file names that merge commit.
**Applies to:** the gold standard for RFC 6902, `corpus/gold/rfc6902.gold.yaml` in `spectrace-dev`.
**Related:** TOR §9, Implementation Plan §8.3, SPEC-11.

---

## 0. How these rules are used

The rules are written before annotation starts and do not change while it runs. If annotation shows a rule to be wrong, annotation stops, the rules change in a new pull request, and annotation starts again under the new commit. A gold file names exactly one rules commit.

The annotator is the student, alone. No model writes, completes or edits an annotation.

Annotation works from a worksheet that code generates by scanning the document for uppercase keywords. It lists every sentence of the de-paginated text that carries one, with its section, lines and keywords. A listed sentence is a candidate, not a requirement: the annotator marks each candidate `keep` or `drop`, and records one entry for each obligation in a kept candidate.

Every entry has four fields. The quote is verbatim, the modality is one of `MUST`, `MUST_NOT`, `SHOULD`, `SHOULD_NOT`, `MAY`, and the testability is one of `testable`, `needs_human_decision`, `not_testable`. The section is left empty unless §5 requires it.

---

## 1. What counts as a requirement

A requirement is a statement carrying an uppercase BCP 14 keyword — `MUST`, `MUST NOT`, `REQUIRED`, `SHALL`, `SHALL NOT`, `SHOULD`, `SHOULD NOT`, `RECOMMENDED`, `NOT RECOMMENDED`, `MAY`, `OPTIONAL` — outside the passage that defines these keywords and outside examples. Lowercase words are not normative.

Uppercase means every letter is a capital. A statement that reads as normative but carries no uppercase keyword is not a requirement under these rules.

Examples are Appendix A, "Examples", in full, and every block the text introduces as an example.

| RFC 6902 | Text | Requirement? |
|---|---|---|
| §4 | `Operation objects MUST have exactly one "op" member, whose value indicates the operation to perform.` | Yes. |
| §2 | `The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].` | No: the passage that defines the keywords. |
| Appendix A.13 | `JSON requires that object member names be unique with a "SHOULD" requirement, and there is no standard error handling for duplicates.` | No: it sits in the examples. |
| Copyright Notice | `Code Components extracted from this document must include Simplified BSD License text` | No: `must` is lowercase. |
| §6 | `Required parameters:  none` | No: `Required` is not uppercase. |
| §4.1 | `it remains an error for that not to be the case` | No: normative in effect, but it carries no keyword. |

---

## 2. One entry per obligation

One entry per obligation. A sentence with two obligations becomes two entries, each quoting verbatim only the minimal clause that carries its own obligation.

§5 holds two obligations in one sentence:

> If a normative requirement is violated by a JSON Patch document, or if an operation is not successful, evaluation of the JSON Patch document SHOULD terminate and application of the entire patch document SHALL NOT be deemed successful.

It becomes two entries:

| Quote | Modality |
|---|---|
| `evaluation of the JSON Patch document SHOULD terminate` | `SHOULD` |
| `application of the entire patch document SHALL NOT be deemed successful` | `MUST_NOT` |

A condition that applies to one obligation alone is part of that obligation's minimal clause. A condition shared by two or more obligations in the same sentence is quoted by none of them, since it could not open both quotes without one quote containing the other obligation. What the quote leaves out is recorded in its testability (§4). The condition opening the §5 sentence is shared, so neither quote above includes it.

A clause that only explains, and carries no obligation of its own, is left out. §4.4:

> The "from" location MUST NOT be a proper prefix of the "path" location; i.e., a location cannot be moved into one of its children.

The quote is `The "from" location MUST NOT be a proper prefix of the "path" location`.

Verbatim means every character as the document has it: case, punctuation and quotation marks included. Whitespace alone may differ, because the resolver collapses every whitespace run to one space. The worksheet's `sentence` field holds the text to copy from.

---

## 3. Modality

An entry's modality is the class of its keyword:

| Keyword | Modality |
|---|---|
| `MUST`, `SHALL`, `REQUIRED` | `MUST` |
| `MUST NOT`, `SHALL NOT` | `MUST_NOT` |
| `SHOULD`, `RECOMMENDED` | `SHOULD` |
| `SHOULD NOT`, `NOT RECOMMENDED` | `SHOULD_NOT` |
| `MAY`, `OPTIONAL` | `MAY` |

§5: `application of the entire patch document SHALL NOT be deemed successful` is `MUST_NOT`.

Outside §2 and Appendix A, RFC 6902 uses only `MUST`, `MUST NOT`, `SHOULD` and `SHALL NOT`. The other rows of the table have no instance in this document.

---

## 4. Testability

Testable if a black-box test can check it from the quote alone; `needs_human_decision` if it depends on context beyond the quote; `not_testable` otherwise.

The annotator reads only the entry's quote when deciding, not the sentence or section around it.

| RFC 6902 | Quote | Testability |
|---|---|---|
| §4.4 | `The "from" location MUST NOT be a proper prefix of the "path" location` | `testable`: a test sends a "move" whose "from" is a proper prefix of its "path" and checks that it is refused. |
| §4.1 | `When the operation is applied, the target location MUST reference one of:` | `needs_human_decision`: what the location must reference is listed after the quote. |

No statement in RFC 6902 was found to be clearly `not_testable`, so this rule has no example here. The value is for a quote that says fully what it requires, but that no black-box test can observe.

---

## 5. A quote that occurs more than once

Each entry identifies one place in the document. A quote must be found exactly once: in the whole document, or in the section the entry names.

Each occurrence that §1 makes a requirement is its own entry, even when the words repeat. §4.2 and §4.3 both say:

> The target location MUST exist for the operation to be successful.

Once of the "remove" operation, once of "replace": two entries.

When an entry's quote occurs more than once in the document, the entry names the section it sits in, and the quote must occur exactly once in that section. When the quote occurs once in the document, the section may be left empty. Either way, the section recorded for a requirement is derived from where its quote is found.

Minimal clauses repeat more often than whole sentences: `The operation object MUST contain a "value" member` occurs in §4.1, §4.3 and §4.6.

A quote that occurs more than once within its own section is extended, within its sentence, until it occurs there once.
