# Limitations and known issues

## Deliberate limitations

These are choices, not gaps waiting to be filled.

**Delta CRLs are not supported.** They are neither indexed nor considered. A scope requiring one
fails closed rather than being approximated, because approximating revocation is worse than
declining to answer.

**FN-DSA is not verified.** The identifiers are declared; verification waits on a FIPS 206
implementation.

**This is not a TLS verifier.** Web PKI-specific behaviour — name matching rules, policy
requirements particular to the CA/Browser Forum baseline — is out of scope by design. PITTv3
validates certification paths per RFC 5280 and RFC 5937; it does not decide whether a certificate is
acceptable for a TLS connection.

## Things worth knowing

**A CRL folder is written as well as read.** Indexing removes any CRL that does not cover the time
of interest, so pointing `--crl-folder` at a directory whose contents matter will prune it. A
superseded CRL generally cannot be fetched again, so this can foreclose validating as of a time it
covered.

**Cleanup moves or deletes, depending on one field.** With an error folder named — and the desktop
application names one by default — certificates that cannot contribute are moved there. With that
field cleared, they are deleted. Use *Report Only* first.

**A store does not know about certificates added after it was built.** Partial paths are computed
when the store is generated; adding certificates to a CA folder afterwards does not change an
existing store. Regenerate it.

**A time of interest of `0` disables validity period checks** rather than meaning the epoch.

**Windows machine stores need elevation to read.** The library asks for write access when opening a
CAPI store, because the wrapper it uses offers no read-only open, so `LocalMachine` fails for an
unelevated process even when only reading. Windows itself does not require that; the constraint is
ours and is being addressed upstream.

## Known issues

TODO — to be filled from the issue tracker at release time.
