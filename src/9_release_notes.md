# Release notes

## 0.2.0 — unreleased

The first published release. Versions 0.1.0 through 0.1.4 were used internally and never published.

This entry describes what the tool is rather than how it changed, because there is no released
version to have changed from; the commit history covers the detail. Release-by-release notes begin
at 0.2.1.

- Validation reaches five interfaces — command line, desktop application, browser application,
  browser with a relay, and a service API — all running the same library, so a verdict does not
  depend on which produced it.
- Certification path building and validation per **RFC 5280** as augmented by **RFC 5937**, with
  certificate policy processing following the graph-based algorithm of **RFC 9618**. Trust anchors
  may be certificates or **RFC 5914** `TrustAnchorInfo` values.
- **Post-quantum signatures** are validated: ML-DSA (FIPS 204) and SLH-DSA (FIPS 205), alongside
  RSA, ECDSA and Ed25519.
- Revocation from CRLs and OCSP: an indexed folder, an in-memory cache, artifacts supplied by the
  caller, and responders reached over the network.
- Trust stores as a portable CBOR pair carrying certificates together with their precomputed
  partial paths.
- Conformance is measured rather than asserted, and both suites run in CI: **x509-limbo** and the
  NIST **PKITS** suite in seventeen editions, including fifteen post-quantum re-issues.
