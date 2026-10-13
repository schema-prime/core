# Extensions

## What this is

PRIME is the core. Extensions are how an enterprise attaches its own structure without changing the core.

Think: PRIME is the socket. Extensions are plugs. The socket never changes.

## Why extensions exist

The core must stay tiny and identical for everyone, or verification breaks. Every enterprise needs different extra fields — its own IDs, its own rules, its own geography, jurisdiction, telemetry.

If we put all of those in the core, the core would bloat and break every time someone needs something new. So we let enterprises bolt their structure on the side, in a governed envelope, without touching the core.

What stays in the core: spans, evidence, model card, extraction quality, classification, claims, passages, sections, references, relationships, entities. What never goes in the core: enterprise-internal IDs, cost, latency, privilege flags, ward numbers, docket numbers, cost centres, pipeline IDs. Those live in extensions.

## What an extension is — and is not

An extension is:

* a named, versioned block of domain structure,
* attached at a declared in-point (`extensions[]`),
* governed by its own schema and its own author,
* accepted by an explicit 512 decision, recorded in the object.

An extension is not:

* a way to change core fields,
* a way to rename core fields,
* a way to make core validation pass or fail differently,
* a fork of PRIME.

If you need to change core meaning, that is a new core version, not an extension.

## Anatomy — the `extension` object

From `core.v5.json`, `$defs/extension`. `additionalProperties: false`.

| Field | Required | Type | Meaning |
| --- | --- | --- | --- |
| `extension_id` | Yes | string, `^[a-z][a-z0-9_.]{2,64}$` | Registered identifier, e.g. `cf.geography`. Unregistered structure belongs under `^x_`, not here. |
| `extension_version` | Yes | semver | Version of the extension contract, independent of the core version. |
| `core_version_bound` | No | semver | Core version this extension declares compatibility with, e.g. `5.0.3`. |
| `accepted` | Yes | boolean | Explicit acceptance under 512. `false` means present but not governed — consumers must treat it as advisory. |
| `accepted_at` | No | date-time | When this extension was admitted to the object. |
| `accepted_by` | No | string | Party that admitted it. |
| `data` | Yes | object | Extension payload. Validated by the extension's own schema, not by the core file. |

Example envelope:

```json
{
  "extension_id": "cf.geography",
  "extension_version": "1.0.0",
  "core_version_bound": "5.0.3",
  "accepted": true,
  "accepted_at": "2026-12-05T00:00:00Z",
  "accepted_by": "512 steward",
  "data": {
    "latitude": 43.6767,
    "longitude": -79.2939,
    "place": "Toronto, Beaches"
  }
}
```

## PRIME stays prime — the rules

1. An extension never writes to a core field. It only adds its own `data`.
2. An extension never renames, retypes, or redefines a core field.
3. Core validation ignores extension internals except the envelope. A bad extension does not make core validation pass, and a good extension does not make bad core pass.
4. An extension has its own schema. That schema is versioned, content-hashed, and governed by its author — not by PRIME.
5. Acceptance is explicit. Data appearing does not imply acceptance. `accepted: false` or absent means advisory only.
6. An extension declares its `core_version_bound`. A consumer bound to a different major core version must not silently accept it.
7. Removing an extension never changes core meaning. The core object must still verify without it.
8. Private or experimental fields go under `^x_`, not in `extensions[]`. `^x_` is ungoverned by design, so it never forces a major version bump. Structure intended for third parties belongs in `extensions[]`, not under `^x_`.

## `^x_` vs `extensions[]` — which one?

* Use `^x_` when: quick experiment, operator-private field, single pipeline, no one else needs to read it, no governance needed.
* Use `extensions[]` when: a third party must read it, it has a schema, it has a version, it needs acceptance, it will be reused across objects, it will be priced, licensed, or audited.

If you are unsure, start under `^x_`. Promote to `extensions[]` when someone else depends on it.

## Lifecycle

1. **Propose.** Name it (`<author>.<domain>`, e.g. `acme.supplier`), write its schema, state what it is for and what it is not for.
2. **Register.** Record `extension_id`, `extension_version`, `core_version_bound`, schema hash, author, and where the schema lives. Registration is a fact, not acceptance.
3. **Implement.** Producers emit it. Consumers validate it against its own schema. Core validation stays unchanged.
4. **Accept.** A 512 decision admits it to an object. Record `accepted: true`, `accepted_at`, `accepted_by`. Acceptance is per object, not global — one object accepting an extension does not accept it for all objects.
5. **Version.** Fix the extension contract by versioning it. A breaking change to the extension's `data` shape is a new `extension_version`. Never edit a released extension version in place — release a new version.
6. **Deprecate.** Mark the old version deprecated, name the replacement, keep old objects verifiable. Deprecation is forward-only — old receipts stay verifiable under the version they recorded.

## Versioning

* Extension version is independent of core version. `cf.geography 1.2.0` can work with core `5.0.3` and core `5.1.0`, as long as `core_version_bound` says so.
* Patch: clarifications, docs, non-breaking constraints.
* Minor: new optional fields in `data`.
* Major: removed fields, renamed fields, changed types, changed meaning.
* Core major bump: re-check every extension's `core_version_bound`. Do not assume compatibility across a core major.

## Validation — two layers

Layer 1 — Core: validate the PRIME object against `core.vN.json`. This checks core fields + extension envelopes, never extension `data` internals.

Layer 2 — Extension: for each entry in `extensions[]`, validate `data` against that extension's own schema, at that `extension_version`.

A consumer must:

* reject an object if Layer 1 fails, whatever the extensions say,
* treat `accepted: false` extensions as advisory — read, do not rely,
* ignore unknown `extension_id`s for core purposes, but declare that it ignored them — never silently pretend they were understood.

## Governance

* The extension author governs the extension's schema. PRIME does not govern enterprise schemas.
* The 512 steward governs acceptance. Acceptance is a 512 decision, recorded in the object.
* PRIME governs only the envelope and the PRIME-stays-prime rules above.
* An extension that tries to smuggle core semantics — e.g. a `verdict` field that overrides core `status`, or a `span` that redefines evidence — is not an extension. Reject it.

## Worked example — full object with extensions

A PRIME object for a hospital report, with one governed enterprise extension, one geography extension, and one advisory extension:

```json
{
  "id": "cko:example:hospital-report-001",
  "schema_version": "5.0.3",
  "title": "Ward occupancy report",
  "source": {
    "source_type": "document",
    "title": "Ward occupancy report"
  },
  "model_card": [
    {
      "pass_id": "pass_01",
      "role": "generator",
      "model_id": "example/model-1",
      "schema_version_bound": "5.0.3",
      "inputs": [{ "from": "source_text" }],
      "eval_status": "pass"
    }
  ],
  "extensions": [
    {
      "extension_id": "hospital.encounter",
      "extension_version": "1.0.0",
      "core_version_bound": "5.0.3",
      "accepted": true,
      "accepted_at": "2026-12-05T00:00:00Z",
      "accepted_by": "hospital steward",
      "data": {
        "ward": "4B",
        "attending": "Dr. Example",
        "encounter_id": "ENC-88421"
      }
    },
    {
      "extension_id": "cf.geography",
      "extension_version": "1.0.0",
      "core_version_bound": "5.0.3",
      "accepted": true,
      "accepted_at": "2026-12-05T00:00:00Z",
      "accepted_by": "512 steward",
      "data": {
        "latitude": 43.6767,
        "longitude": -79.2939,
        "place": "Toronto, Beaches"
      }
    },
    {
      "extension_id": "acme.supplier",
      "extension_version": "0.1.0",
      "core_version_bound": "5.0.3",
      "accepted": false,
      "data": {
        "supplier_id": "SUP-991",
        "cost_centre": "CC-104"
      }
    }
  ],
  "x_pipeline_note": "routed via nightly batch"
}
```

Read it like this: core still verifies on its own. Remove all three extensions and the core object still means the same thing. The hospital and geography extensions are governed and accepted. The `acme.supplier` extension is present but advisory — do not rely on it until it is accepted. The `^x_` field is private, ungoverned, and never forces a core bump.

More examples of the same class:

* **Legal:** `legal.filing` — court, docket number, jurisdiction, privilege flag. Core unchanged.
* **Enterprise:** `acme.supplier` — supplier ID, contract ID, cost centre. Core unchanged.
* **Operations:** `ops.telemetry` — pipeline ID, model cost, latency. Cost and latency describe the operator's economics, not the document — they belong here, never in the core.
* **Geography:** `cf.geography` — latitude, longitude, place hierarchy. Core unchanged.

## Extension schema template — minimal

Every extension ships its own JSON Schema, versioned and hashed:

```json
{
  "$id": "urn:ext:acme.supplier:1.0.0",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Acme Supplier Extension",
  "type": "object",
  "required": ["supplier_id"],
  "properties": {
    "supplier_id": { "type": "string" },
    "contract_id": { "type": "string" },
    "cost_centre": { "type": "string" }
  },
  "additionalProperties": false
}
```

## Authoring checklist

* [ ] `extension_id` matches `^[a-z][a-z0-9_.]{2,64}$`
* [ ] `extension_version` is semver, new version for any breaking change
* [ ] `core_version_bound` is set
* [ ] Own schema exists, versioned, hashed, and published where consumers can fetch it
* [ ] `data` validates against that schema
* [ ] No core field is written, renamed, or redefined
* [ ] Acceptance is recorded (`accepted`, `accepted_at`, `accepted_by`) or explicitly `false`
* [ ] Removal test: object still verifies with the extension stripped

## Common mistakes

* Putting price, cost, or latency in the core. Price lives outside the kernel. Cost and latency live in an extension.
* Using `extensions[]` for a one-off private field. Use `^x_`.
* Editing a released extension version in place. Release a new version.
* Treating presence as acceptance. Acceptance is explicit, recorded, per object.
* Letting an extension override core meaning. If it does, it is not an extension — reject it.
