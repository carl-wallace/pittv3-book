# Command line

`pittv3 --help` lists every option. This chapter covers the shapes of use rather than the options
one by one, because the options make more sense once the shape is clear.

## Validating against a store

The ordinary case. Give it trust anchors, CA certificates and something to validate:

```
pittv3 --ta-folder ./tas --cbor ./pki.cbor --end-entity-file ./cert.der
```

Trust anchors come from a folder (`--ta-folder`), a CBOR trust-anchor store (`--ta-cbor`), the
`webpki-roots` crate (`--webpki-tas`), a Windows certificate store (`--capi-ta`), or any
mixture through `--ta` — which takes a folder, a certificate, a bundle or a store and works out
which from the path and then from the bytes. CA certificates arrive the same way through `--ca`, or
from a store with `--cbor`.

`--validate-all` exercises every path to a target rather than stopping at the first that validates,
which is what you want when the question is "how many ways does this certificate chain to a given
trust anchor store" rather than "does it validate".

`--settings` names the JSON file holding path-validation inputs — the initial policy set, the policy
indicators, name constraints, the time of interest. It is the same file the desktop and browser
applications edit.

## Generating a store

Discovering partial paths is the expensive step; a store is that work saved:

```
pittv3 --generate --ta-folder ./tas --ca-folder ./cas --cbor ./pki.cbor
```

`--chase-aia-and-sia` extends the search by following the URIs the certificates name, writing what
it fetches into `--download-folder`. `--cbor-ta-store` writes a trust-anchor store instead, read
from the CA input.

## Building paths dynamically

`--dynamic-build` chases AIA and SIA during validation rather than in advance, for a target whose
issuers are not in any store you hold. It needs somewhere to put what it fetches, so either
`--ca-folder` or `--download-folder` must be given. `--use-downloaded-cas` folds a previous run's
downloads back in.

## Revocation

`--crl-folder` names a folder of CRLs that is indexed before validation and also receives CRLs
fetched during it. **The folder is written as well as read:** indexing removes any CRL that is not
valid at the time of interest, so do not point it at a directory whose contents matter.

`--rev` supplies CRLs and OCSP responses directly, sorted by content rather than by name.
`--keep-crl-entries-in-memory` caches verified CRLs for the life of the run;
`--no-revocation-cache` makes every path obtain its own revocation data, which is slower but leaves
each path carrying the evidence for its own result — which is what an export needs.

## Interrogating a store

Nothing here validates anything; each reports what a store holds.

| option | reports |
|---|---|
| `--list-partial-paths` | every partial path in the store |
| `--list-partial-paths-for-target` | the partial paths that could serve one certificate |
| `--list-partial-paths-for-leaf-ca` | the partial paths below one CA |
| `--list-buffers` | the certificates the store holds |
| `--list-trust-anchors` | the anchors loaded |
| `--list-name-constraints` | the name constraints in force |
| `--list-aia-and-sia` | the URIs the certificates name |
| `--dump-cert-at-index` | one certificate, by its index in the store |

## Maintaining a folder

`--cleanup` removes certificates a run could not use — unparseable, not valid at the time of
interest, self-signed, or not a CA — and `--ta-cleanup` does the same for trust anchors. Both
**move** rather than delete when `--error-folder` is given. `--report-only` says what would go
without touching anything, and is worth using first.

## Other tools

`--check-uris` fetches and reports every HTTP URI one certificate names, independently of path
processing; see [Checking the URIs in a certificate](7_miscellaneous.md).
`--check-uris-when-validating` runs the same check over every certificate on each path a run builds.
`--validate-self-signed` answers the narrower question of whether one certificate is self-signed.
`--mozilla-csv` parses the Mozilla intermediate CA report into a folder of certificates.
