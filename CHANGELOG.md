# Core Schema — Changelog

## v5.0.3 → v5.1.0

Released file: `core.v5.json`, `$id` `urn:cko:core:5.1.0`, 82,185 bytes, SHA-256 `c4ca5e7f02efa782f28ff3baf6c83aae2dfa91efe7039ea5ea15b8932a7e8c4d`.

**Added — identity declarations (top level, optional).** `original_name` (the source file's name exactly as received at intake, before any slug or hash naming), `document_slug` (the operator-assigned stable slug for the source document), and `run_id` (the run that produced the object or export). PRIME has always carried provenance-by-hash in `content_hash`; it carried no governed provenance-by-name, and consumers that needed the document's original name had nowhere lawful to put it — the Extraction Engine's bench supplied the fields by editing its vendored schema copies, which diverged the bench's schema from core. The fields are adopted into core here, deliberately: provenance of the source is a property of the document, not of any one operator's business, so it is not an extension.

**Added — `x-512-declaration` (schema metadata).** The schema file now declares itself: artifact, canonical URN, version, status, the amendment path, and the evaluation rule a holder applies before use (hash the copy, compare with the pin published in `MANIFEST.md`, refuse and report on mismatch). The declaration deliberately does not carry the schema's own digest — a file cannot contain its own hash; the pin lives in `MANIFEST.md`.

**Compatibility.** Additive only. No field was renamed, retyped, re-scoped or removed; no constraint was loosened or tightened. Every object valid under v5.0.3 remains valid under v5.1.0. Objects carrying the new identity fields require v5.1.0.

**Reconciled — `$comment`.** The `$comment` now notes the one exception to its own rule that process is not recorded in this file: the declaration block is in-file by design, while stewardship, succession and proposal process in full remain in `GOVERNANCE.md`. No other `$comment` text changed.

---

## v4 → v5.0.3

Released file: `core.v5.json`, `$id` `urn:cko:core:5.0.3`, 80,071 bytes, SHA-256 `072fd266d83b183e26e27e01f5dd31ae395b5180cde62d84857cef698192ad8a`.

Full v4 → v5 delta is not yet documented in this file. Do not infer it from the v3 → v4 section below. The v3 → v4 section is preserved verbatim as history.

---
# Core Schema — v3 → v4

Filename unchanged. `$id` moves to `.../core/v4.json`.

Every v3 field description is preserved verbatim except where listed below. No field was renamed.

## Added

**`normalized_text_ref`** (top level, optional)
Locator for the normalised full text that spans index into. If absent, the normalised text is now *defined* as the ordered concatenation of `passages[].text` joined by `\n\n`. Before this, spans pointed at a text the schema never located — every span in a v3 document was formally unverifiable.

**`schema_hash`** (top level, optional now, required once a released hash exists)
Digest of the exact schema-file bytes this object validated against. Makes the object self-describing: the binding is verifiable from the document plus a candidate schema file alone, with no manifest lookup. Ledger transaction id and ledger index are deliberately not carried per-object — they are lookup coordinates that can change, and stamping them into every document guarantees stale references. They live in `MANIFEST.md`.

**`model_card[]`** (top level, min 1 item)
Ordered chain of pipeline passes. Each entry: `pass_index`, `role` (generator | repair | verifier | human), `model_id`, `schema_version_bound`, `constraint_set_id`, `input_hash`, `output_hash`, `truncated`, `eval_status`, `failed_rules[]`, timestamps.

Two gate conditions follow from it, and neither is enforceable in JSON Schema alone — they belong in the gate:
- `input_hash` of pass *n* MUST equal `output_hash` of pass *n−1*. A verifier that breaks the chain did not see what it attests to.
- All passes MUST share one `schema_version_bound`. Divergence is schema drift.

Deliberately excluded: token cost, decoding parameters, latency. Those describe the operator's economics, not the document.

**`extensions[]`** (top level, optional)
Named, versioned extension attached at a declared in-point — `extension_id`, `extension_version`, `core_version_bound`, `accepted`, `accepted_at`, `accepted_by`, `data`. Acceptance is an explicit recorded decision; `accepted: false` means present but ungoverned, and consumers must treat the payload as advisory.

`^x_` survives unchanged for private and experimental fields, now explicitly documented as ungoverned. Structure intended for third parties goes in `extensions[]`.

## Changed

**`passage`** — `text` and `text_ref` are now a `oneOf`. Exactly one, never both, never neither. Dual storage drifts.

**`intelligence`** — description now declares it a PROJECTION rather than extraction: regenerated on its own cadence, independently perishable, and not covered by the durability guarantee the rest of the object carries. Structure unchanged.

## Removed

**`prospect_fit`** — property and `$defs` entry both deleted. It scored a commercial prospect from the operator's point of view inside an object served to that prospect. Rebuild it as a sibling record keyed by document id if still wanted.

## Model card: DAG provenance, not a linear chain

`role` gains **`merge`** — reconciling two or more independent candidate extractions into one. Distinct from
`repair`, which corrects a single output. Collapsing them would erase the strongest provenance claim an ensemble
pipeline can make.

`input_hash` (singular) is **replaced by `inputs[]`** — an array of `{from_pass?, hash}`. A generator reading the
source has one input with `from_pass` absent; a merge over three candidates has three inputs, each naming its
producing pass. A singular field would have forced a merge to name one source and silently discard the rest.

`pass_id` replaces `pass_index` as the identifier. `pass_index` is retained for human reading only and is
explicitly **not unique** — parallel passes share a stage number, so it must never be used to reconstruct
dependency order. `inputs[]` is the authority.

Three gate conditions follow, none expressible in JSON Schema:

- every input hash resolves to the document's `normalized_hash` (when `from_pass` is absent) or to the named pass's `output_hash`
- `pass_id` is unique within the array
- the graph is acyclic

## Publication predicate (closes the last hole)

`model_card` and normalised text are **conditionally required**, expressed in the schema via `allOf`/`if`/`then`:

- `status` of `draft` or `deleted` — object may be incomplete.
- Any other status, **including an absent status** (which defaults to `active`) — the object MUST carry `model_card`, and MUST carry either `normalized_text_ref` or at least one passage.

A draft can be incomplete. Anything that circulates cannot.

`eval_status` is now required on every `model_card` entry. `not_evaluated` is an acceptable and honest value; omission is not.

## Deliberately not in v4

| Change | Where it goes | Why not core |
| --- | --- | --- |
| Geography / place | Extension (`cf.geography`) | Can be added later without breaking existing objects |
| Coverage declaration ("what we looked for") | Extraction layer | Operational reporting, not document structure |
| Confidence threshold | Channel policy | Thresholds are meant to change |
| Model cost / decoding detail | Extraction layer | Describes the business, not the document |
| Enrichment permission | Rights plane | Belongs on the Publisher Launch Commitment |
| On-chain anchor references | Extraction layer, if at all | Additive later; no buyer requires it yet |
| CVS receipts | Never in the object | Receipts reference the document, not the reverse |

## Validation

Passes `Draft202012Validator.check_schema`. All `$refs` resolve; no orphaned definitions. A minimal instance with a generator→verifier chain validates; a passage carrying both `text` and `text_ref` is rejected.

## Before locking

Run v4 against real documents from both target archetypes — a batch of trade articles and a batch of legal filings — and only then anchor.
