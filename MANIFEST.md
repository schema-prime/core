# Manifest

One row per released version. This file answers *when* and *where*; the digest in
each object answers *what*.

| Version | Git tag | SHA-256 of `core.vN.json` | Ledger txn | Ledger index | Mirror |
| --- | --- | --- | --- | --- | --- |
| 4.0.0 | _tag pending_ | `451348f4dc832c6ca364bb2d932a3ba732cf7150579e2f12183008e5090e1ffc` | _pending_ | _pending_ | _pending_ |
| 5.0.3 | _tag pending_ | `072fd266d83b183e26e27e01f5dd31ae395b5180cde62d84857cef698192ad8a` | _pending_ | _pending_ | _pending_ |
| 5.1.0 | _tag pending_ | `c4ca5e7f02efa782f28ff3baf6c83aae2dfa91efe7039ea5ea15b8932a7e8c4d` | _pending_ | _pending_ | _pending_ |

## Governance digest

`GOVERNANCE.md` is versioned and hashed separately from the schema. It must be able
to change — stewardship and succession are not immutable facts — so it is never
embedded in the sealed schema file. The durable design rules that defend the core
(the admission test) travel inside `core.vN.json` itself; process lives here.

| Governance version | Git tag | SHA-256 of `GOVERNANCE.md` | Effective |
| --- | --- | --- | --- |
| 1.0.0 | _unreleased_ | _pending_ | _pending_ |

Digest above is of `core.v4.json` as built, 43,846 bytes.

Digest below is of `core.v5.json` as built, 80,071 bytes. SHA-256 `072fd266d83b183e26e27e01f5dd31ae395b5180cde62d84857cef698192ad8a`. It supersedes candidate
`2d8f3f25f3d908275275e1833b9f1423c52032fc26f6e9ffba2d9fa50627eed0`, which was never
anchored: that build could not truthfully encode an ensemble execution graph.

Digest for v5.1.0 is of `core.v5.json` as amended, 82,185 bytes. SHA-256 `c4ca5e7f02efa782f28ff3baf6c83aae2dfa91efe7039ea5ea15b8932a7e8c4d`. It adds the
identity declarations (`original_name`, `document_slug`, `run_id`) and the
`x-512-declaration` metadata block to v5.0.3; nothing else in the schema changed.
Not yet tagged or anchored.

## Rules

1. **Hash the `.json` file bytes, never an archive.** Zip embeds timestamps and
   compression settings, so re-archiving identical content yields a different
   digest and verification fails for reasons nobody can diagnose.
2. The bytes hashed are those committed at the corresponding signed git tag.
3. Anchoring is performed from one dedicated publishing account, whose address is
   published in this repository and used for nothing else. Without a known
   publishing key, an anchored digest attests to nothing.
4. A released row is never edited. Corrections are new rows.

## Verifying a document

```
sha256sum core.v4.json          # compare to the object's schema_hash
sha256sum core.v5.json          # compare to the object's schema_hash
```

Matching digest means the document was produced under exactly this contract.
The ledger row tells you when that contract was published, and by whom.
