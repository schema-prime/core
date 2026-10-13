# Core Knowledge Object

`urn:cko:core:5.0.3` · Apache-2.0

A schema for representing an ingested source document, its extracted structure,
and its evidentiary graph — in a form a machine can reason over and a human can
audit.

It is not a summarisation format and not an embedding format. It preserves the
source's structure, the entities and claims it contains, and — the part most
formats omit — a character-level trace from every derived assertion back to the
text that supports it.

## Two rules the schema exists to enforce

**Every derived assertion resolves to a span.** A claim that cannot be traced to a
character range in the normalised source text is not publishable. This is why
`normalized_text_ref` is mandatory for any object that circulates: spans that
index into a text nobody can retrieve are decoration.

**Every object says who made it.** `model_card[]` records the pipeline as an
ordered chain — which model acted, in what role, against which schema version,
over exactly which bytes. `input_hash` of each pass must equal `output_hash` of
the one before it, so a verifier that never saw the generator's output cannot
silently claim to have checked it. An object with no verifier pass is honest about
that rather than quiet about it.

## Shape

| Layer | Contents |
| --- | --- |
| Identity | `id`, `tenant_id`, `schema_version`, `schema_hash`, `revision`, `status` |
| Source | origin, hashes, byte size, page and token counts |
| Structure | `sections[]`, `passages[]` |
| Graph | `entities[]`, `claims[]`, `evidence[]`, `references[]`, `relationships[]` |
| Attestation | `model_card[]`, `extraction_quality` |
| Projections | `intelligence` — derived, perishable, declared as such |
| Governed extension | `extensions[]` |
| Private extension | `^x_` — ungoverned by design |

## Publication predicate

Objects with `status` of `draft` or `deleted` may be incomplete. Anything else —
including an object that omits `status`, since it defaults to `active` — must
carry a `model_card` chain and must have retrievable normalised text.

Two further conditions cannot be expressed in JSON Schema and belong in the
implementing gate:

- `input_hash`/`output_hash` continuity across the model-card chain
- one `schema_version_bound` shared by every pass

## Using it

```bash
npm i -g @sourcemeta/jsonschema     # or: pip install jsonschema
jsonschema validate core.v5.json my-document.json
```

Author against it in TypeBox or Pydantic and emit standard JSON Schema; validate
at runtime with AJV or python-jsonschema with `additionalProperties: false`. The
contract is the file, not any one library's opinion of it.

## Extending it

Domain structure attaches through `extensions[]` — a named, versioned payload with
its own schema. Geography, jurisdiction and telemetry all belong there, not in the
core. Anyone may publish an extension; none can alter the core.

See [GOVERNANCE.md](GOVERNANCE.md) for how a change gets into the core, and why
most should not.

## Verification

Every object carries `schema_hash`: the digest of the exact schema-file bytes it
was validated against. A document plus a candidate schema file is enough to verify
the binding — no manifest, no network, no trust in anyone. See
[MANIFEST.md](MANIFEST.md).

## Licence

Apache-2.0, chosen for the patent grant rather than the permissiveness. A schema
offered for anyone to implement is worth little if implementers carry a patent
risk from its author. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
