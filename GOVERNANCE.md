# Governance

## What this is

A public schema. Nobody owns the contract it defines. It is licensed Apache-2.0,
including the patent grant, so that anyone can implement it — commercially or
otherwise — without permission and without exposure to a later patent claim from
its author.

## Unowned is not unmaintained

Those are one step apart, and the gap is where specifications die. So this is
stated plainly rather than left implied:

- **The schema is unowned.** No licence fee, no registration, no permission.
- **The schema is stewarded.** One maintainer arbitrates changes and holds the
  anchoring key. Today that is the original author.

A steward who pretends to be absent produces a spec nobody can rely on. A steward
who behaves like an owner produces a spec nobody adopts. The line is: the steward
decides what enters the core, and has no say in who uses it or how.

## Identity and verification

Canonical identity is a URN — `urn:cko:core:<version>` — which depends on no
domain and no company. URLs are mirrors: locations, not identity.

The durable binding between a document and its contract is the SHA-256 digest of
the exact bytes of the released schema file. Every emitted object carries that
digest in `schema_hash`. A holder with a document and a candidate schema file can
verify the binding with nothing else — no manifest, no network, no trust in the
steward.

Released digests are additionally anchored on a public ledger from a dedicated
publishing account, which adds a third-party-verifiable timestamp. The ledger
stores the digest only; it never stores the file. See `MANIFEST.md`.

## Versioning

- Released versions are **never** modified. Corrections ship as a new version.
- The core is deliberately hard to enter. Any proposal must answer, in writing,
  **"why is this not an extension?"** Most do not survive that sentence.
- Breaking changes increment the major version. Both versions are served in
  parallel until adopters migrate.

### Admission test for the core

A change enters the core only if every answer is yes:

1. Without it, can a consumer no longer verify an object is what it claims to be?
2. Is it equally true of a trade article, a legal brief and a service manual?
3. Will it read the same way in five years?
4. Would adding it later **break** existing objects? (If it could be added later
   cleanly, it belongs in an extension.)
5. Does it describe the document rather than any operator's business?

## Extensions

Domain structure — geography, jurisdiction, telemetry — attaches through
`extensions[]` at declared in-points. Extensions carry their own identifier,
version and schema, and are governed by their authors, not by the core.

Anyone may publish an extension. No permission is required, and no extension can
alter the core.

`^x_` remains available for private and experimental fields. It is ungoverned by
design and is not a route into the standard.

## Preservation

The steward's infrastructure is not a single point of failure, and is not
permitted to become one. Released versions are mirrored to a public repository
with signed tags and to content-addressed permanent storage. Because the digest
is public and the file is self-certifying, any surviving copy anywhere restores
the canonical schema — the holder need not be trusted, only the bytes checked.

## Succession

If the steward becomes unavailable, stewardship passes to whoever maintains the
public repository and can demonstrate control of the anchoring key. Failing that,
the schema remains usable in perpetuity at its last released version, which is the
point of licensing it this way.
